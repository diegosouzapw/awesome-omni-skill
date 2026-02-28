---
name: beads-worker
description: Implement exactly one Beads task in a dedicated git worktree/branch with plan→code→self-review and write full updates back to the task. Use when implementing a Beads task, coding in a worktree, or executing a single task implementation.
compatibility: Requires git with worktree support, Beads CLI (bd)
read_when:
  - implementing a Beads task
  - coding in a worktree
  - single task implementation
  - writing code for a task
  - executing implementation work
metadata:
  short-description: Single-task implementation agent for Beads + worktrees
  version: 0.1.0
---

# Beads Worker

## Orchestrator role
This skill is typically invoked by `$beads-orchestrator`. Users normally start with the orchestrator, not this role.

## Input you must obtain from context
- Task id (e.g., `bd-a3f8.1`)
- Worktree path (e.g., `.worktrees/bd-a3f8-1`)
- Branch name (e.g., `beads/bd-a3f8-1`)

## Rules
- Do not work on multiple tasks.
- Do not commit to any other branch.
- Before writing code: publish a plan to the Beads task.
- Confirm the task already has an **Architect Plan** update and `state=plan-ready` before beginning work.
- After writing code: self-review, run checks, then publish an implementation summary to the task.
- Tests must use the project's existing technologies/infrastructure; do not add new test frameworks without a clear task requirement.
- If UI changes are involved: invoke `$frontend-design` before implementing UI, and follow its guidance.

## Tmux integration
- Prefer running this skill inside its tmux pane created by `scripts/tmux-orchestrator.sh add-worker "$TASK_ID" ...`.
- If you need to launch a worker from scratch, start the pane with `scripts/tmux-orchestrator.sh add-worker "$TASK_ID" "cd $WORKTREE_PATH && <work commands>"`.

## Procedure
1. Enter the worktree and confirm branch:
   - `pwd`, `git status`, `git branch --show-current`
2. Read the task:
   - `bd show "$TASK_ID"`
3. Write “Plan” update to task (use template in `references/worker-checklist.md`).
4. Implement in small commits:
   - commit messages must include the task id.
5. Self-review:
   - review diff vs base branch
   - run tests/build
6. Write “Implemented” update to task:
   - summary, verification steps, tests run, commit SHAs
7. Stop. Do not merge. Hand off to reviewer/integrator.

## If you discover out-of-scope work
- Do not silently expand scope.
- Write a note to the task describing the out-of-scope issue clearly.
- Recommend creating follow-up Beads tasks.

## If implementation goes wrong (rollback)
If your implementation breaks the build, tests, or introduces regressions:
1. **Stop and assess** — do not push more commits hoping to fix forward blindly.
2. **Soft reset option** — if recent commits are the problem:
   - `git log --oneline` to identify the last good commit
   - `git reset --soft "$GOOD_COMMIT"` to unstage bad changes (keeps files)
   - Fix issues, then recommit
3. **Hard reset option** — if you need to discard all changes:
   - Confirm you are in the assigned worktree + branch: `pwd`, `git rev-parse --show-toplevel`, `git branch --show-current`
   - `git reset --hard "$GOOD_COMMIT"` (destructive — loses uncommitted work)
4. **Document in Beads** — write a note explaining what went wrong and what was rolled back.
5. **Autonomous recovery** — if the task is blocked, note it in Beads, attempt a safe workaround or create follow-up tasks, and continue. Halt only if truly catastrophic.
