---
name: design
description: Interactive task/PRD design workflows adapted from the cyotee Claude plugin.
metadata:
  short-description: Create and refine tasks with user stories and acceptance criteria.
---

# design (Codex)

Use this skill to guide users through creating and refining tasks in the `tasks/` backlog structure.

## Responsibilities
- Initialize `tasks/` scaffolding when missing (INDEX.md, templates, archive/).
- Run short, iterative interviews (2–4 questions per round) to gather scope, constraints, and acceptance criteria.
- Produce task directories with PRD/PROGRESS/REVIEW files and update `tasks/INDEX.md`.
- Review existing tasks for clarity, completeness, and dependency hygiene.

## How to Use
- `design:init`: Detect layer from repo/directory name, create `tasks/` if absent with:
  - `tasks/INDEX.md` (layer name, prefix, status table headers)
  - `tasks/0/PRD.md`, `PROGRESS.md`, `REVIEW.md` as templates
  - `tasks/archive/` directory
- `design <feature>` / `design:task <feature>`:
  1) Detect layer + prefix (prefer `tasks/INDEX.md`; else derive from repo name first letter).  
  2) Ask clarifying rounds: scope/purpose, technical approach/dependencies, acceptance/tests.  
  3) Draft user stories and acceptance criteria that are testable and inventory-check friendly.  
  4) Create `tasks/[PREFIX]-[N]/PRD.md` with metadata frontmatter (task number, title, status `pending`, layer, worktree suggestion, dependencies).  
  5) Create empty `PROGRESS.md` (with Checkpoints section) and `REVIEW.md`.  
  6) Append the task row to `tasks/INDEX.md` (status `pending`).
- `design:prd`: Run a focused interview to generate/refresh a repository-level `PRD.md` (vision, goals, constraints, users, success metrics).
- `design:review` [<ID>]: **Audit task definitions** (not code review - use `/backlog:review` for code)
  - For all tasks: scan `tasks/*` (skip `archive/`), flag missing/weak descriptions, acceptance criteria, dependencies, file path accuracy, and test coverage.
  - For a single task: apply the checklist and propose edits; update the files if asked.
  - This reviews TASK.md quality, NOT implementation code.

## Task Template Reminders
- PRD structure: description, dependencies, user stories with acceptance criteria, files/tests to touch, inventory checks, completion criteria.
- Status values: 🆕 `pending`, 🚀 `in_progress`, 📋 `review`, ✅ `complete`.
- Worktree naming: `feature/<kebab-case-title>` unless overridden.

## Notes
- Keep edits additive; never delete user notes unless explicitly requested.
- When numbering tasks, increment the highest existing `N` for the detected prefix.
- Surface any uncertainties or missing inputs before writing files.
