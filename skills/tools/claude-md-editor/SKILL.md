---
name: claude-md-editor
description: Edit and maintain CLAUDE.md and project documentation files. Use when modifying documentation, asking about best practices, or performing documentation maintenance tasks.
---

# CLAUDE.md Writing Guidelines

Guidelines for writing effective CLAUDE.md files, based on [Anthropic's official best practices](https://www.anthropic.com/engineering/claude-code-best-practices).

## Purpose of CLAUDE.md

CLAUDE.md is automatically pulled into context. Document:

- **Common bash commands** - Build, test, lint, deploy
- **Core files and utilities** - Key files Claude should know
- **Code style guidelines** - Formatting, patterns, conventions
- **Testing instructions** - How to run tests, patterns
- **Repository etiquette** - Branch naming, PR conventions
- **Unexpected behaviors** - Known gotchas, things that cause problems

## Core Principles

### 1. Keep It Concise

> "You're writing for Claude, not onboarding a junior dev."

CLAUDE.md consumes tokens on every prompt. Every line should earn its place.

**DO:**
- Write for Claude, not humans
- Use bullet points and tables over prose
- Link to detailed docs instead of duplicating
- Regularly audit and remove outdated content

**DON'T:**
- Include verbose explanations
- Duplicate information from code/comments
- Add documentation that's rarely referenced
- Include Table of Contents (Claude doesn't need navigation)

### 2. Use Emphasis Markers

Claude follows instructions better with explicit emphasis:

- `IMPORTANT:` - Rules that need attention
- `CRITICAL:` - Rules that must never be violated
- `NEVER:` - Absolute prohibitions
- `ALWAYS:` - Absolute requirements

### 3. Use Tables for Quick Reference

Tables are scannable and token-efficient:

```markdown
| Client Context | Import From | Notes |
|----------------|-------------|-------|
| Server Components | `@/lib/server` | Must await |
| Client Components | `@/lib/client` | No await |
```

### 4. Use DO/DON'T Examples

```markdown
// DON'T: Use raw color values
<p className="text-gray-500">Text</p>

// DO: Use semantic tokens
<p className="text-muted">Text</p>
```

### 5. Create a "Danger Zone" Section

Group actions that cause problems:

```markdown
## Danger Zone

| Action | Consequence |
|--------|-------------|
| `npm run dev` automatically | Dev server already running |
| `as any` casts | Defeats TypeScript |
```

## Structure Recommendation

```markdown
# CLAUDE.md

## Project Overview
## Quick Start
## Architecture
## Danger Zone
## Critical Rules
## Common Patterns
## Environment Setup
## Troubleshooting
## Additional Documentation
```

## What to Include vs. Link

### Include Directly

- Commands Claude runs frequently
- Critical rules that must never be violated
- Quick reference tables
- Brief DO/DON'T examples
- Danger Zone items

### Link to External Docs

- Detailed setup guides
- Comprehensive code examples
- Full API documentation
- Any content > 50 lines on a single topic

## Anti-Patterns to Avoid

### Verbose Explanations

```markdown
# Too verbose
The middleware handles authentication by using the server-side
utilities. It's critically important that you understand...

# Concise
**Middleware:** Return exact response from `updateSession()`.
```

### Table of Contents

Claude doesn't need navigation aids. ToCs waste tokens.

### Obvious Instructions

```markdown
# Claude already knows this
Use proper types. Write clean code.

# Project-specific
Use `Tables<'table_name'>` helpers, never verbose syntax.
```

### Duplicated Rules

Audit for rules stated multiple times. Consolidate into one location.

## Modular Rules Architecture

For larger projects, use modular rules in `.claude/rules/`:

```markdown
# CLAUDE.md (lean hub)

## Rules Reference

| Rule File | Contents |
|-----------|----------|
| `architecture.md` | Route structure, data flow |
| `patterns.md` | Common code patterns |
| `danger-zone.md` | All "never do" rules |
```

Benefits:
- Agents can include specific rule bundles
- Easier maintenance
- Reduces main CLAUDE.md size

## Tuning Your CLAUDE.md

### Iterate Like a Prompt

1. **Observe** - Notice when Claude doesn't follow instructions
2. **Strengthen** - Add emphasis markers (IMPORTANT, NEVER)
3. **Simplify** - Remove content Claude doesn't reference
4. **Test** - Try different phrasings

### Regular Audits

- Remove rules Claude consistently follows without prompting
- Strengthen rules Claude frequently ignores
- Update after major codebase changes
- Remove references to deleted files/patterns

## Review Checklist

- [ ] Can each section be shorter?
- [ ] Is every instruction necessary?
- [ ] Are critical rules emphasized?
- [ ] Is verbose content moved to linked docs?
- [ ] Are there redundant sections?
- [ ] Is there a Table of Contents? (Remove it)

## Resources

- [Anthropic's Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [CLAUDE.md Optimization - Arize](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/)
