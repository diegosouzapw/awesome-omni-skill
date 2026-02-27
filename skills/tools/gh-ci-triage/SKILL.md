---
name: gh-ci-triage
description: Triage failing PR checks with gh CLI, identify first actionable root cause, and derive exact local repro commands.
license: Apache-2.0
compatibility: opencode
metadata:
  project: up-streamer-rust
  workflow: ci-triage
---

## When to use

Use this skill when a PR reports failing GitHub checks and you need a fast, evidence-backed diagnosis.

## Workflow

1. Fetch check overview:
   - `gh pr checks <pr-number>`
   - `gh pr view <pr-number> --json statusCheckRollup`
2. Inspect failed jobs only:
   - `gh run view <run-id> --log-failed`
   - If needed, `gh run view <run-id> --job <job-id> --log-failed`
3. Extract first actionable error (not cascaded noise).
4. Map failure to local repro command using the exact workflow flags.
5. After fix, rerun local repro commands before pushing.
6. Re-check status:
   - `gh pr checks <pr-number>`

## Guardrails

- Do not assume root cause from PR title/last commit; use failed logs.
- Distinguish no-feature lint failures from feature-enabled failures.
- Prefer minimal-scope fixes; avoid broad suppression (`#[allow(...)]`) unless required.

## Report template

Always report:

1. Failing workflow/job name
2. First actionable error line(s)
3. Root cause hypothesis
4. Exact local repro command
5. Fix applied
6. Post-fix local verification commands and results
