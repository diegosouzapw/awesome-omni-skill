---
name: agent-development
description: This skill should be used when the user asks to "create an agent", "write an agent", "build an agent", or wants to add new agent capabilities to Claude Code.
---

# Agent Development

Create effective Claude Code agents.

## Agent Colors

Agents support color assignments for visual distinction. Set in YAML frontmatter:

```yaml
---
name: explore
description: Explores codebases
color: purple
---
```

### Color Conventions

| Color | Agent Type | Purpose |
|-------|------------|---------|
| Purple | Explore | Codebase exploration, documentation |
| Blue | Plan | Analysis, planning, architecture |
| Green | Create | Testing, creation, validation |
| Orange | Debug | Debugging, refactoring |
| Yellow | Clean | Optimization, cleanup |
| Red | Reflect | Security, critical review |
| Pink | - | Available |
| Cyan | - | Orchestration, coordination |

## Agent Behavior

### Permission Denial Resilience

Subagents continue after permission denial rather than stopping entirely. When a subagent hits a permissions wall, it tries alternative approaches automatically. This makes autonomous workflows more resilient and reduces the need for human intervention.

## Important

After creating or modifying agents, inform the user:

> **No restart needed.** Agent changes take effect immediately - agents are hot-reloaded.
