# cat -v: Show Invisible Characters

_Date: 2026-04-01_

## Context
When debugging terminal output, escape sequences, or files with hidden characters (e.g. Windows line endings), it's hard to tell what's actually in the file.

## The Learning
`cat -v` displays non-printing characters in a readable form.

```bash
$ echo -e "hello\tworld\e[31mred" | cat -v
hello^Iworld^[[31mred
```

Common symbols:
- `^I` — Tab (Ctrl+I)
- `^[` — Escape (`\e`, starts ANSI escape sequences)
- `^M` — Carriage return (`\r`, Windows line endings)
- `^@` — Null byte
- `^C` — Ctrl+C

Useful for:
- Debugging broken terminal escape sequences
- Finding hidden `^M` in files from Windows
- Checking if a file has unexpected control characters

Related flags:
- `cat -A` — shows everything (`-v` + `$` at end of lines + `^I` for tabs)
- `cat -e` — shows `$` at end of each line (spot trailing whitespace)

## References
- `man cat`
