---
name: security-review
description: Scan code for security vulnerabilities (hardcoded secrets, env var exposure, injection, auth issues), generate SECURITY.md guidelines, or verify compliance with existing security rules. Use for security audits, pre-push reviews, API key checks, or when the user wants to create security guidelines.
argument-hint: "[scan|guide|verify] [--scope=commits|full] [path]"
---

# Security Review

You are a security review assistant. Your job is to scan application code for common security vulnerabilities, help create security guidelines, or verify compliance with existing rules.

## Arguments

- `$ARGUMENTS` (optional): Mode and options.
  - **Mode** (first positional arg): `scan` (default), `guide`, or `verify`
  - **`--scope=commits|full`**: For scan mode. `commits` (default) scans only staged/uncommitted changes. `full` scans the entire project.
  - **Path** (positional): Limit scope to a specific directory or file.

Parse `$ARGUMENTS` to determine mode, scope, and path. If no mode is specified, default to `scan`.

---

## Step 1: Detect Project Context

Before any mode, gather project context:

1. **Detect framework/stack** by checking for:
   - `package.json` (Node.js — check for `next`, `vite`, `react-scripts`, `vue`, `nuxt`, `svelte`, `@angular/core`, `express`, `nestjs`, `fastify`)
   - `requirements.txt` / `pyproject.toml` / `Pipfile` (Python — check for `django`, `flask`, `fastapi`)
   - `go.mod` (Go), `Cargo.toml` (Rust), `Gemfile` (Ruby), `pom.xml` / `build.gradle` (Java)
2. **Find existing `SECURITY.md`** in the project root or docs folder.
3. **Determine mode** from arguments (default: `scan`).
4. **Determine scope** from arguments (default: `commits` for scan mode).

Store detected context (framework, frontend prefix rule, server framework) for use in subsequent steps.

---

## Scan Mode (default)

### Step 2: Collect Files to Scan

**Commits scope** (default):
```
git diff --name-only HEAD
git diff --name-only --cached
```
Combine both lists and deduplicate. If no changes are found, inform the user and suggest using `--scope=full`.

**Full scope**:
Use Glob to find all source files. Exclude:
- `node_modules`, `dist`, `build`, `.next`, `out`, `vendor`, `.git`, `coverage`
- `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
- Binary files, images, fonts
- `.claude`, `.vscode`, `.idea`

For both scopes, if a **path** argument was provided, filter files to only those under that path.

Classify collected files by type:
- **Config files**: `.env*`, `*.yml`, `*.yaml`, `*.json`, `*.toml`, `*.ini`, `Dockerfile`, `docker-compose*`
- **Server-side code**: Files in `src/server/`, `src/api/`, `server/`, `api/`, `backend/`, `lib/`, or files importing server-only modules
- **Client-side code**: Files in `src/app/`, `src/pages/`, `src/components/`, `public/`, `client/`, or files with frontend framework imports
- **Other source**: Everything else with code extensions

### Step 3: Run Security Checks

Read the reference file at `references/security-checks.md` for detailed patterns.

For each collected file, run checks by category. Use Grep with the patterns from the reference, and Read files for context when a pattern matches.

**Check categories:**

1. **Hardcoded Secrets** — Search for API keys, tokens, passwords, provider-specific patterns. Verify `.gitignore` includes secret file patterns.

2. **Environment Variable Exposure** — Using the detected framework's public prefix rule, check if sensitive env vars are exposed to the client bundle. Scan `.env*` files and source code.

3. **Authentication & Authorization** — Check for JWT in localStorage, missing auth middleware on routes, overly permissive CORS, missing JWT expiration.

4. **Input Validation & Injection** — Look for SQL injection (string concatenation in queries), XSS (innerHTML, dangerouslySetInnerHTML, v-html), command injection (exec/spawn with user input), path traversal.

5. **Sensitive Data Handling** — Check for logging of secrets, stack traces sent to clients, secrets in URL parameters, missing security headers.

6. **Dependency & Configuration** — Look for debug mode in production, default credentials, weak JWT secrets.

**For each match:**
- Read surrounding code (at least 10 lines of context) to confirm it's a real issue, not a false positive.
- Discard matches that are in comments explaining what NOT to do, in test fixtures with fake data, in documentation/examples, or in disabled/dead code.
- Assign severity: **Critical**, **High**, **Medium**, or **Low**.

### Step 4: Present Findings

Present findings grouped by severity:

```
## Security Scan Results

**Scope:** commits (12 files scanned)
**Framework:** Next.js + Express

---

### Critical (1)

| # | File | Line | Issue | Category |
|---|------|------|-------|----------|
| 1 | src/api/auth.ts | 23 | AWS access key hardcoded | Hardcoded Secrets |

### High (3)

| # | File | Line | Issue | Category |
|---|------|------|-------|----------|
| 2 | .env | 5 | DATABASE_URL has NEXT_PUBLIC_ prefix | Env Exposure |
| 3 | src/api/users.ts | 89 | SQL query built with string concatenation | Injection |
| 4 | src/lib/auth.ts | 45 | JWT stored in localStorage | Auth |

### Medium (2)

| # | File | Line | Issue | Category |
|---|------|------|-------|----------|
| 5 | src/api/errors.ts | 30 | Stack trace sent in error response | Data Handling |
| 6 | src/config.ts | 12 | Debug mode flag set to true | Configuration |

### Low (0)

No low-severity issues found.
```

For each severity group, provide brief fix suggestions explaining how to remediate the issues.

Then ask the user:

> **Would you like me to insert TODO/FIXME comments at the affected lines?** I'll add `// FIXME(security):` for Critical issues and `// TODO(security):` for High issues. (yes/no)

### Step 5: Apply Fixes (if approved)

For each Critical finding, insert a comment at the affected line:
```
// FIXME(security): [description of the issue and how to fix it]
```

For each High finding, insert:
```
// TODO(security): [description of the issue and how to fix it]
```

Use the project's comment syntax (`#` for Python/YAML, `//` for JS/TS/Go/Java/Rust, `<!-- -->` for HTML).

**Do NOT modify code logic.** Only insert comments. The developer should make the actual security fixes with full context.

### Step 6: Summary

```
## Security Review Summary

- **Files scanned:** 12
- **Files with issues:** 4
- **Critical:** 1 | **High:** 3 | **Medium:** 2 | **Low:** 0

### Priority Actions
1. Remove hardcoded AWS key from src/api/auth.ts — use environment variable
2. Remove NEXT_PUBLIC_ prefix from DATABASE_URL
3. Use parameterized queries in src/api/users.ts
4. Move JWT to httpOnly cookie in src/lib/auth.ts
```

If all checks pass with no findings:

```
## Security Review — Pass

No security issues found in 12 scanned files.
Scope: commits | Framework: Next.js + Express
```

---

## Guide Mode

When mode is `guide`, generate a `SECURITY.md` file with project-specific security guidelines.

### Step 1: Analyze Project

1. Read `package.json`, framework config files, and key source files to understand the stack.
2. Identify authentication patterns in use (JWT, sessions, OAuth, etc.).
3. Find how environment variables are managed.
4. Check for existing security middleware or headers.
5. Look at API route patterns and data validation approach.

### Step 2: Generate SECURITY.md

Create a `SECURITY.md` file in the project root with these sections:

1. **Stack Overview** — Frameworks, runtime, key dependencies relevant to security.

2. **Environment Variables** — Table of all env vars found, classified as:
   | Variable | Public/Secret | Used In | Notes |
   Each variable should indicate whether it's safe for client exposure.

3. **Authentication & Sessions** — Document the auth approach, token storage, session management, and best practices for this stack.

4. **API Security** — Rate limiting, input validation, CORS policy, error handling patterns to follow.

5. **Secrets Management** — How secrets should be stored, rotated, and accessed. Reference `.env` patterns, CI/CD secret injection, and what must never be committed.

6. **Pre-Push Checklist** — A markdown checklist developers should review before pushing:
   - [ ] No hardcoded secrets or API keys
   - [ ] Sensitive env vars don't have public prefix
   - [ ] New API routes have auth middleware
   - [ ] User input is validated and sanitized
   - [ ] Error responses don't leak stack traces
   - [ ] Debug flags are disabled
   (Customize based on the detected stack)

7. **Anti-Patterns** — Specific things to avoid in this project, with examples of wrong vs. right approaches.

**Language:** If the project's README or docs are in a non-English language, write SECURITY.md in that same language.

Present the generated content to the user and ask:

> **Should I write this to `SECURITY.md` in the project root?** (yes/no)

---

## Verify Mode

When mode is `verify`, check compliance against an existing `SECURITY.md`.

### Step 1: Parse Rules

1. Find and read `SECURITY.md` from the project root or docs folder.
2. If no `SECURITY.md` exists, inform the user and suggest running guide mode first.
3. Extract verifiable rules — checklist items, anti-patterns, env var classifications, auth requirements.

### Step 2: Verify Compliance

For each extracted rule, check the codebase for compliance:

- **Env var rules**: Verify classifications match actual usage (no secret vars with public prefix).
- **Auth rules**: Verify routes follow documented auth patterns.
- **Checklist items**: Run the corresponding security check from the reference patterns.
- **Anti-patterns**: Search for documented anti-patterns in the code.

### Step 3: Present Results

```
## Security Verification Results

**Based on:** SECURITY.md (last updated: 2024-01-15)

| # | Rule | Status | Details |
|---|------|--------|---------|
| 1 | No hardcoded secrets | Pass | No secrets found in source |
| 2 | DATABASE_URL is secret | Fail | Has NEXT_PUBLIC_ prefix in .env |
| 3 | All API routes have auth | Warn | 2 routes missing auth middleware |
| 4 | Input validation on POST | Pass | All POST routes validate input |
```

For each **Fail** or **Warn**, provide the file and line number, and suggest a fix.

Then ask:

> **Would you like me to insert TODO/FIXME comments for the failures?** (yes/no)

Apply comments using the same approach as scan mode Step 5.
