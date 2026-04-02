# bunx is bun — Same Binary, Different argv[0]

_Date: 2026-03-24_

## Context

Noticed `bunx` is a symlink to `bun`, but they behave differently. Dug into the Bun source code (Zig) to understand how one binary serves multiple roles.

## The Learning

### bunx is just a symlink to bun

```bash
which bunx
# → ~/.bun/bin/bunx -> ~/.bun/bin/bun

ls -la ~/.bun/bin/
# bun
# bunx -> bun
```

The bun binary checks `argv[0]` at startup to decide its behavior. This is a classic Unix pattern — one binary, multiple personalities based on how it's invoked.

Other examples of the same pattern:

- `gzip` / `gunzip` / `zcat` — same binary, compress vs decompress vs view
- `busybox` — one binary pretends to be `ls`, `cat`, `cp` and hundreds more
- `vim` / `vi` — same binary, `vi` starts in compatible mode

### argv[0] detection (src/cli.zig)

The core logic lives in two functions:

```zig
// Step 1: Check if the executable name ends with "bunx"
pub fn isBunX(argv0: []const u8) bool {
    if (Environment.isWindows) {
        return strings.endsWithComptime(argv0, "bunx.exe") or
               strings.endsWithComptime(argv0, "bunx");
    }
    return strings.endsWithComptime(argv0, "bunx");
}

// Same pattern for Node.js compatibility
pub fn isNode(argv0: []const u8) bool {
    if (Environment.isWindows) {
        return strings.endsWithComptime(argv0, "node.exe") or
               strings.endsWithComptime(argv0, "node");
    }
    return strings.endsWithComptime(argv0, "node");
}
```

```zig
// Step 2: The which() function routes to the right command
pub fn which() Tag {
    var args_iter = ArgsIterator{ .buf = bun.argv };
    const argv0 = args_iter.next() orelse return .HelpCommand;

    // --- Priority 1: Check executable name (symlink detection) ---
    if (isBunX(argv0)) {
        is_bunx_exe = true;
        return .BunxCommand;
    }

    if (isNode(argv0)) {
        pretend_to_be_node = true;    // bun pretends to be Node.js!
        return .RunAsNodeCommand;
    }

    // --- Priority 2: Parse subcommand from argv[1] ---
    var next_arg = args_iter.next() orelse return .AutoCommand;

    // Skip leading flags (but not -e which is eval)
    while (next_arg.len > 0 and next_arg[0] == '-' and
           !(next_arg.len > 1 and next_arg[1] == 'e')) {
        next_arg = args_iter.next() orelse return .AutoCommand;
    }

    return switch (RootCommandMatcher.match(next_arg)) {
        RootCommandMatcher.case("run")     => .RunCommand,
        RootCommandMatcher.case("x")       => .BunxCommand,  // "bun x" subcommand
        RootCommandMatcher.case("install") => .InstallCommand,
        RootCommandMatcher.case("build")   => .BuildCommand,
        RootCommandMatcher.case("test")    => .TestCommand,
        RootCommandMatcher.case("init")    => .InitCommand,
        // ... more commands ...
        else => .AutoCommand,
    };
}
```

So `bunx foo` and `bun x foo` reach the same `.BunxCommand`, but via different paths:

- `bunx foo` → argv[0] ends with "bunx" → symlink name detection
- `bun x foo` → argv[1] is "x" → subcommand parsing

### What BunxCommand actually does (src/cli/bunx_command.zig)

When bunx resolves a package to execute:

```
1. Parse options (package name, --no-install, etc.)
2. Check node_modules/.bin/ for an existing binary (bun.which())
3. If not found, check bunx cache: <temp_dir>/bunx-<uid>-<package>/
4. If still not found (and --no-install not set), run `bun add` in cache dir
5. Read package.json "bin" field to resolve actual executable name
6. Run the binary with Run.runBinary()
```

This is why `bunx` can auto-download packages that aren't installed — it has its own cache directory and install logic.

### `bun run` vs `bunx` — different resolution order

- `bun run <target>`: package.json scripts → node_modules/.bin/ → file path
- `bunx <target>`: node_modules/.bin/ → bunx cache → auto-download

Key difference: `bun run` checks scripts first, `bunx` skips scripts entirely. And `bunx` auto-downloads, `bun run` doesn't.

### Where the symlink gets created (src/cli/install_completions_command.zig)

The symlink is NOT created by the install script (`curl | bash`). It's created by `bun completions`, which the install script calls as a final step.

POSIX (macOS/Linux) — tries these locations in order:

```
1. Same directory as bun binary (e.g. ~/.bun/bin/bunx)
2. $BUN_INSTALL/bin/
3. $HOME/.bun/bin/
4. $HOME/.local/bin/
```

Each uses `std.posix.symlink(exe, bunx_path)`, falls through to the next on failure.

Windows — symlinks need admin privileges, so it uses alternatives:

```
1. Try CreateHardLinkW (hardlink, no admin needed)
2. If hardlink fails, create bunx.cmd wrapper script:
   @%~dp0bun.exe x %*
```

The `.cmd` wrapper just forwards to `bun x`, which reaches the same `BunxCommand` via subcommand parsing.

### Installation method affects the binary form

- `curl -fsSL https://bun.sh/install | bash` → one binary + symlink via `bun completions`
- `npm install -g bun` (e.g. under nvm) → two separate files (`bun.exe` + `bunx.exe`), defined in npm package's `bin` field. `bunx.exe` is typically a hardlink (same size as `bun.exe`)

Either way, argv[0] detection works the same — it only cares about the filename.

## References

- [bun/src/cli.zig](https://github.com/oven-sh/bun/blob/main/src/cli.zig) — `isBunX()`, `isNode()`, `which()` command routing
- [bun/src/cli/bunx_command.zig](https://github.com/oven-sh/bun/blob/main/src/cli/bunx_command.zig) — BunxCommand execution logic
- [bun/src/cli/install_completions_command.zig](https://github.com/oven-sh/bun/blob/main/src/cli/install_completions_command.zig) — symlink/hardlink creation
