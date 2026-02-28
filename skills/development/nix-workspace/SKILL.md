---
name: nix-workspace
description: Node.js dashboard for Nix agents - task management with comments, agent status, action log, and task routing. Start with `node server.js`. Access at http://localhost:5050 or https://nixbot.auromations.com
---

# Nix Workspace Dashboard

A lightweight Node.js dashboard for managing Nix agents and tasks.

## Features

- **Kanban Board**: Tasks with To Do → In Progress → Done columns
- **Comments System**: AI comments on tasks with completion notes
- **Task Routing**: Tasks moved to in_progress auto-route to main agent
- **Agent Status**: View all configured agents and their models
- **Action Log**: Track all dashboard activity
- **Notes**: Leave instructions for agents (checked on heartbeats)

## Quick Start

```bash
cd /data/workspace/skills/nix-workspace/src
npm install  # first time only
node server.js
```

Dashboard runs at:
- **Local**: http://localhost:5050
- **Public**: https://nixbot.auromations.com (via Cloudflare tunnel)

## API Endpoints

### Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/tasks` | List all tasks |
| POST | `/api/tasks` | Create task |
| PUT | `/api/tasks/:id` | Update task (move columns) |
| DELETE | `/api/tasks/:id` | Delete task |
| POST | `/api/tasks/:id/complete` | Mark done with summary |

### Comments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/tasks/:id/comments` | List comments |
| POST | `/api/tasks/:id/comments` | Add comment |

**Comment types**: `note`, `completion`, `update`

```bash
# Add comment
curl -X POST http://localhost:5050/api/tasks/TASK_ID/comments \
  -H "Content-Type: application/json" \
  -d '{"text":"Working on it...","author":"Nix","type":"note"}'

# Complete with summary
curl -X POST http://localhost:5050/api/tasks/TASK_ID/complete \
  -H "Content-Type: application/json" \
  -d '{"summary":"Task completed successfully","author":"Nix"}'
```

### Other

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/status` | Agent status (all agents + models) |
| GET | `/api/notes` | Get notes |
| POST | `/api/notes` | Update notes |
| GET | `/api/action-log` | Recent activity |
| GET | `/api/health` | Health check |

## Task Routing

When a task is moved to `in_progress`, the dashboard automatically sends a system event to `agent:main:main` via webhook. This ensures tasks are executed by the main agent which has access to all channels (Telegram, WhatsApp, etc).

The system event includes:
- Task title and description
- Task ID for completion tracking
- Instructions for marking done and adding comments

## Agent Integration

### Heartbeat Checks

Add to `HEARTBEAT.md`:

```markdown
## Dashboard Tasks
Check http://localhost:5050/api/tasks for in_progress items.
Complete with: curl -X POST http://localhost:5050/api/tasks/ID/complete -d '{"summary":"..."}'
```

### Completion Flow

1. Task created in dashboard → appears in "To Do"
2. User moves to "In Progress" → routed to main agent
3. Agent works on task
4. Agent marks complete with summary:
   ```bash
   curl -X POST http://localhost:5050/api/tasks/ID/complete \
     -H "Content-Type: application/json" \
     -d '{"summary":"Sent email successfully","author":"Nix"}'
   ```

## Configuration

Dashboard reads from `/data/.clawdbot/openclaw.json`:
- `gateway.auth.token` - For API calls
- `hooks.token` - For webhook notifications
- `agents.list` - Agent names and models

## Files

```
src/
├── server.js          # Express backend
├── data/
│   ├── tasks.json     # Tasks with comments
│   ├── notes.json     # Notes for agent
│   └── action_log.json
├── static/
│   ├── css/style.css
│   └── js/dashboard.js
└── templates/
    └── index.html
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DASHBOARD_PORT` | 5050 | Server port |
| `GATEWAY_URL` | http://localhost:18789 | Gateway URL |

## Security

- No auth by default - run on trusted networks
- Public access via Cloudflare tunnel (nixbot.auromations.com)
- Don't expose directly to internet without authentication
