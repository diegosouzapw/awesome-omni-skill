---
name: claude-code-optimizer
description: Audit and optimize Claude Code installations to current best practices. Use when users ask to check, audit, optimize, or improve their Claude Code setup, configuration, hooks, permissions, or workflow. Triggers on requests mentioning Claude Code settings, CLAUDE.md files, hooks configuration, permissions setup, or performance optimization.
---

# Claude Code Optimizer

Audit and optimize Claude Code installations. Requires Claude Code 2.1.3 or later.

## Quick Start

Run the appropriate audit based on user request:

1. **Full audit**: Invoke `audit-orchestrator` agent for comprehensive review
2. **Specific audit**: Invoke specialized agent directly (see Agents below)
3. **Quick check**: Run inline checks from Audit Checklist

## Agents

Invoke via Task tool with appropriate `subagent_type`:

| Agent | Purpose |
|-------|---------|
| `audit-orchestrator` | Coordinates full audit across all areas |
| `config-auditor` | Audits settings.json and CLAUDE.md files |
| `hooks-auditor` | Audits hook configurations |
| `permissions-auditor` | Audits permission patterns |
| `workflow-auditor` | Audits commands, agents, and skills |
| `mcp-auditor` | Audits MCP server configurations |

Agent definitions are in `agents/` directory. Install to `~/.claude/agents/` or `.claude/agents/`.

## Audit Checklist

### Configuration Files
Check these locations exist and are properly configured:

```
~/.claude/settings.json          # User settings
.claude/settings.json            # Project settings
CLAUDE.md                        # Project memory (root)
.claude/CLAUDE.md               # Alternative location
CLAUDE.local.md                 # Local overrides (gitignored)
~/.claude/CLAUDE.md             # User-level memory
```

### Settings.json Structure
Valid top-level keys (2.1.3+):
- `hooks` - Event handlers
- `permissions` - Tool permissions (allow/deny arrays)
- `env` - Environment variables
- `mcpServers` - MCP server configurations
- `model` - Default model
- `theme` - UI theme
- `respectGitignore` - File picker behavior
- `language` - Response language
- `fileSuggestion` - Custom `@` file search command
- `releaseChannel` - `stable` or `latest` channel toggle

### Hook Events (2.1.0+)
Valid event names:
- `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`
- `Notification`, `UserPromptSubmit`
- `SessionStart`, `SessionEnd`
- `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`

### Tool Names for Matchers
- `Bash`, `Read`, `Write`, `Edit`
- `Grep`, `Glob`, `Task`, `TaskOutput`
- `WebFetch`, `WebSearch`
- `NotebookEdit`, `TodoWrite`, `KillShell`
- `LSP` (Language Server Protocol, 2.0.74+)
- `mcp__*` for MCP tools (wildcard: `mcp__server__*`)

### Permission Patterns
```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Glob"],
    "deny": ["Bash(rm -rf:*)"]
  }
}
```

Wildcard syntax: `Bash(git *:*)` allows all git commands.

## Common Issues

| Issue | Fix |
|-------|-----|
| Hooks not firing | Check event name spelling (case-sensitive) |
| Skills not loading | Verify `~/.claude/skills/` or `.claude/skills/` path |
| Permissions ignored | Check allow/deny order; deny takes precedence |
| CLAUDE.md not read | Must be in project root or `.claude/` |

## References

For detailed audit procedures, see:
- `references/config-guide.md` - Configuration deep-dive
- `references/hooks-guide.md` - Hook patterns and examples
- `references/permissions-guide.md` - Permission strategies
- `references/security-guide.md` - Security audit procedures
- `references/troubleshooting.md` - Common problems and solutions

## Version Compatibility

This skill targets Claude Code 2.1.3+ features:
- Skill hot-reload
- Forked sub-agent context (`context: fork`)
- Hook frontmatter in skills/agents with `once: true` option
- YAML-style allowed-tools
- Unreachable permission rule detection
- MCP wildcard permissions (`mcp__server__*`)
- Named sessions (`/rename`, `/resume <name>`)
- Release channel configuration

Check version: `claude --version`
Update: `claude update`
