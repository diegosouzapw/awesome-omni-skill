---
name: shortcut
version: 1.2.0
description: Manage stories on Shortcut.com kanban boards. Use when creating, updating, or listing tasks/stories on Shortcut project management boards. Supports creating stories with descriptions and types (feature/bug/chore), updating story status, and listing active/completed stories. Includes full checklist task management and comment support.
---

# Shortcut Kanban Integration

Manage tasks and stories on Shortcut.com project boards via API.

## Prerequisites

- Shortcut API token configured via one of:
  - Environment variable: `SHORTCUT_API_TOKEN`
  - File: `~/.config/shortcut/api-token`
- Access to a Shortcut workspace with appropriate permissions

### Setup

1. Get your API token from Shortcut.com (Settings → API Tokens)
2. Store it either:
   - As environment variable: `export SHORTCUT_API_TOKEN="your-token"`
   - In a file: `echo "your-token" > ~/.config/shortcut/api-token && chmod 600 ~/.config/shortcut/api-token`
3. Optionally add to `~/.bashrc` for persistence:
   ```bash
   export SHORTCUT_API_TOKEN=$(cat ~/.config/shortcut/api-token 2>/dev/null | tr -d '\n')
   ```

## Available Operations

### List Stories

```bash
scripts/shortcut-list-stories.sh [--active|--completed|--all] [--json]
```

Options:
- `--active` - Show only incomplete stories (default)
- `--completed` - Show only completed stories
- `--all` - Include archived stories
- `--json` - Output raw JSON

### Show Story Details

```bash
scripts/shortcut-show-story.sh <story-id>
```

Displays full story information including:
- Story name and status
- Description (if present)
- Checklist items with completion status

### Create Story

```bash
scripts/shortcut-create-story.sh "Story name" [--description "text"] [--type feature|bug|chore]
```

Story types:
- `feature` (default) - New functionality
- `bug` - Bug fix
- `chore` - Maintenance task

### Update Story

```bash
scripts/shortcut-update-story.sh <story-id> [--complete|--todo|--in-progress] [--description "new text"]
```

Workflow state IDs vary by workspace. Common defaults:
- To Do (typically `500000006`)
- In Progress (typically `500000007`)
- Done (typically `500000010`)

To find your workspace's state IDs, use the Shortcut API or check your board settings.

### Manage Checklist Tasks

**Create a task:**
```bash
scripts/shortcut-create-task.sh <story-id> "task description"
```

**Update task completion status:**
```bash
scripts/shortcut-update-task.sh <story-id> <task-id> [--complete|--incomplete]
```

**Edit task description:**
```bash
scripts/shortcut-edit-task.sh <story-id> <task-id> "new description"
```

**Delete a task:**
```bash
scripts/shortcut-delete-task.sh <story-id> <task-id>
```

Use `shortcut-show-story.sh` to see task IDs.

### Manage Comments

**Add a comment:**
```bash
scripts/shortcut-add-comment.sh <story-id> "comment text"
```

**Update a comment:**
```bash
scripts/shortcut-update-comment.sh <story-id> <comment-id> "new text"
```

**Delete a comment:**
```bash
scripts/shortcut-delete-comment.sh <story-id> <comment-id>
```

Use `shortcut-show-story.sh` to see comment IDs.

## Workflow

1. List existing stories to understand current board state
2. Create new stories with descriptive names and appropriate types
3. Update story status as work progresses

## Notes

- Scripts use `SHORTCUT_API_TOKEN` environment variable or fall back to `~/.config/shortcut/api-token`
- Stories are created in "Unstarted" state by default (workflow_state_id: 500000006)
- If your workspace uses different workflow state IDs, you may need to adjust the scripts
- The token must have permissions for the workspace you want to manage
