---
name: sandbox
description: Execute commands in isolated sandboxes for security. Use when running untrusted code, system commands, or operations that could affect the host system. Automatically detects the right runtime (Python, Node, Rust, Go, Ruby, etc.) from the command.
---

# Sandbox Skill

Execute commands in secure, isolated containers using agentkernel. This provides hardware-level isolation to protect the host system from potentially dangerous operations.

## When to Use This Skill

Use this skill when:

- Running untrusted or generated code
- Executing shell commands that could modify system state
- Testing code in a clean environment
- Running build/test commands that might have side effects
- Executing code from external sources
- Any operation where isolation is beneficial for security

Do NOT use this skill for:

- Simple file reads/writes (use standard tools)
- Git operations (use standard tools)
- Operations that need access to host credentials or SSH keys

## Instructions

### Basic Usage

Run any command in an isolated sandbox:

```bash
agentkernel run <command> [args...]
```

For compound commands with `&&` or `||`, wrap in `sh -c`:

```bash
agentkernel run sh -c 'npm install && npm test'
```

The runtime is auto-detected from the command:

```bash
# Python
agentkernel run python3 script.py
agentkernel run pip install package-name
agentkernel run pytest

# Node.js
agentkernel run node script.js
agentkernel run npm test
agentkernel run npx create-react-app my-app

# Rust
agentkernel run cargo build
agentkernel run cargo test

# Go
agentkernel run go build
agentkernel run go test ./...

# Ruby
agentkernel run ruby script.rb
agentkernel run bundle exec rspec

# And more: PHP, Java, C/C++, Elixir, Lua, Terraform...
```

### Specifying an Image

Override the auto-detected image:

```bash
agentkernel run --image ubuntu:24.04 apt-get update
agentkernel run --image postgres:16-alpine psql --version
```

### Security Profiles

Control sandbox permissions with security profiles:

```bash
# Default: moderate security (network enabled, no mounts)
agentkernel run npm test

# Restrictive: no network, read-only filesystem, all capabilities dropped
agentkernel run --profile restrictive python3 script.py

# Permissive: network, mounts, environment passthrough
agentkernel run --profile permissive cargo build

# Disable network access specifically
agentkernel run --no-network curl example.com  # Will fail
```

| Profile | Network | Mount CWD | Mount Home | Pass Env | Read-only |
|---------|---------|-----------|------------|----------|-----------|
| permissive | Yes | Yes | Yes | Yes | No |
| moderate | Yes | No | No | No | No |
| restrictive | No | No | No | No | Yes |

### Keeping Sandboxes for Debugging

Keep the sandbox after execution:

```bash
agentkernel run --keep npm test
# Later: agentkernel sandbox remove run-<id>
```

### Persistent Sandboxes

For longer-running work:

```bash
# Create and start
agentkernel sandbox create my-sandbox
agentkernel sandbox start my-sandbox

# Execute commands
agentkernel exec my-sandbox npm test
agentkernel exec my-sandbox python -m pytest

# Clean up
agentkernel sandbox stop my-sandbox
agentkernel sandbox remove my-sandbox
```

## Supported Languages

| Language | Commands | Docker Image |
|----------|----------|--------------|
| Python | `python`, `python3`, `pip`, `poetry`, `uv`, `pytest` | `python:3.12-alpine` |
| Node.js | `node`, `npm`, `npx`, `yarn`, `pnpm`, `bun` | `node:22-alpine` |
| Rust | `cargo`, `rustc` | `rust:1.85-alpine` |
| Go | `go`, `gofmt` | `golang:1.23-alpine` |
| Ruby | `ruby`, `bundle`, `rails`, `rake` | `ruby:3.3-alpine` |
| Java | `java`, `javac`, `mvn`, `gradle` | `eclipse-temurin:21-alpine` |
| PHP | `php`, `composer` | `php:8.3-alpine` |
| C/C++ | `gcc`, `g++`, `make`, `cmake` | `gcc:14-bookworm` |
| .NET | `dotnet` | `mcr.microsoft.com/dotnet/sdk:8.0` |
| Terraform | `terraform` | `hashicorp/terraform:1.10` |
| Lua | `lua`, `luajit` | `nickblah/lua:5.4-alpine` |
| Shell | `bash`, `sh`, `zsh` | `alpine:3.20` |

## Security Notes

- Sandboxes run in isolated containers (Docker or Podman)
- On Linux with KVM, Firecracker microVMs provide hardware-level isolation
- Three security profiles: `permissive`, `moderate` (default), `restrictive`
- Use `--profile restrictive` for maximum isolation (no network, read-only fs)
- Use `--no-network` to disable network access specifically
- Host filesystem is NOT mounted by default (use `permissive` profile to enable)
- Each sandbox gets a clean environment

## Agent Compatibility Modes

Agentkernel supports compatibility modes for different AI agents. Each mode configures sandbox behavior optimally for that agent's needs:

```bash
# Claude Code compatibility (default for this plugin)
agentkernel run --compatibility-mode claude python3 script.py

# Codex compatibility (stricter isolation)
agentkernel run --compatibility-mode codex python3 script.py

# Gemini compatibility (Docker-style)
agentkernel run --compatibility-mode gemini python3 script.py
```

Or configure in `agentkernel.toml`:

```toml
[sandbox]
name = "my-project"

[agent]
preferred = "claude"
compatibility_mode = "claude"  # claude, codex, gemini, or native

[security]
profile = "moderate"  # Can override compatibility mode defaults
```

### Compatibility Mode Differences

| Mode | Network Policy | Project Mount | API Access | Isolation |
|------|----------------|---------------|------------|-----------|
| Native | Allow all | No | None | Moderate |
| Claude | Domain allowlist | Yes | Anthropic API | Moderate |
| Codex | Domain allowlist | Yes | OpenAI API | Strict |
| Gemini | Domain allowlist | Yes | Google API | Moderate |

Claude mode allows: `api.anthropic.com`, `cdn.anthropic.com`, common package registries.
Codex mode allows: `api.openai.com`, `cdn.openai.com`, common package registries.
Gemini mode allows: `*.googleapis.com`, common package registries.

All modes block cloud metadata endpoints (169.254.169.254, metadata.google.internal).

## MCP Server Integration

This plugin provides an MCP (Model Context Protocol) server for direct tool integration. When enabled, the following MCP tools are available:

### Command Execution

| Tool | Description |
|------|-------------|
| `sandbox_run` | Run a command in a temporary sandbox with security profile and compatibility mode |
| `sandbox_create` | Create a persistent sandbox |
| `sandbox_exec` | Execute a command in an existing sandbox |
| `sandbox_list` | List all sandboxes |
| `sandbox_remove` | Remove a sandbox |

### File Operations

| Tool | Description |
|------|-------------|
| `sandbox_file_write` | Write a file into a running sandbox |
| `sandbox_file_read` | Read a file from a sandbox (supports binary via base64) |

### Lifecycle Management

| Tool | Description |
|------|-------------|
| `sandbox_start` | Start a stopped sandbox |
| `sandbox_stop` | Stop a running sandbox |

### sandbox_run Options

The `sandbox_run` tool accepts these parameters:

- `command` (required): Command to execute (array of strings)
- `profile`: Security profile (`restrictive`, `moderate`, `permissive`)
- `compatibility_mode`: Agent mode (`native`, `claude`, `codex`, `gemini`)
- `network`: Override network access (boolean)
- `cwd`: Working directory inside sandbox
- `env`: Environment variables (object)
- `timeout_ms`: Command timeout in milliseconds (default: 30000)

Example MCP request:
```json
{
  "method": "tools/call",
  "params": {
    "name": "sandbox_run",
    "arguments": {
      "command": ["python3", "-c", "print('hello')"],
      "compatibility_mode": "claude",
      "timeout_ms": 60000
    }
  }
}
```

The MCP server starts automatically when the plugin is loaded and provides these tools via JSON-RPC over stdio.

## Daemon Mode (Fast Execution)

For high-performance scenarios, agentkernel can run a daemon that maintains a pool of pre-warmed sandboxes:

```bash
# Start daemon with pre-warmed pool
agentkernel daemon start

# Check status
agentkernel daemon status  # Shows warm VMs per agent type

# Commands use warm pool automatically (~50ms vs ~500ms cold start)
agentkernel run python3 script.py

# Pre-warm for specific agent types
agentkernel daemon prewarm --mode claude

# Stop daemon
agentkernel daemon stop
```

The daemon supports per-agent pool configuration for optimal warm starts:

```toml
# Example daemon config (future)
[daemon]
min_warm = 3
max_warm = 5

[daemon.agents.claude]
min_warm = 2
runtime = "python"

[daemon.agents.codex]
min_warm = 1
runtime = "python"
```

## Claude Code Permission Configuration

Agentkernel MCP tools are designed to run safely in isolated sandboxes. You can configure Claude Code's approval behavior for these tools in your project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__agentkernel__sandbox_run",
      "mcp__agentkernel__sandbox_exec",
      "mcp__agentkernel__sandbox_list",
      "mcp__agentkernel__sandbox_file_read"
    ],
    "ask": [
      "mcp__agentkernel__sandbox_file_write",
      "mcp__agentkernel__sandbox_create"
    ],
    "deny": []
  }
}
```

### Tool Risk Levels

| Tool | Risk Level | Recommendation |
|------|------------|----------------|
| `sandbox_list` | Low | Auto-approve (read-only) |
| `sandbox_file_read` | Low | Auto-approve (read from sandbox only) |
| `sandbox_run` | Medium | Auto-approve (sandboxed execution) |
| `sandbox_exec` | Medium | Auto-approve (sandboxed execution) |
| `sandbox_file_write` | Medium | Ask (writes files into sandbox) |
| `sandbox_create` | Medium | Ask (creates persistent resources) |
| `sandbox_start` | Medium | Auto-approve (starts existing sandbox) |
| `sandbox_stop` | Medium | Auto-approve (stops sandbox) |
| `sandbox_remove` | Medium | Ask (removes sandbox) |

All tools run in isolated sandboxes, so even "medium" risk tools cannot affect the host system. The `ask` recommendations are for visibility, not security.

### MCP Server Configuration

Add agentkernel to your Claude Code MCP servers:

```json
// .mcp.json (project-level)
{
  "mcpServers": {
    "agentkernel": {
      "command": "agentkernel",
      "args": ["mcp", "server"],
      "env": {}
    }
  }
}
```

Or globally in `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "agentkernel": {
      "command": "agentkernel",
      "args": ["mcp", "server"]
    }
  }
}
```

### Headless/Automation Mode

For CI/CD or automated workflows, use the `--allowedTools` flag:

```bash
claude --allowedTools "mcp__agentkernel__*" "Run tests in sandbox"
```

Or configure in settings for persistent auto-approval of all agentkernel tools.

## Error Handling

If a command fails, the sandbox is automatically cleaned up. Use `--keep` to preserve it for debugging:

```bash
agentkernel run --keep failing-command
# Inspect: docker exec -it agentkernel-run-<id> sh
# Clean up: agentkernel sandbox remove run-<id>
```
