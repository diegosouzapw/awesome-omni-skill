---
name: beads-integrator
description: Merge Beads task branches into main/master safely (serialized), verify, push to remote, record merge SHAs back into tasks, and clean worktrees. Use when merging branches, integrating code to base branch, or finalizing reviewed task branches.
compatibility: Requires git with worktree support, Beads CLI (bd)
read_when:
  - merging branches
  - integrating code
  - merge to main/master
  - finalizing task branches
  - integration work
  - serialized merge
metadata:
  short-description: Single-threaded integration/merge role for multi-agent work
  version: 0.1.0
---

# Beads Integrator

## Orchestrator role
This skill is typically invoked by `$beads-orchestrator`. Users normally start with the orchestrator, not this role.

## Core rule
You are the *only* entity that merges to the base branch during an orchestration run.

## Inputs
- List of task ids + branch names approved by reviewer
- Base branch (usually `main` or `master`)
  - If unclear, detect with: `git symbolic-ref --short refs/remotes/origin/HEAD`
- Store base name for reuse:
  ```bash
  if git symbolic-ref --short refs/remotes/origin/HEAD >/dev/null 2>&1; then
    BASE="$(git symbolic-ref --short refs/remotes/origin/HEAD | sed 's|^origin/||')"
  elif git show-ref --verify --quiet "refs/heads/main"; then
    BASE=main
  else
    BASE=master
  fi
  ```
- Remote name
  - Detect with:
    ```bash
    REMOTE="$(git remote | head -n 1)"
    if git remote get-url origin >/dev/null 2>&1; then REMOTE=origin; fi
    ```
- Task id (sanitized for branch/worktree)
  ```bash
  TASK_ID_SANITIZED="${TASK_ID//\//-}"
  TASK_ID_SANITIZED="${TASK_ID_SANITIZED//./-}"
  BRANCH="beads/$TASK_ID_SANITIZED"
  WORKTREE_PATH=".worktrees/$TASK_ID_SANITIZED"
  ```

## Procedure (repeat per task branch)
1. Ensure base is clean and current:
   - If dirty: wait 60 seconds, then re-check.
   - If still dirty:
     - If only `.beads/issues.jsonl` changed: continue and commit your updates as usual.
     - Otherwise: stash to unblock:
       ```bash
       git stash push -u -m "beads-integrator-auto"
       ```
   - `git checkout "$BASE"`
   - `git pull --rebase` (if a remote is configured)
2. Update task branch with base:
   - `git checkout "$BRANCH"`
   - `git rebase "$BASE"` (or `git merge "$BASE"` if policy avoids rebasing)
3. Verify:
   - run the agreed verification commands
4. Merge:
   - `git checkout "$BASE"`
   - `git merge --ff-only "$BRANCH"` (or merge-commit policy)
   - `git push "$REMOTE" "$BASE"` (use detected `$REMOTE`, or skip if no remote)
5. Record in Beads:
   - merge commit SHA, method, verification performed, follow-ups if needed
6. Clean worktree:
   - remove `$WORKTREE_PATH` when safe

## If conflicts occur
- Resolve conflicts in the task branch context.
- If conflict complexity suggests architectural ambiguity:
  - pause and request `$beads-architect` consult
  - or defer with follow-up tasks instead of brute forcing a risky merge

## If merge breaks base branch (rollback procedure)
If verification fails after merging to base branch, act immediately:

1. **Revert the merge commit:**
   ```bash
   git revert -m 1 "$MERGE_SHA" --no-edit
   git push "$REMOTE" "$BASE"
   ```
   This creates a new commit that undoes the merge while preserving history.

2. **Document in Beads:**
   - Update the task with: "Merge reverted due to: $REASON"
   - Include the revert commit SHA
   - Change task status back to "needs work"

3. **Notify orchestrator:**
   - The task needs re-review after fixes
   - Consider whether the issue indicates a gap in verification steps

4. **Create follow-up task (if needed):**
   - If the failure reveals a deeper issue, create a new Beads task to address root cause
   - Link it as a blocker to the original task

**Prevention:** Always run full verification on the integrated branch before pushing. If CI is available, wait for CI to pass before considering the merge complete.
