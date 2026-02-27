---
name: create-rule
description: Create persistent AI agent rules and instructions. Use when you want to create a rule, add coding standards, set up project conventions, configure file-specific patterns, or create AGENTS.md/GEMINI.md rule files across Cursor, Gemini CLI, or Codex.
---

# Creating AI Agent Rules

Rules provide persistent instructions the agent follows across sessions. Each platform uses a different mechanism—choose based on the current environment.

## Platform Overview

| Platform   | Format            | Project location | Scope control              |
| ---------- | ----------------- | ---------------- | -------------------------- |
| Cursor     | `.mdc` files      | `.cursor/rules/` | `globs` + `alwaysApply`    |
| Gemini CLI | `.mdc` files      | `.gemini/rules/` | `globs` + `alwaysApply`    |
| Codex      | `AGENTS.md` files | Anywhere in repo | Directory tree of the file |

---

## Gather Requirements

Before creating a rule, determine:

1. **Purpose**: What should this rule enforce or teach?
2. **Scope**: Should it always apply, or only for specific files/directories?
3. **File patterns**: If file-specific, which glob patterns?

If the user hasn't specified scope, ask:

- "Should this rule always apply, or only when working with specific files?"

If they mentioned specific files without providing patterns, ask:

- "Which file patterns should this rule apply to?" (e.g., `**/*.ts`, `backend/**/*.py`)

Use a structured question tool if available, otherwise ask conversationally.

---

## Cursor & Gemini CLI: `.mdc` Rule Files

Rules are `.mdc` files with YAML frontmatter placed in the platform's rules directory:

```
.cursor/rules/      # Cursor
.gemini/rules/      # Gemini CLI
```

### File Structure

```markdown
---
description: Brief description of what this rule does
globs: **/*.ts
alwaysApply: false
---

# Rule Title

Your rule content here...
```

### Frontmatter Fields

| Field         | Type    | Description                                              |
| ------------- | ------- | -------------------------------------------------------- |
| `description` | string  | What the rule does (shown in rule picker)                |
| `globs`       | string  | File pattern — rule applies when matching files are open |
| `alwaysApply` | boolean | If `true`, applies to every session                      |

### Rule Configurations

**Always Apply** — universal standards that apply to every conversation:

```yaml
---
description: Core coding standards for the project
alwaysApply: true
---
```

**Apply to Specific Files** — conventions scoped to file types:

```yaml
---
description: TypeScript conventions for this project
globs: **/*.ts
alwaysApply: false
---
```

---

## Codex: `AGENTS.md` Files

Codex reads `AGENTS.md` files placed anywhere in the repo. A file's scope covers the entire directory tree rooted at its location. More deeply nested files take precedence over parent ones.

```markdown
<!-- AGENTS.md at repo root — applies project-wide -->

- Always use `async/await` over `.then()` chains.
- Run `npm test` after any logic change.
- Follow the error handling pattern defined in `src/errors.ts`.
```

**Placement guide:**

| File location       | Effective scope             |
| ------------------- | --------------------------- |
| `AGENTS.md`         | Entire project              |
| `src/AGENTS.md`     | Everything under `src/`     |
| `src/api/AGENTS.md` | Everything under `src/api/` |

---

## Best Practices

- **Under 50 lines**: Keep rules concise and focused.
- **One concern per rule**: Split large rules into separate files.
- **Actionable**: Write like clear internal docs, not vague guidelines.
- **Concrete examples**: Show the bad pattern and the correct pattern.

### Example (`.mdc`)

```markdown
---
description: TypeScript error handling standards
globs: **/*.ts
alwaysApply: false
---

# Error Handling

\`\`\`typescript
// ❌ BAD
try { await fetchData(); } catch (e) {}

// ✅ GOOD
try {
await fetchData();
} catch (e) {
logger.error('Failed to fetch', { error: e });
throw new DataFetchError('Unable to retrieve data', { cause: e });
}
\`\`\`
```

---

## Checklist

- [ ] Correct format for current platform (`.mdc` vs `AGENTS.md`)
- [ ] Frontmatter configured correctly (Cursor/Gemini CLI only)
- [ ] Content under 500 lines
- [ ] Includes at least one concrete example
