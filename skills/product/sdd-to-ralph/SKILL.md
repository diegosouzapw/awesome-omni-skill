---
name: sdd-to-ralph
description: This skill should be used when the user wants to convert completed CCSDD specs into Ralph execution artifacts. Use when user says "bridge spec to ralph", "convert sdd to ralph", "sdd to ralph", "generate ralph from spec", "prepare spec for ralph", or wants to take a cc-sdd feature spec and run it through Ralph for autonomous implementation.
allowed-tools: Read, Write, Glob, AskUserQuestion
---

# SDD to Ralph Bridge

Convert completed CCSDD (cc-sdd) spec artifacts into Ralph execution artifacts for autonomous implementation.

- **Input**: `.kiro/specs/<feature>/` — requirements.md, design.md, tasks.md
- **Output**: `.ralph/PROMPT.md`, `.ralph/fix_plan.md`, updated `.ralphrc`, `.ralph/specs/`

## Prerequisites

Verify both systems are installed before proceeding:

1. **Ralph enabled**: `.ralph/` directory and `.ralphrc` file exist
   - If missing: instruct user to run `ralph-enable`
2. **CCSDD installed**: `.kiro/` directory exists with at least one spec
   - If missing: instruct user to run `npx cc-sdd@latest --claude`

If either prerequisite fails, stop and report the issue.

## Step 1: Detect and Select Feature Spec

Glob for `.kiro/specs/*/spec.json` to find available specs.

**If 0 specs found:**

- Error: "No CCSDD specs found in `.kiro/specs/`"
- Instruct: "Run `/kiro:spec-init <description>` to create a feature spec first"
- Stop

**If 1 spec found:**

- Read spec.json, extract `feature_name`
- Confirm: "Found spec: **<feature_name>**. Proceeding."

**If multiple specs found:**

- Read each spec.json to get feature names and phases
- Use AskUserQuestion to present the list with phase status:
  - Header: "Feature"
  - Question: "Which CCSDD spec should be bridged to Ralph?"
  - Options: list each feature with its current phase (e.g., "auth-system (tasks-generated)")

## Step 2: Validate Spec Completeness

Read the selected `.kiro/specs/<feature>/spec.json` and check:

1. `approvals.requirements.generated` is `true`
2. `approvals.design.generated` is `true`
3. `approvals.tasks.generated` is `true`

**If all phases complete:** proceed to Step 3.

**If incomplete phases detected:**

- List missing phases with suggested commands:
  - Requirements missing → `/kiro:spec-requirements <feature>`
  - Design missing → `/kiro:spec-design <feature>`
  - Tasks missing → `/kiro:spec-tasks <feature>`
- Use AskUserQuestion:
  - Header: "Incomplete"
  - Question: "Spec has incomplete phases. Proceed anyway?"
  - Options: "Proceed" (continue with available artifacts), "Stop" (exit to complete phases)

## Step 3: Read CCSDD Artifacts

Read all available artifacts from `.kiro/specs/<feature>/`:

1. `requirements.md` — EARS-format requirements
2. `design.md` — technical design
3. `tasks.md` — implementation tasks with parallel markers

Read steering docs if they exist (for additional project context):

1. `.kiro/steering/product.md` — project vision
2. `.kiro/steering/tech.md` — technology stack
3. `.kiro/steering/structure.md` — directory conventions

Read Ralph configuration:

1. `.ralphrc` — current settings

If a file doesn't exist, note it and proceed with what's available. At minimum, `tasks.md` must exist (it provides the fix_plan content).

<critical>
If `tasks.md` does not exist or is empty, stop and report: "Cannot generate fix_plan.md without tasks. Run `/kiro:spec-tasks <feature>` first."
</critical>

## Step 4: Generate PROMPT.md

Create `.ralph/PROMPT.md` with two parts.

### Part 1: Project-Specific Content

Synthesize from the CCSDD artifacts read in Step 3:

```markdown
# Ralph Development Instructions

## Context

[2-3 sentences from design.md overview/summary section describing what is being built]

You are Ralph, building [brief description].

## Technology Stack

[Extract from design.md "Technology Stack" section or .kiro/steering/tech.md]
[Include version numbers when available]

- [Language] [version]
- [Framework] [version]
- [Library] [version] - [purpose]

## Key Principles

[Extract top 5 design decisions/principles from design.md]

- [Principle 1]
- [Principle 2]

## Current Objectives

[Extract high-level requirement titles from requirements.md — NOT full EARS criteria]

1. [Requirement group title] - [brief description]
2. [Requirement group title] - [brief description]

## Requirements Summary

[Numbered list with brief descriptions — point to full specs for detail]

1. [Requirement name] ([N] acceptance criteria)
   - [Key criteria summary]
2. [Requirement name] ([N] acceptance criteria)
   - [Key criteria summary]

Full EARS-format requirements: `.ralph/specs/requirements.md`

## Design Reference

[Key architecture decisions from design.md, ~500 words max]
[Include architecture diagram if present (Mermaid syntax)]

Full technical design: `.ralph/specs/design.md`

## Quality Standards

[Extract testing/quality requirements from design.md or requirements.md]

- [Standard 1]
- [Standard 2]
```

<guidelines type="prompt-context">
- Summarize, don't dump — Ralph reads PROMPT.md every loop, so it must be concise
- Full specs are in `.ralph/specs/` for reference when Ralph needs detail
- Include version numbers for all technologies
- Make principles actionable — things Claude can verify
- Keep Requirements Summary to titles and key criteria only
- Design Reference should focus on architecture decisions that affect implementation
</guidelines>

### Part 2: Ralph Operational Instructions

Read [references/ralph-operational-instructions.md](references/ralph-operational-instructions.md) and append the content block (everything inside the ` ```markdown ` fence, lines 8-139) to the end of the generated PROMPT.md.

<critical>
The operational instructions section is MANDATORY. Without it:
- Claude won't output the RALPH_STATUS block that Ralph parses for exit signals
- Claude won't know when to set EXIT_SIGNAL: true vs false
- Ralph will either never exit or exit prematurely
- The circuit breaker and response analyzer depend on structured status output
</critical>

Write the complete PROMPT.md to `.ralph/PROMPT.md`.

## Step 5: Generate fix_plan.md

Convert CCSDD's `tasks.md` into Ralph's `fix_plan.md` using `/kiro:spec-impl` commands.

### Conversion Algorithm

1. **Parse tasks.md** — extract all checkbox items (`- [ ]` or `- [x]`)
2. **Identify leaf tasks** — tasks with no sub-tasks (e.g., `1.1`, `1.2`, `2.3`). Skip parent-only tasks (e.g., `1.` if it has `1.1`, `1.2` children)
3. **Extract task IDs** — the numeric prefix of each leaf task (e.g., `1.1`, `2.3`)
4. **Generate spec-impl commands** — one per leaf task, in order
5. **Add closing task** — `/kiro:validate-impl` as final item
6. **Add standard sections**: Completed, Discovered

### Output Format

```markdown
# Fix Plan - <feature-name>

## Tasks

- [ ] /kiro:spec-impl <feature-name> 1.1
- [ ] /kiro:spec-impl <feature-name> 1.2
- [ ] /kiro:spec-impl <feature-name> 1.3
- [ ] /kiro:spec-impl <feature-name> 2.1
- [ ] /kiro:spec-impl <feature-name> 2.2
- [ ] /kiro:validate-impl <feature-name>

## Completed

## Discovered

<!-- Ralph will add discovered tasks here -->
```

<guidelines type="fix-plan">
- Only include leaf tasks — parent tasks are organizational groupings, not implementable units
- Preserve the task ordering from tasks.md (respects dependency structure via P0/P1/P2 ordering)
- The /kiro:validate-impl task is always last — it closes the spec→implementation→validation loop
</guidelines>

Write the complete fix_plan.md to `.ralph/fix_plan.md`.

## Step 6: Update .ralphrc

Read `.ralphrc` and update project-specific fields. Preserve all other settings.

**Update PROJECT_NAME:**

- Set to `feature_name` from spec.json

**Detect and update PROJECT_TYPE:**

- Scan design.md tech stack for keywords:
  - "TypeScript", "Node.js", "npm", "React", "Next.js" → `typescript`
  - "Python", "FastAPI", "Django", "Flask" → `python`
  - "Rust", "Cargo" → `rust`
  - "Go", "golang" → `go`
  - No match → leave as current value

**Update ALLOWED_TOOLS based on PROJECT_TYPE:**

| Project Type | Add to ALLOWED_TOOLS                           |
| ------------ | ---------------------------------------------- |
| typescript   | `Bash(npm *),Bash(npx *)`                      |
| python       | `Bash(pip *),Bash(python *),Bash(pytest)`      |
| rust         | `Bash(cargo *)`                                |
| go           | `Bash(go *)`                                   |
| All types    | `Write,Read,Edit,Bash(git *)` (always include) |

**Do NOT modify:** MAX_CALLS_PER_HOUR, CLAUDE_TIMEOUT_MINUTES, SESSION_CONTINUITY, SESSION_EXPIRY_HOURS, circuit breaker thresholds, or any other existing settings.

Write updated `.ralphrc`.

## Step 7: Copy Spec Files to Ralph

Create `.ralph/specs/` directory if it doesn't exist. Copy CCSDD spec files so Ralph can reference them during implementation:

1. `.kiro/specs/<feature>/requirements.md` → `.ralph/specs/requirements.md`
2. `.kiro/specs/<feature>/design.md` → `.ralph/specs/design.md`
3. `.kiro/specs/<feature>/research.md` → `.ralph/specs/research.md` (if exists)

## Step 8: Verify Generated Files

Read back generated files and verify:

### PROMPT.md checks

1. Contains `---RALPH_STATUS---` (exact string)
2. Contains `EXIT_SIGNAL` examples
3. Contains project-specific context sections (Context, Technology Stack, Current Objectives)
4. Is at least 100 lines (both parts present)

### fix_plan.md checks

1. Has `## Tasks` section
2. Has at least one `- [ ] /kiro:spec-impl` item
3. Has `## Completed` section
4. Has `## Discovered` section
5. Contains `/kiro:validate-impl` as final task item

**If any check fails:** report the specific failure but do NOT delete files. Suggest re-running or manual fix.

## Step 9: Present Summary

Display results to the user:

```
Successfully bridged CCSDD spec to Ralph!

Feature: <feature-name>

Tasks: N spec-impl commands + validation
  /kiro:spec-impl <feature-name> 1.1
  /kiro:spec-impl <feature-name> 1.2
  ...
  /kiro:validate-impl <feature-name>

Files Generated:
  .ralph/PROMPT.md
  .ralph/fix_plan.md
  .ralph/specs/requirements.md
  .ralph/specs/design.md

Configuration Updated:
  .ralphrc PROJECT_NAME = "<feature-name>"
  .ralphrc PROJECT_TYPE = "<type>"

[Any warnings about incomplete phases, etc.]

Next Steps:
  1. Review .ralph/PROMPT.md and .ralph/fix_plan.md
  2. Run: ralph --monitor
```

## Error Handling

**No `.ralph/` directory:**

- "Ralph is not enabled. Run `ralph-enable` first."

**No `.kiro/` directory:**

- "CCSDD is not installed. Run `npx cc-sdd@latest --claude` first."

**Empty or missing `tasks.md`:**

- "No tasks found. Run `/kiro:spec-tasks <feature>` to generate tasks."

**Missing steering docs:**

- Warning only. Extract context from design.md alone.

**Tasks without parallel markers:**

- Infer from parent or position. Log in Notes section.

**Corrupted spec.json:**

- Show parse error. Suggest manual inspection.
