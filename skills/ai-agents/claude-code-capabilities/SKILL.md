---
name: claude-code-capabilities
description: Comprehensive guide to Claude Code's extensibility features including subagents, skills, plugins, commands, hooks, MCP servers, permissions, and automation. Use when the user asks about Claude Code features, capabilities, customization, extensibility, how to extend Claude Code, or automation workflows.
---

# Claude Code Capabilities

This skill provides a comprehensive overview of all Claude Code extensibility features and capabilities. Use this as your starting point to discover what Claude Code can do, then dive into specific documents for detailed implementation guidance.

## Overview

Claude Code is a powerful AI-powered development tool that can be customized and extended in many ways. This skill covers all the ways you can enhance and automate your Claude Code experience.

## Core Capabilities

### 1. Extending Claude's Capabilities

**Skills** - Model-invoked modular capabilities that Claude autonomously uses
* Package expertise into discoverable capabilities
* Auto-activation based on context
* Share across projects and teams
* See [skills.md](skills.md) for details

**Plugins** - Packaged extensions with commands, agents, skills, and hooks
* Distribute multiple features together
* Install from marketplaces
* Create organizational plugin catalogs
* See [plugins.md](plugins.md) for details

### 2. Task Delegation

**Subagents** - Specialized AI assistants for specific tasks
* Dedicated context window per subagent
* Custom system prompts and tool access
* Automatic or manual invocation
* Perfect for focused, repeatable tasks
* See [subagents.md](subagents.md) for details

**Headless Mode** - Programmatic Claude Code execution
* Run Claude Code from scripts
* No interactive UI
* JSON output for automation
* Multi-turn conversations
* See [headless.md](headless.md) for details

### 3. Automation

**Hooks** - Event-driven shell commands
* Execute commands at lifecycle events
* PreToolUse, PostToolUse, SessionStart, etc.
* Automatic formatting, validation, logging
* Custom permission checks
* See [hooks.md](hooks.md) for details

**GitHub Actions** - CI/CD integration
* AI-powered PR reviews
* Automated issue resolution
* `@claude` mention triggers
* Custom workflows
* See [github-actions.md](github-actions.md) for details

### 4. Customization

**Slash Commands** - Custom user-invoked commands
* Stored prompts for frequent tasks
* Project and personal commands
* Arguments and file references
* Bash execution support
* See [commands.md](commands.md) for details

**Configuration** - Fine-grained settings control
* Environment variables
* Model selection
* Permission modes
* Sandbox settings
* See [configuration.md](configuration.md) for details

### 5. Integration

**MCP Servers** - External tool integration
* Connect to APIs and services
* Remote and local servers
* OAuth authentication
* Tools, resources, and prompts
* See [mcp.md](mcp.md) for details

**Permissions** - Security and access control
* Tool-level permissions
* File access controls
* Directory restrictions
* Enterprise policies
* See [permissions.md](permissions.md) for details

## Common Use Cases

### Development Workflows

**Automated code review**
1. Create a code-reviewer subagent ([subagent-examples.md](examples/subagent-examples.md))
2. Add a PostToolUse hook to trigger reviews ([hook-examples.md](examples/hook-examples.md))
3. Use GitHub Actions for PR automation ([github-actions.md](github-actions.md))

**Testing automation**
1. Create a test-runner subagent
2. Add a command for running specific test suites
3. Use hooks to auto-run tests after code changes

**Code formatting**
1. Add PostToolUse hooks for formatters (Prettier, Black, gofmt)
2. Configure per-language formatting rules
3. See [hook-examples.md](examples/hook-examples.md)

### Team Collaboration

**Shared workflows**
1. Create project-level commands in `.claude/commands/`
2. Define team subagents in `.claude/agents/`
3. Package everything as a plugin for easy distribution

**Onboarding**
1. Create CLAUDE.md with project context
2. Add project-specific commands for common tasks
3. Configure permissions for safe exploration

### Security and Compliance

**Access control**
1. Use deny rules to protect sensitive files
2. Configure Bash command permissions
3. Set up enterprise policies
4. See [permissions.md](permissions.md)

**Audit logging**
1. Add PreToolUse hooks to log commands
2. Track file modifications
3. Monitor tool usage
4. See [hook-examples.md](examples/hook-examples.md)

### Integration and Automation

**Connect to external services**
1. Add MCP servers for GitHub, Jira, Slack, etc.
2. Configure authentication
3. Control tool permissions
4. See [mcp.md](mcp.md)

**Headless automation**
1. Use `claude -p` for scripting
2. Parse JSON output
3. Chain multi-turn conversations
4. See [headless.md](headless.md)

## Getting Started by Use Case

### I want to...

**...automate repetitive tasks**
→ Start with [commands.md](commands.md) to create custom slash commands

**...add specialized AI behavior**
→ Start with [subagents.md](subagents.md) to create task-specific assistants

**...run code automatically (formatting, testing)**
→ Start with [hooks.md](hooks.md) for event-driven automation

**...package and share customizations**
→ Start with [plugins.md](plugins.md) to bundle everything together

**...connect to external tools/APIs**
→ Start with [mcp.md](mcp.md) to integrate with services

**...control what Claude can access**
→ Start with [permissions.md](permissions.md) for security

**...run Claude Code from scripts**
→ Start with [headless.md](headless.md) for programmatic usage

**...integrate with CI/CD**
→ Start with [github-actions.md](github-actions.md) for workflow automation

## Feature Comparison

### When to use what?

**Skills vs. Subagents vs. Commands**

| Feature | Skills | Subagents | Commands |
|---------|---------|-----------|----------|
| **Invocation** | Model-invoked (automatic) | Model or user-invoked | User-invoked (explicit) |
| **Context** | Main conversation | Separate context | Main conversation |
| **Use case** | Extend capabilities | Specialized tasks | Stored prompts |
| **System prompt** | No (uses main) | Yes (custom) | No (uses main) |
| **Tool restrictions** | Optional | Yes | Optional |

**Hooks vs. MCP Tools**

| Feature | Hooks | MCP Tools |
|---------|-------|-----------|
| **Purpose** | Automation on events | External tool integration |
| **Language** | Shell commands | Server implementation |
| **Timing** | Event-driven | On-demand |
| **Use case** | Formatting, logging, validation | API access, data retrieval |

## Best Practices

### Organization

1. **Start simple**: Begin with commands or skills before building complex plugins
2. **Project vs. personal**: Use `.claude/` for team-shared, `~/.claude/` for personal
3. **Version control**: Check in project-level customizations to share with team
4. **Documentation**: Document your customizations in README or CLAUDE.md

### Security

1. **Least privilege**: Grant minimal permissions needed
2. **Protect secrets**: Use deny rules for `.env` and credential files
3. **Review hooks**: Hooks run automatically - review code carefully
4. **Enterprise policies**: Use managed settings for organization-wide controls

### Performance

1. **Limit subagent use**: Each invocation starts fresh - adds latency
2. **Hook timeouts**: Set timeouts to prevent hanging
3. **Focused skills**: Keep skills specific for better discovery
4. **Cache where possible**: Use SessionStart hooks for one-time setup

## Reference Documentation

### Deep Dives

* [subagents.md](subagents.md) - Creating and managing custom subagents
* [skills.md](skills.md) - Creating and managing skills
* [plugins.md](plugins.md) - Plugin system overview
* [commands.md](commands.md) - Custom slash commands
* [hooks.md](hooks.md) - Event-driven automation
* [mcp.md](mcp.md) - MCP server integration
* [permissions.md](permissions.md) - IAM and permissions
* [configuration.md](configuration.md) - Settings and configuration
* [headless.md](headless.md) - Programmatic usage
* [github-actions.md](github-actions.md) - GitHub Actions integration
* [output-styles.md](output-styles.md) - Output styles (deprecated)
* [cli-reference.md](cli-reference.md) - CLI commands and flags

### Examples

* [examples/subagent-examples.md](examples/subagent-examples.md) - Practical subagent examples
* [examples/hook-examples.md](examples/hook-examples.md) - Hook configuration examples
* [examples/plugin-examples.md](examples/plugin-examples.md) - Plugin development examples

## Quick Reference

### Essential Commands

```bash
# Interactive commands
/agents          # Manage subagents
/hooks           # Configure hooks
/mcp             # Manage MCP servers
/permissions     # View/edit permissions
/config          # Open settings
/plugin          # Manage plugins

# CLI usage
claude -p "query"                    # Headless mode
claude --agents '{...}'              # Dynamic subagents
claude --allowedTools "Bash,Read"    # Restrict tools
claude --permission-mode plan        # Plan mode (read-only)
```

### Configuration Files

```
~/.claude/                    # User-level
├── settings.json            # User settings
├── mcp.json                 # MCP servers
├── agents/                  # User subagents
├── commands/                # User commands
└── skills/                  # User skills

.claude/                     # Project-level
├── settings.json           # Project settings (shared)
├── settings.local.json     # Local settings (gitignored)
├── agents/                 # Project subagents
├── commands/               # Project commands
└── skills/                 # Project skills

.mcp.json                   # Project MCP servers
CLAUDE.md                   # Project memory/instructions
```

## Need Help?

When you have questions about Claude Code capabilities:

1. **Quick lookup**: Use this SKILL.md to find the right topic
2. **Deep dive**: Read the specific document for detailed information
3. **Examples**: Check the examples/ folder for practical code
4. **Ask Claude**: I can help you implement any of these features!

## Next Steps

Based on your needs:

* **New to Claude Code?** Start with [configuration.md](configuration.md) and [commands.md](commands.md)
* **Want automation?** Read [hooks.md](hooks.md) and [examples/hook-examples.md](examples/hook-examples.md)
* **Building for teams?** Check [plugins.md](plugins.md) and [permissions.md](permissions.md)
* **Need integration?** Explore [mcp.md](mcp.md) and [headless.md](headless.md)
