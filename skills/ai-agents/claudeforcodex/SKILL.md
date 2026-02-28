---
name: claudeforcodex
description: "Claude Code CLI configuration guide for Codex. Use when users ask about: claude code config, ~/.claude/settings.json, claude plugins, claude agents, claude skills, claude hooks, claude mcp servers, or integrating tools with Claude Code."
---

# Claude Code CLI Configuration Guide

Configure and extend Anthropic's Claude Code CLI for optimal development workflows.

## Quick Reference

| Task | Command/Location |
|------|------------------|
| Config file | `~/.claude/settings.json` |
| Project config | `.claude/settings.json` (in project root) |
| Plugins | `~/.claude/plugins/` or marketplace |
| Add MCP server | Edit `.mcp.json` or plugin |
| Skills location | Plugin `skills/` directories |

## Core Configuration

Edit `~/.claude/settings.json` for global settings:

```json
{
  "permissions": {
    "allow": ["Bash(*)", "Read(*)", "Write(*)"],
    "deny": []
  },
  "env": {
    "ANTHROPIC_API_KEY": "sk-..."
  }
}
```

**Project-level:** Create `.claude/settings.json` in project root to override.

## Permission System

Claude Code uses explicit tool permissions:

```json
{
  "permissions": {
    "allow": [
      "Bash(*)",           // All bash commands
      "Read(*)",           // All file reads
      "Write(~/projects/*)", // Write only in projects
      "mcp:server_name:*"  // All tools from MCP server
    ],
    "deny": [
      "Bash(rm -rf *)"     // Block dangerous commands
    ]
  }
}
```

**Permission patterns:**
- `Tool(*)` - Allow all uses of tool
- `Tool(/path/*)` - Allow with path prefix
- `mcp:server:tool` - Specific MCP tool

## Plugin System

Claude Code has two plugin patterns:

### Standalone Plugin (entire repo = one plugin)

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # Required for standalone
├── commands/
├── agents/
├── skills/
│   └── my-skill/
│       └── SKILL.md     # ONE level deep only!
├── hooks/
└── .mcp.json
```

### Marketplace Plugin (repo contains multiple plugins)

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json     # Lists all plugins
└── plugins/
    ├── plugin-a/
    │   ├── plugin.json      # AT ROOT (not .claude-plugin/)
    │   └── skills/
    │       └── skill-name/
    │           └── SKILL.md
    └── plugin-b/
        └── plugin.json
```

**Critical Rules:**
1. **Marketplace plugins:** `plugin.json` at plugin root, NOT in `.claude-plugin/`
2. **Skills ONE level deep:** `skills/name/SKILL.md` - NO `skills/a/skills/b/`
3. **No plugin.json per skill** - only one per plugin
4. **Explicit skills array** in marketplace.json (recommended)

**Install plugin:**
```bash
claude plugins add /path/to/plugin
claude plugins add https://github.com/user/plugin

# Install specific branch or tag (v2.0.28+)
claude plugins add https://github.com/user/plugin#develop
claude plugins add https://github.com/user/plugin#v1.0.0
```

**Team sharing:** Add to `.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": [
    "https://github.com/company/plugins#stable"
  ]
}
```

## Commands (Slash Commands)

Create `commands/name.md`:

```markdown
---
description: Brief description for autocomplete
argument-hint: "[optional args]"
model: sonnet
allowed-tools:
  - Bash
  - Read
context: fork
hooks:
  PostToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "${CLAUDE_PLUGIN_ROOT}/scripts/notify.sh"
---

# Command Name

Instructions Claude follows when user types /name
```

**Frontmatter fields:**
- `description` (required) - Help text (<60 chars)
- `argument-hint` - Document expected arguments
- `model` - Override model (opus/sonnet/haiku)
- `allowed-tools` - Restrict available tools (YAML list)
- `context: fork` - Run in forked sub-agent
- `agent` - Agent type for execution
- `hooks` - Command-scoped hooks
- `disable-model-invocation` - Prevent SlashCommand tool calls

**Invoke:** `/plugin-name:command-name` or `/command-name`

## Agents (Subagents)

Create `agents/name.md`:

```markdown
---
name: agent-name
description: "What this agent specializes in. Include trigger phrases."
model: sonnet
tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
skills:
  - skill-name
permissionMode: default
disallowedTools:
  - WebSearch
hooks:
  Stop:
    - hooks:
        - type: prompt
          prompt: "Verify work is complete before stopping."
---

# Agent Name

System prompt and instructions for the agent.
```

**Frontmatter fields:**
- `name` (required) - Agent identifier (lowercase, hyphens)
- `description` (required) - When/why to use, include trigger phrases
- `model` - opus/sonnet/haiku (default: inherit)
- `tools` - Restrict available tools (YAML list, omit for all)
- `skills` - Auto-load specific skills
- `permissionMode` - default/acceptEdits/plan/bypassPermissions
- `disallowedTools` - Explicitly block tools
- `hooks` - Agent-scoped hooks (PreToolUse, PostToolUse, Stop)

**Invoke:** `claude --agent plugin-name:agent-name`

**Spawn from Claude:**
```
Task(subagent_type="plugin:agent", prompt="Do something")
```

## Skills (Auto-Knowledge)

Create `skills/name/SKILL.md`:

```markdown
---
name: skill-name
description: "When to trigger this skill. Be specific about keywords and contexts."
user-invocable: true
model: sonnet
context: fork
allowed-tools:
  - Read
  - Write
  - Bash
hooks:
  PostToolUse:
    - matcher: "Write"
      hooks:
        - type: command
          command: "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
---

# Skill Name

Knowledge and instructions loaded when skill triggers.
```

**Frontmatter fields:**
- `name` (required) - Skill identifier (lowercase, hyphens)
- `description` (required) - Trigger phrases and usage context
- `user-invocable` - Show in slash command menu (default: true)
- `model` - Override model (opus/sonnet/haiku)
- `context: fork` - Run in forked sub-agent
- `agent` - Agent type for execution
- `allowed-tools` - Restrict available tools (YAML list)
- `hooks` - Skill-scoped hooks

**Key features:**
- **Hot-reload** (v2.1.0+): Skills in `~/.claude/skills` or `.claude/skills` reload without restart
- **Unified with commands** (v2.1.3+): Skills visible in `/` menu by default
- **Progress display**: Tool uses shown while skill executes

**Critical:** Skills must be **ONE level deep**:
```
skills/
├── skill-a/          # ✅ Correct
│   └── SKILL.md
└── skill-b/          # ✅ Correct
    └── SKILL.md

# ❌ WRONG - nested skills won't be discovered:
skills/
└── parent/
    └── skills/       # Nesting breaks discovery!
        └── child/
            └── SKILL.md
```

**Note:** Skills auto-load based on conversation context matching the description.

## Hooks (Event Handlers)

Create `hooks/hooks.json` or define in component frontmatter:

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh",
        "timeout": 30000
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Verify all tests pass before stopping.",
        "model": "haiku"
      }]
    }]
  }
}
```

**Hook events:**
| Event | When | Special Features |
|-------|------|------------------|
| `PreToolUse` | Before tool execution | Can modify `updatedInput`, return `ask` |
| `PostToolUse` | After tool execution | Access `tool_use_id` |
| `PermissionRequest` | Permission dialog shown | Can auto-approve/deny |
| `UserPromptSubmit` | User submits prompt | `additionalContext` output |
| `Notification` | Notifications sent | Supports `matcher` values |
| `Stop` | Claude attempts to stop | Prompt-based hooks supported |
| `SubagentStop` | Subagent stops | `agent_id`, `agent_transcript_path` |
| `SubagentStart` | Subagent starts | - |
| `SessionStart` | Session begins | `agent_type` if `--agent` used |
| `SessionEnd` | Session ends | `systemMessage` supported |
| `PreCompact` | Before history compacted | - |

**Hook types:**
- `command` - Execute shell scripts
- `prompt` - LLM-driven evaluation (can specify `model`)

**Hook configuration:**
- `matcher` - Tool/event pattern (regex, `*` wildcard)
- `timeout` - Timeout in ms (default: 10 minutes)
- `once: true` - Run only once per session
- `model` - Model for prompt-based hooks

## MCP Server Integration

Create `.mcp.json` in plugin or project root:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "@company/mcp-server"],
      "env": {
        "API_KEY": "xxx"
      }
    }
  }
}
```

**Use MCP tools:**
```bash
mcp-cli tools                    # List all MCP tools
mcp-cli info server/tool         # Get tool schema
mcp-cli call server/tool '{}'    # Call tool
```

## Claude Code vs Codex

| Aspect | Claude Code | Codex |
|--------|-------------|-------|
| Provider | Anthropic | OpenAI |
| Config | `~/.claude/settings.json` | `~/.codex/config.toml` |
| Plugins | Full plugin system | Skills only |
| Skills | Plugin-based | `~/.codex/skills/` |
| Agents | Plugin agents | N/A |
| Hooks | Plugin hooks | N/A |
| MCP | `.mcp.json` or plugin | `config.toml` sections |
| Permissions | JSON allow/deny | Sandbox modes |

## Common Workflows

### Run with Full Permissions
```bash
claude --dangerously-skip-permissions
```

### Use Specific Agent
```bash
claude --agent conductor:code-reviewer
```

### Install Marketplace Plugin
```bash
claude plugins add marketplace-name
```

### Debug Plugin Loading
```bash
claude --debug
```

## Auditing Plugin Structure

When auditing a Claude Code plugin/marketplace for issues:

```bash
# Find all SKILL.md files
find plugins -name "SKILL.md" | sort

# Find nested skills (WRONG structure)
find plugins -path "*/skills/*/skills/*" -name "SKILL.md"

# Find all plugin.json files
find plugins -name "plugin.json" | sort

# Find orphaned plugin.json in skill dirs (should be 0)
find plugins -path "*/skills/*/plugin.json" | wc -l

# Check marketplace.json has skills arrays
cat .claude-plugin/marketplace.json | jq '.plugins[].skills'
```

**Common Issues:**
| Issue | Symptom | Fix |
|-------|---------|-----|
| Nested skills | Skills not loading | Flatten to `skills/name/SKILL.md` |
| plugin.json in skill dirs | Confusion | Remove, only one per plugin |
| Missing skills array | Skills not discovered | Add to marketplace.json |
| Wrong plugin.json location | Plugin not loading | Move to plugin root |

## Reference Files

- `references/plugin-structure.md` - Complete plugin layout
