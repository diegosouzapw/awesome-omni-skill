---
name: review-fix
description: Read the latest code review, plan fixes for all findings, then execute the fixes — all in one command. Produces a dated plan and execution log in .reviews/.
---

# Review Fix — Plan & Execute from Last Review

Read the most recent `.reviews/YYYY-MM-DD-review.md`, plan fixes for all findings
grouped by priority, then execute every planned fix with atomic commits. Produces
two artifacts: a **review-plan** and a **review-execution** log.

## Instructions

### Phase 0 — Find the Latest Review

1. Glob for `.reviews/*-review.md` and pick the most recent file by date.
2. Read the full review file. Parse every finding into a list of
   `{ severity, reviewer, file, line, description }` entries.
3. If no review file exists, tell the user to run `/code-review` first and stop.

### Phase 1 — Plan

Spawn **4 parallel planning agents** (Task tool, `subagent_type: Plan`) that each
receive the full findings list filtered to their scope. Each agent returns a
structured list of fix actions.

#### Agent scopes

| Agent         | Scope                                                  |
|---------------|--------------------------------------------------------|
| **backend**   | Findings from Tina, Marcus, Priya, Carlos, Oscar that touch `cachibot/` |
| **frontend**  | Findings from Tina, Zoe, Luna that touch `frontend/src/` |
| **data**      | Findings from Carlos (storage/db layer)                |
| **tests**     | Findings from Derek (test coverage gaps)               |

> Findings that span multiple scopes go to every relevant agent.

Each planning agent must:

- Group related findings that can be fixed together (same file, same pattern).
- For each group, produce:
  - **What**: one-line summary of the change
  - **Files**: list of files to modify
  - **How**: 2-3 sentence implementation approach
  - **Severity**: highest severity in the group (Fix > Improve > Note)
  - **Risk**: Low / Medium / High (does this change behavior or just style?)
  - **Depends on**: other group IDs this group must wait for (if any)
- Order groups by: Fix severity first, then by dependency order.
- Skip any finding rated **Note** — notes are informational only.

After all 4 agents return, compile results into `.reviews/YYYY-MM-DD-review-plan.md`:

```markdown
# Review Fix Plan — YYYY-MM-DD

**Source review:** `.reviews/YYYY-MM-DD-review.md`
**Generated:** YYYY-MM-DD HH:MM

## Stats

| Severity | Findings | Planned groups |
|----------|----------|----------------|
| Fix      | N        | N              |
| Improve  | N        | N              |
| Note     | N        | (skipped)      |

## Execution Waves

Wave 1 (no dependencies):
- [B1] Backend: <summary>
- [F1] Frontend: <summary>

Wave 2 (depends on wave 1):
- [B2] Backend: <summary> (depends on B1)

...

## Detailed Plan

### [B1] <Summary>
- **Severity:** Fix
- **Risk:** Low
- **Files:** `path/to/file.py`
- **How:** Description of the fix approach
- **Findings addressed:**
  - Tina: `path/to/file.py:42` — original finding text
  - Marcus: `path/to/file.py:50` — original finding text
```

Print the plan summary (wave count, group count, severity breakdown) to the user,
then **immediately proceed to Phase 2** without waiting for approval.

### Phase 2 — Execute

Read the plan file. Execute fixes **wave by wave** — all groups in a wave run in
parallel, but waves run sequentially (wave 2 waits for wave 1 to finish).

For each group, spawn a Task agent (`subagent_type: general-purpose`) with:

- The group's detailed plan (What, Files, How)
- The original finding text for context
- Instruction to read each file before editing
- Instruction to make the minimal change that addresses the finding
- Instruction to run `ruff check --fix` and `ruff format` on any modified Python file
- Instruction to run `npm run lint -- --fix` on any modified TypeScript file (from `frontend/`)
- Instruction to NOT create new files unless the plan explicitly says to
- Instruction to NOT add tests (Derek findings are tracked but test writing is a separate task)

After each wave completes:

1. Verify no lint errors remain (`ruff check cachibot/` and `cd frontend && npm run lint`).
2. If lint fails, spawn a fix agent for the failing files before continuing.
3. Stage all changed files and create one commit per wave:
   ```
   Review fixes wave N: <comma-separated group IDs>

   Addresses findings from YYYY-MM-DD code review.
   Groups: [B1] summary, [F2] summary, ...

   Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
   ```

After all waves complete, write `.reviews/YYYY-MM-DD-review-execution.md`:

```markdown
# Review Fix Execution — YYYY-MM-DD

**Plan:** `.reviews/YYYY-MM-DD-review-plan.md`
**Source review:** `.reviews/YYYY-MM-DD-review.md`

## Results

| Wave | Groups | Status  | Commit  |
|------|--------|---------|---------|
| 1    | B1, F1 | Done    | abc1234 |
| 2    | B2     | Done    | def5678 |

## Findings Addressed

| Severity | Planned | Fixed | Skipped | Reason          |
|----------|---------|-------|---------|-----------------|
| Fix      | N       | N     | N       |                 |
| Improve  | N       | N     | N       |                 |

## Group Details

### [B1] <Summary> — Done
- **Commit:** abc1234
- **Files changed:** `path/to/file.py`
- **Findings fixed:**
  - Tina: `path/to/file.py:42` — fixed
  - Marcus: `path/to/file.py:50` — fixed

### [F1] <Summary> — Skipped
- **Reason:** File was recently refactored, finding no longer applies
- **Findings skipped:**
  - Zoe: `frontend/src/Component.tsx:100` — N/A after refactor

## Remaining Work

- Derek's test coverage findings were not addressed (test writing is separate)
- N findings skipped due to: ...
```

### Phase 3 — Summary

Print a final summary to the user:

```
Review fix complete.

Waves executed: N
Commits created: N
Findings fixed: N / M planned (X Fix, Y Improve)
Skipped: N (reasons listed in execution log)

Plan:      .reviews/YYYY-MM-DD-review-plan.md
Execution: .reviews/YYYY-MM-DD-review-execution.md
```

## Important Rules

- **Never skip a Fix-severity finding** without logging a clear reason.
- **Read before edit** — every agent must read the target file before modifying it.
- **Minimal changes** — fix exactly what the finding describes, nothing more.
- **No new features** — this is a fix pass, not a feature pass.
- **No test writing** — Derek's findings are logged as "remaining work" but not executed
  here. Test creation deserves its own focused session.
- **Preserve behavior** — if a fix would change external API behavior, mark it as
  Risk: High in the plan and add a note in the execution log.
- If the review file has a "Previous Reviews" section noting still-open findings from
  prior reviews, include those in the plan too.
