---
name: copilot-cli
description: "Use the GitHub Copilot CLI for agentic coding, session management, custom agents, skills, plugins, SDK integration, and Ralph Wiggum loops. Use when: (1) running Copilot CLI sessions interactively or non-interactively, (2) managing custom agents and skills, (3) configuring MCP servers and plugins, (4) integrating with the Copilot SDK programmatically, (5) orchestrating multi-agent workflows, (6) setting up long-running autonomous coding loops (Ralph Wiggum pattern). NOT for: the old `gh copilot` extension (that's deprecated), simple file edits (just edit directly), or GitHub API operations (use `gh` CLI)."
---

# GitHub Copilot CLI

The standalone `copilot` CLI — an agentic coding assistant that runs in your terminal. Supports sessions, custom agents, skills, plugins, MCP servers, and a TypeScript SDK for programmatic control.

## Installation & Auth

```bash
# Install globally
npm install -g @github/copilot

# Authenticate (opens browser for device flow)
copilot login

# Check version
copilot --version
```

**Auth tokens (precedence order):** `COPILOT_GITHUB_TOKEN` > `GH_TOKEN` > `GITHUB_TOKEN` > stored credentials from `copilot login`

---

## Core Usage Patterns

### Interactive Mode (default)

```bash
# Start interactive session in current directory
copilot

# Start with a specific model
copilot --model claude-sonnet-4.6

# Start with a custom agent
copilot --agent code-reviewer

# Resume most recent session
copilot --continue

# Resume a specific session
copilot --resume <session-id>

# Resume with session picker
copilot --resume
```

### Non-Interactive Mode (scripting)

```bash
# One-shot prompt — exits after completion
copilot -p "Fix the bug in main.js" --allow-all-tools

# Silent output (agent response only, no stats)
copilot -p "Explain this error" -s

# Share session to markdown after completion
copilot -p "Refactor auth module" --allow-all --share

# Share to GitHub gist
copilot -p "Review this code" --allow-all --share-gist
```

### Auto-Execute Mode

```bash
# Start interactive and auto-execute a prompt
copilot -i "Fix the failing tests"

# Autopilot mode — agent continues without waiting for input
copilot --autopilot --allow-all
copilot --autopilot --max-autopilot-continues 10 --allow-all
```

### Full Auto (YOLO)

```bash
# All permissions — no confirmations, no restrictions
copilot --yolo
# Equivalent to:
copilot --allow-all-tools --allow-all-paths --allow-all-urls
```

---

## Available Models

Set via `--model`, `/model` command, `COPILOT_MODEL` env, or `model` in config.

```bash
# List available models (check --help for current choices)
copilot --help 2>&1 | grep -A1 'model <model>'

# Or in interactive mode
/model
```

---

## Permissions System

Copilot CLI has granular permission controls. Run `copilot help permissions` for full details. By default, it prompts for confirmation on file writes, shell commands, and URL access.

### Tool Permissions

```bash
# Allow specific tools without prompting
copilot --allow-tool 'shell(git:*)'           # All git commands
copilot --allow-tool 'write'                   # All file writes
copilot --allow-tool 'shell(npm:*)'            # All npm commands
copilot --allow-tool 'MyMCP(tool_name)'        # Specific MCP tool
copilot --allow-tool 'MyMCP'                   # All tools from an MCP server

# Deny specific tools (overrides allow)
copilot --deny-tool 'shell(git push)'          # Block git push
copilot --deny-tool 'shell(rm:*)'              # Block rm commands

# Control which tools the model can even see
copilot --available-tools shell write read      # Only these tools exist
copilot --excluded-tools shell                  # Everything except shell

# Allow all tools
copilot --allow-all-tools
```

### Path Permissions

```bash
# Add directories to allowed list
copilot --add-dir ~/other-project
copilot --add-dir /tmp --add-dir ~/workspace

# Allow any path
copilot --allow-all-paths

# Block temp dir access
copilot --disallow-temp-dir
```

### URL Permissions

```bash
# Allow specific domains (defaults to https://)
copilot --allow-url github.com
copilot --allow-url '*.github.com'

# Deny specific domains (overrides allow)
copilot --deny-url https://malicious-site.com

# Allow all URLs
copilot --allow-all-urls
```

---

## Interactive Commands

When running in interactive mode, use `/help` to see all available commands. Key ones:

### Agent & Customization
| Command | Description |
|---|---|
| `/agent` | Browse and select custom agents |
| `/skills` | Manage skills (list, enable/disable, add, reload, info, remove) |
| `/mcp` | Manage MCP server configuration |
| `/plugin` | Manage plugins and marketplaces |
| `/init` | Initialize Copilot instructions for the repo |
| `/instructions` | View and toggle custom instruction files |

### Session Management
| Command | Description |
|---|---|
| `/resume [id]` | Switch to a different session |
| `/rename` | Rename current session |
| `/context` | Show context window token usage |
| `/usage` | Display session usage metrics |
| `/session` | Show session info and workspace summary |
| `/compact` | Summarize history to reduce context usage |
| `/share` | Share session to file or gist |
| `/clear` | Clear conversation history |

### Code & Review
| Command | Description |
|---|---|
| `/diff` | Review changes in current directory |
| `/review` | Run code review agent |
| `/plan` | Create an implementation plan before coding |
| `/ide` | Connect to an IDE workspace |
| `/lsp` | Manage language server configuration |

### Multi-Agent
| Command | Description |
|---|---|
| `/model` | Select AI model |
| `/fleet` | Enable fleet mode for parallel subagent execution |
| `/tasks` | View and manage background tasks (subagents + shell sessions) |

### Permissions (runtime)
| Command | Description |
|---|---|
| `/allow-all` | Enable all permissions |
| `/add-dir` | Add directory to allowed list |
| `/list-dirs` | Show all allowed directories |
| `/cwd` | Change or show working directory |
| `/reset-allowed-tools` | Reset allowed tools list |

---

## Custom Agents, Skills, Plugins & MCP

For creating custom agents (`.agent.md`), managing skills, installing plugins, configuring MCP servers, and setting up custom instructions:

📄 **[`authoring/GUIDE.md`](authoring/GUIDE.md)**

---

## Docker Sandboxes

Run Copilot in an isolated Docker sandbox — **recommended for `--yolo` / Ralph Wiggum loops** where the agent has unrestricted tool access.

Official docs: [Docker Sandboxes for Copilot](https://docs.docker.com/ai/sandboxes/agents/copilot/)

### Quick Start

```bash
# Create and run a sandbox for a project directory
docker sandbox run copilot ~/my-project

# Or from within the project directory
cd ~/my-project
docker sandbox run copilot
```

The workspace is mounted at `/workspace` inside the sandbox by default.

### Authentication

The sandbox daemon doesn't inherit shell env vars. Set your token **globally** in your shell config:

```bash
# ~/.bashrc or ~/.zshrc
export GH_TOKEN=ghp_xxxxx
# or: export GITHUB_TOKEN=ghp_xxxxx
```

Then: `source ~/.zshrc`, restart Docker Desktop, and create the sandbox.

### Trusted Folders

Configure `~/.copilot/config.json` to skip safety prompts for sandbox workspaces:

```json
{
  "trusted_folders": ["/workspace"]
}
```

### Pass Options at Runtime

Use `--` separator to pass CLI flags through to Copilot inside the sandbox:

```bash
# Run with --yolo (no confirmations)
docker sandbox run <sandbox-name> -- --yolo

# Run with a specific model
docker sandbox run <sandbox-name> -- --model claude-sonnet-4.6
```

### Custom Templates

Base image: `docker/sandbox-templates:copilot`. See [Custom templates](https://docs.docker.com/ai/sandboxes/templates/) to build your own.

---

## Configuration

Config lives in `~/.copilot/config.json` (override with `XDG_CONFIG_HOME`).

Key settings (discover all with `cat ~/.copilot/config.json` or `copilot help config`):

| Setting | Default | Description |
|---|---|---|
| `model` | — | Default AI model |
| `auto_update` | `true` | Auto-download updates |
| `banner` | `"once"` | Banner display frequency |
| `beep` | `true` | Beep on attention needed |
| `include_coauthor` | `true` | Add Co-authored-by to commits |
| `trusted_folders` | `[]` | Pre-approved directories |
| `allowed_urls` | `[]` | Pre-approved URLs |
| `denied_urls` | `[]` | Blocked URLs |
| `streamer_mode` | `false` | Hide model names and quota |
| `parallel_tool_execution` | `true` | Run tools in parallel |
| `alt_screen` | `false` | Use alternate screen buffer |
| `experimental` | `false` | Enable experimental features |

---

## Environment Variables

Run `copilot help environment` for the full list. Key ones:

| Variable | Description |
|---|---|
| `COPILOT_MODEL` | Override default model |
| `COPILOT_ALLOW_ALL` | Set `"true"` for full auto mode |
| `COPILOT_AUTO_UPDATE` | Set `"false"` to disable auto-update |
| `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` | Comma-separated extra instruction directories |
| `COPILOT_EDITOR` / `VISUAL` / `EDITOR` | Editor for interactive editing (e.g., plan) |
| `XDG_CONFIG_HOME` | Override config directory (default: `~/.copilot`) |
| `XDG_STATE_HOME` | Override state directory (default: `~/.copilot`) |
| `USE_BUILTIN_RIPGREP` | Set `"false"` to use system ripgrep |
| `PLAIN_DIFF` | Set `"true"` to disable rich diffs |

---

## SDK Integration (TypeScript)

For programmatic control via the `@github/copilot-sdk` package (JSON-RPC, sessions, streaming events):

📄 **[`sdk/GUIDE.md`](sdk/GUIDE.md)**

---

## Orchestration Patterns

### Multi-Model Comparison

```bash
# Run same prompt across models, compare output
copilot --model gpt-5.3-codex -p "Implement X" -s --share ./codex-result.md
copilot --model claude-sonnet-4.6 -p "Implement X" -s --share ./claude-result.md
```

### Fleet Mode (parallel subagents)

```bash
# In interactive mode
/fleet
# Copilot spawns multiple subagents working in parallel
/tasks  # Monitor them
```

### Delegation to Copilot Coding Agent

```bash
# In interactive mode — push session to GitHub for background execution
/delegate complete the API integration tests

# Or prefix with &
& complete the API integration tests
```

### Non-Interactive Batch

```bash
# Process multiple files
for f in src/*.ts; do
  copilot -p "Review $f for security issues" -s --allow-all >> reviews.md
done
```

---

## Tips

- **`@ file paths`** — Reference files in prompts: `Explain @src/auth.ts`
- **`! shell`** — Run shell commands directly: `!git status`
- **`Esc`** — Stop current operation while thinking
- **`Shift+Tab`** — Toggle plan mode (plan before coding)
- **Steer while thinking** — Send follow-up messages while agent is working
- **`/compact`** — When context gets long, summarize to free up space
- **Config dir override** — `copilot --config-dir ./project-copilot-config` for project-specific settings

---

## Ralph Wiggum Loop (Long-Running Autonomous Coding)

For setting up autonomous coding loops that run indefinitely with fresh context each iteration, see the detailed guide:

📄 **[`ralph-wiggum/GUIDE.md`](ralph-wiggum/GUIDE.md)** — full pattern, templates, loop script, and orchestration options

Based on [@GeoffreyHuntley's Ralph Wiggum technique](https://github.com/ghuntley/how-to-ralph-wiggum).
