---
name: portable-agents
description: "Defines agent configuration formats across Claude Code, Cursor, OpenCode, and Cline. Covers frontmatter fields, tool restrictions, model selection, and cross-tool portability. ALWAYS read before creating or editing agent definitions. This skill should be used when 'writing an agent', 'creating a subagent', 'porting agents between tools', 'agent frontmatter', 'agent configuration', or setting up agents across multiple AI tools."
user-invocable: true
disable-model-invocation: false
---

# Cross-Tool Agent Interoperability

Write agent definitions that work across Claude Code, Cursor, OpenCode, and Cline without duplication.

## Quick Start

All four tools define agents as markdown files with YAML frontmatter. The body is the system prompt.

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
---

You are a code reviewer. Analyze code for security, performance, and maintainability.
```

Store agents in tool-specific directories. Unlike skills, there is **no single shared directory** -- each tool has its own agent location.

## Agent Directories

| Scope | Claude Code | Cursor | OpenCode | Cline |
|-------|------------|--------|----------|-------|
| **Project** | `.claude/agents/` | `.cursor/agents/` | `.opencode/agents/` | `.clinerules/agents/` |
| **User** | `~/.claude/agents/` | `~/.cursor/agents/` | `~/.config/opencode/agents/` | N/A |
| **Plugin** | Plugin `agents/` dir | N/A | N/A | N/A |

**Priority** (when names collide): CLI flags > project > user > plugin (Claude Code). Project > user (Cursor, OpenCode).

## Frontmatter Compatibility Matrix

| Field | Claude Code | Cursor | OpenCode | Cline |
|-------|------------|--------|----------|-------|
| `name` | Required | Required | Filename = name | Filename = name |
| `description` | Required | Required | Required | Required |
| `model` | `sonnet`/`opus`/`haiku`/`inherit` | `inherit` only | `provider/model-id` | N/A |
| `tools` | Allowlist (tool names) | Not supported | Boolean map | Not supported |
| `disallowedTools` | Denylist (tool names) | Not supported | Not supported | Not supported |
| `mode` | Implicit (subagent by location) | Implicit | `primary`/`subagent`/`all` | Implicit |
| `temperature` | Not in frontmatter | Not in frontmatter | Supported | Not supported |
| `maxTurns`/`steps` | `maxTurns` | Not supported | `steps` | Not supported |
| `permissionMode` | 6 modes | Not supported | `permission` (per-tool) | Not supported |
| `skills` | Preload skill content | Not supported | Not supported | Not supported |
| `mcpServers` | Scope MCP per agent | Not supported | Not supported | Not supported |
| `hooks` | Full lifecycle hooks | Not supported | Not supported | Not supported |
| `memory` | `user`/`project`/`local` | Not supported | Not supported | Not supported |
| `color` | Hex or theme color | Not supported | Hex or theme color | Not supported |
| `hidden` | Not supported | Not supported | Hide from `@` menu | Not supported |

## Portable Subset (All Tools)

The only fields guaranteed portable across all four tools:

```yaml
---
name: my-agent          # or use filename in OpenCode/Cline
description: What this agent does and when to use it
---

System prompt in markdown body.
```

Everything beyond `name`, `description`, and the markdown body is tool-specific. Write the core agent logic in the body, then add tool-specific frontmatter as needed.

## Cross-Tool Setup

### Strategy: Canonical Source + Symlinks

Maintain one source directory with canonical agent definitions, then symlink or copy to each tool's expected location.

```bash
# Canonical source
mkdir -p ~/.agents/

# Symlink to each tool
ln -sfn ~/.agents ~/.claude/agents
ln -sfn ~/.agents ~/.cursor/agents
ln -sfn ~/.agents ~/.config/opencode/agents
```

**Caveat**: OpenCode uses the filename as the agent name (no `name` field needed), while Claude Code and Cursor require `name` in frontmatter. Include `name` in frontmatter for maximum compatibility -- OpenCode ignores it.

### Strategy: Transpiler Script

For agents that need tool-specific features (model routing, tool restrictions, permissions), maintain a canonical format and transpile:

```bash
#!/bin/bash
# sync-agents.sh - Generate tool-specific agent files from canonical source
CANONICAL_DIR="./agents-canonical"
for agent in "$CANONICAL_DIR"/*.md; do
  name=$(basename "$agent" .md)
  # Copy to each tool dir, stripping unsupported fields
  cp "$agent" ".claude/agents/$name.md"
  cp "$agent" ".cursor/agents/$name.md"
  cp "$agent" ".opencode/agents/$name.md"
done
```

## Model Selection Across Tools

Each tool handles model assignment differently:

| Tool | How Models Work |
|------|----------------|
| **Claude Code** | Aliases: `sonnet`, `opus`, `haiku`, `inherit`. Subagents default to `inherit` |
| **Cursor** | Subagents always inherit parent model. No per-agent model selection |
| **OpenCode** | Full `provider/model-id` format. Per-agent overrides in JSON config or frontmatter |
| **Cline** | Model set globally. No per-agent selection |

**Portable approach**: Omit `model` from frontmatter (defaults to inherit/parent in all tools). Override per-tool via config files when needed.

## Tool Restrictions Across Tools

| Tool | Mechanism | Format |
|------|-----------|--------|
| **Claude Code** | `tools` (allowlist) or `disallowedTools` (denylist) | Comma-separated tool names: `Read, Grep, Glob` |
| **Cursor** | Not supported in agent frontmatter | N/A |
| **OpenCode** | `tools` (boolean map) | `tools: { write: false, edit: false, bash: false }` |
| **Cline** | Not supported | N/A |

**Portable approach**: Express tool restrictions in the system prompt body as instructions ("Do NOT modify files. Only read and analyze."). This works everywhere, though it's advisory rather than enforced.

## Subagent Spawning

All four tools support subagents, but the spawning mechanism differs:

| Aspect | Claude Code | Cursor | OpenCode | Cline |
|--------|------------|--------|----------|-------|
| **Spawn mechanism** | `Task` tool | `Task` tool | `Task` tool / `@mention` | `@mention` |
| **Max concurrent** | 10 (queues beyond) | Limited by model | Configurable per-provider | Limited |
| **Nesting** | Subagents cannot spawn subagents | Subagents can nest (2026) | Subagents cannot nest | Cannot nest |
| **Background** | Yes (Ctrl+B or explicit) | Yes (async subagents) | Yes (`run_in_background`) | No |
| **Resume** | Yes (agent ID) | Yes (agent ID) | Yes (session navigation) | No |
| **Restrict spawning** | `Task(worker, researcher)` in tools | Not supported | `permission.task` glob patterns | Not supported |

## Orchestration Patterns

### Pattern 1: AGENTS.md Orchestration (Most Portable)

Put orchestration rules in `AGENTS.md` (read by all tools). The root agent decides when to delegate based on instructions, not config.

```markdown
## Orchestration Rules

When the task involves:
- **Multiple files or modules**: Delegate to explore agent first
- **External libraries**: Delegate to librarian agent
- **Complex implementation**: Create a plan, then delegate to implementer
- **Architecture decisions**: Consult oracle agent
```

### Pattern 2: Orchestrator Rule (Cursor-Specific)

Cursor supports `.cursor/rules/*.mdc` with `alwaysApply: true` for always-on orchestration. This is the pattern oh-my-cursor uses.

### Pattern 3: Plugin Orchestration (OpenCode-Specific)

oh-my-opencode uses a plugin system with JSON config for model routing, categories, and provider fallback chains. See [references/opencode-agents.md](references/opencode-agents.md).

## Validation Checklist

Before finalizing a portable agent:

- [ ] `name` is lowercase with hyphens, matches filename
- [ ] `description` clearly states when to delegate to this agent
- [ ] System prompt body uses imperative form
- [ ] No tool-specific features in core prompt logic
- [ ] Tool-specific frontmatter clearly commented
- [ ] `model` omitted or set to `inherit` for portability
- [ ] Tool restrictions expressed in prompt body (advisory) AND frontmatter (enforced) where supported
- [ ] Tested delegation in target tools

## References

- **[references/claude-code-agents.md](references/claude-code-agents.md)** - Claude Code agent configuration and capabilities
- **[references/cursor-agents.md](references/cursor-agents.md)** - Cursor agent configuration and capabilities
- **[references/opencode-agents.md](references/opencode-agents.md)** - OpenCode agent configuration and capabilities
- **[references/agent-roster.md](references/agent-roster.md)** - Standard agent roles and their system prompts
