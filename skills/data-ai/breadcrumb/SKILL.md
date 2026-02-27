---
name: breadcrumb
description: Leave notes on files for other agents to see in future sessions. Use after making non-obvious changes, fixing tricky bugs, or when code looks wrong but is intentional.
license: MIT
metadata:
  author: tylergibbs1
  version: "3.5.0"
  keywords:
    - agent-communication
    - file-warnings
    - session-coordination
    - context-sharing
---

# Breadcrumb

Leave notes on files that persist across agent sessions.

## When to use

After making changes that future agents might misunderstand:
- Non-obvious code that looks like it could be simplified
- Bug fixes for edge cases
- Intentional workarounds
- Security-critical patterns
- Performance tuning

## Core workflow

**1. Before editing, check for warnings:**
```bash
breadcrumb check ./src/api/users.ts
```
- Exit 0 = safe to proceed (clear/info)
- Exit 1 = warning exists, read the `suggestion` field

**2. After non-obvious changes, leave a note:**
```bash
breadcrumb add ./src/api/users.ts "Retry logic tuned for rate limits"
```

## Command reference

| Command | Purpose |
|---------|---------|
| `breadcrumb check <path>` | Check path for notes (`-r` for recursive) |
| `breadcrumb add <path> <message>` | Leave a note (`-s` severity, `--ttl` expiration) |
| `breadcrumb edit <path-or-id>` | Edit a note (`-m` message, `-a` append, `-s` severity) |
| `breadcrumb verify [path]` | Check if notes are stale (`--update` to refresh hashes) |
| `breadcrumb search <query>` | Find notes by content (`-r` for regex) |
| `breadcrumb coverage [path]` | Show breadcrumb coverage stats |
| `breadcrumb ls` | List all notes (`-s` filter by severity) |
| `breadcrumb status` | Quick overview (counts) |
| `breadcrumb rm <path>` | Remove a note (`-i` by ID) |
| `breadcrumb prune` | Remove expired notes |

## Staleness detection

Notes track file content hashes. When you see `[STALE]` prefix:
- The file has changed since the note was written
- The note may no longer be accurate
- Use judgment: the warning might still apply, or might be outdated

```
📝 BREADCRUMB: [STALE] Don't simplify this regex
                ↑ Code changed - verify note still applies
```

After reviewing stale notes, update hashes with:
```bash
breadcrumb verify --update
```

## Output format

All commands output JSON. Key fields:
- `status`: "clear", "info", or "warn"
- `suggestion`: Actionable guidance when warnings exist
- `breadcrumbs`: Array of matching breadcrumb objects
- `staleness`: "verified", "stale", or "unknown" (per breadcrumb)
