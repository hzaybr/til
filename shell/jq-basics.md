# jq Basics for Shell Scripts

_Date: 2025-03-12_

## Context

`jq` is the standard tool for parsing and creating JSON in shell scripts. Essential for writing Claude Code hooks, where stdin/stdout communicate via JSON.

## The Learning

### Reading JSON

```bash
# Extract a field (-r for raw string, no quotes)
echo '{"name":"Alice"}' | jq -r '.name'
# → Alice

# Nested field
echo '{"a":{"b":1}}' | jq '.a.b'
# → 1

# Default value (// is the alternative operator — if left is null/false, use right)
echo '{}' | jq '.name // "default"'
# → "default"
```

### Creating JSON

```bash
# -n: don't read stdin, create from scratch
jq -n '{ key: "value" }'

# --arg: create a string variable (auto-escapes special chars)
jq -n --arg name "Alice" '{ name: $name }'
# → {"name":"Alice"}

# --argjson: create a non-string variable (number, boolean, object)
jq -n --argjson count 42 '{ count: $count }'
# → {"count":42}
```

### Why `--arg` Instead of String Interpolation

```bash
# ❌ If $MSG contains quotes or newlines, the JSON breaks
jq -n "{ reason: \"$MSG\" }"

# ✅ --arg handles escaping automatically
jq -n --arg reason "$MSG" '{ reason: $reason }'
```

### Common Operations

```bash
echo '[1,2,3]' | jq 'length'                      # → 3
echo '{"a":1,"b":2}' | jq 'keys'                  # → ["a","b"]
echo '[1,2,3,4,5]' | jq '[.[] | select(. > 3)]'   # → [4,5]
echo '[1,2,3]' | jq 'map(. * 2)'                   # → [2,4,6]
echo '{"items":[1,2,3]}' | jq '.items | length'   # → 3 (internal pipe)
```

### Key Flags

- `-r` — raw output (no quotes around strings)
- `-n` — null input (don't read stdin)
- `-e` — exit with error if output is null or false
- `--arg name val` — bind string variable
- `--argjson name val` — bind non-string variable

## References

- [jq Manual](https://jqlang.github.io/jq/manual/)
- [jq Playground](https://jqplay.org/)
