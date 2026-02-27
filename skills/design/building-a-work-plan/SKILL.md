---
name: building-a-work-plan
description: Use when orchestrating the creation of a work plan from a design document. Manages branch setup, codebase investigation, milestone planning, and execution handoff.
command: build-work-plan
---

# Building a Work Plan

## Overview

This skill orchestrates the end-to-end process of converting a design document into an executable work plan. It coordinates branch setup, codebase investigation, milestone-by-milestone planning, and handoff to execution. Think of it as the "conductor" that invokes sub-skills and agents in the right order.

## Required Sub-Skills

Load these during execution:

| Skill | Purpose | When Loaded |
|-------|---------|-------------|
| [writing-a-work-plan](../writing-a-work-plan/SKILL.md) | Task decomposition patterns, templates | Phase 2 |
| [writing-code](../writing-code/SKILL.md) | Engineering principles (via writing-a-work-plan) | Phase 2 |

## Entry Point

When the user invokes `/build-work-plan`:

1. **Announce**: "I'm using the building-a-work-plan skill to create a work plan from your design document."

2. **Get design document path**:
   - If user provided a path, use it
   - If not, use `AskUserQuestion` to request the path
   - **Never guess the path**

3. **Choose review mode** (MANDATORY):

   **STOP. You MUST get user preference before creating orchestration tasks.**

   Use `AskUserQuestion` with these options:
   - **Interactive** (Recommended): Plan one milestone, get approval, proceed. Catches errors early.
   - **Batch**: Plan all milestones, then review all. Faster but riskier.

   Record the user's choice. This determines behavior in Phase 2 and Phase 4.

4. **Create orchestration tasks** (see Task Tracking section)

5. **Proceed through phases**

## Task Tracking

Create these tasks at the start of each session. Update status as you progress.

```
◻ #1 Setup: Confirm design doc and optional branch setup
◻ #2 Investigate: Codebase testing patterns
◻ #3 Investigate: Design assumptions verification
◻ #4 Plan: Write milestone tasks (one sub-task per milestone)
◻ #5 Verify: Post-planning codebase check
◻ #6 Review: Code review of plan
◻ #7 Approve: User approval
◻ #8 Write: Save plan to disk
◻ #9 Handoff: Execution instructions
```

**Dependencies**:
- #2 blocked by #1
- #3 blocked by #2
- #4 blocked by #3
- #5 blocked by #4
- #6 blocked by #5
- #7 blocked by #6
- #8 blocked by #7
- #9 blocked by #8

Use `TaskCreate` and `TaskUpdate` to manage these. Mark each task `in_progress` when starting, `completed` when done.

## Phases

### Phase 0: Setup (Task #1)

**Step 1: Confirm design document**

Read the design document. Verify it contains:
- [ ] Fleshed-out milestones (job stories, descriptions, AC, demos)
- [ ] Milestone count ≤ 8

If milestones are vague or missing AC, stop and suggest running `/start-milestone-review` first.

**Step 2: Optional branch setup**

Ask the user:

```
Do you need to set up a branch for this work?
- Yes, create a new branch
- No, I'm already on the correct branch
```

If yes:
1. Determine base branch (`main` or `master`)
2. Generate branch name from design doc title (e.g., `feat/trackman-scraper`)
3. Create and checkout branch: `git checkout -b <branch-name>`
4. Confirm branch created

Mark Task #1 complete.

### Phase 1: Codebase Investigation (Tasks #2, #3)

**Task #2: Testing patterns**

Dispatch `codebase-investigator` agent:

```
Investigate the testing patterns in this codebase:
- How are tests structured? (tests/, test/, colocated?)
- What testing framework is used? (pytest, jest, etc.)
- What mocking patterns exist?
- What fixtures are available?
- Any test utilities or helpers?

Report findings in a structured format.
```

Document findings. Mark Task #2 complete.

**Task #3: Design assumptions** (sequential after #2)

Dispatch `codebase-investigator` agent:

```
Verify these design assumptions against the current codebase:

Design doc: [path]
Milestones: [list milestone titles]

Check:
- Do files exist where the design expects them?
- Do expected features/dependencies exist?
- Is there drift between design doc and current code?
- What naming conventions should we follow?

Report any mismatches or concerns.
```

Document findings. Mark Task #3 complete.

### Phase 2: Milestone Planning (Task #4)

**CHECKPOINT: Verify sub-skill is loaded.**

Before processing any milestone, confirm you have loaded `writing-a-work-plan` skill. If you have not loaded it, STOP and load it now using the Skill tool.

This sub-skill provides:
- Milestone plan header template (Context, Codebase Verification, References)
- Task templates by type (Infrastructure, Functionality, Integration)
- TDD-aligned implementation step patterns

**Load sub-skill**: Activate `writing-a-work-plan` skill.

**Process each milestone** following the `writing-a-work-plan` Core Pattern:

For each milestone M:
1. Create sub-task: "Plan: Milestone M - [title]"
2. Read milestone from design doc
3. **Write complete milestone plan header** (see `writing-a-work-plan` Document Format)
4. Classify each AC (infrastructure, functionality, integration)
5. Create scaffold task if needed
6. Create one task per AC with implementation steps
7. **If Interactive mode**: Use `AskUserQuestion` to get approval:
   - **Approve**: Milestone plan looks good, proceed to next milestone
   - **Revise**: I have feedback on specific tasks (user provides details via "Other")

   Do not proceed until user approves or revisions are complete.
8. **Write milestone to disk immediately after approval** (enables session recovery)
9. **If Batch mode**: Continue to next milestone
10. Mark sub-task complete

Mark Task #4 complete when all milestones are planned.

### Phase 3: Post-Planning Verification (Task #5)

Dispatch `codebase-investigator` agent:

```
Verify the planned tasks are implementable:

Work plan location: [path]

Check:
- Do target directories exist (or can be created)?
- Are import paths valid?
- Any conflicts with existing code?
- Are test file locations correct?

Report any issues that would block implementation.
```

If issues found:
- Present to user
- Update tasks to match reality, or
- Flag as known issues in the plan

Mark Task #5 complete.

### Phase 4: Code Review (Task #6)

**This phase is MANDATORY. Do not skip code review.**

Even for "simple" plans, code review catches:
- Missing tasks for AC
- Vague verification steps that cause confusion during implementation
- Inconsistent commit message formats
- Dependency ordering issues

Dispatch `code-reviewer` agent over all milestone plan files:

```
Review these work plan files for quality:

Files: [list of milestone_##.md files]

Check:
- All AC have corresponding tasks
- Tasks include concrete test code where applicable
- Verification steps are explicit
- Commit messages follow conventions
- Dependencies are noted
- No ambiguous or vague steps

Report issues by file and task number.
```

**Handle review results based on review mode** (chosen in Entry Point step 3):
- **Interactive mode**: Present issues to user; let user decide what to fix
- **Batch mode**: Automatically fix issues; user sees clean plan

**Verification**: Before marking Task #6 complete, confirm:
- [ ] Code review agent was dispatched
- [ ] All reported issues were addressed (fixed or user-approved as-is)
- [ ] Results were presented to user

Mark Task #6 complete.

### Phase 5: User Approval (Task #7)

Present the complete work plan summary:

```markdown
## Work Plan Summary

**Design doc**: [path]
**Plan location**: [path]
**Milestones**: [count]

| Milestone | Tasks | Type Breakdown |
|-----------|-------|----------------|
| M1: [title] | N tasks | X infra, Y func, Z integ |
| M2: [title] | N tasks | ... |
| ... | ... | ... |

**Total tasks**: [count]

**Codebase verification**: [status]
**Code review**: [status]

Ready to write to disk?
```

Get explicit approval. Mark Task #7 complete.

### Phase 6: Write to Disk (Task #8)

Create directory and files using this EXACT structure:

```
docs/work-plans/YYYY-MM-DD-<plan-name>/
├── milestone_01.md
├── milestone_02.md
├── ...
└── milestone_NN.md
```

**Concrete example** (for a plan named "golf-data-loader" on 2026-01-15):
```
docs/work-plans/2026-01-15-golf-data-loader/
├── milestone_01.md
├── milestone_02.md
├── milestone_03.md
```

**Naming rules**:
- Directory: `YYYY-MM-DD-<plan-name>` where plan-name is lowercase, hyphenated
- Files: `milestone_01.md`, `milestone_02.md`, etc. (zero-padded two digits)
- **NOT**: `m1-*.md`, `M1_*.md`, `milestone-1.md`, or any other variant

**Verification** (REQUIRED before marking complete):
```bash
ls -la docs/work-plans/YYYY-MM-DD-<plan-name>/
```

Confirm output shows:
- Directory name matches `YYYY-MM-DD-<plan-name>` format
- All files are named `milestone_##.md`

Mark Task #8 complete only after verification passes.

### Phase 7: Execution Handoff (Task #9)

**Capture absolute paths**:
```bash
git rev-parse --show-toplevel  # Working root
```

**Present handoff instructions**:

```markdown
## Work Plan Complete

Your work plan is ready at:
`[absolute-path-to-plan-directory]`

### Next Steps

To execute this work plan:

```
/execute-work-plan [absolute-path-to-plan-directory]
```

This will:
1. Read milestones just-in-time (one at a time)
2. Dispatch `code-worker` agents for each task
3. Run code review once per milestone
4. Fix all issues before proceeding to next milestone
5. Provide a full implementation report when complete

### To Resume Later

If you need to resume work on this plan in a new session:
1. Run `/clear` to reset context
2. Run `/execute-work-plan [plan-directory-path]`
3. The skill will detect progress and resume from the next incomplete task

### Files Created

[List all milestone_##.md files with paths]
```

Mark Task #9 complete.

## Resumption

If the user returns to continue a work plan:

1. Ask for the plan directory path
2. Read existing milestone files
3. Identify which milestones are complete vs pending
4. Resume from the next pending milestone

## Common Mistakes

| Mistake | Why It Fails | Correct Approach |
|---------|--------------|------------------|
| Guessing design doc path | Wrong file, wasted effort | Always ask if not provided |
| Skipping review mode choice | User loses control over workflow | AskUserQuestion at session start |
| Skipping codebase investigation | Tasks may not be implementable | Always investigate before planning |
| Not tracking orchestration tasks | Lose progress on interruption | Create tasks at session start |
| Relative paths in handoff | Paths break across sessions | Always use absolute paths |
| Batch planning all milestones | Early mistakes compound | Interactive milestone-by-milestone |
| Skipping code review phase | Plan quality issues compound during implementation | Code review is MANDATORY |
| Wrong file naming (`m1-*.md`) | Breaks execution skill expectations | Use `milestone_01.md` format |
| Incomplete milestone headers | Implementers lack context | All header fields are required |

## Anti-Rationalizations

- "I know where the design doc is" — Ask anyway. Confirmation prevents errors.
- "Codebase investigation is slow" — It's faster than planning tasks for nonexistent files.
- "Task tracking is overhead" — It's insurance against interruption. Do it.
- "The user knows the paths" — Capture absolute paths explicitly. Context resets.
- "Batch mode is faster" — User loses opportunity to catch early mistakes. Compounding errors cost more than time saved. Ask for their preference.
- "Code review is overkill for a plan" — Plans with issues compound during implementation. A 10-minute review saves hours of debugging.
- "The user will notice the file naming" — They won't until the execution skill fails. Follow the spec exactly.
- "The milestone header is just boilerplate" — It's context for the implementer (human or LLM). Missing context means wrong assumptions.

## Summary

1. **Announce the skill.** User knows what's happening.
2. **Create orchestration tasks.** Track progress through phases.
3. **Investigate before planning.** Codebase reality grounds the plan.
4. **Interactive milestone review.** Approve each before proceeding.
5. **Capture absolute paths.** Handoff survives context reset.
6. **Execution is future work.** Plan is the deliverable for now.
