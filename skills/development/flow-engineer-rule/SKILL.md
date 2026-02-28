---
name: flow-engineer-rule
description: Guide for creating persistent AI rules (coding standards, project conventions, file-specific patterns). Use when users want to create a rule, add coding standards, set up project conventions, configure file-specific patterns, or ask about rules placement. Works across IDEs (Cursor, Claude Code, Antigravity, OpenAI Codex, OpenCode).
disable-model-invocation: true
license: Based on https://github.com/anthropics/skills
---

# Rule Creator

This skill guides through creating persistent rules — instructions that automatically apply to AI agent sessions, enforcing coding standards, project conventions, and file-specific patterns.

## About Rules

Rules are markdown files with metadata that provide persistent context to AI agents. They differ from skills in that rules are **automatically injected** into every relevant session without explicit invocation.

Rules serve two purposes:
1. **Always-apply rules** — universal standards for every conversation (coding style, architecture decisions, project conventions)
2. **Conditional rules** — file-specific patterns triggered when matching files are open (TypeScript conventions for `*.ts`, React patterns for `*.tsx`)

## IDE Detection and Rule Placement

Rules work across multiple IDEs but use different file formats and locations. Before creating a rule, determine the current environment.

### Control Primitives Map by IDE

| Primitive | Scope | Claude Code | Antigravity | Cursor | OpenAI Codex | OpenCode |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Persistent Instructions** | User | `~/.claude/CLAUDE.md` | `~/.gemini/GEMINI.md` | - | `~/.codex/AGENTS.md`<br>`~/.codex/AGENTS.override.md` | `~/.config/opencode/AGENTS.md`<br>`~/.claude/CLAUDE.md` (fallback) |
| | Project | `CLAUDE.md`<br>`.claude/rules/*.md` | `.agent/rules/*.md` | `AGENTS.md`<br>`.cursor/rules/*/RULE.md`<br>~~`.cursor/rules/*.mdc`~~ | `AGENTS.md`<br>`AGENTS.override.md` | `AGENTS.md`<br>`CLAUDE.md` (fallback)<br>`opencode.json` `instructions` |
| | Folder | `subdir/CLAUDE.md`<br>`CLAUDE.local.md` | - | `subdir/AGENTS.md` | `subdir/AGENTS.md`<br>`subdir/AGENTS.override.md` | - |
| **Conditional Instructions** | Project | `.claude/rules/*.md` | - | `.cursor/rules/*/RULE.md`<br>~~`.cursor/rules/*.mdc`~~ | - | `opencode.json` `instructions` (globs) |
| **Custom Commands** | User | `~/.claude/commands/*.md` | `~/.gemini/antigravity/global_workflows/global-workflow.md` | `~/.cursor/commands/*.md` | `~/.codex/prompts/*.md` | `~/.config/opencode/commands/*.md` |
| | Project | `.claude/commands/*.md`<br>`.claude/commands/<namespace>/*.md` | `.agent/workflows/*.md` | `.cursor/commands/*.md` | - | `.opencode/commands/*.md` |
| **Event Hooks** | User | `~/.claude/settings.json` | - | `~/.cursor/hooks.json` | - | `~/.config/opencode/plugins/*.{js,ts}` |
| | Project | `.claude/settings.json`<br>`.claude/settings.local.json` | - | `.cursor/hooks.json` | - | `.opencode/plugins/*.{js,ts}` |
| **MCP Integration** | User | `settings.json`<br>`managed-mcp.json` | - | `~/.cursor/mcp.json` | `~/.codex/config.toml` | `opencode.json` `mcp` |
| | Project | `.mcp.json` | - | `.cursor/mcp.json` | - | `opencode.json` `mcp` |
| **Context Ignoring** | User | `.claude/settings.json` | - | - | - | - |
| | Project | - | `.gitignore` | `.cursorignore` | `.gitignore` | `.gitignore`<br>`.ignore`<br>`opencode.json` `watcher.ignore` |

### Rule-Specific Paths

| IDE | Always-Apply Rules | Conditional Rules | Format |
|-----|-------------------|-------------------|--------|
| **Cursor** | `.cursor/rules/*/RULE.md` with `alwaysApply: true` | `.cursor/rules/*/RULE.md` with `globs:` | YAML frontmatter + Markdown |
| **Claude Code** | `CLAUDE.md`, `.claude/rules/*.md` | `.claude/rules/*.md` with `paths:` | YAML frontmatter + Markdown |
| **Antigravity** | `.agent/rules/*.md` | Not supported | Plain Markdown |
| **OpenAI Codex** | `AGENTS.md`, `AGENTS.override.md` | Not supported | Plain Markdown |
| **OpenCode** | `AGENTS.md`, `opencode.json` `instructions` | `opencode.json` `instructions` (globs) | `AGENTS.md`: Plain Markdown; `opencode.json`: JSON array of paths/globs/URLs |

### Detection Strategy

1. Check for IDE-specific markers in the project:
   - `.cursor/` directory -> Cursor
   - `.claude/` directory -> Claude Code
   - `.opencode/` directory or `opencode.json` -> OpenCode
   - `.agent/` directory -> Antigravity
   - `.codex/` directory or `AGENTS.md` without `.cursor/` -> OpenAI Codex
2. If multiple detected or none -> ask the user
3. Ask: always-apply or conditional (file-specific)?

## Gather Requirements

Before creating a rule, determine:

1. **Purpose**: What should this rule enforce or teach?
2. **Scope**: Always apply, or only for specific files?
3. **File patterns**: If conditional, which glob patterns? (e.g., `**/*.ts`, `backend/**/*.py`)

### Inferring from Context

If prior conversation context exists, infer rules from what was discussed. Create multiple rules if the conversation covers distinct topics. Don't ask redundant questions if context provides answers.

### Required Questions

If scope not specified:
- "Should this rule always apply, or only when working with specific files?"

If file-specific but no concrete patterns:
- "Which file patterns should this rule apply to?"

## Rule Format by IDE

### Cursor

Directory-based rules in `.cursor/rules/<rule-name>/RULE.md`:

```markdown
---
description: Brief description of what this rule does
globs: "**/*.ts"
alwaysApply: false
---

# Rule Title

Rule content here...
```

**Legacy format** (`.cursor/rules/*.mdc`) still works but is deprecated. Prefer directory format.

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | What the rule does (used for agent discovery when `alwaysApply: false`) |
| `globs` | string | File pattern — rule applies when matching files are in context |
| `alwaysApply` | boolean | If `true`, applies to every session regardless of files |

### Claude Code

Rules in `.claude/rules/*.md` with frontmatter:

```markdown
---
description: TypeScript conventions
paths: src/**/*.ts
---

# Rule Title

Rule content here...
```

Or project-wide in `CLAUDE.md` (no frontmatter, always applies).

### Antigravity

Rules in `.agent/rules/*.md` (plain markdown, always apply):

```markdown
# Rule Title

Rule content here...
```

### OpenAI Codex

Rules in `AGENTS.md` or `AGENTS.override.md` (plain markdown, always apply):

```markdown
# Rule Title

Rule content here...
```

Subdirectory rules: `subdir/AGENTS.md` applies when working in that directory.

### OpenCode

Always-apply rules in `AGENTS.md` (project root, plain markdown):

```markdown
# Rule Title

Rule content here...
```

Or via `opencode.json` field `instructions` (array of paths, globs, or URLs):

```jsonc
{
  "instructions": [
    "AGENTS.md",
    ".cursor/rules/**/*.md",
    "https://example.com/standards.md"
  ]
}
```

Global user rules: `~/.config/opencode/AGENTS.md`. Falls back to `~/.claude/CLAUDE.md` unless `OPENCODE_DISABLE_CLAUDE_CODE_PROMPT=1`.

## Core Principles

### Keep Rules Concise

- **Under 50 lines** preferred, max 500 lines
- **One concern per rule**: split large rules into focused pieces
- **Actionable**: write like clear internal docs
- **Concrete examples**: show good/bad patterns with code

### Structure

Every rule should contain:
1. **Clear title** — what aspect it covers
2. **Brief context** — why this rule exists (1-2 sentences max)
3. **The rules themselves** — concrete, actionable instructions
4. **Examples** — good/bad code patterns when applicable

### Naming

- Cursor: directory name = `kebab-case` (e.g., `typescript-standards/RULE.md`)
- Claude Code: filename = `kebab-case.md` (e.g., `typescript-standards.md`)
- Others: section heading in the shared file

## Rule Creation Process

### Phase 1: Discovery

1. Determine IDE (auto-detect or ask)
2. Determine scope (always-apply vs conditional)
3. Determine file patterns if conditional
4. Gather the actual standards/conventions to encode

### Phase 2: Implementation

1. Create the rule file in the correct location for the detected IDE
2. Write frontmatter with appropriate fields
3. Write concise, actionable rule content
4. Include concrete code examples (good/bad patterns)

### Phase 3: Verification

Run validation:

```bash
scripts/validate_rule.py <path/to/rule-file-or-directory>
```

Checklist:
- [ ] Correct file format for target IDE
- [ ] Frontmatter configured correctly (description, globs/paths, alwaysApply)
- [ ] Content under 500 lines (prefer under 50)
- [ ] Includes concrete examples
- [ ] One concern per rule
- [ ] No redundancy with existing rules

## Common Rule Categories

| Category | Scope | Example Patterns |
|----------|-------|-----------------|
| Coding standards | Always or per-language | Error handling, naming, imports |
| Architecture | Always | Layer boundaries, dependency rules |
| Framework patterns | File-specific | React components, API routes |
| Testing | File-specific | Test structure, mocking conventions |
| Documentation | File-specific | JSDoc format, comment standards |
| Security | Always | Auth patterns, input validation |

## Example Rules

### Error Handling (TypeScript)

```markdown
---
description: TypeScript error handling standards
globs: "**/*.ts"
alwaysApply: false
---

# Error Handling

Always use typed errors with context:

\`\`\`typescript
// BAD
try {
  await fetchData();
} catch (e) {}

// GOOD
try {
  await fetchData();
} catch (e) {
  logger.error('Failed to fetch', { error: e });
  throw new DataFetchError('Unable to retrieve data', { cause: e });
}
\`\`\`
```

### React Patterns

```markdown
---
description: React component patterns
globs: "**/*.tsx"
alwaysApply: false
---

# React Patterns

- Use functional components
- Extract custom hooks for reusable logic
- Colocate styles with components
- Props interface named `{ComponentName}Props`
```

### Project Architecture (Always Apply)

```markdown
---
description: Core architecture rules
alwaysApply: true
---

# Architecture

- Domain logic in `src/domain/` — no framework imports
- API handlers in `src/api/` — thin, delegate to domain
- Shared types in `src/types/` — no runtime code
```
