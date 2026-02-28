---
name: lint-and-format
description: Runs linting and formatting checks before committing. Use this skill after writing or modifying code to ensure it passes all linters and formatters before creating a commit.
globs: "{lib,test,config}/**/*.{ex,exs},cli/**/*.{ts,tsx,js,json}"
---

# Lint & Format Skill

You ensure all code passes the project's linters and formatters before it gets committed. This mirrors the lefthook pre-commit hooks and CI checks so problems are caught immediately.

## When to act

- After writing or modifying any source file, **before creating a commit**.
- When the user asks to fix lint or formatting issues.
- When a commit or CI fails due to formatting.

## Commands

### Elixir (lib/, test/, config/)

```bash
# Check formatting (what CI runs)
mise run lint:core

# Auto-fix formatting
mise run format:core
```

### TypeScript/JavaScript (cli/)

```bash
# ESLint check
cd cli && pnpm lint

# ESLint auto-fix
cd cli && pnpm lint:fix

# Prettier check
cd cli && pnpm format:check

# Prettier auto-fix
cd cli && pnpm format
```

### Run everything

```bash
# Check all (what CI runs)
mise run lint

# Fix all
mise run format
```

## Workflow

1. After making code changes, run the relevant lint/format check commands.
2. If there are failures, auto-fix them using the fix variants.
3. If auto-fix changes files, review the diff to make sure nothing unexpected changed.
4. Only then proceed with staging and committing.

## Rules

1. **Never commit code that fails linting or formatting checks.**
2. Prefer auto-fix (`mise run format`, `lint:fix`) over manual edits when possible.
3. If a lint rule seems wrong for a specific case, discuss with the user before adding a suppression comment.
4. Do not disable or weaken lint rules without explicit approval.
