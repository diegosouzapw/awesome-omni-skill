---
name: swarm-dashboard
description: Live real-time dashboard for monitoring AI agent swarms with smart completion detection, clickable agent detail views, idle time tracking, and activity streaming. Auto-launches a beautiful web UI. V6 features embedded HTML, line-by-line JSON parsing, and capybara-inspired colors.
version: 6.1.0
author: HappyCapy
triggers:
  - launch dashboard
  - swarm dashboard
  - monitor agents
  - agent dashboard
  - live dashboard
  - show agent progress
  - track swarm
---

# Swarm Dashboard Skill

> Real-time monitoring for AI agent swarms with a beautiful, auto-refreshing web UI.

## Quick Start

```python
from swarm_dashboard import launch_dashboard, update_agent_task_id

# Launch dashboard
url = launch_dashboard(
    swarm_name="My Project",
    swarm_dir="/workspace/my-project",
    agents={
        "agent-1": {"role": "Core Architect", "wave": 1, "mission": "Setup"},
        "agent-2": {"role": "Backend Dev", "wave": 2, "mission": "Build API"},
    }
)

# After launching each agent with Task tool, update task ID
update_agent_task_id(
    swarm_dir="/workspace/my-project",
    agent_id="agent-1",
    task_id="abc123"  # From Task tool response
)
```

## Features

- **Real-time Updates** - Auto-refreshes every 2 seconds
- **Smart Completion Detection** - Auto-detects finished agents
- **Capybara-Inspired UI** - Natural color palette with light/dark themes
- **Clickable Details** - Live activity feed, tools used, files created
- **Zero Dependencies** - Pure Python stdlib

## CLI Usage

```bash
# Start server
swarm-dashboard serve --port 8080 --swarm-dir /workspace

# Launch in background
swarm-dashboard launch --name "My Swarm" --dir /workspace

# Check status
swarm-dashboard status --dir /workspace

# Stop
swarm-dashboard stop --dir /workspace
```

## Configuration

Environment variables:
- `SWARM_DIR` - Directory with agent folders
- `TASK_DIR` - Directory with task outputs
- `DASHBOARD_PORT` - Server port (default 8080)
- `SWARM_NAME` - Display name

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `GET /` | Dashboard HTML |
| `GET /api/status` | Swarm status JSON |
| `GET /api/agent/{id}` | Agent details |
| `GET /health` | Health check |
