---
name: things3-manager
description: Full task management for Things3 app on macOS. Use when the user wants to view, create, update, complete, delete, or organize tasks, projects, and areas in Things3. Triggers include "show my tasks", "add a todo", "what's on my Today list", "mark X as done", "delete task", "cancel project", "move task to project", "what's overdue", "help me reorganize my tasks", "reschedule", "batch update", or any reference to Things3, todos, projects, deadlines, or task management.
---

# Things3 Manager

Manage Things3 tasks, projects, and areas via Python scripts.

## Requirements

1. **Things3 for Mac** installed and opened at least once
2. **Enable Things URLs**: Things → Settings → General → Enable Things URLs
3. **Install things.py**: `pip install things.py`

## Architecture

- **Reading**: `scripts/things3_dashboard.py` and `scripts/things3_read.py` use things.py (direct SQLite queries)
- **Writing**: `scripts/things3_write.py` uses Things URL scheme (`things:///`) and AppleScript for delete
- **Utilities**: `scripts/things3_utils.py` for sync and token checking

## Important Limitations

### Areas Must Exist First
The Things URL scheme **cannot create new areas**. If you want to assign a project/task to a new area:
1. Ask the user to create the area manually in Things3
2. Verify it exists: `python3 -c "import things; print([a['title'] for a in things.areas()])"`
3. Then assign projects/tasks to it

### Tags Must Exist First
Tags must already exist in Things3 to be applied. The URL scheme cannot create new tags.

## Quick Reference

### Dashboard (Fast Full Overview)

For quick overview of everything, use the dashboard script (single call, much faster):

```bash
# Human-readable dashboard
python3 scripts/things3_dashboard.py

# JSON output (for processing)
python3 scripts/things3_dashboard.py --json

# Skip sync check (faster but may have stale data)
python3 scripts/things3_dashboard.py --no-sync
```

Returns: inbox, today, upcoming, anytime, someday, projects, areas, overdue, deadlines — all in one call.

**Prefer dashboard for full overviews.** Use individual read commands only when you need specific filtered data.

### Read Operations (Individual Queries)

```bash
# Lists
python3 scripts/things3_read.py inbox
python3 scripts/things3_read.py today
python3 scripts/things3_read.py upcoming
python3 scripts/things3_read.py anytime
python3 scripts/things3_read.py someday
python3 scripts/things3_read.py logbook --limit 10
python3 scripts/things3_read.py deadlines
python3 scripts/things3_read.py overdue

# Organization
python3 scripts/things3_read.py projects
python3 scripts/things3_read.py projects --area "Work"
python3 scripts/things3_read.py areas --include-items
python3 scripts/things3_read.py tags
python3 scripts/things3_read.py project-tasks "Project Name"

# Search & lookup
python3 scripts/things3_read.py search "keyword"
python3 scripts/things3_read.py get UUID

# Check existing areas (for assignment)
python3 -c "import things; print([a['title'] for a in things.areas()])"
```

### Write Operations

```bash
# Add to-do (with notes for context)
python3 scripts/things3_write.py add "Task title"
python3 scripts/things3_write.py add "Call Alex" --when today --tags personal --notes "Re: project deadline"
python3 scripts/things3_write.py add "Buy groceries" --checklist milk eggs bread
python3 scripts/things3_write.py add "Review doc" --list "Work Project" --deadline 2025-01-20

# Add project (with notes for context)
python3 scripts/things3_write.py add-project "Q1 Planning" --area Work --notes "Key deliverables for Q1"
python3 scripts/things3_write.py add-project "Trip" --todos "Book flights" "Reserve hotel" "Pack bags"

# Update (requires UUID from read operations)
python3 scripts/things3_write.py update UUID --when tomorrow
python3 scripts/things3_write.py update UUID --tags urgent important
python3 scripts/things3_write.py update UUID --list "Different Project"
python3 scripts/things3_write.py update UUID --notes "New notes (replaces existing)"
python3 scripts/things3_write.py update UUID --append-notes "Additional context"
python3 scripts/things3_write.py update UUID --prepend-notes "Important: "

# Complete / Cancel / Delete
python3 scripts/things3_write.py complete UUID
python3 scripts/things3_write.py cancel UUID      # Moves to Logbook as canceled
python3 scripts/things3_write.py delete UUID      # Moves to Trash

# Show in app
python3 scripts/things3_write.py show --list today
python3 scripts/things3_write.py show --id UUID
```

### Batch Operations

Update multiple items at once:

```bash
# Complete multiple tasks
python3 scripts/things3_write.py batch complete UUID1 UUID2 UUID3

# Cancel multiple tasks
python3 scripts/things3_write.py batch cancel UUID1 UUID2

# Reschedule multiple tasks
python3 scripts/things3_write.py batch reschedule UUID1 UUID2 --when tomorrow

# Move multiple tasks to a project
python3 scripts/things3_write.py batch move UUID1 UUID2 --list "Project Name"

# Add tag to multiple tasks
python3 scripts/things3_write.py batch add-tag UUID1 UUID2 --tag "urgent"
```

### Utilities

```bash
# Ensure Things is running and synced
python3 scripts/things3_utils.py ensure-running

# Check if URL scheme is enabled
python3 scripts/things3_utils.py check-token
```

## Best Practices

### Adding Context with Notes
When creating tasks/projects, always add helpful notes:
- **Projects**: Include purpose, timeline, dependencies, phase info
- **Tasks**: Include links, context from conversation, why it matters

Example:
```bash
# Good - has context
python3 scripts/things3_write.py add-project "Bass Buzz" --notes "1-month hardcore schedule. Jazz rehearsal Jan 31."

# Bad - no context
python3 scripts/things3_write.py add-project "Bass Buzz"
```

### Using Tags Effectively
Check existing tags first, then apply relevant ones:
```bash
# See available tags
python3 scripts/things3_read.py tags

# Apply existing tags
python3 scripts/things3_write.py update UUID --add-tags "priority" "learning"
```

### Phased Planning
For users with many goals, help organize into phases:
1. **Phase 1**: Immediate priorities (this week/month)
2. **Phase 2**: After Phase 1 completes, add next priorities
3. **Someday**: Ideas to revisit later

Use notes to indicate phase: `--notes "Phase 1 priority. Deadline: Jan 31"`

## Workflows

### Finding and completing a task

1. Search: `python3 scripts/things3_read.py search "task name"`
2. Get UUID from output
3. Complete: `python3 scripts/things3_write.py complete UUID`

### Moving tasks between projects

1. Get task UUID: `python3 scripts/things3_read.py search "task"`
2. Get project UUID: `python3 scripts/things3_read.py projects`
3. Move: `python3 scripts/things3_write.py update TASK_UUID --list-id PROJECT_UUID`

### Assigning to a new area

1. Ask user to create area in Things3
2. Verify: `python3 -c "import things; print([a['title'] for a in things.areas()])"`
3. Assign: `python3 scripts/things3_write.py update-project UUID --area "New Area Name"`

### Bulk reorganization

1. Get full picture: `python3 scripts/things3_dashboard.py --json`
2. For each task to update: `python3 scripts/things3_write.py update UUID --when DATE`
3. Or use batch: `python3 scripts/things3_write.py batch reschedule UUID1 UUID2 --when DATE`

### Deleting multiple tasks

1. Get UUIDs: `python3 scripts/things3_read.py search "old project"`
2. Delete each: Run delete for each UUID (no batch delete to prevent accidents)

## When Parameters

| Value | Effect |
|-------|--------|
| `today` | Moves to Today |
| `tomorrow` | Schedules for tomorrow |
| `evening` | Today's evening section |
| `someday` | Moves to Someday |
| `YYYY-MM-DD` | Schedules for specific date |
| `anytime` | Removes from Today, keeps in Anytime |

## Confirmation Behavior

- **Auto-execute**: Simple adds with clear parameters
- **Confirm first**: Updates, completions, deletes, batch operations, or any ambiguous request

## Tips

- Use dashboard for full overviews (faster than multiple individual calls)
- UUIDs are required for all update/complete/delete operations
- Projects can have headings; specify with `--heading "Heading Name"` when adding tasks
- **Areas and tags must already exist** in Things to be assigned
- Use `--dry-run` to preview what a command will do without executing
- Always add notes for context when creating projects/tasks
- For URL scheme details, see `references/url_scheme.md`
