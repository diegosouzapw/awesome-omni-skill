---
name: careerswarm
description: CareerSwarm repo: monitor, code sweeps, debugging, and sync with Cursor. Workspace is this repo; read OPENCLAW.md and CONTEXT/todo before big edits.
---

# CareerSwarm skill

**Model:** Use only **Claude Sonnet 4.5** for this project. Do not use Opus, Haiku, or other models unless the user explicitly requests. If your agent or session is set to a different model, the user should set the primary model to Sonnet 4.5 in OpenClaw config (see docs/OPENCLAW_INTEGRATION.md).

You are working in the **CareerSwarm** repo (AI career evidence platform: Roast, Master Profile, onboarding, Railway + MySQL + OpenAI). Stay in parallel with Cursor by reading **CONTEXT_FOR_NEW_CHAT.md** and **todo.md** before making changes.

## Before any code or file edits

1. Read **OPENCLAW.md** (project summary, key paths).
2. Read **CONTEXT_FOR_NEW_CHAT.md** (current state, last session, next steps).
3. Read **todo.md** (priorities). Do not duplicate work Cursor is doing.

**Goal shortcut:** If the user says "production today" or "deploy-ready," run only `pnpm run ship:check:full` and hand off (do not run code sweep or work-until-fixed). If the user says "fix lead-magnet" or "work until fixed," use OPENCLAW_START / OPENCLAW_ORCHESTRATION work-until-fixed flow.

## Commands (run from repo root)

- **Full ship gate (local):** `pnpm run ship:check` (check + build + test).
- **Full ship gate + prod tests:** `pnpm run ship:check:full` (ship:check + production smoke + E2E).
- **Pre-commit:** `pnpm precommit` (secrets scan + check + format:check + lint).
- **Monitor:** `pnpm run monitor` (CI, Railway, app health, Cloudflare); or `pnpm run monitor:watch` for polling + notifications.
- **Checks:** `pnpm check`, `pnpm test`, `pnpm lint`, `pnpm format:check`.
- **Production smoke:** `npx playwright test tests/production-smoke.spec.ts --config=playwright.production.config.ts`.
- **Production E2E:** `npx playwright test tests/production-e2e.spec.ts --config=playwright.production.config.ts`.
- **Dependency audit:** `pnpm audit`; `pnpm outdated` (report only, do not upgrade).

## When asked to “monitor” or “check status”

Run `pnpm run monitor` and report pass/fail If something fails, follow the "When you find errors" section: work through to a solution, then hand off for commit.

## When asked to “code sweep” or “review server”

Read docs/DEBUGGING.md (order of operations: platform limits → env → minimal fix). Then read key paths from OPENCLAW.md (server/\_core, server/roast.ts, server/db.ts, drizzle/schema.ts). Suggest only minimal, aligned changes; do not add instrumentation before platform/env are ruled out.

## When asked to “sync check” or “what’s Cursor doing”

Read CONTEXT_FOR_NEW_CHAT.md and todo.md; summarize “current state,” “last session,” and “next steps” so the human can align Cursor and you.

## When acting as the Business agent (GTM, strategy, in-app business agents)

Focus on **GTM, strategy, positioning, pricing, and the in-app business/GTM agents**. Read docs/CAREERSWARM_GTM_STRATEGY.md, docs/GTM_PLAN.md, and server/agents/gtm/ when asked. Suggest improvements to positioning, prompts, pipeline steps, or metrics; explain why. Do not edit code unless the user asks; otherwise hand off a clear summary (what to improve, why, where). Use docs/BUSINESS_AGENT_IMPROVEMENTS.md for a checklist of improvements (prompt alignment, observability, testing, pipeline steps). For **future needs and shoestring budget** (what to do next on the cheap), follow docs/OPENCLAW_FUTURE_SHOESTRING.md and append a short summary to OPENCLAW_HANDOFF.md.

## When asked to suggest better code and explain why (Review role)

Your job is to **identify better code and solutions** and **explain why**. Read the file(s) or path(s) the user asks about. For each improvement: (1) say **what** could be better (readability, pattern, performance, consistency with DEBUGGING/docs, safety), (2) say **why** it’s better, (3) optionally give a short code or approach example. Do not apply changes unless the user explicitly asks; otherwise hand off a clear summary (what could be better, why, where) so the user or Cursor can decide what to adopt. You are the “explain the why” agent.

## Debugging (production-only)

Follow docs/DEBUGGING.md: (1) platform limits, (2) env (e.g. OPENAI_API_KEY, DATABASE_URL), (3) minimal code fix, (4) instrumentation only if still stuck. Use `railway logs` only if the user has Railway CLI linked to the project.

## Other use cases (when asked)

- **Regression triage:** Run production smoke + E2E; list which tests failed and which passed. Report only; do not fix.
- **Dependency / audit:** Run `pnpm audit` and `pnpm outdated`; summarize high/critical and major updates. Report only; do not upgrade.
- **Docs consistency:** Read README, CONTEXT_FOR_NEW_CHAT.md, docs/SHIP_CHECKLIST.md, docs/CRITICAL_SETUP_CHECKLIST.md; list contradictions or outdated steps.
- **Log analysis:** User pastes logs/errors. Summarize errors and which DEBUGGING.md section or code path applies. No secrets in your response.
- **Test coverage gaps:** List critical flows (Roast, onboarding, login) from README; for each say if we have smoke or E2E in tests/. Report gaps.
- **Release notes / changelog:** Read recent git log (e.g. last 10–20 commits); draft a short CHANGELOG or release summary for the user to edit.
- **On-call runbook:** For a given symptom, use docs/DEBUGGING.md and docs/MONITORING.md to give step-by-step what to do.
- **Migration safety:** Read latest migration in drizzle/ and drizzle/schema.ts; list any new env or deploy steps before running in prod.
- **API coverage check:** List public tRPC procedures from server/routers; check production-smoke and production-e2e for which are exercised; list critical API not covered.
- **Local dev checklist:** From README and docs/CRITICAL_SETUP_CHECKLIST.md, produce a minimal “local dev only” setup checklist (no deploy).
- **Rollback decision:** Summarize what CONTEXT says was last deployed; if user rollbacks, what to verify first.
- **Sentry / error triage:** User pastes sanitized Sentry error. Map to server/ or client/ paths and suggest DEBUGGING.md or next steps.
- **Run everything / full gate:** Run `pnpm precommit` then `pnpm run ship:check:full` then `pnpm run monitor`. Report pass/fail for each stage.

## When you find errors: work through to a solution, then hand off for commit

When any check or run fails (monitor, ship:check, lint, tests, etc.): (1) **Investigate** — read docs/DEBUGGING.md (platform limits → env → minimal fix) and identify the cause. (2) **Fix** — apply minimal code or config changes only (no .env or secrets; suggest those for the human); no instrumentation until platform/env are ruled out. (3) **Re-run** the relevant check(s) to verify. (4) **Repeat** if still failing, or escalate (e.g. "needs env change" or "needs human decision"). (5) **When a solution is determined** (checks pass or clear handoff) — summarize what you found, what you changed, and that it's **ready for the human to review and commit**; do not commit or push yourself. Optionally append a short handoff note to OPENCLAW_HANDOFF.md (when, agent role, what ran, what changed, ready for review). The human (or Cursor) does the commit after reviewing.

## Do not

- **Commit or push** — You work to a solution and hand off; the human reviews and commits. Only commit if the user explicitly says "commit" or "push."
- Change .env or secrets; suggest env changes only.
- Add logging/instrumentation before checking platform limits and env.
