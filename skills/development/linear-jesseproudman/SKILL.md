---
name: linear
description: Interact with Linear project management. Use for creating, searching, updating issues and projects. Triggers on "linear", "issue", "ticket", "task", "backlog", "sprint", "cycle", or project management requests.
---

# Linear Integration

Query and manage Linear issues, projects, and teams via the Linear SDK.

## Setup

Requires `LINEAR_API_KEY` in your Moltbot config env section:
```json
"env": {
  "LINEAR_API_KEY": "lin_api_..."
}
```

## Usage

Run scripts from this skill's `scripts/` directory:

### List Issues
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs issues [--limit N] [--team TEAM_ID]
```

### Search Issues
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs search "query text" [--limit N]
```

### Get Issue by ID
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs get ISSUE_ID
```

### Create Issue
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs create --title "Title" --team TEAM_ID [--description "..."] [--priority 1-4]
```

### Update Issue
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs update ISSUE_ID [--title "..."] [--state STATE_ID] [--priority 1-4]
```

### List Teams
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs teams
```

### List Projects
```bash
LINEAR_API_KEY="$LINEAR_API_KEY" node scripts/linear.mjs projects
```

## Output

All commands output JSON for easy parsing. Use `jq` for formatting if needed.

## Priority Levels
- 0: No priority
- 1: Urgent
- 2: High
- 3: Medium
- 4: Low
