---
name: mini-agent
description: Personal AI Agent with Memory + Proactivity - Multi-instance support
version: 1.0.0
author: miles990
tags:
  - ai
  - agent
  - memory
  - proactive
  - personal-assistant
metadata:
  requires:
    bins:
      - claude
      - node
      - pnpm
  install:
    - id: bash
      kind: script
      command: curl -fsSL https://raw.githubusercontent.com/miles990/mini-agent/main/install.sh | bash
      bins:
        - mini-agent
---

# Mini-Agent

Personal AI Agent with Memory + Proactivity. A lightweight agent that maintains long-term memory, schedules proactive tasks, and supports multi-instance deployment.

## Features

- **Long-term Memory**: File-based memory using Markdown (MEMORY.md)
- **Daily Notes**: Automatic logging of conversations (daily/YYYY-MM-DD.md)
- **Task Management**: HEARTBEAT.md for active and scheduled tasks
- **Proactive Mode**: Cron-based task checking and execution
- **Multi-Instance**: Run multiple isolated instances with different configurations
- **HTTP API**: RESTful API for integration
- **Claude Integration**: Uses Claude Code CLI for AI processing

## Quick Start

```bash
# Install via script (recommended)
curl -fsSL https://raw.githubusercontent.com/miles990/mini-agent/main/install.sh | bash

# Or manually
git clone https://github.com/miles990/mini-agent.git ~/.mini-agent
cd ~/.mini-agent && pnpm install && pnpm build && npm link
```

## Usage

### Interactive Chat

```bash
mini-agent
```

Commands in chat:
- `/help` - Show available commands
- `/search <query>` - Search memory
- `/remember <text>` - Save to memory
- `/heartbeat` - Show active tasks
- `/trigger` - Execute pending tasks
- `/proactive on|off` - Toggle proactive mode
- `/quit` - Exit

### Process Files

```bash
mini-agent readme.md "summarize this"
mini-agent src/app.ts "review this code"
mini-agent a.txt b.txt "compare these files"
```

### Pipe Mode

```bash
echo "Hello" | mini-agent "translate to Chinese"
git diff | mini-agent "write commit message"
```

### HTTP Server

```bash
mini-agent server --port 3001
```

API endpoints:
- `POST /chat` - Send a message
- `GET /memory` - Read memory
- `GET /memory/search?q=` - Search memory
- `POST /memory` - Add to memory
- `GET /tasks` - List tasks
- `POST /tasks` - Add a task
- `POST /heartbeat/trigger` - Trigger tasks

## Multi-Instance Support

### Create Instances

```bash
mini-agent instance create --name "Research" --port 3002
mini-agent instance create --name "Coding" --port 3003 --role worker
```

### List Instances

```bash
mini-agent instance list
```

### Use Specific Instance

```bash
mini-agent --instance abc12345 "hello"
mini-agent --instance abc12345 server
```

### Start Instance Server

```bash
mini-agent instance start abc12345
```

## Data Structure

```
~/.mini-agent/
├── config.yaml                 # Global configuration
├── instances/
│   ├── default/                # Default instance
│   │   ├── instance.yaml       # Instance config
│   │   ├── MEMORY.md           # Long-term memory
│   │   ├── HEARTBEAT.md        # Tasks
│   │   ├── SKILLS.md           # Available skills
│   │   └── daily/              # Daily notes
│   └── {uuid}/                 # Custom instances
└── shared/
    └── GLOBAL_MEMORY.md        # Shared memory (optional)
```

## Configuration

### Global Config (~/.mini-agent/config.yaml)

```yaml
defaults:
  port: 3001
  proactiveSchedule: "*/30 * * * *"
  claudeTimeout: 120000
  maxSearchResults: 5

instances:
  - id: default
    role: standalone
```

### Instance Config (instance.yaml)

```yaml
id: "abc12345"
name: "Research Assistant"
role: standalone
port: 3002

persona:
  description: "A technical research assistant"
  systemPrompt: |
    You are a helpful research assistant...

proactive:
  enabled: true
  schedule: "0 9,12,18 * * *"

memory:
  maxSize: "50MB"
  syncToGlobal: false
```

## Memory Tags

The agent uses special tags in responses:

- `[REMEMBER]...[/REMEMBER]` - Content to save to memory
- `[TASK schedule="..."]...[/TASK]` - Task to add to heartbeat

Example:
```
User: Remember that I prefer TypeScript
Assistant: [REMEMBER]User prefers TypeScript over JavaScript[/REMEMBER]
I'll remember that you prefer TypeScript!
```

## Integration

### As Claude Code Skill

Copy this SKILL.md to your project's `.claude/skills/mini-agent/` directory.

### As API Service

```javascript
// Chat with agent
const response = await fetch('http://localhost:3001/chat', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ message: 'Hello!' })
});
const data = await response.json();
console.log(data.content);

// Add to memory
await fetch('http://localhost:3001/memory', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ content: 'Important fact', section: 'Facts' })
});
```

## Requirements

- Node.js >= 20
- Claude Code CLI (`claude` command available)

## License

MIT
