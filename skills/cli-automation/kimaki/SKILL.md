---
name: kimaki
description: Control OpenCode agents from Discord via Kimaki CLI
license: MIT
compatibility: opencode
metadata:
  audience: developers
  workflow: automation
  platform: any
  homepage: https://kimaki.xyz
  repository: https://github.com/remorses/kimaki
---

## What I do

[Kimaki](https://kimaki.xyz) connects Discord to [OpenCode](https://github.com/opencode-ai/opencode). Send a message in Discord → AI agent edits code on your machine.

## Installation

```bash
# Run directly (recommended)
npx -y kimaki@latest

# Or install globally
npm install -g kimaki
```

## Setup

```bash
# Run the interactive setup wizard
npx -y kimaki@latest
```

This will guide you through:
1. Creating a Discord bot
2. Configuring bot settings (Message Content intent)
3. Generating an invite link
4. Selecting project directories

## Architecture

- **One bot per machine** - Each Discord bot is tied to one computer
- **Channels = projects** - Each Discord channel maps to a local project directory
- **Run on startup** - Keep `kimaki` running to receive messages

## Commands

### Send prompts

```bash
# Send to channel (creates new thread)
npx -y kimaki@latest send --channel CHANNEL_ID --prompt "Your prompt"

# Continue existing thread
npx -y kimaki@latest send --thread THREAD_ID --prompt "Follow-up"

# Continue existing session
npx -y kimaki@latest send --session SESSION_ID --prompt "Prompt"
```

### Session management

```bash
# List sessions
npx -y kimaki@latest session list
npx -y kimaki@latest session list --project /path/to/project

# Read session conversation
npx -y kimaki@latest session read SESSION_ID
```

### Files

```bash
# Upload file to session
npx -y kimaki@latest upload-to-discord file.md --session SESSION_ID
```

### Projects

```bash
# List projects
npx -y kimaki@latest project list

# Add new project
npx -y kimaki@latest project add /path/to/project
```

### Tunnel

```bash
# Create tunnel for dev server
npx -y kimaki@latest tunnel --port 3000
npx -y kimaki@latest tunnel --port 3000 --tunnel-id my-app
```

### Update

```bash
npx -y kimaki@latest upgrade
```

## Options

| Flag | Description |
|------|-------------|
| `-c, --channel` | Discord channel ID |
| `-p, --prompt` | Message content |
| `-d, --project` | Project directory |
| `-n, --name` | Thread name |
| `--thread` | Continue existing thread |
| `--session` | Continue existing session |
| `--notify-only` | Create thread without starting AI |
| `--wait` | Wait for response |
| `--worktree` | Create git worktree |
| `--model` | Specify model (e.g., anthropic/claude-3-5-sonnet-20241022) |
| `--agent` | Specify agent type |

## Tips

- Use `--notify-only` for scheduled tasks - creates a notification thread you can reply to when ready
- Sessions accumulate history - continue via `--thread` or `--session` for context
- Set up multiple bots for multiple machines - add all bots to the same Discord server
- Run data in separate directories for multiple instances: `kimaki --data-dir ~/work-bot`
