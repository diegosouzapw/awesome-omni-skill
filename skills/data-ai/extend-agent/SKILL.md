---
name: extend-agent
description: Create new skills, commands, hooks, or subagents for your AI agent. Use when you need to add new capabilities to Claude Code or Cursor — any type of extension.
---

<essential_principles>

## Types of Agent Extensions

| Type | Claude Code | Cursor | Purpose |
|------|-------------|--------|---------|
| **Skill** | `~/.claude/skills/<name>/SKILL.md` | `.cursor/rules/<name>.mdc` | Domain knowledge + workflows |
| **Command** | `~/.claude/commands/<name>.md` | `.cursor/rules/<name>.mdc` (on-demand rule) | Quick action shortcuts |
| **Hook** | `.claude/hooks.json` | `alwaysApply: true` rules | Event-driven automation |
| **Subagent** | `~/.claude/agents/<name>.md` | `.cursor/agents/<name>.md` | Specialist AI instances |

Both Claude Code and Cursor use the same `.claude/agents/` path for subagents.

### Frontmatter Format by Type

**Skill — Claude Code (SKILL.md):**
```yaml
---
name: skill-name
description: What it does and when to use it.
---
```

**Skill — Cursor (.mdc):**
```yaml
---
description: What it does and when it applies
globs: ["**/*.py"]      # File patterns (empty = on-demand)
alwaysApply: false       # true = always in context
---
```

**Subagent — Claude Code vs Cursor differences:**
| Field | Claude Code | Cursor |
|-------|-------------|--------|
| `name` | ✅ Required | ✅ Required |
| `description` | ✅ Required | ✅ Required |
| `model` | `sonnet/opus/haiku/inherit` | `fast/inherit/<model-id>` |
| `tools` | ✅ Restricts tool access | Ignored |
| `readonly` | Ignored | ✅ Read-only mode |
| `is_background` | Ignored | ✅ Background execution |

</essential_principles>

<intake>
What would you like to create?

1. **Skill** — Domain expertise module (SKILL.md for Claude / .mdc for Cursor)
2. **Command** — Quick action shortcut
3. **Hook** — Event-driven automation
4. **Subagent** — Specialist AI instance

**Wait for response before proceeding.**
</intake>

<routing>
| Choice | Instructions |
|--------|-------------|
| 1, "skill", "rule", "mdc" | Read `workflows/create-simple-skill.md` |
| 2, "command", "slash" | See `<create_command>` below |
| 3, "hook", "automation" | See `<create_hook>` below |
| 4, "subagent", "agent" | Read `workflows/create-subagent.md` |
</routing>

<create_command>

### Creating a Command

**Claude Code** (`.claude/commands/<name>.md`):
```yaml
---
description: What this command does (appears in /help)
argument-hint: [optional-arg]
allowed-tools: Bash(git add:*), Bash(git commit:*)  # Optional
---

<objective>
What to accomplish. Use $ARGUMENTS if taking user input.
</objective>

<context>
Current status: !`git status`
Relevant file: @package.json
</context>

<process>
1. Step one
2. Step two
3. Verify
</process>

<success_criteria>
Definition of done.
</success_criteria>
```
Install: `cp command.md ~/.claude/commands/`
Invoke: `/command-name [args]`

**Cursor** (`.cursor/rules/<name>.mdc` with `alwaysApply: false`):
```yaml
---
description: When asked to [action], follow this workflow
globs: []
alwaysApply: false
---

<process>
1. Step one
2. Step two
</process>
```
Key difference: Cursor has no `/` invocation — the rule activates when the user describes the action.

**Arguments:**
- `$ARGUMENTS` — all args as one string: `/fix-issue 123` → `$ARGUMENTS = "123"`
- `$1`, `$2`, `$3` — positional: `/review-pr 456 high` → `$1=456`, `$2=high`
- No args — operates on current context (git status, open file, etc.)

</create_command>

<create_hook>

### Creating a Hook (Claude Code)

Hooks are event-driven automation in `.claude/hooks.json`.

**Hook events:**
| Event | When | Blocking? |
|-------|------|-----------|
| `PreToolUse` | Before tool runs | Yes |
| `PostToolUse` | After tool runs | No |
| `UserPromptSubmit` | User submits prompt | Yes |
| `Stop` | Agent tries to stop | Yes |
| `SubagentStop` | Subagent tries to stop | Yes |
| `SessionStart` | Session begins | No |
| `SessionEnd` | Session ends | No |
| `PreCompact` | Before context compaction | Yes |
| `Notification` | Claude needs input | No |

**Command hook template:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/my-hook.sh",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

**Blocking hook output** (stdout from your script):
```json
{"decision": "block", "reason": "Why this was blocked"}
```

**Prompt hook template** (LLM evaluates instead of a script):
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{"type": "prompt", "prompt": "Is this command safe: $ARGUMENTS\nReturn JSON: {\"decision\": \"approve\" or \"block\", \"reason\": \"...\"}"}]
    }]
  }
}
```
Use `prompt` type when decision requires reasoning; `command` type for deterministic checks.
Test hooks with: `claude --debug`

**Matchers:**
```
"Bash"           → Exact tool name
"Write|Edit"     → Multiple tools (regex OR)
"mcp__.*"        → All MCP tools
(omit matcher)   → Fires for all tools
```

**Cursor equivalent:** No native hook system. Use `alwaysApply: true` rules:
```yaml
---
alwaysApply: true
---

NEVER run: rm -rf, git push --force, DROP TABLE.
ALWAYS check git status before committing.
```

See `skills/security/hooks/` for working hook script examples.

</create_hook>

<create_skill_and_subagent_notes>

**Skill complexity rule:**
- Simple (4-10 lines of instructions) → single `SKILL.md` file
- Complex (multi-step workflows, multiple domains) → router pattern with `workflows/`, `references/`, `templates/` subdirs

**Subagent execution model — critical constraints:**
- Subagents **cannot** use `AskUserQuestion` or wait for user input
- Run isolated — user sees only final output, not intermediate steps
- Invoked automatically (Claude matches `description` field) or explicitly via the Task tool
- Project subagents (`.claude/agents/`) override user subagents (`~/.claude/agents/`) on name conflict
- For multi-stage orchestration (research → plan → execute pipeline): use the planning skill + Task tool, not a single subagent

</create_skill_and_subagent_notes>

<reference_index>
- `references/skill-structure.md` — Complete skill patterns and XML tags
- `references/cursor-format.md` — Cursor .mdc format in depth
- `references/core-principles.md` — Prompting principles for all extension types
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| `workflows/create-simple-skill.md` | Build a single-file skill or a complex router skill with subdirectories |
| `workflows/convert-to-cursor.md` | Transform a SKILL.md to Cursor .mdc format |
| `workflows/create-subagent.md` | Build a specialist agent for Claude Code or Cursor |
</workflows_index>

<success_criteria>
- Valid YAML frontmatter for the target platform
- XML-structured body (no markdown headings in body)
- Description specific enough to trigger at the right time
- Installed in the correct location for the platform
- Tested with a representative user request
</success_criteria>
