---
name: build-and-test
description: Build, test, lint, and validate the Phoenix Agentic Website Frontend. Use when user asks to build, compile, test, lint, run checks, fix test failures, or validate changes in the Website Frontend repo.
---

# Build & Test — Phoenix Agentic Website Frontend (Next.js)

## Mandatory first step: terminal scope check

Before build/test commands, verify terminal scope:

1. `Set-Location "C:\Users\rivie\vsCodeProjects\Phoenix-Agentic-Website-Frontend"`
2. `Get-Location`
3. `git rev-parse --show-toplevel`
4. `git branch --show-current`

If scope is wrong, open a fresh Website Frontend-scoped terminal and retry.

## Repo Identity

This is the **public website frontend**. Next.js app with TypeScript, React, and Tailwind CSS.

## Quick Commands

| Task label | Command | Purpose |
|------------|---------|---------|
| `website: dev: setup` | `npm install` | Install dependencies |
| `website: dev: run` | `npm run dev` | Start dev server |
| `website: dev: lint` | `npm run lint` | Run ESLint |
| `website: dev: typecheck` | `npm run typecheck` | Run tsc --noEmit |
| `website: dev: test` | `npm run test` | Run vitest |
| `website: dev: build` | `npm run build` | Production build |
| `website: dev: check` | lint + typecheck + test + build | Full validation |
| `website: dev: clean` | Remove .next, coverage, dist | Clean artifacts |

## Using VS Code Tasks

Prefer using `run_task` or `create_and_run_task` to execute these tasks. Task IDs:

- `shell: website: dev: setup`
- `shell: website: dev: run`
- `shell: website: dev: lint`
- `shell: website: dev: typecheck`
- `shell: website: dev: test`
- `shell: website: dev: build`
- `shell: website: dev: check`
- `shell: website: dev: clean`

## Manual Commands (if needed)

```bash
# Install
npm install

# Lint
npm run lint

# Typecheck
npm run typecheck

# Test
npm run test

# Build
npm run build
```

## Validation checklist

1. **Lint passes** — `npm run lint`
2. **Typecheck passes** — `npm run typecheck`
3. **Tests pass** — `npm run test`
4. **Build succeeds** — `npm run build`
5. **Full check** — `npm run lint; npm run typecheck; npm run test; npm run build`

## Testing rules

- Tests must be deterministic — no network calls, no randomness without seed control
- Use vitest for all tests
- Always run full check before considering work complete

## Common issues

| Error | Fix |
|-------|-----|
| `npm ci` failure | Delete `node_modules` and `package-lock.json`, run `npm install` |
| ESLint errors | Run `npm run lint` and fix violations |
| TypeScript errors | Run `npm run typecheck` locally |
| Next.js build failure | Check for SSR-incompatible imports |
| Test failure | Run `npm run test` locally, check assertions |
