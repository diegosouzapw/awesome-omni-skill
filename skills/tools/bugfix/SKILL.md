---
name: bugfix
description: Create and manage non-urgent bugfix branches in a git worktree with standard PR workflow (squash merge).
---
## What I do
- Turn a short task title into a deterministic `bugfix/<slug>` branch name.
- Create a git worktree at `../.worktrees/pluto/<branch_sanitized>`.
- Guide sync with `origin/main`.
- Guide PR creation (GitHub, squash merge; do not merge automatically).
- Guide safe cleanup (remove worktree only).

## Naming
- Base branch: `origin/main`
- Branch: `bugfix/<slug>`
- Slug rules: same as `feature` (max 60).
- Worktree path: same as `feature`.

## Workflows
Use the same steps as `feature`.
- Sync: prefer `git rebase origin/main`
- PR: recommend Conventional Commit PR title (often `fix(...)`).
- Cleanup: remove worktree only.
