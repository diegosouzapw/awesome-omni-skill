---
name: ai-agents-dashboard
description: Live real-time dashboard for monitoring AI agent swarms with smart completion detection, clickable agent detail views, idle time tracking, and activity streaming. Auto-launches a beautiful web UI with capybara-inspired colors.
version: 6.0.0
author: HappyCapy
triggers:
  - launch dashboard
  - agent dashboard
  - monitor agents
  - live dashboard
  - show agent progress
  - track swarm
  - ai dashboard
---

# AI Agents Dashboard Skill

> Real-time monitoring for AI agent swarms with a beautiful, auto-refreshing web UI.

## Features

- **Real-time Updates**: Auto-refreshes every 2 seconds
- **Smart Completion Detection**: Auto-detects when agents finish
- **Last Activity Tracking**: Shows "Xs/Xm/Xh ago" for each agent
- **Idle State Detection**: Agents idle >60s shown as "IDLE"
- **Beautiful UI**: Capybara-inspired color palette with light/dark themes
- **Clickable Agent Details**: Click any agent for live activity feed
- **Zero Dependencies**: Pure Python stdlib + vanilla HTML/CSS/JS
- **Single File**: Self-contained with embedded HTML

## Quick Start

### 1. Create Swarm Configuration

Create `swarm-config.json` in your project directory:

```json
{
  "swarm_name": "My Project Swarm",
  "start_time": "2026-02-05T14:00:00Z",
  "agents": {
    "agent-1": {
      "role": "Core Architect",
      "wave": 1,
      "task_id": "abc123",
      "mission": "Set up project structure"
    },
    "agent-2": {
      "role": "Backend Developer",
      "wave": 2,
      "task_id": "def456",
      "mission": "Build API endpoints"
    }
  }
}
```

### 2. Launch Dashboard

```bash
# Copy server to your workspace
cp ~/.claude/skills/ai-agents-dashboard/ai-agents-dashboard-server.py /workspace/my-project/

# Set environment variables
export SWARM_DIR="/workspace/my-project"
export SWARM_NAME="My Project Swarm"
export DASHBOARD_PORT=8080

# Start server
python3 /workspace/my-project/ai-agents-dashboard-server.py &

# Export port for web access
/app/export-port.sh 8080
```

### 3. View Dashboard

Open the exported URL in your browser. Dashboard auto-refreshes every 2 seconds.

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SWARM_DIR` | Current directory | Directory containing swarm-config.json |
| `TASK_DIR` | `/tmp/claude-1000` | Directory containing task output files |
| `DASHBOARD_PORT` | `8080` | HTTP server port |
| `SWARM_NAME` | From config | Display name for the swarm |

## API Endpoints

### GET /
Returns the dashboard HTML page.

### GET /api/status
Returns JSON with full swarm status.

### GET /api/agent/{agent-id}
Returns detailed information about a specific agent.

### GET /health
Returns `{"status": "ok", "version": "v6"}` for health checks.

## Status States

| Status | Color | Condition |
|--------|-------|-----------|
| `pending` | Gray | No output file exists |
| `running` | Orange (pulse) | Active output, <60s since last event |
| `idle` | Teal | 60-120s since last event |
| `completed` | Green | >120s idle OR completion marker found |
| `failed` | Red | status.json has `"status": "failed"` |

## Files in This Skill

```
~/.claude/skills/ai-agents-dashboard/
├── SKILL.md                         # This file
├── ai-agents-dashboard-server.py    # Main server (self-contained)
└── launch-dashboard.py              # Launcher utility script
```

## Using the Launcher Utility

```python
from launch_dashboard import launch_dashboard, update_agent_task_id

# Launch dashboard
url = launch_dashboard(
    swarm_name="my-project",
    swarm_dir="/workspace/my-project",
    agents={
        "agent-1-core": {"role": "Core", "wave": 1, "mission": "Setup"},
        "agent-2-api": {"role": "API", "wave": 2, "mission": "Build API"},
    }
)

# Update agent task IDs after launching
update_agent_task_id(
    swarm_dir="/workspace/my-project",
    agent_id="agent-1-core",
    task_id="abc123def456"
)
```

## Troubleshooting

### Port already in use?
```bash
lsof -i :8080
export DASHBOARD_PORT=8081
```

### Live Activity Feed Not Showing?
Call `update_agent_task_id()` after launching each agent to create symlinks.

## Credits

Built for monitoring AI agent swarms in real-time.
