---
name: brutal-agent
description: "Orchestrate subsystem-by-subsystem project hardening by combining brutal-project-review and task-worker in a strict loop: review one subsystem, create CRITICAL/MAJOR tasks, run all tasks to completion, then move to the next subsystem. Continue full review/task passes until a complete pass finds no new CRITICAL/MAJOR issues. Use when the user wants both deep subsystem review and autonomous task execution with no instruction loss."
allowed-tools: Bash(ls:*), Bash(find:*), Bash(wc:*), Bash(date:*), Bash(git:*), Bash(./run:*), Bash(mkdir:*), Bash(rm:*), Bash(cat:*), Bash(mv:*), Task, Read, Write, Edit, Grep, Glob
---

Run a strict orchestration loop that combines these two skills without dropping any instructions:
- `/home/user/.codex/skills/brutal-project-review/SKILL.md`
- `/home/user/.codex/skills/task-worker/SKILL.md`

## Non-Negotiable Inheritance Rules

1. Read both source `SKILL.md` files in full before starting.
2. Treat every instruction in both source files as authoritative and preserved.
3. Do not summarize away, simplify, or omit any requirement from either source skill.
4. Only add orchestration logic for ordering. If there is tension, preserve source-skill behavior and use this skill only to decide sequence.

## Orchestration Workflow

### Step 0: Load Source Skills
- Read:
  - `/home/user/.codex/skills/brutal-project-review/SKILL.md`
  - `/home/user/.codex/skills/task-worker/SKILL.md`
- Keep their instructions active for the rest of execution.

### Step 0.5: Load Project Target Context (Before Feature Planning)
- Before planning any feature work, check for `TARGET.md` in the project root directory.
- If `TARGET.md` exists, read it in full and treat it as required planning context.
- Do not start feature planning until this check/read has been completed.

### Step 1: Drain Existing Tasks First (Resume Safety)
- If any task exists in either:
  - `workspace/tasks/in-progress/`
  - `workspace/tasks/todo/`
- Run `task-worker` exactly as specified, until it reports no more tasks.
- This prevents reviewing additional subsystems while previous findings remain unaddressed.

### Step 1.5: Initialize Pass Tracking
- Define a "pass" as reviewing all currently discovered subsystems to `done` once.
- Read `.claude/review-state/manifest.json` and record:
  - `pass_started_at`
  - `pass_baseline_tasks_created = len(tasks_created)` (use `0` if manifest does not yet exist)
- Use this baseline to determine whether the pass discovered any new CRITICAL/MAJOR issues.

### Step 2: Review One Subsystem
- Run `brutal-project-review` exactly as specified.
- Execute one full subsystem cycle (including report, task creation, manifest update, completion marking, and cleanup) for the next pending subsystem.

### Step 3: Run All Tasks to Completion
- Immediately run `task-worker` exactly as specified.
- Let it process continuously until both are empty:
  - `workspace/tasks/todo/`
  - `workspace/tasks/in-progress/`
- Respect all TDD, self-review, fix-loop, verification, state, and lifecycle requirements from `task-worker`.

### Step 4: Loop Control
- Check `brutal-project-review` manifest state.
- If any subsystem remains pending, go back to Step 2.
- If all subsystems are done:
  - Compute `pass_new_tasks = len(tasks_created) - pass_baseline_tasks_created`.
  - If `pass_new_tasks > 0`, start another pass:
    - Reinitialize pass tracking (Step 1.5)
    - Go back to Step 2
  - If `pass_new_tasks == 0`, go to Step 5.

### Step 5: Final Drain and Completion Report
- Run one final `task-worker` pass to ensure no residual tasks remain.
- Stop only when this condition is true:
  - Last complete pass produced `pass_new_tasks == 0` (no new CRITICAL/MAJOR issues found)
- Report completion summary:
  - Passes executed
  - New CRITICAL/MAJOR tasks created in last pass (`0`)
  - Subsystems reviewed (done/total)
  - Remaining tasks (must be zero unless explicitly blocked/needs-human-review)
  - Any blocked or needs-human-review tasks

## Execution Contract

When this skill says “run `brutal-project-review`” or “run `task-worker`”, it means:
- Apply the full, original instructions from each referenced source skill.
- Preserve all required formats, severity definitions, state handling, history updates, commits, and review rigor.
- Preserve all resume logic and guardrails from both skills.

This skill only defines the macro-ordering loop:
1. Review one subsystem
2. Run all tasks
3. When a full pass completes, repeat passes until no new CRITICAL/MAJOR tasks are created
