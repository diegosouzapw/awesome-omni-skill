---
name: backlog
description: Manage tasks/ backlogs and worktrees created by the cyotee Claude plugin set.
metadata:
  short-description: Backlog/status/launch/complete/prune workflows for tasks/
---

# backlog (Codex)

Use this skill to mirror the Claude `/backlog` plugin flows inside Codex. It assumes the repository follows the `tasks/` structure described in the plugin README.

## Responsibilities
- Enumerate tasks by reading `tasks/INDEX.md` when present, otherwise infer from `tasks/[PREFIX]-[N]` directories.
- Display task details by reading `tasks/[ID]/PRD.md`, `PROGRESS.md`, and `REVIEW.md`.
- Launch work by creating/updating a worktree and generating `PROMPT.md` + `PROGRESS.md` guidance.
- Move tasks through `pending -> in_progress -> review -> complete` and archive when done.

## How to Use
- `status`: Summarize tasks by status with dependencies and worktrees. Prefer `tasks/INDEX.md`; if missing, scan `tasks/*` except `archive/`.
- `read <ID>`: Normalize IDs (allow `P-5` or `5` with detected prefix). Output PRD highlights (title, goals, acceptance criteria) and the latest progress/review notes.
- `work <ID>`: Start working on a task in the current session (no worktree). Creates PROMPT.md, marks status `in_progress`. Use for quick/simple tasks.
- `launch <ID>`:
  1) Resolve repo root (`git rev-parse --show-toplevel`) and expected worktree branch (commonly `feature/<kebab>` from PRD).  
  2) Create or reuse worktree via `plugins/backlog/scripts/wt-create.sh <branch> [repo-root]`.  
  3) Generate/refresh `PROMPT.md` in the worktree root with: task summary, completion promise (`TASK_COMPLETE`), instructions to update `tasks/[ID]/PROGRESS.md`, and memory/compact guidance.  
  4) Ensure `tasks/[ID]/PROGRESS.md` exists with a Checkpoints section; append a new entry noting the launch.  
  5) Mark status `in_progress` in `tasks/INDEX.md` (or add frontmatter to PRD if no index).  
  6) Print launch steps for the user (`cd <worktree>`, run Claude/Codex, start agent loop).
- `complete <ID>`: Complete a task. Supports two modes:
  - **In-session mode** (from `/backlog:work`): Commits changes, rebases if on feature branch, merges to main, removes PROMPT.md, marks complete.
  - **Worktree mode** (from `/backlog:launch`): Two-phase workflow - Phase 1 from worktree (rebase, mark pending), Phase 2 from main (merge, archive, cleanup).
- `prune [<ID>|--all]`: Move completed/reviewed tasks to `tasks/archive/`, update `INDEX.md`, and list any stale worktrees to delete (use `plugins/backlog/scripts/wt-remove.sh <branch> [repo-root]`).
- `review <ID>`: Transition task to **code review mode** (not task definition audit - use `/design:review` for that). Updates PROMPT.md in worktree to switch from implementation to code review.
- `list`: Read `tasks/INDEX.md` and display formatted table with columns: ID, Title, Status, Dependencies, Worktree. Include summary counts and next actions. Use `--worktrees-only` to show only active worktrees.

## Key Files
- `tasks/INDEX.md` — canonical task table (layer name, prefix, status, worktree, deps).
- `tasks/[ID]/PRD.md`, `PROGRESS.md`, `REVIEW.md` — task docs to read/write.
- `PROMPT.md` (in worktrees) — agent instructions; regenerate when launching if stale.
- Scripts: `plugins/backlog/scripts/wt-create.sh`, `wt-remove.sh` (worktree management), `deps.sh` (dependency checks).

## Conventions
- Layer/prefix detection: read `tasks/INDEX.md` for `layer`/`prefix`; else derive prefix from repo/dir name first letter.
- Status icons: 🆕 `pending`, 🚀 `in_progress`, 📋 `review`, ✅ `complete`. Respect any existing icon mapping.
- Dependencies: mark tasks blocked if prerequisites are not `complete`/`review`.
- Be non-destructive with user files; append to `PROGRESS.md` instead of overwriting unless asked.
