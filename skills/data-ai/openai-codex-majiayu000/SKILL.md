---
name: openai-codex
description: "OpenAI Codex CLI usage patterns, configuration, sandboxing, and best practices for AI-assisted development."
---

# OpenAI Codex CLI Development

> **Source:** https://github.com/openai/codex

Codex CLI is OpenAI's AI-powered coding agent that runs locally. This skill provides quick reference for effective usage, configuration, and integration.

## Best Practices

### Getting Started

- **Install via npm**: `npm install -g @openai/codex` or `brew install --cask codex`
- **Authenticate with ChatGPT** - Sign in with your ChatGPT Plus/Pro/Team account
- **Create an AGENTS.md** in project roots with project-specific context
- **Use trusted directories** to grant file system access

### Effective Usage

- **Be specific with requests** - "Refactor Dashboard to React Hooks" beats "fix code"
- **Use resume for long tasks** - `codex resume --last` continues previous sessions
- **Leverage non-interactive mode** - `codex exec "..."` for automation

### Model Selection

Codex uses GPT-4o by default. Override with `--model`:

```bash
codex --model gpt-4o          # Default, balanced
codex --model gpt-4o-mini     # Faster, cheaper
codex --model o1-preview      # Advanced reasoning
```

### Configuration

Configuration lives in `~/.codex/config.toml`:

```toml
[model]
name = "gpt-4o"

[sandbox]
enabled = true
permissions = ["read", "write", "execute"]

[mcp_servers]
my-server = { command = "node", args = ["./server.js"] }
```

### AGENTS.md Files

Create project context files:

```markdown
<!-- AGENTS.md -->
# Project Context

This is a React TypeScript project using:
- Vite for bundling
- Tailwind CSS for styling
- React Query for data fetching

## Conventions
- Use functional components with hooks
- Place components in src/components/
- Run `npm test` before committing
```

### Sandbox & Security

- **Sandbox mode** restricts file system and network access
- **Execpolicy** defines rules for allowed commands
- **Approval modes**: `suggest` (default), `auto-edit`, `full-auto`

```bash
# Run with auto-approval for safe commands
codex --approval-mode auto-edit

# Strict sandbox mode
codex --sandbox
```

### Non-Interactive Mode

Use `codex exec` for automation and CI/CD:

```bash
# Single command
codex exec "fix lint errors in src/"

# With specific model
codex exec -m gpt-4o-mini "add TypeScript types to utils/"

# JSON output for scripting
codex exec --json "list all TODO comments"
```

## Quick Reference

### Common Commands

```bash
# Interactive session
codex

# With initial prompt
codex "explain this codebase"

# Non-interactive
codex exec "fix the bug in auth.ts"

# Resume previous session
codex resume --last
codex resume <SESSION_ID>

# Show status
codex status
```

### Slash Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/clear` | Clear conversation |
| `/status` | Show session info |
| `/compact` | Compress context |
| `/model` | Change model |
| `/bug` | Report a bug |

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+C` | Cancel operation |
| `Ctrl+D` | Exit CLI |
| `Ctrl+L` | Clear screen |
| `Enter` | Send message |
| `Shift+Enter` | New line |

### Environment Variables

```bash
OPENAI_API_KEY         # API key (if using API billing)
CODEX_MODEL            # Default model
CODEX_SANDBOX          # Enable sandbox (true/false)
CODEX_HOME             # Config directory (~/.codex)
```

## Documentation Index

Detailed documentation synced from the official GitHub repository.

### Getting Started

| Resource | When to Consult |
|----------|-----------------|
| [readme.md](resources/readme.md) | Overview, installation, quickstart |
| [getting-started.md](resources/getting-started.md) | CLI usage, tips, example prompts |
| [install.md](resources/install.md) | Installation options |
| [authentication.md](resources/authentication.md) | Auth methods, API keys, headless login |

### Configuration

| Resource | When to Consult |
|----------|-----------------|
| [config.md](resources/config.md) | Full configuration reference |
| [example-config.md](resources/example-config.md) | Example configurations |
| [agents-md.md](resources/agents-md.md) | Root AGENTS.md file |
| [agents-md-guide.md](resources/agents-md-guide.md) | Writing effective AGENTS.md |

### Security & Sandboxing

| Resource | When to Consult |
|----------|-----------------|
| [sandbox.md](resources/sandbox.md) | Sandbox mode, permissions |
| [execpolicy.md](resources/execpolicy.md) | Command execution policies |
| [platform-sandboxing.md](resources/platform-sandboxing.md) | Platform-specific sandboxing |
| [windows-sandbox.md](resources/windows-sandbox.md) | Windows sandbox security |
| [zero-data-retention.md](resources/zero-data-retention.md) | ZDR mode |

### Features

| Resource | When to Consult |
|----------|-----------------|
| [exec.md](resources/exec.md) | Non-interactive mode |
| [prompts.md](resources/prompts.md) | Custom prompts |
| [skills.md](resources/skills.md) | Codex skills |
| [slash-commands.md](resources/slash-commands.md) | Slash commands |
| [advanced.md](resources/advanced.md) | MCP, tracing, advanced features |

### Reference

| Resource | When to Consult |
|----------|-----------------|
| [faq.md](resources/faq.md) | Frequently asked questions |
| [changelog.md](resources/changelog.md) | Version history |
| [contributing.md](resources/contributing.md) | Contributing guidelines |
| [sdk/typescript.md](resources/sdk/typescript.md) | TypeScript SDK |

## Syncing Documentation

Resources are synced from the official OpenAI Codex GitHub repository:

```bash
cd skills/openai-codex
bun run scripts/sync-docs.ts
```
