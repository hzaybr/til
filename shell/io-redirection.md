# Bash I/O Redirection

_Date: 2025-03-12_

## Context

Every process has three standard I/O streams. Understanding redirection is essential for shell scripts — especially when you need to capture output, discard errors, or pipe data between commands.

## The Learning

### The Three Standard Streams

```
0 (stdin)  ← input   — default: keyboard
1 (stdout) → output  — default: screen
2 (stderr) → errors  — default: screen
```

These are **file descriptors (fd)** — just numbers the OS uses to identify I/O channels. `0`, `1`, `2` are pre-assigned; you can open more with `exec 3>file.txt`.

### Output Redirection

```bash
command > file.txt       # stdout to file (overwrite). Same as 1>file.txt
command >> file.txt      # stdout to file (append)
command 2> errors.txt    # stderr to file
command 2>/dev/null      # stderr discarded (/dev/null = black hole)
command &> file.txt      # stdout + stderr to file (bash shorthand)
```

### Merging Streams with `2>&1`

`&` tells bash "this is a fd number, not a filename":

```bash
2>&1     # stderr → wherever stdout currently points
2>1      # stderr → a FILE named "1" (not what you want!)
```

**Order matters** — bash processes redirections left to right:

```bash
# ✅ Both to file
command > file.txt 2>&1
# Step 1: stdout → file.txt
# Step 2: stderr → follows stdout → file.txt

# ❌ Only stdout to file
command 2>&1 > file.txt
# Step 1: stderr → follows stdout → screen (stdout still points to screen)
# Step 2: stdout → file.txt (but stderr is already locked to screen)
```

### Pipes

Connect stdout of one command to stdin of the next:

```bash
cat file | grep "error" | wc -l
#  stdout →→ stdin/stdout →→ stdin
```

Pipes only carry **stdout**. stderr bypasses the pipe and goes to screen:

```bash
command | grep "x"         # only stdout piped
command 2>&1 | grep "x"   # merge stderr first, then both piped
```

### Capturing Output

```bash
OUT=$(command)             # capture stdout only
OUT=$(command 2>&1)        # capture stdout + stderr
OUT=$(command 2>/dev/null) # capture stdout, discard stderr
```

### Input Redirection

```bash
command < file.txt          # stdin from file
grep "hi" <<< "hi there"   # here string
cat <<EOF                   # here document
multi-line
input
EOF
```

## References

- `man bash` — section "REDIRECTION"
