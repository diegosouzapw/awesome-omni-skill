---
name: ai-context
description: Generates AI IDE context files (AGENTS.md, CLAUDE.md, .cursorrules, copilot-instructions.md) from codebase analysis. Creates project-specific instructions for AI coding assistants so they understand conventions, architecture, and workflows. Use when setting up AI tool context for a repository.
---

# AI Context File Generator

## Philosophy

AI coding assistants work better when they understand a project's conventions, architecture, and constraints. Context files tell AI tools **how** to work with the codebase — coding standards, test patterns, import conventions, key file paths, and deployment workflows.

This skill generates context files for multiple AI tools from a single codebase analysis. The same scan produces output for Claude Code, Codex CLI, Cursor, GitHub Copilot, and Gemini CLI.

## Supported Context Files

| File | AI Tool | Purpose | Adoption |
|------|---------|---------|----------|
| `AGENTS.md` | Codex CLI, Cursor, Gemini CLI, Claude Code | Cross-tool agent context — identity, capabilities, conventions | 40,000+ repos |
| `CLAUDE.md` | Claude Code | Project-specific instructions loaded on every session | Native to Claude Code |
| `.cursorrules` | Cursor | Editor-specific rules for code generation | Native to Cursor |
| `.github/copilot-instructions.md` | GitHub Copilot | Repository-level instructions for Copilot suggestions | Native to Copilot |

## Codebase Analysis Workflow

### Step 1: Detect Project Profile

```bash
# Language and runtime
ls package.json pyproject.toml Cargo.toml go.mod pom.xml build.gradle 2>/dev/null

# Framework detection
cat package.json 2>/dev/null | grep -E '"(react|next|express|fastify|hono|astro|svelte)"'
cat pyproject.toml 2>/dev/null | grep -E '(fastapi|django|flask|starlette)'

# Test runner
ls jest.config* vitest.config* pytest.ini pyproject.toml .mocharc* 2>/dev/null
grep -l "test" package.json 2>/dev/null

# Linter/formatter
ls .eslintrc* eslint.config* .prettierrc* biome.json ruff.toml .flake8 2>/dev/null

# TypeScript configuration
ls tsconfig*.json 2>/dev/null

# Monorepo detection
ls pnpm-workspace.yaml lerna.json nx.json turbo.json 2>/dev/null

# CI/CD
ls .github/workflows/*.yml 2>/dev/null
```

### Step 2: Extract Conventions

From the codebase analysis, extract:

1. **Language version** — Node.js version from `.nvmrc`/`engines`, Python from `requires-python`, Go from `go.mod`
2. **Import conventions** — ESM vs CommonJS, absolute vs relative imports, path aliases (`@/`)
3. **Naming patterns** — camelCase/snake_case for variables, PascalCase for types, file naming
4. **Directory structure** — where source, tests, config, and docs live
5. **Test patterns** — test file location (`__tests__/`, `*.test.ts`, `tests/`), test runner, assertion style
6. **Build/deploy** — build command, deploy target (Cloudflare, Vercel, AWS, etc.)
7. **Error handling** — custom error classes, Result types, try-catch patterns
8. **Security rules** — .gitignore patterns, secret management, input validation

### Step 3: Generate Context Files

#### AGENTS.md Structure

```markdown
# AGENTS.md

## Identity

[Project name] is a [brief description]. Built with [language/framework].

## Project Structure

```
[Key directories and their purpose]
```

## Coding Conventions

- [Language]: [version]
- Style: [naming conventions, import order]
- Types: [strict mode, no any, explicit returns — if TypeScript]
- Tests: [runner, location, naming pattern]
- Commits: [conventional commits, branch naming]

## Key Commands

```bash
[install command]
[test command]
[build command]
[lint command]
[deploy command]
```

## Architecture

[2-3 sentences on architecture: patterns used, key abstractions, data flow]

## Important Files

- [key config file] — [purpose]
- [main entry point] — [purpose]
- [key module] — [purpose]

## Rules

- [Critical rule 1 — e.g., never commit secrets]
- [Critical rule 2 — e.g., all public functions need tests]
- [Critical rule 3 — e.g., use direnv exec for deploy commands]
```

#### CLAUDE.md Structure

Claude Code loads this file at the start of every session. Keep it concise — under 200 lines.

```markdown
# [Project Name]

## Quick Reference

- **Language**: [X] with [framework]
- **Test**: `[test command]`
- **Build**: `[build command]`
- **Deploy**: `[deploy command]`

## Coding Standards

[3-5 bullet points on the most important conventions]

## Architecture

[Key patterns, file organisation, data flow — 3-5 bullet points]

## Key Paths

| Path | Purpose |
|------|---------|
| `src/` | Source code |
| `tests/` | Test files |
| ... | ... |

## Rules

[Critical do/don't rules that prevent common mistakes]
```

#### .cursorrules Structure

Cursor rules are plain text, loaded when editing files in the project.

```
You are working on [project name], a [description].

Language: [X] with [framework]
Style: [conventions]

Key rules:
- [Rule 1]
- [Rule 2]
- [Rule 3]

When writing code:
- [Pattern 1]
- [Pattern 2]

When writing tests:
- [Test pattern 1]
- [Test pattern 2]

File structure:
- src/ — source code
- tests/ — test files
```

#### .github/copilot-instructions.md Structure

```markdown
# Copilot Instructions

## Project Context

This is a [description] built with [language/framework].

## Coding Standards

- [Convention 1]
- [Convention 2]
- [Convention 3]

## Patterns to Follow

- [Pattern 1]
- [Pattern 2]

## Patterns to Avoid

- [Anti-pattern 1]
- [Anti-pattern 2]
```

## Staleness Audit

When running in `audit` mode, check existing context files for drift:

1. **Version mismatch** — Does the context file reference the correct language/framework version?
2. **Missing commands** — Are test/build/deploy commands still accurate? Run them to verify.
3. **Stale paths** — Do referenced file paths still exist?
4. **New conventions** — Has the project adopted new patterns (e.g., added ESLint, switched to Vitest) that aren't reflected?

Report format:
```
AI Context Audit:
  ✓ AGENTS.md — up to date (last modified: 2 days ago)
  ⚠ CLAUDE.md — references jest but vitest.config.ts detected
  ✗ .cursorrules — references src/index.ts but file moved to src/main.ts
  · .github/copilot-instructions.md — not present
```

## Anti-Patterns

- **Don't dump entire codebase** — context files should be concise summaries, not file listings
- **Don't include secrets** — no API keys, tokens, or credentials in context files
- **Don't repeat framework docs** — reference framework conventions, don't reproduce them
- **Don't over-constrain** — provide patterns, not rigid rules that prevent creative problem-solving
- **Don't include session-specific state** — context files should be durable across sessions
