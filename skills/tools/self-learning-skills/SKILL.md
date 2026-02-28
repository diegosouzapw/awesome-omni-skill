---
name: self-learning-skills
description: "Memory sidecar for agent work: recall before tasks, record learnings after tasks, review recommendations, optional backport bundles."
---

# Self-learning sidecar

Use this skill to **recall** prior shortcuts before you start work, and to **record** durable “aha” moments + recommendations after you finish.

Critical rule: if no learnings exist (cold start), say so and proceed with standard tools — **do not invent memories**.

## 1) PRE-RUN: Recall (before starting work)
**When to use:** Before any non-trivial task.

**Action:**
1. Locate the project store: `<repo-root>/.agent-skills/self-learning/v1/users/<user>/`
2. Read `<project_store>/INDEX.md` (quick skim).
3. If you need targeted recall, run:
   - `python scripts/self_learning.py list --query "<keywords>"`
   - Optional filters: `--skill <name>`, `--tag skill:<name>`
4. Summarize **3–7** directly actionable bullets relevant to the current task (titles + IDs only; no long dumps).

## 2) POST-RUN: Record (after finishing work)
**When to use:** You discovered something durable (schema, fix, command sequence, constraint, etc.).

**Action:**
1. Capture **1–5** Aha Cards (durable, reusable, specific, non-sensitive). Format: `references/FORMAT.md`.
   - Ensure every Aha Card and Recommendation has `primary_skill` (use `unknown` if unsure).
   - Set `scope` to `project` (repo/run-specific) or `portable` (generally reusable; a backport candidate).
   - If you rediscovered the same learning, treat it as reinforcement (signal) rather than duplicating the full card.
2. Capture **1–5** concrete recommendations (what to change and where).
3. Persist:
   - `python scripts/self_learning.py record --json payload.json` (or stdin)

**Output requirement:** print a short summary + top 3 items, then point to “view more” (`INDEX.md` / `review --format json`). Do not dump long JSON by default.

## 3) REVIEW: Dashboard / Next actions
**When to use:** “What’s still open?”, “What’s stale?”, “What should we backport?”, “Most useful learnings this week?”

**Action:**
- `python scripts/self_learning.py review --days 7`
- Full JSON: add `--format json`
- Filters: `--skill <name>`, `--scope project|portable`, `--status proposed,accepted,in_progress`, `--query "<keywords>"`

## 4) MAINTENANCE / Governance
- Repair store hygiene (append-only): `python scripts/self_learning.py repair --apply`
- Update recommendation status/scope: `python scripts/self_learning.py rec-status --id rec_... --status done --scope portable --note "..."`  
- Optional backport bundle (explicit + auditable): `python scripts/self_learning.py export-backport --skill-path <skill-dir> --ids <aha_ids> [--make-diff] [--apply]`
- Inspect backport markers in a skill: `python scripts/self_learning.py backport-inspect --skill-path <skill-dir>`

## Docs
- Setup/background: `README.md`
- Integration templates (no hooks): `references/INTEGRATION.md`
- Rubric/format/portability: `references/RUBRIC.md`, `references/FORMAT.md`, `references/PORTABILITY.md`
