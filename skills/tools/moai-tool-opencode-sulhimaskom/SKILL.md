---
name: moai-tool-opencode
description: OpenCode.ai open-source AI coding agent comprehensive reference. Use when working with OpenCode TUI, CLI, IDE integration, configuring agents, tools, MCP servers, creating plugins, or developing with the SDK.
version: 2.0.0
user-invocable: false
status: active
updated: 2026-01-08
allowed-tools:
  - Read
  - Grep
  - Glob
  - WebFetch
  - Bash
---

# OpenCode.ai Tool Skill

Comprehensive reference for OpenCode.ai - an open-source AI coding agent available as terminal interface, desktop application, and IDE extension.

## Quick Reference

**Installation:**
```bash
curl -fsSL https://opencode.ai/install | bash
# or
npm install -g opencode
```

**Start OpenCode:**
```bash
opencode              # Launch TUI in current directory
opencode /path/to/dir # Launch in specific directory
opencode run "prompt" # Non-interactive mode
opencode serve        # Start headless server
```

**Essential Commands:**
- `/connect` - Configure API keys
- `/init` - Generate AGENTS.md
- `/models` - List available models
- `/share` - Share session
- `/undo` - Undo changes
- `Tab` - Switch agents
- `Ctrl+P` - Command palette

**Configuration Files:**
- `~/.config/opencode/opencode.json` - Global config
- `./opencode.json` - Project config
- `AGENTS.md` - Project instructions
- `.opencode/agent/` - Custom agents
- `.opencode/skill/` - Agent skills
- `.opencode/tool/` - Custom tools
- `.opencode/command/` - Custom commands

---

## Documentation Index

This skill contains 32 comprehensive documentation modules organized by category.

### Core (7 modules)
- [Introduction](modules/core/intro.md) - Installation, setup, primary use cases
- [Configuration](modules/core/config.md) - JSON configuration options and merge order
- [Providers](modules/core/providers.md) - 75+ LLM providers setup
- [Network](modules/core/network.md) - Proxy and certificate configuration
- [Enterprise](modules/core/enterprise.md) - SSO, central config, AI gateway
- [Troubleshooting](modules/core/troubleshooting.md) - Common issues and solutions
- [Migration 1.0](modules/core/migration-1.0.md) - Upgrade guide

### Usage (7 modules)
- [TUI](modules/usage/tui.md) - Terminal interface reference
- [CLI](modules/usage/cli.md) - Command-line interface
- [IDE](modules/usage/ide.md) - VS Code/Cursor integration
- [Zen](modules/usage/zen.md) - Curated model gateway
- [Share](modules/usage/share.md) - Session sharing
- [GitHub](modules/usage/github.md) - GitHub Actions integration
- [GitLab](modules/usage/gitlab.md) - GitLab CI/CD integration

### Configuration (14 modules)
- [Tools](modules/configure/tools.md) - Built-in and custom tools
- [Rules](modules/configure/rules.md) - AGENTS.md instructions
- [Agents](modules/configure/agents.md) - Custom agent configuration
- [Models](modules/configure/models.md) - Model selection and variants
- [Themes](modules/configure/themes.md) - Custom theme creation
- [Keybinds](modules/configure/keybinds.md) - Keyboard shortcuts
- [Commands](modules/configure/commands.md) - Custom slash commands
- [Formatters](modules/configure/formatters.md) - Code formatting
- [Permissions](modules/configure/permissions.md) - Tool approval system
- [LSP Servers](modules/configure/lsp-servers.md) - Language server setup
- [MCP Servers](modules/configure/mcp-servers.md) - Model Context Protocol
- [ACP Support](modules/configure/acp.md) - Agent Client Protocol
- [Skills](modules/configure/skills.md) - Reusable instructions
- [Custom Tools](modules/configure/custom-tools.md) - TypeScript tools

### Development (4 modules)
- [SDK](modules/develop/sdk.md) - JavaScript/TypeScript client
- [Server](modules/develop/server.md) - HTTP API reference
- [Plugins](modules/develop/plugins.md) - Plugin development
- [Ecosystem](modules/develop/ecosystem.md) - Community projects

See [index.md](index.md) for complete navigation.

---

## Key Concepts

### Agent Architecture

**Primary Agents** - Main assistants you interact with directly:
- Build: Full tools enabled (default)
- Plan: Restricted, analysis-focused

**Subagents** - Specialized assistants invoked by primary agents:
- General: Research and multi-step tasks
- Explore: Fast codebase exploration

### Configuration Hierarchy

1. Command-line flags (highest priority)
2. Environment variables
3. Project config (`./opencode.json`)
4. Global config (`~/.config/opencode/opencode.json`)
5. Built-in defaults

### Tool Permission Levels

| Level | Description |
|-------|-------------|
| `allow` | Auto-approve execution |
| `ask` | Request user approval |
| `deny` | Block execution |

---

## Common Patterns

### Basic Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4",
  "agent": {
    "security": {
      "description": "Security auditor",
      "permission": { "edit": "deny" }
    }
  }
}
```

### Custom Command

```markdown
---
description: Review current changes
agent: plan
---
Review the following changes:
!`git diff --staged`
```

### MCP Server Setup

```json
{
  "mcp": {
    "github": {
      "type": "local",
      "command": ["npx", "-y", "@modelcontextprotocol/server-github"],
      "environment": { "GITHUB_TOKEN": "{env:GITHUB_TOKEN}" }
    }
  }
}
```

### Custom Tool

```typescript
import { tool } from "@opencode-ai/plugin"

export default tool({
  description: "Query project database",
  args: {
    query: tool.schema.string().describe("SQL query")
  },
  async execute(args) {
    return `Executed: ${args.query}`
  }
})
```

---

## Resources

- Documentation: https://opencode.ai/docs
- GitHub: https://github.com/anomalyco/opencode
- Discord: https://opencode.ai/discord

Storage Locations:
- Config: `~/.config/opencode/`
- Data: `~/.local/share/opencode/`
- Cache: `~/.cache/opencode/`
- Logs: `~/.local/share/opencode/log/`

---

## Works Well With

- moai-lang-typescript - TypeScript development patterns
- moai-lang-python - Python development patterns
- moai-domain-backend - Backend API development
- moai-domain-frontend - Frontend development
- moai-workflow-testing - Testing workflows


---

## Referenced Files

> The following files are referenced in this skill and included for context.

### modules/core/intro.md

```markdown
---
source: https://opencode.ai/docs/
fetched: 2026-01-08
title: Introduction to OpenCode
---

# Introduction to OpenCode

## Overview

OpenCode is an open source AI coding agent available in multiple formats: terminal interface, desktop application, and IDE extension.

## Installation Methods

### Quick Install

```bash
curl -fsSL https://opencode.ai/install | bash
```

### Package Managers

**Node.js (npm, Bun, pnpm, Yarn)**

```bash
npm install -g opencode
# or
bun install -g opencode
# or
pnpm install -g opencode
# or
yarn global add opencode
```

**Homebrew (macOS/Linux)**

```bash
brew install opencode
```

**Paru (Arch Linux)**

```bash
paru -S opencode
```

**Chocolatey (Windows)**

```bash
choco install opencode
```

**Scoop (Windows)**

```bash
scoop install opencode
```

**Mise**

```bash
mise use -g opencode
```

**Docker**

Docker support is available for containerized environments.

## Core Setup Steps

### 1. Prerequisites

- Modern terminal emulator (WezTerm, Alacritty, Ghostty, Kitty)
- LLM API keys

### 2. Configuration

Users connect via `/connect` command, authenticate at opencode.ai/auth, and input API credentials.

### 3. Project Initialization

Navigate to project directory and run `opencode`, then execute `/init` to analyze the codebase and generate `AGENTS.md`.

## Primary Use Cases

The documentation highlights four main workflows:

### Question Asking

Query codebase understanding using `@` syntax for file references.

```
@src/main.ts What does this file do?
```

### Feature Planning

Toggle Plan mode (Tab key) to review implementation approaches before building.

### Direct Changes

Request specific modifications with detailed context.

### Version Control

Undo/redo changes via `/undo` and `/redo` commands.

## Additional Features

### Sharing

`/share` command creates shareable conversation links.

### Customization

- Theme selection
- Keybind configuration
- Code formatter setup
- Custom commands

### Documentation

References to config, providers, and troubleshooting resources.

## Best Practices

The page emphasizes providing detailed context and treating the AI agent like a junior developer for optimal results.

```

### modules/core/config.md

```markdown
---
source: https://opencode.ai/docs/config/
fetched: 2026-01-08
title: Configuration
---

# OpenCode Configuration Guide

## Overview

OpenCode uses JSON/JSONC config files to customize the tool's behavior across TUI, CLI, IDE, and other interfaces.

## Configuration Locations (Merge Order)

Configuration files are merged together, not replaced. Settings from the following config locations are combined:

1. **Global**: `~/.config/opencode/opencode.json`
2. **Per-project**: `opencode.json` in project root (merged with global)
3. **Custom path**: Via `OPENCODE_CONFIG` environment variable
4. **Custom directory**: Via `OPENCODE_CONFIG_DIR` environment variable

## Core Configuration Options

### Models & Providers

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4-5",
  "small_model": "anthropic/claude-haiku-3-5",
  "provider": {
    "anthropic": {
      "timeout": 30000,
      "setCacheKey": true,
      "region": "us-east-1",
      "profile": "default",
      "endpoint": "https://api.anthropic.com"
    }
  }
}
```

- `model`: Primary model (e.g., "anthropic/claude-sonnet-4-5")
- `small_model`: Lightweight tasks like title generation
- `provider`: Provider-specific settings (timeout, setCacheKey, region, profile, endpoint)

### Interface Settings

```json
{
  "theme": "dark",
  "tui": {
    "scroll_speed": 3,
    "scroll_acceleration": 1.5,
    "diff_style": "unified"
  },
  "server": {
    "port": 3000,
    "hostname": "localhost",
    "mdns": true,
    "cors": true
  }
}
```

- `theme`: Visual appearance configuration
- `tui`: Terminal UI options (scroll_speed, scroll_acceleration, diff_style)
- `server`: Server settings (port, hostname, mdns, cors)

### Functionality

```json
{
  "tools": {
    "write": true,
    "bash": true
  },
  "agent": {
    "code-review": {
      "description": "Code review specialist",
      "model": "anthropic/claude-sonnet-4",
      "prompt": "You are a code review expert..."
    }
  },
  "default_agent": "code-review",
  "command": {
    "test": {
      "description": "Run tests",
      "command": "npm test"
    }
  },
  "keybinds": {
    "submit": "ctrl+enter"
  },
  "formatter": {
    "typescript": "prettier"
  },
  "permission": {
    "write": "ask",
    "bash": "allow"
  }
}
```

- `tools`: Enable/disable tools like write, bash
- `agent`: Define specialized agents for specific tasks
- `default_agent`: Agent used when none specified
- `command`: Custom commands for repetitive workflows
- `keybinds`: Keyboard shortcuts customization
- `formatter`: Code formatter configuration
- `permission`: Tool approval requirements ("ask", "allow", etc.)

### Features & Behavior

```json
{
  "share": "manual",
  "autoupdate": true,
  "compaction": "auto",
  "watcher": {
    "ignore": ["node_modules", ".git"]
  },
  "mcp": {
    "servers": {}
  },
  "plugin": [],
  "instructions": ["AGENTS.md", ".opencode/*.md"],
  "disabled_providers": [],
  "enabled_providers": [],
  "experimental": {}
}
```

- `share`: Sharing mode ("manual", "auto", "disabled")
- `autoupdate`: Auto-update behavior (true/false/"notify")
- `compaction`: Context management (auto, prune)
- `watcher`: File watching ignore patterns
- `mcp`: Model Context Protocol server configuration
- `plugin`: Load custom plugins from npm or local files
- `instructions`: Array of instruction files/glob patterns
- `disabled_providers`: Blocklist of providers
- `enabled_providers`: Allowlist of providers
- `experimental`: Unstable features under development

## Variable Substitution

### Environment Variables

Use `{env:VARIABLE_NAME}` syntax:

```json
{
  "model": "{env:OPENCODE_MODEL}"
}
```

### File Contents

Use `{file:path/to/file}` syntax:

```json
{
  "apiKey": "{file:~/.secrets/openai-key}"
}
```

File paths support relative (to config) and absolute paths (`/` or `~`).

## Provider-Specific Options

### Amazon Bedrock

Amazon Bedrock supports the following options:

- `region`: AWS region (defaults to AWS_REGION env or us-east-1)
- `profile`: AWS named profile
- `endpoint`: Custom VPC endpoint URL

**Note:** Bearer tokens take precedence over profile-based authentication.

```json
{
  "provider": {
    "amazon-bedrock": {
      "region": "us-west-2",
      "profile": "production",
      "endpoint": "https://bedrock.us-west-2.amazonaws.com"
    }
  }
}
```

## Agent Configuration

Agents can be defined inline in the config or via markdown files:

- Global: `~/.config/opencode/agent/`
- Project: `.opencode/agent/`

Configuration includes:
- `description`: What the agent does
- `model`: Which model to use
- `prompt`: System prompt for the agent
- Tool restrictions

## Important Notes

- The schema is defined at `opencode.ai/config.json`
- Experimental options are not stable. They may change or be removed without notice.
- `disabled_providers` takes priority over `enabled_providers`
- Sharing defaults to manual mode requiring explicit `/share` command

```

### modules/core/providers.md

```markdown
---
source: https://opencode.ai/docs/providers/
fetched: 2026-01-08
title: Providers
---

# OpenCode Providers Documentation

## Overview

OpenCode supports **75+ LLM providers** through the AI SDK and Models.dev, including local model execution. Setting up a provider requires two steps:

1. Add API keys via the `/connect` command (stored in `~/.local/share/opencode/auth.json`)
2. Configure the provider in your OpenCode config file

## Base URL Customization

Users can override default endpoints:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "anthropic": {
      "options": {
        "baseURL": "https://api.anthropic.com/v1"
      }
    }
  }
}
```

## OpenCode Zen

OpenCode Zen provides curated, tested models from the OpenCode team. Setup requires:

1. Run `/connect`, select opencode, visit opencode.ai/auth
2. Sign in and copy your API key
3. Use `/models` to view recommendations

## Provider Directory

### Amazon Bedrock

- Requires Model Catalog access
- Authentication via AWS credentials, profiles, or bearer tokens
- Supports VPC endpoints with custom endpoint configuration
- Priority: Bearer Token > AWS Credential Chain

```json
{
  "provider": {
    "amazon-bedrock": {
      "region": "us-west-2",
      "profile": "production",
      "endpoint": "https://bedrock-runtime.us-west-2.amazonaws.com"
    }
  }
}
```

### Anthropic

- Recommends Claude Pro/Max subscription
- Supports OAuth flow or manual API key entry
- Available via `/connect` command

```json
{
  "provider": {
    "anthropic": {
      "options": {
        "apiKey": "{env:ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

### Azure OpenAI

- Requires resource creation and model deployment
- Deployment names must match model names
- Set `AZURE_RESOURCE_NAME` environment variable

**Note:** "I'm sorry" errors may indicate content filter issues—switch from DefaultV2 to Default.

```json
{
  "provider": {
    "azure-openai": {
      "options": {
        "resourceName": "{env:AZURE_RESOURCE_NAME}",
        "apiKey": "{env:AZURE_OPENAI_API_KEY}"
      }
    }
  }
}
```

### Google Vertex AI

- Requires Google Cloud project with Vertex AI API enabled
- Key environment variables: `GOOGLE_CLOUD_PROJECT`, `VERTEX_LOCATION`
- Supports service account authentication
- "Global" region recommended for availability

```json
{
  "provider": {
    "google-vertex": {
      "options": {
        "project": "{env:GOOGLE_CLOUD_PROJECT}",
        "location": "us-central1"
      }
    }
  }
}
```

### Local Models (Ollama, LM Studio, llama.cpp)

Custom provider configuration example:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Ollama (local)",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      },
      "models": {
        "llama2": {
          "name": "Llama 2"
        }
      }
    }
  }
}
```

### Helicone

Observability platform supporting 17+ providers with automatic routing. Optional custom configuration:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "helicone": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Helicone",
      "options": {
        "baseURL": "https://ai-gateway.helicone.ai",
        "headers": {
          "Helicone-Cache-Enabled": "true",
          "Helicone-User-Id": "opencode"
        }
      }
    }
  }
}
```

Common headers include:
- `Helicone-Cache-Enabled`
- `Helicone-User-Id`
- `Helicone-Property-[Name]`
- `Helicone-Prompt-Id`
- `Helicone-Session-Id`

### OpenRouter & ZenMux

Support custom model additions via config:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {
        "somecoolnewmodel": {}
      }
    }
  }
}
```

OpenRouter supports provider routing:

```json
{
  "provider": {
    "openrouter": {
      "models": {
        "anthropic/claude-sonnet-4": {
          "options": {
            "order": ["baseten"],
            "allow_fallbacks": false
          }
        }
      }
    }
  }
}
```

### Vercel AI Gateway

```json
{
  "provider": {
    "vercel": {
      "models": {
        "anthropic/claude-sonnet-4": {
          "options": {
            "order": ["anthropic", "vertex"]
          }
        }
      }
    }
  }
}
```

Routing options: `order`, `only`, `zeroDataRetention`.

## Custom Provider (OpenAI-Compatible)

1. Run `/connect` and select "Other"
2. Enter unique provider ID
3. Enter API key
4. Configure in `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "myprovider": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Display Name",
      "options": {
        "baseURL": "https://api.myprovider.com/v1",
        "apiKey": "{env:CUSTOM_API_KEY}",
        "headers": {
          "Authorization": "Bearer token"
        }
      },
      "models": {
        "model-id": {
          "name": "Display Name",
          "limit": {
            "context": 200000,
            "output": 65536
          }
        }
      }
    }
  }
}
```

Key options: `baseURL`, `apiKey`, `headers`, and model `limit` (context/output tokens).

## Additional Providers Listed

- Amazon Bedrock
- Anthropic
- Azure OpenAI
- Azure Cognitive Services
- Baseten
- Cerebras
- Cloudflare AI Gateway
- Cortecs
- DeepSeek
- Deep Infra
- Fireworks AI
- GitHub Copilot
- Google Vertex AI
- Groq
- Hugging Face
- Helicone
- llama.cpp
- IO.NET
- LM Studio
- Moonshot AI
- MiniMax
- Nebius Token Factory
- Ollama
- Ollama Cloud
- OpenAI
- OpenRouter
- SAP AI Core
- OVHcloud AI Endpoints
- Together AI
- Venice AI
- Vercel AI Gateway
- xAI
- Z.AI
- ZenMux

## Troubleshooting

- Verify credentials: `opencode auth list`
- Confirm provider ID matches across `/connect` and config
- Validate npm package (use provider-specific or `@ai-sdk/openai-compatible`)
- Check `baseURL` correctness

```

### modules/core/network.md

```markdown
---
source: https://opencode.ai/docs/network/
fetched: 2026-01-08
title: Network Configuration
---

# Network Configuration Guide

OpenCode provides enterprise-grade networking support through proxy and certificate management.

## Proxy Setup

OpenCode recognizes standard proxy environment variables for routing traffic through corporate proxies:

```bash
export HTTPS_PROXY=https://proxy.example.com:8080
export HTTP_PROXY=http://proxy.example.com:8080
export NO_PROXY=localhost,127.0.0.1
```

**Critical requirement:** The `NO_PROXY` variable must exclude localhost addresses. The TUI relies on local HTTP server communication, and proxying this traffic creates routing loops.

## Authentication

For credential-protected proxies, embed credentials directly in the proxy URL:

```bash
export HTTPS_PROXY=http://username:password@proxy.example.com:8080
```

### Security Best Practices

The documentation advises against hardcoding passwords, recommending:
- Environment variables
- Secure credential management systems

### Advanced Authentication

For advanced authentication schemes like NTLM or Kerberos, using an LLM Gateway with native support is suggested.

## Custom Certificates

Enterprise environments using custom certificate authorities can configure OpenCode to trust them:

```bash
export NODE_EXTRA_CA_CERTS=/path/to/ca-cert.pem
```

This setting applies to both proxy connections and direct API communications.

## Server Configuration

OpenCode supports configurable server ports and hostnames via CLI flags:

```bash
opencode --port 3000 --hostname localhost
```

## Complete Example

```bash
# Proxy configuration
export HTTPS_PROXY=https://proxy.corporate.com:8080
export HTTP_PROXY=http://proxy.corporate.com:8080
export NO_PROXY=localhost,127.0.0.1,.internal.corp

# Custom CA certificate
export NODE_EXTRA_CA_CERTS=/etc/ssl/certs/corporate-ca.pem

# Start OpenCode
opencode
```

```

### modules/core/enterprise.md

```markdown
---
source: https://opencode.ai/docs/enterprise/
fetched: 2026-01-08
title: Enterprise
---

# OpenCode Enterprise

## Overview

OpenCode Enterprise enables organizations to keep code and data within their own infrastructure through centralized configuration, SSO integration, and internal AI gateway support.

## Trial & Data Security

**Key Statement:** "OpenCode does not store any of your code or context data."

All processing occurs locally or via direct API calls to the provider. The only exception is the optional `/share` feature, which sends conversations to opencode.ai's CDN.

### Recommended Trial Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "share": "disabled"
}
```

### Code Ownership

Users retain all rights to code produced by OpenCode with no licensing restrictions.

## Enterprise Features

### Central Config

Single organizational configuration managing SSO and AI gateway access.

### SSO Integration

Authentication through existing identity management systems to obtain gateway credentials.

### Internal AI Gateway

Route all requests exclusively through approved organizational infrastructure.

### Self-Hosting

Currently on roadmap; option to host share pages on internal infrastructure.

## Pricing Model

- Per-seat licensing
- No token charges if organizations maintain their own LLM gateway
- Custom quotes available upon contact

## Private NPM Registry Support

Organizations using private registries (JFrog Artifactory, Nexus, etc.) must authenticate developers before running OpenCode:

```bash
npm login --registry=https://your-company.jfrog.io/api/npm/npm-virtual/
```

**Important Requirement:** "You must be logged into the private registry before running OpenCode."

## Contact

Organizations interested in implementation should email contact@anoma.ly for pricing and deployment discussions.

```

### modules/core/troubleshooting.md

```markdown
---
source: https://opencode.ai/docs/troubleshooting/
fetched: 2026-01-08
title: Troubleshooting
---

# OpenCode Troubleshooting Guide

## Overview

OpenCode provides debugging tools through logs and local storage to help resolve issues.

## Logs Location

Log files are stored at:

| Platform | Path |
|----------|------|
| macOS/Linux | `~/.local/share/opencode/log/` |
| Windows | `%USERPROFILE%\.local\share\opencode\log\` |

Files use timestamp naming (e.g., `2025-01-09T123456.log`) with the 10 most recent retained.

### Enable Detailed Logging

```bash
opencode --log-level DEBUG
```

## Storage Location

Application data resides at:

| Platform | Path |
|----------|------|
| macOS/Linux | `~/.local/share/opencode/` |
| Windows | `%USERPROFILE%\.local\share\opencode` |

### Contents

- `auth.json` - API keys and OAuth tokens
- `log/` - Application logs
- `project/` - Session and message data
  - `./<project-slug>/storage/` for Git repos
  - `./global/storage/` otherwise

## Getting Help

### GitHub Issues

Report bugs at github.com/anomalyco/opencode/issues

### Discord Community

Real-time assistance available at opencode.ai/discord

## Common Issues & Solutions

### Won't Start

- Check logs
- Run with `--print-logs`
- Verify latest version via `opencode upgrade`

```bash
opencode --print-logs
opencode upgrade
```

### Authentication Problems

- Use `/connect` command in TUI
- Validate API keys
- Confirm network access to provider APIs

### Model Unavailable

- Confirm provider authentication
- Verify correct model naming format (`<providerId>/<modelId>`)
- Use `opencode models` command to check access

```bash
opencode models
```

### ProviderInitError

- Verify provider setup
- Clear configuration via `rm -rf ~/.local/share/opencode`
- Re-authenticate

```bash
rm -rf ~/.local/share/opencode
opencode
/connect
```

### API Call Errors

- Clear provider cache with `rm -rf ~/.cache/opencode`
- Restart to reinstall latest packages

```bash
rm -rf ~/.cache/opencode
opencode
```

### Linux Copy/Paste Issues

Install clipboard utilities based on your environment:

| Environment | Required Package |
|-------------|------------------|
| X11 | `xclip` or `xsel` |
| Wayland | `wl-clipboard` |
| Headless | `xvfb` |

```bash
# For X11
sudo apt install xclip

# For Wayland
sudo apt install wl-clipboard
```

```

### modules/core/migration-1.0.md

```markdown
---
source: https://opencode.ai/docs/1-0/
fetched: 2026-01-08
title: Migrating to 1.0
---

# OpenCode 1.0 Migration Guide

## Key Overview

OpenCode 1.0 represents "a complete rewrite of the TUI," transitioning from a Go/Bubbletea framework to an in-house solution built with Zig and SolidJS. The new version maintains compatibility with the existing OpenCode server while improving performance and capabilities.

## Upgrade Instructions

### Manual Upgrade

```bash
opencode upgrade 1.0.0
```

### Revert to Previous Version

```bash
opencode upgrade 0.15.31
```

**Note:** Some older versions automatically fetch the latest release.

## Notable UX Improvements

### Command Bar

The interface now includes "a command bar which almost everything flows through," accessible via `Ctrl+P`.

### Session History

Session history display has been simplified to focus on edit and bash tool details.

### Session Sidebar

A toggleable session sidebar provides additional context.

## Breaking Changes

### Renamed Keybinds

Four keybinds were renamed:

| Old Name | New Name |
|----------|----------|
| `messages_revert` | `messages_undo` |
| `switch_agent` | `agent_cycle` |
| `switch_agent_reverse` | `agent_cycle_reverse` |
| `switch_mode` | `agent_cycle` |

### Removed Keybinds

Twelve keybinds were removed entirely, including functionality for:

- Layout toggling
- File operations
- Help display
- Thinking block management

## Feedback

The documentation encourages users to report missing features via GitHub issues.

## Technical Details

### Framework Changes

- **Previous:** Go/Bubbletea
- **Current:** Zig and SolidJS (in-house solution)

### Compatibility

The new version maintains compatibility with the existing OpenCode server.

### Performance

The rewrite focuses on improving performance and capabilities.

```

### modules/usage/tui.md

```markdown
---
source: https://opencode.ai/docs/tui/
fetched: 2026-01-08
title: Terminal User Interface (TUI)
---

# OpenCode TUI Documentation

## Overview

OpenCode provides an interactive terminal interface (TUI) for working on projects with an LLM. Launch it by running `opencode` in your project directory or specify a path:

```bash
opencode
# or
opencode /path/to/project
```

## File References

Users can reference files in messages using `@` to perform fuzzy file searching within the current working directory.

**Example:**
```
How is auth handled in @packages/functions/src/api/index.ts?
```

The file content is automatically included in conversations.

## Bash Commands

Messages beginning with `!` execute shell commands. The command output is then added to the conversation as a tool result.

**Example:**
```
!npm test
```

## Commands (Slash Commands)

All commands use `/` prefix and most include `ctrl+x` keybind shortcuts (ctrl+x is the default leader key):

| Command | Aliases | Description | Keybind |
|---------|---------|-------------|---------|
| `/connect` | - | Add a provider and configure API keys | - |
| `/compact` | `/summarize` | Compact current session | `ctrl+x c` |
| `/details` | - | Toggle tool execution details | `ctrl+x d` |
| `/editor` | - | Open external editor for message composition | `ctrl+x e` |
| `/exit` | `/quit`, `/q` | Exit OpenCode | `ctrl+x q` |
| `/export` | - | Export conversation to Markdown | `ctrl+x x` |
| `/help` | - | Show help dialog | `ctrl+x h` |
| `/init` | - | Create/update AGENTS.md file | `ctrl+x i` |
| `/models` | - | List available models | `ctrl+x m` |
| `/new` | `/clear` | Start new session | `ctrl+x n` |
| `/redo` | - | Redo previously undone message | `ctrl+x r` |
| `/sessions` | `/resume`, `/continue` | List/switch sessions | `ctrl+x l` |
| `/share` | - | Share current session | `ctrl+x s` |
| `/themes` | - | List available themes | `ctrl+x t` |
| `/undo` | - | Undo last message and revert file changes | `ctrl+x u` |
| `/unshare` | - | Unshare current session | - |

## Editor Setup

The `EDITOR` environment variable controls which editor opens for `/editor` and `/export` commands.

### Linux/macOS

```bash
export EDITOR=nano
export EDITOR=vim
export EDITOR="code --wait"
```

### Windows CMD

```cmd
set EDITOR=notepad
set EDITOR="code --wait"
```

### Windows PowerShell

```powershell
$env:EDITOR = "notepad"
$env:EDITOR = "code --wait"
```

### Popular Editors

| Editor | Command |
|--------|---------|
| VS Code | `code` |
| Cursor | `cursor` |
| Windsurf | `windsurf` |
| Neovim | `nvim` |
| Vim | `vim` |
| Nano | `nano` |
| Sublime | `subl` |

**Note:** Some editors like VS Code need to be started with the `--wait` flag to block until closed.

## Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "tui": {
    "scroll_speed": 3,
    "scroll_acceleration": {
      "enabled": true
    }
  }
}
```

### Options

| Option | Description |
|--------|-------------|
| `scroll_acceleration` | Enable macOS-style scroll acceleration; takes precedence over scroll_speed |
| `scroll_speed` | Controls scroll speed (minimum: 1); defaults to 1 on Unix, 3 on Windows; ignored if scroll_acceleration is enabled |

## Customization

Users can customize TUI view settings through the command palette (`ctrl+x h` or `/help`). Settings persist across restarts.

### Username Display

Toggle whether your username appears in chat messages through the command palette by searching "username" or "hide username."

```

### modules/usage/cli.md

```markdown
---
source: https://opencode.ai/docs/cli/
fetched: 2026-01-08
title: Command Line Interface (CLI)
---

# OpenCode CLI Documentation

## Overview

OpenCode is a CLI tool that defaults to launching a terminal user interface (TUI) when run without arguments. It supports both interactive and programmatic modes, allowing users to interact with the system through various commands and flags.

## Basic Usage

```bash
# Launch TUI in current directory
opencode

# Launch TUI in specific directory
opencode /path/to/project

# Run non-interactive with a prompt
opencode run "Your prompt here"
```

## Global Flags

| Flag | Description |
|------|-------------|
| `--help` | Display help information |
| `--version` | Display version information |
| `--print-logs` | Print logs to stdout |
| `--log-level` | Set logging level (DEBUG, INFO, WARN, ERROR) |

## Core Commands

### Agent Management

Create, list, and manage custom agents with configurable system prompts and tool configurations.

```bash
opencode agent list
opencode agent create my-agent
```

### Authentication

Handle credentials across multiple providers.

```bash
opencode auth login
opencode auth list
opencode auth logout
```

Credentials are stored in `~/.local/share/opencode/auth.json`.

### Session Control

List, export, and import session data in JSON format.

```bash
opencode session list
opencode session export <session-id>
opencode session import <file.json>
```

Supports OpenCode share URLs for importing sessions.

### Model Operations

Display available models from configured providers in `provider/model` format.

```bash
opencode models
opencode models --refresh  # Refresh cache
```

### Server Modes

**Headless HTTP API Server:**
```bash
opencode serve
```

**HTTP Server with Web Browser Interface:**
```bash
opencode web
```

**Agent Client Protocol Server (stdin/stdout):**
```bash
opencode acp
```

### GitHub Integration

Install workflows and run agents within GitHub Actions environments.

```bash
opencode github install
opencode github run
```

### MCP Server Management

Add, list, and authenticate Model Context Protocol servers with OAuth support.

```bash
opencode mcp add <server-name>
opencode mcp list
opencode mcp auth <server-name>
```

## Non-Interactive Mode

The `opencode run` command enables non-interactive execution:

```bash
opencode run "Your prompt here"
```

"Run opencode in non-interactive mode by passing a prompt directly."

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OPENCODE_CONFIG_DIR` | Custom configuration directory |
| `OPENCODE_DISABLE_AUTOUPDATE` | Disable automatic updates |

## Advanced Features

### Attachment Capability

Connect to running servers to avoid cold boot times on subsequent invocations.

### Experimental Flags

Advanced features may be available through experimental flags. Check `--help` for current options.

## Examples

```bash
# Start TUI with debug logging
opencode --log-level DEBUG

# Run a quick prompt
opencode run "Explain the authentication flow in this codebase"

# Start headless server on custom port
opencode serve --port 8080

# List all available models
opencode models

# Export current session
opencode session export my-session > session.json
```

```

### modules/usage/ide.md

```markdown
---
source: https://opencode.ai/docs/ide/
fetched: 2026-01-08
title: IDE Integration
---

# OpenCode IDE Documentation

## Overview

This documentation covers the OpenCode extension for VS Code, Cursor, and other IDEs that support terminal integration.

## Keyboard Shortcuts

| Action | macOS | Windows/Linux |
|--------|-------|---------------|
| Quick launch | `Cmd+Esc` | `Ctrl+Esc` |
| New session | `Cmd+Shift+Esc` | `Ctrl+Shift+Esc` |
| Insert file reference | `Cmd+Option+K` | `Alt+Ctrl+K` |

### File References

The file reference shortcut enables quick insertion like `@File#L37-42`. The tool automatically shares current selection or active tab with OpenCode.

## Installation

### Automatic Installation

The extension installs automatically when running the `opencode` command in VS Code's integrated terminal.

### Manual Installation

1. Open the Extension Marketplace in your IDE
2. Search for "OpenCode"
3. Click Install

## Supported IDEs

- VS Code
- Cursor
- Windsurf
- Codium

## Troubleshooting

If automatic installation fails:

1. **Run in integrated terminal:** Run `opencode` specifically in the integrated terminal (not an external terminal)

2. **Verify CLI tools:** Ensure IDE CLI tools are installed:
   - `code` (VS Code)
   - `cursor` (Cursor)
   - `windsurf` (Windsurf)
   - `codium` (Codium)

3. **Install shell commands:** Use command palette (`Cmd+Shift+P` or `Ctrl+Shift+P`) to install shell commands in PATH

4. **Check permissions:** Confirm VS Code extension installation permissions are enabled

## Editor Configuration

For using custom editors with `/editor` or `/export` commands, set the `EDITOR` environment variable:

```bash
export EDITOR="code --wait"
```

The `--wait` flag ensures the editor blocks until the file is closed.

## Usage Tips

- Use file references to quickly share code context
- The extension automatically detects active selections
- Works with multiple IDE instances simultaneously

```

### modules/usage/zen.md

```markdown
---
source: https://opencode.ai/docs/zen/
fetched: 2026-01-08
title: OpenCode Zen
---

# OpenCode Zen Documentation

## Overview

OpenCode Zen is "a list of tested and verified models provided by the OpenCode team." It operates as an optional AI gateway offering curated models vetted for coding agent performance.

## Getting Started

### Authentication

1. Visit OpenCode Zen and authenticate
2. Add billing details
3. Retrieve your API key
4. Connect via the `/connect` command in the TUI

```bash
opencode
/connect
# Select "opencode"
# Enter your API key
```

## Available Models

The platform provides access to multiple model families:

### Premium Models
- GPT 5 series
- Claude variants (Sonnet, Haiku, Opus)
- Gemini options

### Open Source Models
- Qwen3
- Kimi

### Free Models (Beta)
- Grok Code
- GLM 4.7
- MiniMax M2.1
- Big Pickle

**Note:** Free models are available during beta periods.

## Pricing Structure

Pay-as-you-go model charging per 1M tokens:

| Model Tier | Input Token Price |
|------------|-------------------|
| Economy | $0.40 per 1M tokens |
| Standard | $3.00 per 1M tokens |
| Premium | $15.00 per 1M tokens |

Exact pricing varies by model.

## Team Features

### Workspaces

Workspaces support role-based access:

| Role | Capabilities |
|------|--------------|
| Admin | Manage models, members, and billing |
| Member | Control only personal API keys |

### Admin Controls

Admins can:
- Disable specific models
- Set spending limits per team member
- Manage workspace billing

### Bring Your Own Key (BYOK)

Users can integrate personal API keys while accessing other Zen models:

1. Add your OpenAI or Anthropic API key
2. Direct billing goes to your provider account
3. Access other Zen models with Zen billing

## Privacy & Data Handling

### Data Retention Policies

| Provider | Retention |
|----------|-----------|
| Most models | Zero retention |
| OpenAI | 30-day retention |
| Free-tier models (beta) | May use data for improvement |

### Hosting Location

All models are hosted in the US.

## Configuration

To use Zen models in your config:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "opencode/claude-sonnet-4"
}
```

## Best Practices

1. Start with recommended models from `/models`
2. Monitor usage through the Zen dashboard
3. Set spending limits for team members
4. Review data retention policies for sensitive projects

```

### modules/usage/share.md

```markdown
---
source: https://opencode.ai/docs/share/
fetched: 2026-01-08
title: Share Feature
---

# OpenCode Share Feature Documentation

## Overview

OpenCode enables users to create public links for conversations, facilitating collaboration.

**Important:** "Shared conversations are publicly accessible to anyone with the link."

## How It Works

The sharing mechanism operates in three steps:

1. Generates a unique public URL for the session
2. Synchronizes conversation history to OpenCode servers
3. Makes content accessible via the format `opncd.ai/s/<share-id>`

## Sharing Modes

### Manual (Default)

Users must explicitly invoke `/share` to generate a shareable URL copied to clipboard.

```json
{
  "$schema": "https://opencode.ai/config.json",
  "share": "manual"
}
```

### Auto-Share

Enabling automatic sharing shares all new conversations automatically:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "share": "auto"
}
```

### Disabled

Completely prevents sharing functionality:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "share": "disabled"
}
```

## Un-sharing

The `/unshare` command removes public access and deletes associated conversation data.

```bash
/unshare
```

## Privacy Considerations

### Data Retention

Shared conversations persist until explicitly unshared, including:
- Full conversation history
- Metadata

### Key Recommendations

1. **Review before sharing:** Check content for sensitive information
2. **Avoid sensitive data:** Do not share conversations containing:
   - Proprietary code
   - Confidential data
   - API keys or credentials
3. **Unshare after collaboration:** Remove access when no longer needed
4. **Disable for sensitive projects:** Use `"share": "disabled"` for confidential work

## Enterprise Options

For organizational deployments, sharing can be:

| Option | Description |
|--------|-------------|
| Disabled | Completely prevent sharing for compliance |
| SSO-restricted | Allow only authenticated users |
| Self-hosted | Host share pages on internal infrastructure |

## Commands Reference

| Command | Description |
|---------|-------------|
| `/share` | Generate shareable link (copies to clipboard) |
| `/unshare` | Remove public access and delete data |

## URL Format

Shared conversations use the format:
```
https://opncd.ai/s/<share-id>
```

```

### modules/usage/github.md

```markdown
---
source: https://opencode.ai/docs/github/
fetched: 2026-01-08
title: GitHub Integration
---

# OpenCode GitHub Integration

## Overview

OpenCode enables AI-powered automation within GitHub workflows. The platform integrates with issues, pull requests, and scheduled tasks through GitHub Actions.

## Core Capabilities

### Triggering OpenCode

Mention `/opencode` or `/oc` in your comment, and OpenCode will execute tasks within your GitHub Actions runner.

### Common Use Cases

- Request issue explanations
- Implement fixes on new branches with PR submissions
- Make targeted code changes
- Automate code reviews
- Schedule repository maintenance

## Installation Methods

### Automated Setup

```bash
opencode github install
```

### Manual Configuration

1. Install the GitHub app at `github.com/apps/opencode-agent`
2. Add a workflow YAML file to `.github/workflows/opencode.yml`
3. Store API keys in GitHub Secrets

### Workflow YAML Example

```yaml
name: OpenCode Agent

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  issues:
    types: [opened, edited]
  pull_request:
    types: [opened, synchronize]
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

jobs:
  opencode:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: opencode-ai/opencode-action@v1
        with:
          model: anthropic/claude-sonnet-4
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

## Configuration Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `model` | Yes | Specifies the AI model in `provider/model` format |
| `agent` | No | Selects the processing agent; defaults to `default_agent` from config |
| `share` | No | Controls session sharing (defaults to true for public repos) |
| `prompt` | No | Customizes behavior through instructions |
| `token` | No | Provides GitHub authentication; uses installation token by default |

## Supported Event Types

### Issue Comments

Responds to comments on issues with full thread context.

```yaml
on:
  issue_comment:
    types: [created]
```

**Usage:**
```
/opencode Explain this issue and suggest a fix
```

### Pull Request Review Comments

Responds to code review comments with line numbers and diffs.

```yaml
on:
  pull_request_review_comment:
    types: [created]
```

**Usage:**
```
/oc Can you improve this function?
```

### Issues Creation/Editing

Automatically processes new or updated issues.

```yaml
on:
  issues:
    types: [opened, edited]
```

### Pull Request Events

Responds to PR opens and updates.

```yaml
on:
  pull_request:
    types: [opened, synchronize]
```

### Scheduled Tasks (Cron)

Runs on a schedule for maintenance tasks.

```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
```

### Manual Dispatch

Trigger workflows manually from GitHub UI.

```yaml
on:
  workflow_dispatch:
```

## Common Applications

### Code Review Automation

```
/opencode Review this PR for security issues and best practices
```

### Issue Triage

```
/oc Categorize this issue and suggest priority
```

### Feature Implementation

```
/opencode Implement the feature described in this issue and create a PR
```

### Repository Maintenance

Schedule daily checks for outdated dependencies, security vulnerabilities, or code quality issues.

## Best Practices

1. **Store secrets securely:** Use GitHub Secrets for API keys
2. **Limit permissions:** Use minimal required permissions
3. **Review generated code:** Always review AI-generated changes before merging
4. **Set appropriate triggers:** Choose triggers that match your workflow

```

### modules/usage/gitlab.md

```markdown
---
source: https://opencode.ai/docs/gitlab/
fetched: 2026-01-08
title: GitLab Integration
---

# GitLab Integration for OpenCode

## Overview

OpenCode provides two integration methods for GitLab workflows:

1. GitLab CI/CD Pipeline
2. GitLab Duo Integration

## GitLab CI/CD Pipeline

OpenCode functions as a standard GitLab pipeline component using the community-maintained [nagyv/gitlab-opencode](https://gitlab.com/nagyv/gitlab-opencode) component.

### Key Capabilities

- Custom configuration per job invocation
- Streamlined setup with minimal configuration needs
- Flexible input parameters for behavior customization

### Implementation

1. Store authentication JSON as masked CI variables
2. Add component reference to `.gitlab-ci.yml`
3. Specify configuration directory and prompts

### Example Configuration

```yaml
include:
  - component: gitlab.com/nagyv/gitlab-opencode/opencode@main
    inputs:
      config_dir: .opencode
      prompt: "Review this merge request"

variables:
  OPENCODE_AUTH: $OPENCODE_AUTH_JSON
```

## GitLab Duo Integration

OpenCode operates within GitLab CI/CD pipelines, activated by mentioning `@opencode` in comments.

### Supported Functions

- Issue analysis and explanation
- Automated fixes and feature implementation via merge requests
- Code review capabilities

### Setup Requirements

1. **GitLab Environment Configuration**
   - Configure GitLab CI/CD environment
   - Set up pipeline triggers

2. **API Key Acquisition**
   - Obtain API key from an AI model provider

3. **Service Account Creation**
   - Create a dedicated service account for OpenCode

4. **CI Variable Configuration**
   - Store credentials as protected CI variables

5. **Flow Configuration**
   - Create configuration file with authentication steps

### Usage Examples

#### Request Issue Explanation

```
@opencode Explain this issue and identify the root cause
```

#### Implement Fixes

```
@opencode Implement a fix for this issue
```

The assistant handles:
- Branch creation
- Code implementation
- Merge request generation

#### Review Merge Requests

```
@opencode Review this merge request for security and performance
```

## Configuration File Example

```yaml
# .gitlab-ci.yml
stages:
  - opencode

opencode:
  stage: opencode
  image: node:20
  script:
    - npm install -g opencode
    - opencode run "$PROMPT"
  variables:
    ANTHROPIC_API_KEY: $ANTHROPIC_API_KEY
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
```

## Best Practices

1. **Use protected variables:** Store API keys as protected CI variables
2. **Limit branch access:** Restrict which branches can trigger OpenCode
3. **Review generated code:** Always review AI-generated merge requests
4. **Set appropriate permissions:** Use minimal required GitLab permissions

```

### modules/configure/tools.md

```markdown
---
source: https://opencode.ai/docs/tools/
fetched: 2026-01-08
title: Tools Configuration
---

# OpenCode Tools Documentation

## Overview

OpenCode provides a comprehensive tools system enabling LLMs to interact with your codebase. The platform includes built-in tools and supports extensibility through custom tools and MCP servers.

## Configuration Options

### Global Setup

Tools are enabled by default. Disable specific tools via the config:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "tools": {
    "write": false,
    "bash": false,
    "webfetch": true
  }
}
```

Wildcard patterns control multiple tools:

```json
{
  "tools": {
    "mymcp_*": false
  }
}
```

This disables all tools from that MCP server.

### Per-Agent Override

Agent-specific configurations supersede global settings:

```json
{
  "agent": {
    "plan": {
      "tools": {
        "write": false,
        "bash": false
      }
    }
  }
}
```

## Built-in Tools

| Tool | Purpose |
|------|---------|
| `bash` | Execute shell commands in your project environment |
| `edit` | Precise file modifications through exact string replacement |
| `write` | Create new or overwrite existing files |
| `read` | Retrieve file contents with line-range support |
| `grep` | Search file contents using regular expressions |
| `glob` | Pattern-based file discovery, sorted by modification time |
| `list` | Directory enumeration with glob pattern filtering |
| `lsp` | Code intelligence (experimental; requires flag) |
| `patch` | Apply patch files to codebases |
| `skill` | Load skill files (SKILL.md) into conversations |
| `todowrite` | Track multi-step task progress |
| `todoread` | Retrieve current task list state |
| `webfetch` | Retrieve web content for documentation lookup |

## Tool Descriptions

### bash

Execute shell commands in your project environment. Useful for running tests, build commands, and system operations.

### edit

Precise file modifications through exact string replacement. Matches and replaces specific text patterns.

### write

Create new files or overwrite existing files with specified content.

### read

Retrieve file contents with optional line-range support for large files.

### grep

Search file contents using regular expressions. Supports pattern matching across multiple files.

### glob

Pattern-based file discovery. Returns files matching glob patterns, sorted by modification time.

### list

Directory enumeration with optional glob pattern filtering.

### lsp

Code intelligence features including:
- Go to definition
- Find references
- Symbol search

**Note:** Experimental feature requiring `--experimental-lsp` flag.

### patch

Apply unified diff patch files to codebases.

### skill

Load skill files (SKILL.md) into conversations for specialized knowledge.

### todowrite / todoread

Track and retrieve multi-step task progress.

### webfetch

Retrieve web content for documentation lookup and research.

## Advanced Configuration

### Ignore Patterns

The system uses ripgrep, respecting `.gitignore` by default. Override with `.ignore`:

```
!node_modules/
!dist/
!build/
```

### Extensibility

**Custom Tools:**
User-defined functions in config files. See Custom Tools documentation.

**MCP Servers:**
External services and database integrations. See MCP Servers documentation.

## Permissions

Permissions can be configured separately to require approval before tool execution:

```json
{
  "permission": {
    "edit": "ask",
    "bash": "ask",
    "write": "allow"
  }
}
```

Options:
- `"ask"`: Require user approval
- `"allow"`: Auto-approve
- `"deny"`: Block tool usage

```

### modules/configure/rules.md

```markdown
---
source: https://opencode.ai/docs/rules/
fetched: 2026-01-08
title: Rules and Instructions
---

# Rules Documentation - OpenCode

## Overview

OpenCode allows developers to customize LLM behavior through custom instructions. The primary mechanism is an `AGENTS.md` file that functions similarly to "CLAUDE.md" or Cursor's rules, containing project-specific guidance.

## Key Features

### Initialization

Running the `/init` command scans your project and generates an `AGENTS.md` file with auto-detected context.

```bash
/init
```

### File Locations

| Type | Location | Purpose |
|------|----------|---------|
| Project-level | `AGENTS.md` in repository root | Version-controlled team rules |
| Global | `~/.config/opencode/AGENTS.md` | Personal rules across projects |

### Configuration Support

The `opencode.json` file accepts an `instructions` field that references external rule files:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    "packages/*/AGENTS.md",
    ".opencode/instructions/*.md"
  ]
}
```

Supports glob patterns for flexible file discovery.

## Implementation Example

A typical project `AGENTS.md` establishes:
- Monorepo structure
- Code standards
- Naming conventions
- Import conventions
- Framework-specific guidance

### Example AGENTS.md

```markdown
# Project Instructions

## Architecture
This is an SST v3 monorepo with the following structure:
- `packages/core/` - Shared business logic
- `packages/functions/` - Lambda handlers
- `packages/web/` - React frontend

## Code Standards
- Use TypeScript strict mode
- Prefer functional components
- Use barrel exports from index.ts

## Conventions
- Import order: external, internal, relative
- Use named exports over default exports
- File names: kebab-case
```

## Precedence Rules

OpenCode searches hierarchically:

1. Local files first (traversing upward from current directory)
2. Then global configuration

Both global and project rules combine when present.

## Advanced Patterns

### Lazy Loading

Developers can implement "lazy loading" of external references by instructing the AI to read specific files "on a need-to-know basis" rather than preemptively.

```markdown
## Extended Documentation
When working on authentication, read @docs/auth-patterns.md
When working on database, read @docs/db-conventions.md
```

This keeps configuration modular while maintaining focused context.

### Multiple Instruction Files

Use glob patterns to load multiple instruction files:

```json
{
  "instructions": [
    "AGENTS.md",
    ".opencode/rules/*.md",
    "packages/*/AGENTS.md"
  ]
}
```

### Conditional Rules

Structure rules for specific contexts:

```markdown
## When Working on API
- Use zod for validation
- Follow REST conventions
- Include error handling

## When Working on Frontend
- Use React Query for data fetching
- Follow component composition patterns
- Include loading states
```

```

### modules/configure/agents.md

```markdown
---
source: https://opencode.ai/docs/agents/
fetched: 2026-01-08
title: Agents Configuration
---

# OpenCode Agents Documentation

## Overview

OpenCode provides specialized AI assistants called agents that can be configured for specific tasks and workflows. They enable focused tools with custom prompts, models, and tool access.

## Agent Types

### Primary Agents

Main assistants you interact with directly. Switch between them using Tab or configured keybinds.

**Built-in Primary Agents:**
- **Build**: Default agent with all tools enabled for full development work
- **Plan**: Restricted agent for analysis (file edits and bash set to "ask")

### Subagents

Specialized assistants that primary agents can invoke. Manually invoked via @ mentions.

**Built-in Subagents:**
- **General**: Research and multi-step tasks
- **Explore**: Fast codebase exploration

## Built-in Agents Summary

| Agent | Type | Description |
|-------|------|-------------|
| Build | Primary | All tools enabled for full development work |
| Plan | Primary | Restricted permissions for analysis |
| General | Subagent | Researching questions and executing multi-step tasks |
| Explore | Subagent | Quick codebase exploration |

## Usage

### Switching Primary Agents

- Press `Tab` to cycle through primary agents
- Use `switch_agent` keybind

### Invoking Subagents

- Automatic: Primary agents invoke subagents as needed
- Manual: Use `@general` or `@explore` syntax

### Navigation

- `<Leader>+Right`: Navigate to child session
- `<Leader>+Left`: Navigate to parent session
- Configure with `session_child_cycle` keybinds

## Configuration Methods

### JSON Configuration

In `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "security-auditor": {
      "description": "Security vulnerability scanner",
      "model": "anthropic/claude-sonnet-4",
      "temperature": 0.3,
      "maxSteps": 20,
      "tools": {
        "write": false,
        "bash": false
      },
      "permission": {
        "edit": "deny"
      }
    }
  }
}
```

### Markdown Files

Create markdown files in:
- Global: `~/.config/opencode/agent/`
- Per-project: `.opencode/agent/`

**Example: `.opencode/agent/docs-writer.md`**

```markdown
---
description: Technical documentation writer
model: anthropic/claude-sonnet-4
temperature: 0.7
---

You are a technical documentation specialist.

Focus on:
- Clear, concise writing
- Code examples
- API documentation
- User guides
```

## Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| `description` | Brief description (required) | - |
| `temperature` | Randomness (0.0-1.0) | Model default |
| `maxSteps` | Max iterations before text-only response | Unlimited |
| `disable` | Set true to disable | false |
| `prompt` | Custom system prompt or file path | - |
| `model` | Override global model | Global model |
| `tools` | Enable/disable specific tools | All enabled |
| `permission` | Tool permissions (ask/allow/deny) | - |
| `mode` | "primary", "subagent", or "all" | "all" |
| `hidden` | Hide from @ autocomplete | false |
| `taskPermissions` | Control subagent invocation | - |

### Tools Configuration

```json
{
  "agent": {
    "readonly": {
      "tools": {
        "write": false,
        "edit": false,
        "bash": false,
        "mcp_*": false
      }
    }
  }
}
```

Supports wildcards for pattern matching.

### Permissions Configuration

```json
{
  "agent": {
    "careful": {
      "permission": {
        "edit": "ask",
        "bash": "ask",
        "webfetch": "allow"
      }
    }
  }
}
```

## Creating Agents

### Interactive Wizard

```bash
opencode agent create
```

The wizard handles:
1. Location selection (global/project)
2. Description input
3. System prompt generation
4. Tool selection
5. Markdown file creation

## Example Use Cases

### Documentation Agent

```json
{
  "agent": {
    "docs": {
      "description": "Technical writing specialist",
      "model": "anthropic/claude-sonnet-4",
      "temperature": 0.7,
      "tools": {
        "bash": false
      }
    }
  }
}
```

### Security Auditor

```json
{
  "agent": {
    "security": {
      "description": "Vulnerability identification",
      "model": "anthropic/claude-opus-4",
      "temperature": 0.1,
      "permission": {
        "edit": "deny",
        "write": "deny"
      }
    }
  }
}
```

### Code Reviewer

```json
{
  "agent": {
    "reviewer": {
      "description": "Code review with read-only access",
      "tools": {
        "write": false,
        "edit": false,
        "bash": false
      }
    }
  }
}
```

### Debug Agent

```json
{
  "agent": {
    "debug": {
      "description": "Issue investigation specialist",
      "model": "anthropic/claude-sonnet-4",
      "prompt": "Focus on debugging and root cause analysis."
    }
  }
}
```

```

### modules/configure/models.md

```markdown
---
source: https://opencode.ai/docs/models/
fetched: 2026-01-08
title: Models Configuration
---

# OpenCode Models Documentation

## Overview

OpenCode explains how to configure LLM providers and models, supporting 75+ providers through the AI SDK and Models.dev integration.

## Providers

OpenCode preloads popular providers by default. Users can add credentials via the `/connect` command to enable additional providers.

```bash
/connect
```

## Model Selection

Access model selection through the `/models` command after configuring your provider:

```bash
/models
```

## Recommended Models

The following models are recommended for code generation and tool calling:

| Model | Provider |
|-------|----------|
| GPT 5.2 | OpenAI |
| GPT 5.1 Codex | OpenAI |
| Claude Opus 4.5 | Anthropic |
| Claude Sonnet 4.5 | Anthropic |
| Minimax M2.1 | Minimax |
| Gemini 3 Pro | Google |

## Default Configuration

Set default models via the config file using the format `provider_id/model_id`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4",
  "small_model": "anthropic/claude-haiku-3-5"
}
```

For local models:

```json
{
  "model": "lmstudio/google/gemma-3n-e4b"
}
```

## Model Options Configuration

### Global Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "openai/gpt-5",
  "provider": {
    "openai": {
      "models": {
        "gpt-5": {
          "options": {
            "reasoningEffort": "high"
          }
        }
      }
    }
  }
}
```

### Agent-Specific Configuration

```json
{
  "agent": {
    "deep-thinker": {
      "model": "openai/gpt-5",
      "modelOptions": {
        "reasoningEffort": "xhigh"
      }
    }
  }
}
```

## Variants System

Models support multiple configuration variants for different use cases.

### Built-in Variants

**Anthropic:**
- `high` (default): Standard thinking budget
- `max`: Maximum thinking budget

**OpenAI:**
- `none`: No reasoning
- `low`: Low reasoning effort
- `medium`: Medium reasoning effort
- `high`: High reasoning effort (default)
- `xhigh`: Extra high reasoning effort

**Google:**
- `low`: Low effort level
- `high`: High effort level

### Custom Variants

Override existing variants or create custom ones:

```json
{
  "provider": {
    "anthropic": {
      "models": {
        "claude-sonnet-4": {
          "variants": {
            "quick": {
              "options": {
                "thinkingBudget": 1000
              }
            },
            "deep": {
              "options": {
                "thinkingBudget": 50000
              }
            }
          }
        }
      }
    }
  }
}
```

### Switching Variants

Use the `variant_cycle` keybind to switch between variants during a session.

## Model Loading Priority

Models are selected in the following order:

1. **Command-line flag**: `--model` or `-m`
2. **Config file**: `model` field in `opencode.json`
3. **Last used model**: Persisted from previous session
4. **Internal default**: Built-in priority list

## Examples

### Using Multiple Models

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "anthropic/claude-sonnet-4",
  "small_model": "anthropic/claude-haiku-3-5",
  "agent": {
    "complex-tasks": {
      "model": "anthropic/claude-opus-4"
    },
    "quick-tasks": {
      "model": "anthropic/claude-haiku-3-5"
    }
  }
}
```

### Local Model Configuration

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/llama2",
  "provider": {
    "ollama": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "http://localhost:11434/v1"
      }
    }
  }
}
```

```

### modules/configure/themes.md

```markdown
---
source: https://opencode.ai/docs/themes/
fetched: 2026-01-08
title: Themes Configuration
---

# OpenCode Themes Documentation

## Overview

OpenCode allows users to select and customize themes. Users can choose from built-in themes, use a system-adaptive theme, or create custom ones.

## Terminal Requirements

Your terminal must support **truecolor** (24-bit color) for proper theme display.

### Verify Support

```bash
echo $COLORTERM
```

Should output `truecolor` or `24bit`.

### Enable Truecolor

Add to your shell profile:

```bash
export COLORTERM=truecolor
```

### Supported Terminals

Modern terminals with truecolor support:
- iTerm2
- Alacritty
- Kitty
- Windows Terminal
- WezTerm
- Ghostty

## Built-in Themes

| Theme | Description |
|-------|-------------|
| `system` | Adapts to terminal background |
| `tokyonight` | Based on popular editor theme |
| `everforest` | Nature-inspired color palette |
| `ayu` | Clean and modern design |
| `catppuccin` | Soothing pastel theme |
| `catppuccin-macchiato` | Catppuccin variant |
| `gruvbox` | Retro groove color scheme |
| `kanagawa` | Japanese-inspired palette |
| `nord` | Arctic, north-bluish palette |
| `matrix` | Green-on-black hacker style |
| `one-dark` | Atom-inspired theme |

## System Theme

The system theme automatically adapts to your terminal's color scheme:
- Generates custom gray scales
- Uses ANSI colors (0-15)
- Preserves terminal defaults using `none` values

## Configuration

### Using /theme Command

```bash
/themes
```

Select from available themes interactively.

### Config File

```json
{
  "$schema": "https://opencode.ai/config.json",
  "theme": "tokyonight"
}
```

## Custom Theme Creation

### Directory Hierarchy (Priority Order)

1. Built-in themes (highest priority)
2. `~/.config/opencode/themes/*.json` (global)
3. `./.opencode/themes/*.json` (project root)
4. `./.opencode/themes/*.json` (current directory)

### JSON Format

Custom themes support:
- Hex colors: `#ffffff`
- ANSI values: `0-255`
- Color references: Reference defined colors
- Dark/light variants
- `"none"`: Terminal defaults

### Example Theme Structure

```json
{
  "$schema": "https://opencode.ai/theme.json",
  "defs": {
    "nord0": "#2E3440",
    "nord1": "#3B4252",
    "nord2": "#434C5E",
    "nord3": "#4C566A",
    "nord4": "#D8DEE9",
    "nord5": "#E5E9F0",
    "nord6": "#ECEFF4",
    "nord7": "#8FBCBB",
    "nord8": "#88C0D0",
    "nord9": "#81A1C1",
    "nord10": "#5E81AC",
    "nord11": "#BF616A",
    "nord12": "#D08770",
    "nord13": "#EBCB8B",
    "nord14": "#A3BE8C",
    "nord15": "#B48EAD"
  },
  "theme": {
    "primary": {
      "dark": "nord8",
      "light": "nord10"
    },
    "secondary": {
      "dark": "nord9",
      "light": "nord9"
    },
    "accent": {
      "dark": "nord7",
      "light": "nord7"
    },
    "error": {
      "dark": "nord11",
      "light": "nord11"
    },
    "warning": {
      "dark": "nord13",
      "light": "nord13"
    },
    "success": {
      "dark": "nord14",
      "light": "nord14"
    },
    "text": {
      "dark": "nord4",
      "light": "nord0"
    },
    "background": {
      "dark": "nord0",
      "light": "nord6"
    },
    "border": {
      "dark": "nord3",
      "light": "nord4"
    },
    "diff": {
      "add": {
        "dark": "nord14",
        "light": "nord14"
      },
      "remove": {
        "dark": "nord11",
        "light": "nord11"
      }
    },
    "syntax": {
      "keyword": "nord9",
      "string": "nord14",
      "number": "nord15",
      "comment": "nord3",
      "function": "nord8",
      "variable": "nord4"
    }
  }
}
```

### Theme Properties

| Property | Description |
|----------|-------------|
| `primary` | Main accent color |
| `secondary` | Secondary accent |
| `accent` | Highlight color |
| `error` | Error messages |
| `warning` | Warning messages |
| `success` | Success messages |
| `text` | Default text |
| `background` | Background color |
| `border` | Border color |
| `diff.add` | Added lines in diffs |
| `diff.remove` | Removed lines in diffs |
| `syntax.*` | Syntax highlighting colors |

```

### modules/configure/keybinds.md

```markdown
---
source: https://opencode.ai/docs/keybinds/
fetched: 2026-01-08
title: Keybinds Configuration
---

# OpenCode Keybinds Documentation

## Overview

OpenCode allows customization of keyboard shortcuts through the configuration file.

## Leader Key Concept

OpenCode implements a leader key system, defaulting to `ctrl+x`, which users press before the actual command shortcut.

This design "avoids conflicts in your terminal."

## Configuration Method

Customize keybinds through the `opencode.json` file using the `"keybinds"` object:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "keybinds": {
    "leader": "ctrl+x",
    "submit": "return",
    "cancel": "escape"
  }
}
```

## Notable Default Bindings

| Action | Default Keybind |
|--------|-----------------|
| Leader key | `ctrl+x` |
| New session | `<leader>n` |
| Model list | `<leader>m` |
| Agent list | `<leader>a` |
| Command list | `ctrl+p` |
| Input submission | `return` |
| Add newline | `shift+return`, `ctrl+return`, `alt+return`, `ctrl+j` |

## Common Keybinds

| Action | Keybind |
|--------|---------|
| `session_new` | `<leader>n` |
| `session_list` | `<leader>l` |
| `model_list` | `<leader>m` |
| `agent_list` | `<leader>a` |
| `command_list` | `ctrl+p` |
| `help` | `<leader>h` |
| `compact` | `<leader>c` |
| `details` | `<leader>d` |
| `editor` | `<leader>e` |
| `export` | `<leader>x` |
| `share` | `<leader>s` |
| `themes` | `<leader>t` |
| `undo` | `<leader>u` |
| `redo` | `<leader>r` |
| `quit` | `<leader>q` |

## Disabling Bindings

Set any keybind value to `"none"` to disable it:

```json
{
  "keybinds": {
    "share": "none",
    "export": "none"
  }
}
```

## Built-in Readline/Emacs Shortcuts

The desktop prompt supports standard text editing shortcuts. These are **not configurable** through `opencode.json`:

| Shortcut | Action |
|----------|--------|
| `ctrl+a` | Move to line start |
| `ctrl+e` | Move to line end |
| `ctrl+w` | Delete word backward |
| `ctrl+u` | Delete to line start |
| `ctrl+k` | Delete to line end |
| `ctrl+b` | Move backward one character |
| `ctrl+f` | Move forward one character |
| `alt+b` | Move backward one word |
| `alt+f` | Move forward one word |

## Platform-Specific Configuration

### Windows Terminal

Users may need to manually configure escape sequence handling for `Shift+Enter` functionality.

Add to Windows Terminal `settings.json`:

```json
{
  "actions": [
    {
      "command": {
        "action": "sendInput",
        "input": "\u001b[13;2u"
      },
      "keys": "shift+enter"
    }
  ]
}
```

## Custom Keybind Examples

### Minimal Configuration

```json
{
  "keybinds": {
    "submit": "ctrl+enter",
    "cancel": "ctrl+c"
  }
}
```

### Vim-Style Configuration

```json
{
  "keybinds": {
    "leader": "space",
    "session_new": "<leader>n",
    "session_list": "<leader>b"
  }
}
```

## Full Keybind Reference

OpenCode provides 80+ configurable actions. Use `/help` or check the documentation for the complete list.

## Best Practices

1. **Keep leader key default**: `ctrl+x` avoids most conflicts
2. **Test in your terminal**: Some terminals intercept certain key combinations
3. **Document custom bindings**: Keep a reference for team consistency

```

