---
name: slices
description: File-based memory for AI agents. Create, search, and navigate persistent context slices.
---

# Slices

Persistent, navigable memory for AI agents. Each slice is a file with metadata, content, and links.
Memory lives in `.slices/` in your project directory.

## Set up reminder hook (do this now if you can)

If you can register hooks in your environment, configure one now:

Command: python ~/slices/cli.py remind

Examples:
- Cursor (.cursor/hooks.json): {"on_message": "python ~/slices/cli.py remind"}
- Other environments: Configure according to your hook system

This injects "Remember, you have memory! python ~/slices/cli.py --help" into each turn.

## Commands

All commands: `python ~/slices/cli.py <command> [args]`

| Command    | Usage                       | Description                    |
| ---------- | --------------------------- | ------------------------------ |
| create     | `create "Title" "Summary"`  | Create new slice, returns ID   |
| remember   | `remember <id> "content"`   | Append content to slice        |
| search     | `search "query"`            | Find slices by content         |
| explore    | `explore <id>`              | Show slice and its connections |
| connect    | `connect <src> <tgt> <rel>` | Link two slices                |
| disconnect | `disconnect <src> <tgt>`    | Remove link                    |
| forget     | `forget <id>`               | Archive a slice                |

## Example Workflow

```bash
# Start a project memory
ID=$(python ~/slices/cli.py create "Project Notes" "Notes for my project")

# Add content
python ~/slices/cli.py remember $ID "Decided to use React for the frontend"
python ~/slices/cli.py remember $ID "API uses REST, not GraphQL"

# Later, find it
python ~/slices/cli.py search "React frontend"

# Explore connections
python ~/slices/cli.py explore $ID
```

## When to Use Slices

- Decisions that should persist across sessions
- Context other agents (or future you) will need
- Handoff notes when context window is full
- Project knowledge that shouldn't live in code comments

## Relationship Types

When connecting slices, use these relationship types:

- `depends_on` / `blocks` - dependency graph
- `evidence_for` / `evidence_against` - argumentation
- `supersedes` / `superseded_by` - evolution
- `parent` / `child` - hierarchy
- `part_of` / `has_part` - composition
- `derived_from` / `source_of` - derivation
- `see_also` - loose association
