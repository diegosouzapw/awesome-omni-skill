---
name: agent-discovery
description: Discovers all Claude Code agents in the system including built-in, plugin, project, and user-level agents. Use when you need to find which agents are available, understand the agent ecosystem, or prepare agents for Actoris registration.
allowed-tools: Read, Glob, Grep, Bash
---

# Agent Discovery Skill

Use this skill to discover and catalog all Claude Code agents.

## What You Can Find

### Built-in Agents (Always Available)
| Agent | Model | Purpose |
|-------|-------|---------|
| Explore | haiku | Fast codebase exploration |
| Plan | sonnet | Implementation planning |
| general-purpose | sonnet | Complex multi-step tasks |
| claude-code-guide | sonnet | Claude Code documentation |
| statusline-setup | haiku | Status line configuration |

### Custom Agent Locations

1. **Project Agents**: `.claude/agents/*.md`
2. **User Agents**: `~/.claude/agents/*.md`
3. **Plugin Agents**: `{plugin-root}/agents/*.md`

### Agent File Format

Custom agents are Markdown files with YAML frontmatter:

```yaml
---
name: my-agent
description: What this agent does and when to use it
tools: Read, Write, Bash  # Comma-separated
model: sonnet  # sonnet, opus, haiku, or inherit
skills: skill1, skill2  # Optional
---

# Agent Instructions

Your agent prompt goes here...
```

## Discovery Commands

### Find Project Agents
```bash
ls -la .claude/agents/*.md 2>/dev/null
```

### Find User Agents
```bash
ls -la ~/.claude/agents/*.md 2>/dev/null
```

### Parse Agent Metadata
```bash
grep -A 5 "^---" .claude/agents/my-agent.md | head -10
```

## MCP Integration

Use the Actoris MCP tool for structured discovery:
```
list_claude_code_agents(source="all", detailed=true)
```

Returns JSON with:
- Total agent count by source
- Agent name, description, tools, model
- File path for custom agents

## After Discovery

Once agents are discovered, they can be:
1. Registered with Actoris via `register_agent_with_actoris`
2. Monitored for trust scores
3. Tracked in the Darwinian leaderboard
4. Included in AGDP calculations
