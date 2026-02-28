---
name: agent-builder
description: Create custom Claude Code sub-agents with specialized expertise and tool access. Use when you need to build reusable agents for specific tasks like code review, debugging, data analysis, or domain-specific workflows.
version: 1.0.0
---

# Build Custom Claude Code Sub-Agents

## When to Use

- Creating a specialized agent for recurring tasks (code review, debugging, testing)
- Need an agent with specific tool permissions or limited scope
- Want to share reusable agents across projects or with your team
- Building domain-specific agents (data science, DevOps, security)
- Need to preserve main conversation context while delegating complex tasks

## What This Skill Does

Guides you through creating custom sub-agents that:

- **Specialize**: Focused expertise for specific domains or tasks
- **Isolate**: Separate context windows prevent main conversation pollution
- **Reuse**: Deploy across projects and share with teams
- **Control**: Granular tool access and model selection per agent

## Quick Start

### 1. Use the Built-In Agent Creator

```bash
# Run the agents command
/agents
```

Then:
1. Select "Create New Agent"
2. Choose project-level (`.claude/agents/`) or user-level (`~/.claude/agents/`)
3. Generate with Claude or manually define configuration
4. Save and test

### 2. Manual Agent Creation

Create a markdown file in `.claude/agents/` (project) or `~/.claude/agents/` (user):

```markdown
---
name: my-agent-name
description: Use this agent when [specific trigger condition]
tools: Read, Edit, Bash
model: sonnet
---

# Agent System Prompt

Your detailed instructions for the agent go here.

Be specific about:
- What tasks this agent handles
- How to approach problems
- What outputs to produce
- Any constraints or guardrails
```

### 3. Invoke Your Agent

**Automatic**: Claude detects matching tasks based on description
**Explicit**: "Use the my-agent-name agent to [task]"

## Agent Configuration

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Lowercase with hyphens | `code-reviewer` |
| `description` | When to use this agent (triggers routing) | `Use PROACTIVELY to review code changes for quality and security` |

### Optional Fields

| Field | Description | Default |
|-------|-------------|---------|
| `tools` | Comma-separated tool list | All tools inherited |
| `model` | Model alias (sonnet/opus/haiku) or 'inherit' | Inherits from main |

**See**: [Configuration Reference](./reference/configuration-guide.md)

## Agent Structure

```
.claude/agents/           # Project-level agents
├── code-reviewer.md
├── debugger.md
└── custom-agent.md

~/.claude/agents/         # User-level agents (global)
├── my-helper.md
└── data-analyzer.md
```

**Priority**: Project agents override user agents with same name

## Common Agent Types

### Code Reviewer
Reviews code for quality, security, and best practices

**Triggers**: After code changes, before commits

### Debugger
Analyzes errors, identifies root causes, proposes fixes

**Triggers**: Test failures, runtime errors

### Data Scientist
Writes SQL queries, performs analysis, generates reports

**Triggers**: Data questions, BigQuery tasks

**See**: [Agent Examples](./reference/agent-examples.md)

## Best Practices

1. **Single Responsibility**: One focused task per agent
2. **Descriptive Triggers**: Use "PROACTIVELY" or "MUST BE USED" for automatic delegation
3. **Detailed Prompts**: Specific instructions yield better results
4. **Limit Tools**: Only grant necessary permissions
5. **Version Control**: Commit project agents for team collaboration

**Full Guide**: [Best Practices](./reference/best-practices.md)

## Available Tools

Sub-agents can access:
- **File Operations**: Read, Write, Edit, Glob, Grep
- **Execution**: Bash
- **MCP Tools**: Any installed MCP server tools

Use `/agents` interface to visually select tools.

## Outputs

This skill helps you create:
- Agent configuration files (`.md` with YAML frontmatter)
- Specialized system prompts
- Tool permission configurations
- Reusable agent templates

## Guardrails

- Agents must have focused, well-defined purposes
- Use lowercase-with-hyphens naming convention
- Always specify clear trigger conditions in description
- Grant minimal tool access (principle of least privilege)
- Test agents thoroughly before sharing with team

## Advanced Topics

- [Configuration Guide](./reference/configuration-guide.md) - Complete field reference
- [Agent Examples](./reference/agent-examples.md) - Real-world templates
- [Best Practices](./reference/best-practices.md) - Design patterns
- [Troubleshooting](./reference/troubleshooting.md) - Common issues

## Triggers

This skill activates when you mention:
- "create an agent" or "build an agent"
- "sub-agent" or "subagent"
- "agent configuration"
- "Task tool" or "custom agent"
- "agent best practices"

## Testing

To test your agent:

```bash
# Ask Claude to use it explicitly
"Use the [agent-name] agent to [task]"

# Or test automatic triggering
"[Describe a task matching agent's description]"
```

Verify:
- [ ] Agent triggers correctly
- [ ] Has necessary tool access
- [ ] Produces expected outputs
- [ ] Maintains scope/focus

---

## Reference Documentation

- **[PRISM Agent Strategy](./reference/prism-agent-strategy.md)** - Artifact-centric agent design patterns for PRISM workflows

---

**Last Updated**: 2025-10-27
**Version**: 1.0.0
