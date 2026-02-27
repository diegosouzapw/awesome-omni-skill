---
name: ci-failure
description: Diagnose GitHub Actions CI failures and suggest fixes. Use when the user reports CI failed, GitHub job failed, Actions failed, build failed on remote, upload artifact failed, or pastes a failure from the Actions tab.
---

# CI Failure Diagnosis

When the user reports a **remote GitHub Actions / CI failure** (or pastes a failure from the Actions tab), apply this skill.

## 1. Load Context First

Read these files **immediately** to understand this repo’s CI:

- **Workflow:** `.github/workflows/ci.yml` (and `.github/workflows/security.yml` if the failure mentions security)
- **Pipeline summary:** `docs/ci.md`

Use them to map the failure to a job and step and to apply repo-specific fixes.

## 2. Identify Job and Step

From the user’s message or pasted log, determine:

- **Job name** (e.g. Lint, Type Check, Test, Build, Docker Build, Notify Slack)
- **Step name** (e.g. "Run ESLint", "Build application", "Upload build artifacts")

If the user only pastes an error line (e.g. "No files were found with the provided path: .next"), infer the step from the message and the workflow.

## 3. Map to Fix

| Failure / Log | Job | Step | Action |
|--------------|-----|------|--------|
| ESLint error, file:line | Lint | Run ESLint | Fix the reported file/line; run `npm run lint` locally. |
| Type error, tsc | Type Check | Run TypeScript compiler | Fix the type; run `npx tsc --noEmit` locally. |
| Test failure, vitest | Test | Run tests | Fix test or app code; run `npm test` locally. |
| Build error, Next.js, OOM | Build | Build application | Check "Build application" logs; confirm `NODE_OPTIONS=--max-old-space-size=4096` in workflow; fix build error or increase memory. |
| No files .next, upload artifact | Build | Build application / Upload | Build didn’t produce `.next`; check "Build application" step logs. Ensure workflow has List build output, "Fail if build did not produce .next", and conditional upload (see docs/ci.md). |
| Docker build failed | Docker Build | Build Docker image | Check Dockerfile and build-args (MONGODB_URI, MONGODB_DB). |
| Notify Slack / health | Notify Slack | — | Check Secrets (SLACK_WEBHOOK_URL) and Variables (APP_URL); see docs/ci.md and readme. |

## 4. Respond

1. **Say which job and step failed** (and link to workflow if helpful).
2. **Give a short cause** (e.g. "Build didn’t produce `.next`" or "ESLint error in `src/foo.ts`").
3. **Suggest a concrete fix** (workflow change, code change, or env/secret), using `docs/ci.md` and `.github/workflows/ci.yml` as the source of truth.
4. If the user pasted only one line, **ask for the full job log** for the failing step if you need it to be precise (e.g. "Paste the full 'Build application' step log from the Build job").

## 5. Optional: Cursor Rule

For more detail on mapping failures and repo-specific CI, see `.cursor/rules/github-ci.mdc` (it aligns with this skill and docs/ci.md).
