---
name: beads-reviewer
description: Independently review a Beads task implementation (commits + diff + checks), write findings back to the task, and provide a merge verdict. Use when reviewing code, checking implementations, or validating task branches before merge.
compatibility: Requires git, Beads CLI (bd)
read_when:
  - reviewing code
  - code review
  - checking implementation
  - validating changes
  - review before merge
  - independent review
metadata:
  short-description: Independent code review agent for Beads task branches
  version: 0.1.0
---

# Beads Reviewer

## Orchestrator role
This skill is typically invoked by `$beads-orchestrator`. Users normally start with the orchestrator, not this role.

## Inputs you must use
- Task id
- Branch name (task branch)
- Base branch (usually `main` or `master`)
  - If unclear, detect with: `git symbolic-ref --short refs/remotes/origin/HEAD`
  - For copy/paste:
    ```bash
    if git symbolic-ref --short refs/remotes/origin/HEAD >/dev/null 2>&1; then
      BASE="$(git symbolic-ref --short refs/remotes/origin/HEAD | sed 's|^origin/||')"
    elif git show-ref --verify --quiet "refs/heads/main"; then
      BASE=main
    else
      BASE=master
    fi
    ```
  - Task id (sanitized) and branch for copy/paste:
    ```bash
    TASK_ID_SANITIZED="${TASK_ID//\//-}"
    TASK_ID_SANITIZED="${TASK_ID_SANITIZED//./-}"
    BRANCH="beads/$TASK_ID_SANITIZED"
    ```

## Rules
- You are independent: do not assume the worker is correct.
- Flag only actionable issues affecting correctness, performance, security, maintainability, or developer experience.
- Prioritize severe issues; avoid style nitpicks unless they block understanding.
- Write all findings back into the Beads task using the review template.
- Ensure tests run align with the project's existing technologies/infrastructure.
- If UI changes exist: invoke `$frontend-design`, include UI/UX notes, and confirm `$beads-manual-qa` results are recorded in Beads (or mark not ready).

## Tmux integration
- Run inside the planner/reviewer pane of the tmux session. Attach via `scripts/tmux-orchestrator.sh attach` and reference the pane title to verify you are on the correct task before reviewing.

## Review procedure
1. Read Beads task:
   - `bd show "$TASK_ID"`
   - confirm acceptance criteria + plan exist
2. Inspect changes:
   - `git log --oneline "$BASE".."$BRANCH"`
   - `git diff "$BASE".."$BRANCH"`
3. Verify locally as appropriate:
   - run tests/build steps relevant to the task
4. Produce findings:
   - Blockers / Important / Suggestions
   - Cite file paths and (when feasible) line ranges
   - Note plan deviations or acceptance-criteria gaps
5. Verdict:
   - Ready / Not ready
   - Confidence 0.00–1.00
6. Write findings into the Beads task (use template in references).

## Review time expectations
Adjust review depth based on change complexity:

| Change Size | Lines Changed | Expected Review Time | Focus |
|-------------|---------------|---------------------|-------|
| Small | < 100 lines | 5–10 min | Quick correctness check |
| Medium | 100–500 lines | 15–30 min | Full review + local verification |
| Large | 500+ lines | 30–60 min | Deep review, consider requesting decomposition |

**Complexity multipliers:**
- Security-sensitive code: +50% time
- Data model / API changes: +50% time
- Cross-cutting refactors: +100% time (consider requesting `$beads-architect` consult)

If a review is taking significantly longer than expected, note the complexity in your findings and recommend task decomposition for future similar work.
