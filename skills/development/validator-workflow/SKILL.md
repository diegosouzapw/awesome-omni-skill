---
name: validator-workflow
description: "Phase-level validation workflow for validator agents. Handles loading project rules, reading phase files, running /code-review, conditional verification (unit tests + typecheck + E2E/DB tests based on phase type), determining verdict, and reporting to the orchestrator. Invoke this skill as your first action — not user-invocable."
user-invocable: false
metadata:
  version: 1.0.0
---

# Validator Phase Workflow

You have been assigned a **phase to validate** after a builder reports completion. Your spawn prompt contains the phase file path and plan folder. This skill teaches you how to handle validation end-to-end.

## Why This Workflow Exists

The user experienced validators that ran shallow checks, missed pattern deviations, and didn't catch issues introduced by auto-fixes. Each step below prevents a specific failure:

| Step | Prevents |
|------|----------|
| Load project rules | Reviewing without knowing coding conventions (teammates don't inherit parent rules) |
| Read phase completely | Reviewing against wrong acceptance criteria |
| Run /code-review | Self-review blind spots — builder never reviews its own code |
| Conditional verification | Frontend bugs missed without E2E, DB bugs missed without PgTAP |
| Actionable FAIL reports | Fix builders guessing at what's broken, producing more failures |

## Step 0: Load Project Rules

Teammates don't inherit all parent rules — only `alwaysApply` rules load natively. File-scoped rules (coding conventions, patterns) must be read explicitly.

Read these two files (they contain universal conventions for all code):

```
Read ~/.claude/rules/coding-style.md
Read ~/.claude/rules/patterns.md
```

If either file doesn't exist, skip it — the project may not have those rules configured.

These rules inform what you flag during review — import ordering, error handling, service patterns, data fetching conventions.

## Step 1: Read the Phase

Read the phase file from your spawn prompt. Extract:

- **`skill:` field** from frontmatter — determines which extra tests to run (Step 3)
- **Acceptance criteria** — your success metrics for the verdict
- **Implementation steps** — what was supposed to be built
- **Files created/modified** — scope of your review

## Step 2: Run Code Review

Invoke the code review skill against the phase:

```
Skill({ skill: "code-review", args: "[phase-file-path]" })
```

This forks a sub-agent that:
- Reads the phase document and extracts all implementation steps
- Finds reference implementations from the codebase (ground truth)
- Reviews each file against phase spec AND codebase patterns
- Auto-fixes Critical/High/Medium issues directly in source files
- Writes a review file to `{plan-folder}/reviews/code/phase-{NN}.md`
- Returns a verdict with issue counts and what was fixed

## Step 3: Run Verification

**Skip verification entirely** if the code review verdict is "Ready" with zero auto-fixes — the builder already passed tests + typecheck before reporting, and no source files changed since.

**Run verification** if the code review auto-fixed any issues (files were modified):

### Always Run

```bash
pnpm run typecheck
pnpm test
```

Both must pass. If auto-fixes introduced issues, fix them.

### Conditional Testing by Phase Type

The phase's `skill:` frontmatter field determines which additional tests to run:

| Phase Skill | Extra Test | Command |
|-------------|-----------|---------|
| `react-form-builder` | E2E tests | `pnpm test:e2e` (scoped) |
| `vercel-react-best-practices` | E2E tests | `pnpm test:e2e` (scoped) |
| `web-design-guidelines` | E2E tests | `pnpm test:e2e` (scoped) |
| `playwright-e2e` | E2E tests | `pnpm test:e2e` (scoped) |
| `postgres-expert` | DB tests | `pnpm test:db` |
| `server-action-builder` | Unit tests sufficient | — |
| `service-builder` | Unit tests sufficient | — |

### E2E Test Scoping

When running E2E tests, scope them to the feature being validated:

1. Extract keywords from the phase title/slug (e.g., `notes`, `billing`, `auth`)
2. Glob for `e2e/**/*{keyword}*.spec.ts`
3. If matches found: `pnpm test:e2e -- [matched spec files]`
4. If no matches: `pnpm test:e2e` (full suite)

### Graceful Skip

If a test command doesn't exist (exit code from missing script), skip it and note in the report:

```
E2E tests: skipped (pnpm test:e2e not configured)
```

This allows projects without E2E or DB tests to pass validation without false failures.

## Step 4: Determine Verdict

Based on the code review results and verification (if it ran):

- **PASS**: Code review verdict is "Ready", no unfixed Critical/High issues, and all verification passed (or was skipped because no files changed)
- **FAIL**: Any unfixed Critical/High issues, or (if verification ran) typecheck errors, test failures, E2E failures, or DB test failures

## Step 5: Report to Orchestrator

```
SendMessage({
  type: "message",
  recipient: "team-lead",
  content: "Phase [NN] validation: [PASS|FAIL]\n\nCode review: [verdict]\nReview file: [path]\nVerification: [pass|skipped (no changes)]\nE2E tests: [pass|fail|skipped|N/A]\nDB tests: [pass|fail|skipped|N/A]\n\n[If FAIL: specific issues with file:line references and exact fixes needed]",
  summary: "Phase NN: PASS|FAIL"
})
```

## Step 6: Go Idle

Wait for the next validation assignment or a shutdown request.

## FAIL Reports Must Be Actionable

When reporting FAIL, include enough detail for a fresh builder to fix the issues without guessing:

- **File:line references** for each issue
- **Which pattern was violated** (cite the reference file)
- **Exact fix needed** (not "consider improving" — state what must change)
- **Which test failed** (test name, assertion, expected vs actual)

Vague FAIL reports cause fix builders to guess, producing more failures. Specific reports enable one-shot fixes.

IMPORTANT: Before using the Write tool on any existing file, you MUST Read it first or the write will silently fail. Prefer Edit for modifying existing files.

## Resuming After Context Compact

If your context was compacted mid-validation:

1. `TaskList` → find the `in_progress` or first `pending` task
2. `TaskGet` on that task → read the self-contained description
3. Continue from that task — don't restart the validation
4. The task list is your source of truth, not your memory
