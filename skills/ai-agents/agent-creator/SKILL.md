---
name: agent-creator
description: Create and optimize Claude Code subagent files (.claude/agents/*.md) following official best practices. Use when user asks to create a subagent, agent file, custom agent, or needs help structuring agent files with YAML frontmatter.
license: MIT
---

# Creating Subagents (Agent Files)

Guide for creating Claude Code subagents following Anthropic's official best practices (January 2026).

## File Format

Subagents are Markdown files with YAML frontmatter:

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer ensuring high standards.

When invoked:

1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately
```

## Required Fields

Only two fields are required:

- `name`: Unique identifier, lowercase letters and hyphens only (kebab-case)
- `description`: When Claude should delegate to this subagent. Include what it does and when to use it.

## Directory Locations

Priority hierarchy (higher priority wins):

1. `--agents` CLI flag (session only)
2. `.claude/agents/` (project-level, version-controlled)
3. `~/.claude/agents/` (user-level, all projects)
4. Plugin `agents/` (lowest)

## Frontmatter Fields

| Field             | Required | Description                                                      |
| ----------------- | -------- | ---------------------------------------------------------------- |
| `name`            | ✅ Yes   | Unique identifier (kebab-case)                                   |
| `description`     | ✅ Yes   | Delegation trigger description                                   |
| `tools`           | No       | Allowlist (inherits all if omitted)                              |
| `disallowedTools` | No       | Denylist to remove                                               |
| `model`           | No       | `sonnet`, `opus`, `haiku`, or `inherit` (default)                |
| `permissionMode`  | No       | `default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan` |
| `skills`          | No       | Skills to preload at startup                                     |
| `hooks`           | No       | Lifecycle hooks scoped to this subagent                          |

## Creating Subagents

**Three methods:**

1. **Interactive**: `/agents` in Claude Code TUI
2. **Manual**: Create `.claude/agents/my-agent.md` or `~/.claude/agents/my-agent.md`
3. **CLI**: `claude --agents '{"agent-name": {...}}'`

After manual creation, restart session or run `/agents` to load.

## Writing Effective Descriptions

Critical for automatic delegation. Include:

1. What the subagent does
2. When to use it
3. Proactive triggers (recommended)

**Good:**

```yaml
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
```

**Bad:**

```yaml
description: Code reviewer # Too vague, missing triggers
```

## Tool Access Control

**Allowlist:**

```yaml
tools: Read, Grep, Glob, Bash
```

**Denylist:**

```yaml
disallowedTools: Write, Edit
```

**Default**: Inherits all tools from main conversation (including MCP tools) if `tools` omitted.

## Model Selection

```yaml
model: haiku      # Fast, read-only exploration
model: sonnet     # Balanced (code review, debugging)
model: opus       # Complex reasoning
model: inherit    # Match main conversation (default)
```

## Permission Modes

| Mode                | Behavior                    |
| ------------------- | --------------------------- |
| `default`           | Standard permission prompts |
| `acceptEdits`       | Auto-accept file edits only |
| `dontAsk`           | Auto-deny prompts           |
| `bypassPermissions` | Skip all checks             |
| `plan`              | Read-only (Plan mode)       |

**Note**: Parent `bypassPermissions` takes precedence.

## Preloading Skills

```yaml
skills:
  - api-conventions
  - error-handling-patterns
```

**Key**: Full content injected at startup. Subagents don't inherit skills from parent conversation.

## System Prompt Patterns

The Markdown body becomes the system prompt. Use:

- **Workflow steps**: "When invoked: 1. Do X 2. Do Y"
- **Checklists**: Structured review criteria
- **Output formats**: Priority-based feedback structure
- **Explicit constraints**: "You cannot modify data..."

## Automatic Delegation

Claude delegates based on task description, `description` field, and context.

**Encourage proactive delegation**: Include "use proactively" in descriptions.

**Explicit invocation:**

```
Use the code-reviewer subagent to review my changes
```

## When to Use Subagents

**Use subagents when:**

- Task produces verbose output
- Need to enforce tool restrictions
- Work is self-contained with summary return
- Want to isolate context

**Use main conversation when:**

- Frequent back-and-forth needed
- Multiple phases share significant context
- Quick, targeted changes
- Latency matters

**Limitation**: Subagents cannot spawn other subagents. Chain from main conversation or use Skills.

## File Naming

- **Filename**: Any name ending in `.md` (e.g., `my-agent.md`)
- **Name field**: kebab-case (lowercase, hyphens only)
- Must be unique within scope

## Loading Behavior

- Loaded at session start
- Restart session or `/agents` to load manually-created files
- Changes require restart

## Version Control

**Project subagents** (`.claude/agents/`): ✅ Check into version control, share with team

**User subagents** (`~/.claude/agents/`): Personal preferences, keep out of version control

## Reference Files

- **Examples**: See [examples.md](examples.md)
- **Hooks**: See [hooks.md](hooks.md) for conditional tool validation
- **Advanced**: See [advanced.md](advanced.md) for background execution, resuming, auto-compaction

## Fundamental Principle

**Focused, specialized assistants with clear delegation triggers and appropriate tool restrictions.** Design them to solve specific, well-defined problems that benefit from isolated context or enforced constraints.
