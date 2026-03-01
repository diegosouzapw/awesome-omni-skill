---
name: cdd
description: Context Driven Development pipeline for release workflows
---

# CDD Pipeline

This project calls its release workflow Context Driven Development (CDD)

## Workspace Configuration

**All CDD commands and scripts require the `$CDD_DIR` environment variable.**

```bash
export CDD_DIR=/path/to/workspace
```

This allows parallel workspaces without VCS conflicts. Each workspace is independent - different branches can use different workspace directories simultaneously.

Scripts will abort with a clear error if `$CDD_DIR` is not set.

## Core Philosophy

Use user/agent interactive development time during the Research and Plan phases to prepare tightly focused work that pre-loads the exact required context to run autonomously during the Execute phase.

## Pipeline Phases

  * Research
  * Plan
  * Execute

### 1. Research

**What:** Build foundational understanding before design decisions.

**Artifacts:**
- `$CDD_DIR/README.md` - High-level description of release features
- `$CDD_DIR/user-stories/` - User actions and expected system responses
- `$CDD_DIR/research/` - Facts gathered from internet, docs, codebase

**Why this phase exists:** Decisions made without research are assumptions. Research surfaces constraints, prior art, and edge cases that would otherwise appear during execution (when fixing is expensive).

**Key insight:** Research artifacts become reference material for execution. Sub-agents can read `$CDD_DIR/research/` without re-fetching from the internet. The research phase pays the lookup cost once.

### 2. Plan

**What:** Transform research into implementation decisions.

**Artifacts:**
- `$CDD_DIR/plan/` - Implementation decisions, architecture choices, detailed designs

**Why this phase exists:** Planning is the translation layer between "what we want" (research) and "what to do". It makes architectural decisions explicit.

### 3. Verify (Sub-phase of Plan)

**What:** Iterative verification of plan alignment.

**Artifacts:**
- `$CDD_DIR/gap.md` - List of issues to investigate and resolve
- `$CDD_DIR/verified.md` - Log of issues that have been fixed

**Verification:**
```
/cdd:gap  # Verify README/user-stories → plan alignment, writes gap.md
fix gaps  # Address issues in gap.md, update plan documents
update verified.md  # Log what was fixed with references to gap.md
/cdd:gap  # Repeat until clean
```

Checks alignment down the pyramid:
- README/research/user-stories → plan
- Naming, error handling, memory management conventions
- Integration points, build changes, migrations

**Why this phase exists:** This is the **last moment of full context visibility**. Cross-cutting issues and inconsistencies can only be caught while the complete picture is visible.

**Key insights:**
- Specific checklists prevent vague "find issues" prompts
- `gap.md` contains current issues to investigate
- `verified.md` tracks what's been fixed, prevents re-checking completed items

**Convergence pattern:** Each gap-finding run identifies issues and writes gap.md. After fixing, log in verified.md and re-run. When no substantive gaps remain, verification is complete.

### 4. Execute

**What:** Unattended execution via the Ralph harness script.

**Artifacts:**
- `$CDD_DIR/goals/*-goal.md` - Goal file(s) for ralph loop execution
- `$CDD_DIR/goals/*-progress.jsonl` - Auto-generated progress log
- `$CDD_DIR/goals/*-summary.md` - Auto-generated progress summary

**Ralph script:**
```bash
.claude/harness/ralph/run \
  --goal=$CDD_DIR/goals/implementation-goal.md \
  --duration=4h \
  --model=sonnet \
  --reasoning=low
```

**Process:**
Ralph iteratively works toward the goal until DONE or time expires:
1. Read goal + progress history
2. Make incremental progress (code changes, tests, fixes)
3. Commit progress via `jj commit`
4. Append progress to JSONL file
5. Repeat until objective achieved

**Why Ralph:** The plan phase prepared complete context. Ralph execution is mechanical - read plan, implement, test, commit. No research, no exploration, just steady progress toward the goal using the plan as a reference.

**Post-execution:**
After Ralph completes (or during iterations), quality checks and refactoring may be needed:
```
ralph → /check:quality → /refactor → /check:quality
```

## Directory Structure

```
$CDD_DIR/
├── README.md          # High-level release description (PRD)
├── user-stories/      # User actions and system responses
│   └── README.md      # Index/overview
├── research/          # Internet facts for reference
│   └── README.md      # Index/overview
├── plan/              # Implementation decisions
│   └── README.md      # Index/overview
├── gap.md             # Current issues to investigate and resolve
├── verified.md        # Log of issues that have been fixed
├── goals/             # Ralph goal files and execution state
│   ├── *-goal.md      # Goal definition(s)
│   ├── *-progress.jsonl   # Progress tracking (auto-generated)
│   └── *-summary.md   # Progress summary (auto-generated)
├── tmp/               # Temp files during execution
└── ...                # Other files permitted, ignored by pipeline
```

## Abstraction Levels

Each directory serves a distinct abstraction level. Content should not leak between levels.

### $CDD_DIR/plan/ - Coordination Layer

Plan documents **coordinate everything shared**.

**The plan owns:**
- **Public symbols** - Function names, struct names, enum names
- **Function signatures** - Arguments, argument order, return types
- **Struct members** - Field names, field order, field types
- **Enums** - Value names and meanings
- **Allowed libraries** - Which dependencies can be used, which are forbidden
- **Architectural choices** - Module boundaries, data flow, ownership rules

**The plan is the contract.**

**DO NOT include:** Implementation code, detailed algorithms, function bodies.

## Artifact Authority

Artifacts form a hierarchy of authority:

```
$CDD_DIR/README.md (authoritative)
        ↓
user-stories + research (derived)
        ↓
      plan (derived)
```

**Rules:**

1. Lower-level artifacts derive from higher-level ones
2. Contradictions between levels must be resolved immediately
3. Resolution is always top-down: align lower artifacts to higher authority
4. Higher-level documents may change through iteration—but when they do, all derived artifacts must be reviewed and realigned

**Why this matters:** Consistency is enforced during authoring because it cannot be enforced during execution.

## Writing Ralph Goal Files

After creating plan documents, write goal files in `$CDD_DIR/goals/*-goal.md`.

**Use `/load goal-authoring` for detailed guidance.** Key points:

- Ralph has unlimited context through iteration - reference all relevant docs
- Specify WHAT to achieve (outcomes), never HOW (steps/order)
- One cohesive objective per goal - trust Ralph to discover the path
- Complete acceptance criteria - Ralph needs to know when done

## Lifecycle

```
Create empty $CDD_DIR/ → Research → Plan → Verify → Execute → Delete $CDD_DIR/
```

The workspace directory is ephemeral. It exists only for the duration of the release workflow. After successful execution, it's deleted. The work lives in the codebase; the pipeline artifacts are disposable.

## Efficiency Principles

1. **Load skills on-demand** - Don't preload "just in case"
2. **Complete authoring** - Spend tokens researching during authoring so execution doesn't need to
3. **Reference vs working knowledge** - Large docs go in separate skills, loaded when needed

## Design Tradeoffs

| Decision | Tradeoff | Rationale |
|----------|----------|-----------|
| Full context in verify | Expensive per-run | Last chance to catch cross-cutting issues |
| Verification is human-terminated | Subjective | Models don't converge to "done" naturally |
| Research cached in files | Stale if release spans days | Avoids re-fetching during execution |

## When to Load This Skill

- Designing or modifying the release workflow
- Debugging why execution sub-agents are failing
- Optimizing token usage in the pipeline
- Understanding why a phase exists before changing it
