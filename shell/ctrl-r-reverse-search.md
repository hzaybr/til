# Ctrl+R: Reverse Search Command History

_Date: 2026-03-11_

## Context
When working in the terminal, you often need to re-run a command you typed earlier. Scrolling through history with the up arrow is slow and tedious.

## The Learning
Press `Ctrl+R` in the terminal to activate **reverse incremental search**. This lets you search through your command history by typing a substring.

```bash
# Press Ctrl+R, then start typing part of the command
(reverse-i-search)`docker': docker compose up -d
```

- **Ctrl+R** again: cycle to the next (older) match
- **Enter**: execute the matched command
- **Tab** or **Right Arrow**: place the command on the line for editing
- **Ctrl+G** or **Ctrl+C**: cancel and return to an empty prompt

This works in both Bash and Zsh.

## References
- [Bash Manual - Searching](https://www.gnu.org/software/bash/manual/html_node/Searching.html)
