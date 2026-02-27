---
name: obsidian
description: "Read, write, search, and manage Obsidian vault notes. Use when: (1) Reading/writing markdown notes, (2) Searching vault content, (3) Managing daily/periodic notes, (4) Tracking tasks or oncall incidents. Supports filesystem access and Local REST API."
---

# Obsidian Vault Integration

## Configuration

```bash
export OBSIDIAN_VAULT_PATH="/path/to/your/vault"
export OBSIDIAN_API_KEY="your-api-key-here"           # From: Obsidian Settings → Local REST API
export OBSIDIAN_DAILY_FORMAT="Journal/Daily/%Y-%m-%d.md"  # Optional
export OBSIDIAN_TODO_FILE="Inbox/Tasks.md"            # Optional
```

## CLI Tools

### Filesystem (obsidian.sh)

```bash
./scripts/obsidian.sh fs-read <path>            # Read note
./scripts/obsidian.sh fs-write <path> <content> # Write note
./scripts/obsidian.sh fs-list [dir]             # List .md files
./scripts/obsidian.sh fs-search <query>         # Grep search
./scripts/obsidian.sh fs-daily-append <content> # Append to daily note
```

### Thought (Daily Notes)

```bash
thought "Great idea for the app"
thought "Meeting went well" meeting work
```

### Todo Tracking

```bash
todo add "Review PR" work --due tomorrow --priority high
todo done 1                    # Complete by number
todo done "PR"                 # Complete by search
todo delete 2                  # Remove task
todo list                      # Show pending
todo list work                 # Filter by tag
```

See: [references/todo.md](references/todo.md)

### Oncall Tracking

```bash
oncall start                   # Start shift
oncall log "Alert fired" incident database
oncall resolve "Fixed it" database
oncall summary                 # View current shift
oncall end                     # End and archive
```

See: [references/oncall.md](references/oncall.md)

### REST API (obsidian.sh)

```bash
./scripts/obsidian.sh status              # Check connection
./scripts/obsidian.sh read <path>         # Read via API
./scripts/obsidian.sh write <path> <content>
./scripts/obsidian.sh daily               # Get daily note
./scripts/obsidian.sh daily-append <content>
./scripts/obsidian.sh search <query>      # Simple search
```

See: [references/api-reference.md](references/api-reference.md)

## Quick Filesystem Access

```bash
# Read
cat "$OBSIDIAN_VAULT_PATH/folder/note.md"

# Write
cat > "$OBSIDIAN_VAULT_PATH/folder/note.md" << 'EOF'
# My Note
Content here
EOF

# Search
grep -r "term" "$OBSIDIAN_VAULT_PATH" --include="*.md"
```

## Decision Guide

| Need                  | Method        |
| --------------------- | ------------- |
| Fast read/write       | Filesystem    |
| Quick thoughts/notes  | `thought` CLI |
| Task management       | `todo` CLI    |
| Oncall/incidents      | `oncall` CLI  |
| Search by frontmatter | REST API      |
| Dataview queries      | REST API      |
| Execute commands      | REST API      |
| No Obsidian running   | Filesystem    |

## Reference Docs

- [API Reference](references/api-reference.md) - REST API endpoints and curl examples
- [Thought Reference](references/thought.md) - Quick notes to daily journal
- [Todo Reference](references/todo.md) - Task management with Obsidian Tasks format
- [Oncall Reference](references/oncall.md) - Incident tracking and shift management
