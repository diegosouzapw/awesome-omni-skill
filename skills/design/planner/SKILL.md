---
name: planner
description: Interactive sequential planning for complex tasks. Use when breaking down multi-step projects, system designs, migration strategies, or architectural decisions. Invoked via python script that outputs required actions between steps.
---

# Planner Skill

## Purpose

Two-phase planning workflow with forced reflection pauses:

1. **PLANNING PHASE**: Break down complex tasks into milestones with concrete specifications
2. **REVIEW PHASE**: Orchestrate TW annotation and QR validation before execution

## When to Use

Use the planner skill when the task has:

- Multiple milestones with dependencies
- Architectural decisions requiring documentation
- Migration steps that need coordination
- Complexity that benefits from forced reflection pauses

## When to Skip

Skip the planner skill when the task is:

- Single-step with obvious implementation
- A quick fix or minor change
- Already well-specified by the user

## Workflow Overview

```
PLANNING PHASE (steps 1-N)
    |
    v
Write plan to file
    |
    v
REVIEW PHASE (steps 1-2)
    |-- Step 1: @agent-technical-writer (plan-annotation)
    |-- Step 2: @agent-quality-reviewer (plan-review)
    v
APPROVED --> /plan-execution
```

---

## PLANNING PHASE

### Preconditions

Before invoking step 1, you MUST have:

1. **Plan file path** - If user did not specify, ASK before proceeding
2. **Clear problem statement** - What needs to be accomplished

### Invocation

Script location: `scripts/planner.py` (relative to this skill)

```bash
python3 scripts/planner.py \
  --step-number 1 \
  --total-steps <estimated_steps> \
  --thoughts "<your thinking about the problem>"
```

### Arguments

| Argument        | Description                                      |
| --------------- | ------------------------------------------------ |
| `--phase`       | Workflow phase: `planning` (default) or `review` |
| `--step-number` | Current step (starts at 1)                       |
| `--total-steps` | Estimated total steps for this phase             |
| `--thoughts`    | Your thinking, findings, and progress            |

### Planning Workflow

1. Confirm preconditions (plan file path, problem statement)
2. Invoke step 1 immediately
3. Complete REQUIRED ACTIONS from output
4. Invoke next step with your thoughts
5. Repeat until `STATUS: phase_complete`
6. Write plan to file using format below

---

## Phase Transition: Planning to Review

When planning phase completes, the script outputs an explicit `ACTION REQUIRED` marker:

```
============================================
>>> ACTION REQUIRED: INVOKE REVIEW PHASE <<<
============================================
```

**You MUST invoke the review phase before proceeding to /plan-execution.**

The review phase ensures:

- Temporally contaminated comments are fixed (via @agent-technical-writer)
- Code snippets have WHY comments (via @agent-technical-writer)
- Plan is validated for production risks (via @agent-quality-reviewer)
- Documentation needs are identified

**Why TW is mandatory**: The planning phase naturally produces temporally contaminated comments -- change-relative language ("Added...", "Replaced..."), baseline references ("Instead of...", "Previously..."), and location directives ("After line 425"). These make sense during planning but are inappropriate for production code. TW transforms them to timeless present form before @agent-developer transcribes them verbatim.

Without review, @agent-developer will transcribe contaminated comments directly into production code.

---

## REVIEW PHASE

After writing the plan file, transition to review phase:

```bash
python3 scripts/planner.py \
  --phase review \
  --step-number 1 \
  --total-steps 2 \
  --thoughts "Plan written to [path/to/plan.md]"
```

### Review Step 1: Technical Writer Review and Fix

Delegate to @agent-technical-writer with mode: `plan-annotation`

TW will:

- **Review and fix** temporally contaminated comments (see `resources/temporal-contamination.md`)
- Read ## Planning Context section
- Add WHY comments to code snippets
- Enrich plan prose with rationale
- Add documentation milestone if missing

**This step is never skipped.** Even if plan prose seems complete, code comments from the planning phase require temporal contamination review.

### Review Step 2: Quality Reviewer Validation

Delegate to @agent-quality-reviewer with mode: `plan-review`

QR will:

- Check production reliability (RULE 0)
- Check project conformance (RULE 1)
- Verify TW annotations are sufficient
- Exclude risks already documented in Planning Context
- Return verdict: PASS | PASS_WITH_CONCERNS | NEEDS_CHANGES

### After Review

- **PASS / PASS_WITH_CONCERNS**: Ready for `/plan-execution`
- **NEEDS_CHANGES**: Return to planning phase to address issues

---

## Plan Format

Write your plan using this structure:

```markdown
# [Plan Title]

## Overview

[Problem statement, chosen approach, and key decisions in 1-2 paragraphs]

## Planning Context

This section is consumed VERBATIM by downstream agents (Technical Writer, Quality Reviewer).
Quality matters: vague entries here produce poor annotations and missed risks.

### Decision Log

| Decision           | Reasoning Chain                                            |
| ------------------ | ---------------------------------------------------------- |
| [What you decided] | [Multi-step reasoning: premise → implication → conclusion] |

Each rationale must contain at least 2 reasoning steps. Single-step rationales are insufficient.

INSUFFICIENT: "Polling over webhooks | Webhooks are unreliable"
SUFFICIENT: "Polling over webhooks | Third-party API has 30% webhook delivery failure in testing → unreliable delivery would require fallback polling anyway → simpler to use polling as primary mechanism"

INSUFFICIENT: "500ms timeout | Matches upstream latency"
SUFFICIENT: "500ms timeout | Upstream 95th percentile is 450ms → 500ms covers 95% of requests without timeout → remaining 5% should fail fast rather than queue"

Include BOTH architectural decisions AND implementation-level micro-decisions:

- Architectural: "Event sourcing over CRUD | Need audit trail + replay capability → CRUD would require separate audit log → event sourcing provides both natively"
- Implementation: "Mutex over channel | Single-writer case → channel coordination adds complexity without benefit → mutex is simpler with equivalent safety"

Technical Writer sources ALL code comments from this table. If a micro-decision isn't here, TW cannot document it.

### Rejected Alternatives

| Alternative          | Why Rejected                                                        |
| -------------------- | ------------------------------------------------------------------- |
| [Approach not taken] | [Concrete reason: performance, complexity, doesn't fit constraints] |

Technical Writer uses this to add "why not X" context to code comments.

### Constraints & Assumptions

- [Technical: API limits, language version, existing patterns to follow]
- [Organizational: timeline, team expertise, approval requirements]
- [Dependencies: external services, libraries, data formats]

### Known Risks

| Risk            | Mitigation                                    |
| --------------- | --------------------------------------------- |
| [Specific risk] | [Concrete mitigation or "Accepted: [reason]"] |

Quality Reviewer excludes these from findings - be thorough.

## Invisible Knowledge

This section captures information NOT visible from reading the code. Technical Writer uses this for README.md documentation during post-implementation.

### Architecture
```

[ASCII diagram showing component relationships]

Example:
User Request
|
v
+----------+ +-------+
| Auth |---->| Cache |
+----------+ +-------+
|
v
+----------+ +------+
| Handler |---->| DB |
+----------+ +------+

```

### Data Flow
```

[How data moves through the system - inputs, transformations, outputs]

Example:
HTTP Request --> Validate --> Transform --> Store --> Response
|
v
Log (async)

````

### Why This Structure
[Reasoning behind module organization that isn't obvious from file names]
- Why these boundaries exist
- What would break if reorganized differently

### Invariants
[Rules that must be maintained but aren't enforced by code]
- Ordering requirements
- State consistency rules
- Implicit contracts between components

### Tradeoffs
[Key decisions with their costs and benefits]
- What was sacrificed for what gain
- Performance vs. readability choices
- Consistency vs. flexibility choices

## Milestones

### Milestone 1: [Name]
**Files**: [exact paths - e.g., src/auth/handler.py, not "auth files"]

**Flags** (if applicable): [needs TW rationale, needs error handling review, needs conformance check]

**Requirements**:
- [Specific: "Add retry with exponential backoff", not "improve error handling"]

**Acceptance Criteria**:
- [Testable: "Returns 429 after 3 failed attempts" - QR can verify pass/fail]
- [Avoid vague: "Works correctly" or "Handles errors properly"]

**Code Changes** (for non-trivial logic, use unified diff format):

See `resources/diff-format.md` for specification.

```diff
--- a/path/to/file.py
+++ b/path/to/file.py
@@ -123,6 +123,15 @@ def existing_function(ctx):
   # Context lines (unchanged) serve as location anchors
   existing_code()

+  # WHY comment explaining rationale - transcribed verbatim by Developer
+  new_code()

   # More context to anchor the insertion point
   more_existing_code()
````

### Milestone N: ...

### Milestone [Last]: Documentation

**Files**:

- `path/to/CLAUDE.md` (index updates)
- `path/to/README.md` (if Invisible Knowledge section has content)

**Requirements**:

- Update CLAUDE.md index entries for all new/modified files
- Each entry has WHAT (contents) and WHEN (task triggers)
- If plan's Invisible Knowledge section is non-empty:
  - Create/update README.md with architecture diagrams from plan
  - Include tradeoffs, invariants, "why this structure" content
  - Verify diagrams match actual implementation

**Acceptance Criteria**:

- CLAUDE.md enables LLM to locate relevant code for debugging/modification tasks
- README.md captures knowledge not discoverable from reading source files
- Architecture diagrams in README.md match plan's Invisible Knowledge section

**Source Material**: `## Invisible Knowledge` section of this plan

## Milestone Dependencies (if applicable)

```
M1 ---> M2
   \
    --> M3 --> M4
```

Independent milestones can execute in parallel during /plan-execution.

````

---

## Resources

| Resource                              | Purpose                                                 |
| ------------------------------------- | ------------------------------------------------------- |
| `resources/diff-format.md`            | Authoritative specification for code change format      |
| `resources/temporal-contamination.md` | Terminology for detecting/fixing temporally contaminated comments |

---

## Quick Reference

```bash
# Start planning
python3 scripts/planner.py --step-number 1 --total-steps 4 --thoughts "..."

# Continue planning
python3 scripts/planner.py --step-number 2 --total-steps 4 --thoughts "..."

# Backtrack if needed
python3 scripts/planner.py --step-number 2 --total-steps 4 --thoughts "New info invalidated prior decision..."

# Start review (after plan written)
python3 scripts/planner.py --phase review --step-number 1 --total-steps 2 --thoughts "Plan at ..."

# Continue review
python3 scripts/planner.py --phase review --step-number 2 --total-steps 2 --thoughts "TW done ..."
````
