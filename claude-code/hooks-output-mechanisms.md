# Claude Code Hooks: Output Mechanisms

_Date: 2025-03-12_

## Context

Claude Code hooks can return results via JSON to control behavior. There are multiple output fields, each with different visibility (user vs Claude) and purpose. The docs describe this with subtle wording — "shown to the user" vs "for Claude to consider" — so it's easy to miss.

## The Learning

### Who Sees What

**`additionalContext`** — Claude sees it, user sees it in expandable diagnostic

Use for: lint errors, console.log detection — things Claude should act on.

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "unused import found in file.py"
  }
}
```

**`systemMessage`** — user sees it as a warning, Claude does NOT see it

Use for: operational notifications that don't need Claude's attention.

```json
{ "systemMessage": "Config sync completed" }
```

**`decision: "block"` + `reason`** — Claude sees the reason, user sees it in diagnostic

Use for: errors Claude must fix (type errors, test failures).

```json
{ "decision": "block", "reason": "TypeScript errors in file.ts:\n..." }
```

**`permissionDecision`** (PreToolUse only) — controls whether the tool runs

- `"allow"` — bypass permission system
- `"deny"` — block, reason shown to Claude
- `"ask"` — show confirmation dialog to user

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "ask",
    "permissionDecisionReason": "Review before pushing"
  }
}
```

**`echo "$INPUT"` (passthrough)** — silent, no feedback to anyone

Use for: auto-formatters (Prettier, Ruff format) that just modify the file.

### Exit Codes

- `exit 0` — success, stdout parsed as JSON
- `exit 2` — blocking error, stderr shown to Claude as feedback
- Other — non-blocking error, stderr only in verbose mode (Ctrl+O)

### Matcher

The `matcher` field is a **regex on the tool name** only (e.g. `"Edit"`, `"Edit|Write"`, `"Bash"`). To filter by file extension or command content, do it inside the hook script:

```bash
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // ""')
if echo "$FILE" | grep -qE '\.py$'; then
  # only run for Python files
fi
```

## References

- [Hooks Reference](https://code.claude.com/docs/en/hooks) — full event schemas and JSON output fields
- [Hooks Guide](https://code.claude.com/docs/en/hooks-guide) — examples and common patterns
