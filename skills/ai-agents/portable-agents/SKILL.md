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
| `model` | `sonnet`/`opus`/`haiku`/`inherit` | Cursor model names | `provider/model-id` | N/A |
| `tools` | Allowlist (tool names) | Not supported | Boolean map | Not supported |
| `disallowedTools` | Denylist (tool names) | Not supported | Not supported | Not supported |
| `mode` | Implicit (subagent by location) | Implicit | `primary`/`subagent`/`all` | Implicit |
| `temperature` | Not in frontmatter | Supported | Supported | Not supported |
| `maxTurns`/`steps` | `maxTurns` | Not supported | `steps` | Not supported |
| `permissionMode` | 6 modes | Not supported | `permission` (per-tool) | Not supported |
| `skills` | Preload skill content | Not supported | Not supported | Not supported |
| `mcpServers` | Scope MCP per agent | Not supported | Not supported | Not supported |
| `hooks` | Full lifecycle hooks | Not supported | Not supported | Not supported |
| `memory` | `user`/`project`/`local` | Not supported | Not supported | Not supported |
| `color` | Hex or theme color | Not supported | Hex or theme color | Not supported |
| `hidden` | Not supported | Not supported | Hide from `@` menu | Not supported |

## Agent Interoperability Summary

**OpenCode IS compatible with Cursor and Claude Code agent specifications** at the core level. All three tools use markdown files with YAML frontmatter and support the same fundamental structure.

### What Works Across All Tools

The **portable subset** that works in Claude Code, Cursor, OpenCode, and Cline:

```yaml
---
name: my-agent          # Required for Claude Code/Cursor, optional for OpenCode/Cline (uses filename)
description: What this agent does and when to use it
---

System prompt in markdown body.
```

This minimal format works **without modification** across all tools. Place the same file in each tool's agent directory via symlinks (see Cross-Tool Setup below).

### What's Tool-Specific

Beyond the portable subset, each tool has unique capabilities that won't transfer:

| Feature | Portability |
|---------|-------------|
| `name` + `description` + markdown body | ✅ Works everywhere |
| Model selection syntax | ❌ Different formats per tool |
| Tool restrictions | ❌ Different mechanisms per tool |
| Permissions (bash, edit, webfetch) | ❌ OpenCode only |
| Temperature in frontmatter | ❌ OpenCode only |
| Max iterations control | ❌ Different field names |
| MCP server scoping | ❌ Claude Code only |
| Skills preloading | ❌ Claude Code only |
| Hooks | ❌ Claude Code only |

### Maximizing Portability

**Strategy 1: Keep it simple (Recommended)**

Use only the portable subset for shared agents. Express constraints in the system prompt body:

```yaml
---
name: code-reviewer
description: Reviews code for quality without making changes
---

You are a read-only code reviewer.

**CONSTRAINTS:**
- Do NOT modify files (no write, edit operations)
- Do NOT run bash commands except read-only git commands
- Focus on analysis and recommendations only
```

This works everywhere, though enforcement is advisory (model follows instructions) rather than technical (tool blocks execution).

**Strategy 2: Tool-specific overlays**

Maintain canonical agents with portable core, then add tool-specific frontmatter:

```yaml
---
name: code-reviewer
description: Reviews code for quality without making changes

# OpenCode-specific
tools:
  write: false
  edit: false
permission:
  bash:
    "*": "ask"
    "git diff*": "allow"

# Claude Code-specific  
disallowedTools: Write, Edit

# Note: Cursor ignores tools/permissions in frontmatter
---

System prompt (portable across all tools).
```

OpenCode and Claude Code will enforce tool restrictions. Cursor will rely on prompt instructions only.

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

**Recommended Strategy**: Symlink portable markdown agents without `model` field, configure tool-specific models in each tool's JSON config. This achieves full interoperability between OpenCode and Cursor.

### Canonical Setup (OpenCode + Cursor)

**Step 1: Create portable agent markdown**

`~/.agents/code-reviewer.md`:
```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
mode: subagent
temperature: 0.1
# NO model field - configure per-tool in JSON
---

You are a code reviewer focusing on security, performance, and maintainability.
```

**Step 2: Symlink to tool directories**

```bash
# Symlink canonical agents to each tool
ln -sfn ~/.agents ~/.cursor/agents
ln -sfn ~/.agents ~/.config/opencode/agents
```

**Step 3: Configure tool-specific models**

**Cursor config** (per Cursor's configuration method):
```json
{
  "agent": {
    "code-reviewer": {
      "model": "claude-sonnet-4.5"
    }
  }
}
```

**OpenCode config** `opencode.json` (project-level):
```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "code-reviewer": {
      "model": "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

### Result

| Tool | Model Used | Source |
|------|------------|--------|
| **Cursor** | `claude-sonnet-4.5` | Cursor JSON config |
| **OpenCode** | `anthropic/claude-sonnet-4-20250514` | OpenCode JSON config |

Both tools read the same symlinked markdown file (no `model` field), then apply their respective JSON configs for model selection.

**Critical:** Do NOT include `model` in YAML frontmatter when using this pattern. YAML frontmatter takes precedence over JSON config in OpenCode, which would prevent the JSON override from working.

**Note**: Include `name` in frontmatter even though OpenCode can use filename - ensures compatibility when symlinked.

## Model Selection Across Tools

Each tool handles model assignment differently with **incompatible formats**:

| Tool | Model Format | Example |
|------|--------------|---------|
| **OpenCode** | Full provider/model-id | `anthropic/claude-sonnet-4-20250514` |
| **Cursor** | Cursor model names | `claude-sonnet-4.5`, `gpt-4o`, `o1` |
| **Claude Code** | Aliases | `sonnet`, `opus`, `haiku`, `inherit` |
| **Cline** | Not supported | N/A |

### Achieving Interoperability (OpenCode + Cursor)

**The model field uses incompatible formats** - you cannot write both in the same YAML frontmatter. The solution: **omit `model` from YAML frontmatter entirely**, then specify tool-specific models in their respective JSON configs.

**Portable agent markdown** (works for both tools):
```yaml
---
name: code-reviewer
description: Reviews code for quality
# NO model field - omit entirely for portability
---
```

**Cursor model config** (in Cursor settings or project config):
```json
{
  "agent": {
    "code-reviewer": {
      "model": "claude-sonnet-4.5"
    }
  }
}
```

**OpenCode project config** `opencode.json`:
```json
{
  "agent": {
    "code-reviewer": {
      "model": "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

**Why this works:**
- **Cursor** reads markdown, finds no `model` field, uses model from Cursor's JSON config
- **OpenCode** reads markdown, finds no `model` field, uses model from `opencode.json`
- Same symlinked markdown file works for both tools
- Each tool's JSON config specifies tool-specific model format

**Critical:** If you specify `model` in YAML frontmatter, OpenCode will use that value and ignore the JSON config. The markdown frontmatter takes precedence over JSON config for agents loaded from `.opencode/agents/` directories.

### Including Claude Code

For Claude Code, also omit `model` from YAML and use its config:

```json
{
  "agent": {
    "code-reviewer": {
      "model": "sonnet"
    }
  }
}
```

## Tool Restrictions Across Tools

| Tool | Mechanism | Format |
|------|-----------|--------|
| **Claude Code** | `tools` (allowlist) or `disallowedTools` (denylist) | Comma-separated tool names: `Read, Grep, Glob` |
| **Cursor** | Not supported in agent frontmatter | N/A |
| **OpenCode** | `tools` (boolean map) | `tools: { write: false, edit: false, bash: false }` |
| **Cline** | Not supported | N/A |

**Portable approach**: Express tool restrictions in the system prompt body as instructions ("Do NOT modify files. Only read and analyze."). This works everywhere, though it's advisory rather than enforced.

## OpenCode Deep Dive

OpenCode provides the most comprehensive agent configuration system. This section covers OpenCode-specific features in detail.

### Configuration Methods

OpenCode supports two configuration approaches. **When using both methods, YAML frontmatter takes precedence over JSON config** - contrary to typical config merging patterns.

**Method 1: JSON Configuration (`opencode.json`)**

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "build": {
      "description": "Full development with all tools",
      "mode": "primary",
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.3,
      "tools": {
        "write": true,
        "edit": true,
        "bash": true
      },
      "permission": {
        "edit": "ask",
        "bash": {
          "*": "ask",
          "git status": "allow"
        }
      }
    }
  }
}
```

**Method 2: Markdown Files**

Place in `.opencode/agents/` (project) or `~/.config/opencode/agents/` (user):

```yaml
---
description: Full development with all tools
mode: primary
model: anthropic/claude-sonnet-4-20250514
temperature: 0.3
tools:
  write: true
  edit: true
  bash: true
permission:
  edit: ask
  bash:
    "*": ask
    "git status": allow
---

System prompt in markdown body.
```

Filename becomes the agent name. `build.md` → `build` agent.

**Merge Behavior - CRITICAL**: OpenCode loads config sources in precedence order:
1. `opencode.json` (project config) - loaded first
2. `.opencode/agents/*.md` (markdown files) - loaded second

**YAML frontmatter takes precedence over JSON config**. If a field exists in both locations, the YAML value wins. For portable agents that need tool-specific models, **omit the `model` field from YAML entirely**.

**Example - Portable agent with OpenCode-specific model:**

`.opencode/agents/code-reviewer.md`:
```yaml
---
description: Reviews code for quality and best practices
mode: subagent
temperature: 0.1
# NO model field - MUST omit to allow JSON config to work
---

You are a code reviewer. Focus on security, performance, and maintainability.
```

`opencode.json`:
```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "code-reviewer": {
      "model": "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

Result: OpenCode uses the model from JSON since YAML has no `model` field. The markdown file remains portable (works in Cursor/Claude Code via symlinks).

**Important: OpenCode is unique** - Claude Code and Cursor only support markdown files with YAML frontmatter. OpenCode's dual JSON/markdown support allows tool-specific configuration, but YAML frontmatter takes precedence over JSON when fields conflict.

### Agent Modes

| Mode | Purpose | Invocation |
|------|---------|------------|
| `primary` | Main conversation agents | Tab key to cycle |
| `subagent` | Specialized task agents | `@mention` or Task tool |
| `all` | Can be used as either | Both methods |

Omit `mode` to default to `all`.

### Built-in Agents

| Agent | Mode | Purpose | Tool Access |
|-------|------|---------|-------------|
| `build` | primary | Full development (default) | All tools enabled |
| `plan` | primary | Read-only planning/analysis | File edits = ask, bash = ask |
| `general` | subagent | Multi-step execution | Full access (no todo) |
| `explore` | subagent | Fast read-only exploration | Read/search only |
| `compaction` | hidden | Context summarization | System agent |
| `title` | hidden | Session title generation | System agent |
| `summary` | hidden | Session summarization | System agent |

### Model Configuration

Use the full `provider/model-id` format:

```json
{
  "agent": {
    "plan": {
      "model": "anthropic/claude-haiku-4-20250514"
    },
    "build": {
      "model": "anthropic/claude-sonnet-4-20250514"
    }
  }
}
```

If omitted:
- **Primary agents**: Use globally configured model
- **Subagents**: Inherit from parent agent's model

Run `opencode models` to see available models.

### Tool Control

Control tool access with boolean maps:

```yaml
tools:
  write: true     # Enable
  edit: false     # Disable
  bash: true
```

**Wildcard support** for MCP tools:

```yaml
tools:
  mymcp_*: false  # Disable all tools from 'mymcp' server
  write: false
  edit: false
```

Agent-specific `tools` overrides global config.

### Permission System

Fine-grained control over tool behavior. Three modes:

| Mode | Behavior |
|------|----------|
| `allow` | Execute without approval |
| `ask` | Prompt before execution |
| `deny` | Block entirely |

#### File Edit Permissions

```json
{
  "permission": {
    "edit": "ask"  // Prompt before all edits
  }
}
```

**Limitation**: No path-based restrictions. Applies to all file edits.

#### Bash Command Permissions

Supports **glob patterns** for command-specific control:

```json
{
  "permission": {
    "bash": {
      "*": "ask",              // Default: ask for all commands
      "git status": "allow",   // Allow specific command
      "git log*": "allow",     // Allow with wildcards
      "grep *": "allow",       // Allow command with args
      "git push": "deny",      // Block dangerous command
      "rm -rf*": "deny"        // Block destructive patterns
    }
  }
}
```

**Pattern matching rules:**
- Last matching rule wins
- Put `*` wildcard first, then specific overrides
- Patterns match full command string including arguments

**Example: Safe git agent**

```yaml
permission:
  bash:
    "*": "deny"            # Block everything by default
    "git status*": "allow"
    "git log*": "allow"
    "git diff*": "allow"
    "git show*": "allow"
  edit: "deny"
  webfetch: "deny"
```

#### WebFetch Permissions

```json
{
  "permission": {
    "webfetch": "ask"  // Prompt before fetching URLs
  }
}
```

#### Subagent Spawning Permissions

Control which subagents an agent can invoke via Task tool:

```json
{
  "agent": {
    "orchestrator": {
      "permission": {
        "task": {
          "*": "deny",                 // Block all by default
          "orchestrator-*": "allow",   // Allow specific prefix
          "code-reviewer": "ask"       // Prompt for specific agent
        }
      }
    }
  }
}
```

When set to `deny`, the subagent is **removed from Task tool description entirely**, preventing the model from attempting invocation.

**Note**: Users can always invoke any subagent via `@mention`, regardless of task permissions.

### Temperature & Top-P

Control response randomness:

```yaml
temperature: 0.1  # 0.0-1.0 (lower = focused, higher = creative)
top_p: 0.9        # 0.0-1.0 (alternative to temperature)
```

**Temperature guidelines:**
- **0.0-0.2**: Code analysis, planning, deterministic tasks
- **0.3-0.5**: General development
- **0.6-1.0**: Brainstorming, creative writing

If unspecified, defaults to model-specific values (typically 0, or 0.55 for Qwen models).

### Max Steps

Limit agentic iterations:

```yaml
steps: 5  # Max tool-use iterations before text-only response
```

When limit reached, agent receives a system prompt instructing it to summarize work and recommend remaining tasks.

Omit to allow unlimited iterations until model stops or user interrupts.

**Note**: Legacy `maxSteps` field is deprecated. Use `steps`.

### Visual Customization

```yaml
color: "#ff6b6b"  # Hex color
# or
color: "accent"   # Theme color: primary, secondary, accent, success, warning, error, info
```

Affects agent appearance in UI.

### Hidden Subagents

Hide from `@` autocomplete menu (subagents only):

```yaml
mode: subagent
hidden: true
```

Still invokable via Task tool. Useful for internal helper agents.

### Provider-Specific Options

Any unrecognized frontmatter fields are **passed through to the provider** as model options.

**Example: OpenAI reasoning models**

```yaml
model: openai/gpt-5
reasoningEffort: high  # OpenAI-specific
textVerbosity: low     # OpenAI-specific
```

Check provider docs for available parameters.

### Disabling Agents

```json
{
  "agent": {
    "review": {
      "disable": true
    }
  }
}
```

### Custom Prompts

Reference external prompt files:

```json
{
  "agent": {
    "review": {
      "prompt": "{file:./prompts/code-review.txt}"
    }
  }
}
```

Path is relative to config file location (works for both global and project configs).

### Complete Example: Read-Only Review Agent

**Markdown approach:**

`.opencode/agents/review.md`

```yaml
---
description: Reviews code for quality without making changes. Use after implementation.
mode: subagent
model: anthropic/claude-sonnet-4-20250514
temperature: 0.1
tools:
  write: false
  edit: false
permission:
  bash:
    "*": "ask"
    "git diff*": "allow"
    "git log*": "allow"
    "grep *": "allow"
  webfetch: "deny"
color: "warning"
---

You are a code reviewer. Focus on:

- Security vulnerabilities
- Performance issues
- Best practices violations
- Potential bugs

Provide constructive feedback without making changes.
```

**JSON approach:**

`opencode.json`

```json
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "review": {
      "description": "Reviews code for quality without making changes. Use after implementation.",
      "mode": "subagent",
      "model": "anthropic/claude-sonnet-4-20250514",
      "temperature": 0.1,
      "prompt": "{file:./.opencode/prompts/review.txt}",
      "tools": {
        "write": false,
        "edit": false
      },
      "permission": {
        "bash": {
          "*": "ask",
          "git diff*": "allow",
          "git log*": "allow",
          "grep *": "allow"
        },
        "webfetch": "deny"
      },
      "color": "warning"
    }
  }
}
```

### Creating Agents via CLI

Interactive agent creation:

```bash
opencode agent create
```

Prompts for:
1. Save location (global vs project)
2. Agent description
3. Tool access preferences
4. Generates markdown file with configuration

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
