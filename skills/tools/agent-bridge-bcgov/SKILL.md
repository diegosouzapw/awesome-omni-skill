---
name: agent-bridge
description: Bridge plugin capabilities (commands, skills, agents, hooks, MCP) to specific agent environments (Claude Code, GitHub Copilot, Gemini, Antigravity). Use this skill when converting or installing a plugin to a target runtime.
allowed-tools: Bash, Write, Read
---

# Agent Bridge

## Overview
This skill **adapts and transforms** plugin content into the specific formats required by different AI agent environments. It ensures each runtime can see and use the plugin's capabilities in its native format.

## Prerequisite
The auto-detect mode only targets **existing** directories. Create them first:
```bash
mkdir .agent .github .gemini .claude
```
> If no directories are found, the installer will print this exact error with the mkdir command.

## Usage

> **CRITICAL**: You must **never** use `--target auto`. You must explicitly specify your own runtime environment (e.g., `antigravity`, `claude`, `gemini`, `github`) to avoid polluting other IDEs.

**Universal Target Support (Agent Awareness)**:
You are a Universal Translator. You are not limited to the primary examples. If you identify your host system as ANY of the following platforms (or similar ones), you MUST invoke the installer with that specific target name (e.g. `--target roo`).
*Supported Environments:* amp, codex, cursor, gemini cli, github copilot, kimi code cli, opencode, antigravity, augment, claude code, openclaw, cline, codebuddy, command code, continue, cortex code, crush, droid, goose, junie, iflow cli, kiko code, kiro cli, kode, mcpjam, mistral vibe, mux, openhands, pi, qoder, qwen code, roo code, trae, trae cn, windsurf, zencoder, neovate, pochi, adal.

### Bridge a Single Plugin
```bash
# Bridge to Claude Code specifically
python ${CLAUDE_PLUGIN_ROOT}/skills/agent-bridge/scripts/bridge_installer.py --plugin <plugin-path> --target claude

# Bridge to Antigravity specifically
python ${CLAUDE_PLUGIN_ROOT}/skills/agent-bridge/scripts/bridge_installer.py --plugin <plugin-path> --target antigravity
```

**Example:**
```bash
python ${CLAUDE_PLUGIN_ROOT}/skills/agent-bridge/scripts/bridge_installer.py --plugin plugins/my-plugin --target antigravity
```

### Bridge All Plugins
```bash
# Explicitly bridge all plugins to your specific environment
python ${CLAUDE_PLUGIN_ROOT}/skills/agent-bridge/scripts/install_all_plugins.py --target gemini
```

---

## Component Mapping Matrix

The bridge intelligently maps plugin source components to the correct file extensions, directories, and architectures expected by the agent environment.

| Target Environment | `commands/*.md` | `skills/` | `agents/*.md` | `rules/` | `hooks/hooks.json` | `.mcp.json` |
|-------------------|----------------|-----------|---------------|----------|-------------------|-------------|
| **Claude Code** (`.claude/`) | `commands/*.md` | `skills/` | `skills/<plugin>/agents/` | Appended to `./CLAUDE.md` | `hooks/<plugin>-hooks.json` | Merged (`./.mcp.json`) |
| **GitHub Copilot** (`.github/`) | `prompts/*.prompt.md` | `skills/` | `agents/*.agent.md` | Appended to `.github/copilot-instructions.md` | *(Ignored)* | Merged (`./.mcp.json`) |
| **Google Gemini** (`.gemini/`) | `commands/*.toml` | `skills/` | `skills/<plugin>/agents/` | Appended to `./GEMINI.md` | *(Ignored)* | Merged (`./.mcp.json`) |
| **Antigravity** (`.agent/`) | `workflows/*.md` | `skills/` | `skills/<plugin>/agents/` | `.agent/rules/` | *(Ignored)* | Merged (`./.mcp.json`) |
| **Universal Generic** (`.<target>/`) | `commands/*.md` | `skills/` | `skills/<plugin>/agents/` | `.<target>/rules/` | *(Ignored)* | Merged (`./.mcp.json`) |

> **GitHub Copilot — Two Agent Types:** The `agents/*.agent.md` column for GitHub Copilot covers two distinct use cases:
> - **IDE / UI Agents**: `.github/agents/name.agent.md` + `.github/prompts/name.prompt.md` — invokable by human via Copilot Chat slash command or agent dropdown in VS Code / GitHub.com.
> - **CI/CD Autonomous Agents**: `.github/agents/name.agent.md` + `.github/workflows/name-agent.yml` — triggered automatically by GitHub Actions on PR/push/schedule with a Kill Switch quality gate.
>
> The `commands/*.md` → `prompts/*.prompt.md` mapping handles the slash-command pointer only. The full rich instruction body should live in the `.agent.md` file, not the `.prompt.md`. Use the `create-agentic-workflow` skill to scaffold either or both agent types from an existing Skill.

## Supported Environments (In-Depth)


### Gemini TOML Format
Command `.md` files are wrapped in TOML. Frontmatter is parsed — the `description` field is extracted and used as the TOML `description`. The frontmatter block is stripped from the prompt body.

---

## Skills vs Workflows (Commands) Caution

> **CRITICAL**: The bridge processes `skills/` and `commands/` (or `workflows/` in older plugins) as distinct directories. **Algorithms/Logic can be deployed to either, but be careful of duplicating them!** 
> - `skills/` are typically for passive knowledge, tools, and persistent behavior.
> - `commands/` are for active, slash-command execution workflows.
> 
> Do not place identical markdown files in both directories within the same plugin, or the bridge will blindly duplicate the logic into the target environments (e.g. into `.agent/workflows/` and `.agent/skills/` simultaneously, causing contextual bloat).

```toml
command = "plugin-name:command-name"
description = "Description from frontmatter"
prompt = """
# Command content without frontmatter
...
"""
```

---

## When to Use
- **Installing a new plugin**: Run bridge after dropping a plugin into `plugins/`.
- **Adding a new target environment**: Existing plugins need to be re-bridged after adding `.gemini/` etc.
- **Upgrading a plugin**: Re-run bridge to overwrite with latest command content.
