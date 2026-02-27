---
name: asset-forge
description: Creates new skills, rules, and MCPs for ai-driven-dev-system or project-specific use. Use when user requests a new reusable component, wants to add coding standards, needs to document a workflow, or asks to create a skill or rule.
---

# Asset Forge

Creates assets following system standards. Works for both global (ai-driven-dev-system) and project-specific assets.

## Before Creating: Gather Requirements

### 1. First: Infer from Context

**ALWAYS try to infer information from:**

- Current conversation history
- User's recent corrections or feedback
- Open files and code context
- Previous agent responses

### 2. Only Ask if Missing

If you cannot infer the following, then ask:

1. **Asset type**: skill, rule, or MCP?
2. **Purpose**: What should this asset do?
3. **Scope**: Always apply, or file-specific? (for rules)

**Principle**: The user's time is valuable. Extract maximum info from context before asking questions.

## Deciding Where to Create

### Decision Flow

1. **Is this reusable across multiple projects?**
   - YES → Create in ai-driven-dev-system (global) via PR
   - NO → Create in current project's `.cursor/`

2. **Is this specific to this codebase?**
   - YES → Create in project's `.cursor/skills/` or `.cursor/rules/`
   - NO → Create in global system

### Location Summary

| Scope   | Skills                         | Rules                         |
| :------ | :----------------------------- | :---------------------------- |
| Global  | `ai-driven-dev-system/skills/` | `ai-driven-dev-system/rules/` |
| Project | `.cursor/skills/`              | `.cursor/rules/`              |

**Note**: Global assets require PR workflow (use system-gardener). Project assets can be committed directly.

## Creating Skills

### Directory Structure

```text
skills/[skill-name]/
├── SKILL.md              # Required
├── reference.md          # Optional - detailed docs
├── scripts/              # Optional - utilities
└── tests/                # Required - validation and unit tests
```

### SKILL.md Template

```markdown
---
name: skill-name
description: Third-person description. Include WHAT it does and WHEN to use it.
---

# Skill Title

Brief overview.

## Before You Begin

Prerequisites or requirements.

## Instructions

Step-by-step guidance.

## Examples

Concrete examples with input/output.

## Checklist

- [ ] Verification items
```

### Frontmatter Rules

- `name`: max 64 chars, lowercase, hyphens only (e.g., `code-review`)
- `description`: max 1024 chars, third person, include trigger terms

### Description Best Practices

Write in third person (injected into system prompt):

- ✅ "Reviews code for quality and security issues. Use when..."
- ❌ "I can help you review code"

Include both WHAT and WHEN:

- WHAT: Specific capabilities
- WHEN: Trigger scenarios

### Testing Requirements

1. **Create `tests/` directory**.
2. **Create `[skill-name].test.ts`** using `assets/test.template.ts`.
3. **If `scripts/` exist**: Add dedicated unit tests for logic.
4. **Run `pnpm test`** to verify compliance.

## Creating Rules

### Rule File Format

```markdown
---
description: What the rule does
globs: **/*.ts
alwaysApply: false
---

# Rule Title

Guidelines and examples.

## Correct

\`\`\`typescript
// ✅ GOOD
example
\`\`\`

## Incorrect

\`\`\`typescript
// ❌ BAD
example
\`\`\`
```

### Frontmatter Fields

| Field         | Type    | Description                 |
| :------------ | :------ | :-------------------------- |
| `description` | string  | Shown in rule picker        |
| `globs`       | string  | File pattern for activation |
| `alwaysApply` | boolean | If true, always active      |

### Configuration Options

- `alwaysApply: true` → Universal, applies to every session
- `globs: "**/*.ts"` → Applies when matching files are open

## Core Principles

1. **Concise**: Under 500 lines, only add what agent doesn't know.
2. **Progressive disclosure**: Essential in main file, details in references.
   - **Agent Best Practice**: Use `npx tsx scripts/extract-frontmatter.ts` (or `pnpm tsx ...`) to inspect skills before reading full files.
3. **Concrete examples**: Show don't tell.
4. **One concern per asset**: Split large topics.

## CRITICAL: Git Protocol for Global Assets

When creating assets in ai-driven-dev-system:

1. **NEVER create directly** - Use system-gardener skill
2. Create in feature branch
3. Submit PR for review
4. Wait for human approval

For project assets, commit directly to project repo.

## Checklist Before Creating

- [ ] Determined scope (global vs project)
- [ ] Correct location and format
- [ ] Description is specific with trigger terms
- [ ] Content under 500 lines
- [ ] Includes concrete examples
- [ ] Created unit tests in tests/
- [ ] Verified with pnpm test
- [ ] If global: using PR workflow
