---
name: agentic-orchestrating
description: Provides the "how-to" for workflow / task execution orchestration. Defines methods for planning multi-phase task/workflows, implementing them through agent delegation, and managing artifacts.
---

# Workflow Orchestration

Knowledge base for orchestrating multi-phase workflow executions with standardized folder structures and agent delegation.

## Core Concepts

This skill embodies seven key orchestration principles:

1. **[Planning](references/planning.md)** - HOW to create execution plans with phase breakdown and artifact definition
2. **[Implementation](references/implementation.md)** - HOW to execute plans through agent delegation and artifact management
3. **[Executing](references/executing.md)** - HOW to run pre-defined workflows using EXECUTE.md blueprints
4. **[Workflow to Execute](references/workflow-to-execute.md)** - HOW to convert plain-English WORKFLOW.md into structured EXECUTE.md
5. **[Resuming](references/resuming.md)** - HOW to restart workflows from specific phases with selective regeneration
6. **[Delegation](references/delegation.md)** - HOW to maintain focused context through strategic agent handoffs
7. **[Artifacts](references/artifacts.md)** - HOW to standardize outputs that flow between phases

### Progressive Disclosure

A core architectural principle that emerges from delegation: **pass file paths, not content**. This enables agents to load context when needed, maintaining high signal-to-noise ratio. Instead of inlining everything upfront, each agent reads only the files it needs for its specific task.

## Standard Execution Folder

All task/workflow executions follow this structure:

```
/{base-dir}/{task/workflow-name}/{YYYY-MM-DD_HH:MM}/
├── data/              # Input files (optional)
├── artifacts/         # Phase outputs
│   ├── 01-*.md
│   ├── 02a-*.md
│   ├── 02b-*.md
│   └── 03-*.md
├── PLAN.md           # Initial plan
└── RESEARCH.md/STRATEGY.md/CONTENT.md       # Final output
```

Where `base-dir` is either `research`, `strategy` or `content`.


See [Artifacts](references/artifacts.md) for naming conventions and folder structure details.

## Orchestration Workflows

### Custom Task Execution (Planning → Implementation)

For one-off, custom tasks that require bespoke execution plans:

#### Planning Phase
**Purpose:** Create a structured execution plan

**Inputs:**
- Task description or workflow definition
- User requirements and context

**Process:** See [Planning](references/planning.md)

**Outputs:**
- Execution folder structure
- PLAN.md with phase breakdown
- Expected artifact definitions

#### Implementation Phase
**Purpose:** Execute the plan through [agent delegation](references/delegation.md)

**Inputs:**
- PLAN.md from execution folder
- data/ (optional, created in planning phase)

**Process:** See [Implementation](references/implementation.md)

**Outputs:**
- Phase artifacts in execution folder
- Final RESEARCH.md/STRATEGY.md/CONTENT.md

---

### Standardized Workflow Execution

For repeatable, standardized workflows defined by Marketing Architects:

#### Workflow Conversion (One-time Setup)
**Purpose:** Convert plain-English WORKFLOW.md into structured EXECUTE.md

**Inputs:**
- WORKFLOW.md from skill folder
- Understanding of agent types and capabilities

**Process:** See [Workflow to Execute](references/workflow-to-execute.md)

**Outputs:**
- EXECUTE.md in same folder as WORKFLOW.md
- Ready for repeated execution via `/execute` command

#### Execution Phase
**Purpose:** Run pre-defined workflow using EXECUTE.md blueprint

**Inputs:**
- EXECUTE.md from skill workflow folder
- User inputs (slug, topic, context)
- data/ (optional, if EXECUTE.md specifies)

**Process:** See [Executing](references/executing.md)

**Outputs:**
- Execution folder in appropriate location
- Phase artifacts
- Final RESEARCH.md/STRATEGY.md/CONTENT.md

---

### Resumption Phase
**Purpose:** Restart execution from specific phase with selective regeneration

**Inputs:**
- Existing execution folder with PLAN.md (or EXECUTE.md)
- User specification of resume point
- Existing artifacts/ (some preserved, some regenerated)

**Process:** See [Resuming](references/resuming.md)

**Outputs:**
- Regenerated artifacts from resume point forward
- Preserved artifacts from before resume point
- Updated final output

## Tools and Resources

### Scripts
- `scripts/create_execution_folder.py` - Creates timestamped execution folders

### Templates
- `assets/PLAN_EXECUTE_template.md` - Plan/Execute blueprint template