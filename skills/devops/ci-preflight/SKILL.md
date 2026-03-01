---
name: ci-preflight
description: Runs the full CI check suite locally before pushing. Use this skill before pushing a branch or when the user asks to verify everything passes, to catch issues before they hit CI.
globs: "**/*"
---

# CI Preflight Skill

You run the same checks that GitHub Actions CI performs, but locally, to catch failures before pushing. This saves time and keeps the commit history clean.

## When to act

- Before pushing a branch to remote.
- When the user asks to "run CI", "check everything", or "preflight".
- After a large set of changes to verify nothing is broken.

## Check sequence

Run these in order. Stop and fix issues at each step before proceeding.

### 1. Elixir core

```bash
# Compile with warnings-as-errors (matches CI)
mise run build:core

# Check formatting
mise run lint:core

# Run tests
mise run test:core
```

### 2. TypeScript CLI

```bash
# Lint (ESLint)
mise run lint:cli

# Verify codegen is current
mix run scripts/codegen_ts.exs --check

# Build (typecheck + compile)
mise run build:cli
```

### 3. Quick summary

Or run everything at once:

```bash
mise run lint && mise run build && mise run test
```

## Rules

1. **Every check must pass before pushing.** If something fails, fix it first.
2. Report results clearly — list what passed and what failed.
3. For failures, diagnose the root cause and suggest or apply fixes.
4. If a test is flaky (passes on retry), flag it to the user rather than silently retrying.
