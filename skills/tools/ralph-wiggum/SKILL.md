---
name: ralph-wiggum
description: Iterative AI development loops - start, monitor, review, and iterate on tasks using the Ralph Wiggum technique with Kiro CLI - use only when explicitly requested by the user
license: MIT
---

# Ralph Wiggum - Iterative Development Loops

## Overview

Iterative AI development where an agent works across multiple iterations until task complete. Each iteration builds on previous work in a self-referential feedback loop.

**Core principle:** Iteration > Perfection. Persistence wins.

**Your role:** Orchestrator. Start Ralph, review results, decide next steps.

**Context separation:** Ralph runs in Kiro CLI with separate conversation history from OpenCode. Enables autonomous iteration without cluttering your context.

**Only activate when explicitly requested by user.** Like TDD, don't use by default.

## When to Use

**Use for:**
- Multi-step tasks requiring iteration/refinement
- Tasks with clear, verifiable completion criteria
- Feature development requiring testing/debugging
- Bug fixes needing investigation/validation
- Refactoring with test coverage requirements
- "Try, fail, fix, repeat" workflows

**Don't use for:**
- Quick one-shot tasks (do directly)
- Unclear completion criteria
- Requires human judgment/design decisions
- No concrete deliverables

## Mode Selection

**Decision tree:**
```
Multi-task feature/backlog? → PRD mode
Single task + tests exist?  → Verify mode
Single task, no tests?      → Promise mode
Open-ended exploration?     → Free run
```

**PRD mode is recommended** for most feature work - provides structure, atomic commits, clear progress tracking.

## Modes of Operation

### PRD Mode (Recommended for multi-task work)

```bash
ralph-loop "Implement user authentication" \
  --prd plans/auth-prd.json \
  --verify-command "npm test"
```

- Works through structured task list one at a time
- Each task has explicit verification steps
- Auto-commits after each completed task
- Exits when all tasks have `passes: true`
- Best for feature implementation with multiple components

### Verify Mode (Single task with tests)

```bash
ralph-loop "Fix failing tests" --verify-command "npm test"
```

- Runs verify after EVERY iteration
- Exits when verify passes
- No promise needed

### Promise Mode (Single task, no tests)

```bash
ralph-loop "Refactor auth module" --completion-promise "DONE"
```

- Exits when model outputs `<promise>DONE</promise>`
- Model self-reports completion

### Verify + Promise (Belt and suspenders)

```bash
ralph-loop "Add user feature" \
  --completion-promise "DONE" \
  --verify-command "npm test"
```

- Waits for promise, then runs verify to confirm
- Best of both worlds

### Free Run (Exploration)

```bash
ralph-loop "Improve codebase" --max-iterations 10
```

- Just runs N iterations
- No completion detection
- Good for open-ended exploration

### Resume (Continue existing session)

```bash
ralph-loop --resume "New instructions here"
```

- Continues previous conversation
- Requires new prompt
- Flags override previous state

## PRD File

### What is a PRD?

A JSON array of tasks with explicit verification steps. User must provide it - Ralph doesn't create PRDs.

### Schema

```json
[
  {
    "category": "string - task grouping (ui, functional, api, backend, etc.)",
    "description": "string - what to implement or verify",
    "steps": ["array", "of", "verification", "steps"],
    "passes": false
  }
]
```

| Field | Required | Description |
|-------|----------|-------------|
| `category` | Yes | Task grouping (ui, functional, api, backend, etc.) |
| `description` | Yes | What to implement or verify |
| `steps` | Yes | Explicit verification steps |
| `passes` | Yes | Completion status (start: false) |

### Writing Good PRD Tasks

- **One deliverable per task** - each task should be completable in 1-3 iterations
- **Explicit steps** - list what to verify, not just what to build
- **Order by dependency** - tasks that depend on others come later
- **Testable** - each step should be verifiable (test output, UI check, etc.)
- **Avoid vague descriptions** - "improve code" is bad, "add input validation for email field" is good

### Example: User Authentication PRD

```json
[
  {
    "category": "backend",
    "description": "User registration creates account with hashed password",
    "steps": [
      "POST /register with email and password",
      "Verify 201 response with user object",
      "Verify password is not returned in response",
      "Verify password is hashed in database",
      "Verify duplicate email returns 409 conflict"
    ],
    "passes": false
  },
  {
    "category": "backend",
    "description": "User login returns valid JWT token",
    "steps": [
      "POST /login with valid credentials",
      "Verify 200 response with token",
      "Verify token is valid JWT format",
      "POST /login with invalid password",
      "Verify 401 unauthorized response"
    ],
    "passes": false
  },
  {
    "category": "backend",
    "description": "Protected routes require valid JWT",
    "steps": [
      "GET /me without token returns 401",
      "GET /me with invalid token returns 401",
      "GET /me with valid token returns user object"
    ],
    "passes": false
  },
  {
    "category": "ui",
    "description": "Login form validates input and shows errors",
    "steps": [
      "Navigate to /login",
      "Submit empty form",
      "Verify validation errors shown",
      "Enter invalid email format",
      "Verify email validation error",
      "Enter valid credentials and submit",
      "Verify redirect to dashboard"
    ],
    "passes": false
  }
]
```

### Example: UI Feature PRD

```json
[
  {
    "category": "ui",
    "description": "Delete video shows confirmation dialog before deleting",
    "steps": [
      "Navigate to a video",
      "Click delete button",
      "Verify confirmation dialog appears",
      "Click cancel",
      "Verify video is not deleted",
      "Click delete again and confirm",
      "Verify video is deleted"
    ],
    "passes": false
  },
  {
    "category": "ui",
    "description": "Video thumbnail shows duration overlay",
    "steps": [
      "Navigate to video list",
      "Verify each thumbnail shows duration in bottom-right",
      "Verify format is MM:SS for videos under 1 hour",
      "Verify format is HH:MM:SS for videos over 1 hour"
    ],
    "passes": false
  }
]
```

## Progress File

### When to Use

- Long-running loops (20+ iterations expected)
- Complex tasks where context matters across iterations
- Work you expect to resume later
- Tasks where learnings from failed attempts matter

### What It Does

- Agent reads at start of each iteration
- Agent appends summary at end of each iteration
- Survives context window compaction
- Acts as cross-session memory

### Usage

```bash
ralph-loop "Build feature" \
  --prd plans/prd.json \
  --progress plans/progress.md
```

Not required - optional enhancement for complex work.

## Auto-Commit

### Behavior

- **Default ON** when using `--prd`
- **Default OFF** otherwise
- Commits after each task marked `passes: true`
- Format: `feat: <description>` (truncated to 72 chars)

### What Gets Committed

- Code changes from the iteration
- PRD file update (`passes: true`)
- Progress file update (if used)

### Disabling

```bash
ralph-loop "Task" --prd prd.json --no-auto-commit
```

### Requirements

- Must be in git repo (auto-disabled if not)
- Uncommitted changes before Ralph starts are fine

## Starting a Ralph Loop

### Step 1: Create PRD (for PRD mode)

Write your PRD file with tasks and verification steps. See examples above.

### Step 2: Craft the Prompt

Keep it focused - one objective per loop.

**Good:**
```bash
ralph-loop "Implement user authentication per the PRD" \
  --prd plans/auth-prd.json \
  --verify-command "npm test"
```

**Bad:**
```bash
ralph-loop "Build auth, add tests, refactor DB, update docs" --max-iterations 40
```

### Step 3: Set Max-Iterations

| Complexity | Recommended |
|------------|-------------|
| Simple (1-3 tasks) | 10-15 |
| Medium (4-7 tasks) | 20-30 |
| Complex (8+ tasks) | 40-50 |

Default is 50. Ralph exits early on success.

### Step 4: Execute

```bash
cd /path/to/project
ralph-loop "YOUR_PROMPT" \
  --prd plans/prd.json \
  --progress plans/progress.md \
  --verify-command "npm test" \
  --max-iterations 30
```

Ralph runs autonomously. Ctrl+C interrupts cleanly, state preserved for --resume.

## Reviewing Ralph's Work

### For PRD Mode

1. **Check PRD file** - how many tasks have `passes: true`?
2. **Review git log** - see atomic commits per completed task
3. **Read progress file** (if used) - understand decisions and blockers
4. **Run tests yourself** - verify independently
5. **Spot-check completed tasks** - do they actually work?

### For Other Modes

1. **Read modified files** - use Read tool on changed files
2. **Run tests yourself** - `npm test` (or equivalent)
3. **Code review** - use code-reviewer for complex changes
4. **Verify requirements** - all met? Quality acceptable?

Can't verify all? Don't claim success. Resume or take over.

## Workflow

```
ASSESS → CREATE PRD → START → WAIT → REVIEW → ITERATE or COMPLETE
```

**Assess:** Is task suitable? Clear completion criteria? Needs iteration?

**Create PRD:** Break down into tasks with verification steps.

**Review:** Check PRD progress, git log, run tests yourself.

**Always verify independently** - don't trust passes field alone.

## Iteration Patterns

### Complete and Successful

```
All PRD tasks pass: true
Tests pass independently
Requirements met
ACTION: Report success
```

### Incomplete - Resume

```
Max iterations reached
Some tasks still passes: false
Progress was made
ACTION: Use --resume with refined instructions
```

```bash
ralph-loop --resume "Focus on the remaining UI tasks, the backend is complete"
```

### Failed - Take Over

```
Ralph stuck on same task
Multiple failed approaches
Requires human judgment
ACTION: Complete task directly
```

Know when to take over. Don't force Ralph on unsolvable problems.

## Common Patterns

### PRD-Driven Feature

```bash
ralph-loop "Implement user authentication system" \
  --prd plans/auth-prd.json \
  --progress plans/auth-progress.md \
  --verify-command "npm test" \
  --max-iterations 30
```

### Bug Fix with TDD

```bash
ralph-loop "Fix bug: empty comments accepted. Write failing test first, then fix." \
  --verify-command "npm test" \
  --max-iterations 15
```

### Refactoring with Safety

```bash
ralph-loop "Refactor database layer to use repository pattern. Keep all tests passing." \
  --verify-command "npm test" \
  --max-iterations 25
```

## Troubleshooting

### Max Iterations Reached

**Causes:** Task too complex, unclear PRD steps, missing dependencies

**Solutions:**
1. Check PRD - which tasks are still `passes: false`?
2. Check progress file - what's blocking?
3. Use --resume with clearer instructions
4. Break remaining tasks into smaller steps
5. Take over if Ralph is stuck

### PRD Task Won't Pass

Ralph may not understand the verification steps.

**Solutions:**
1. Check if steps are specific enough
2. Use --resume with clarification
3. Manually verify and mark `passes: true` if actually working

### State File Issues

**Orphaned state:** Delete `.kiro/ralph-state.json` and start fresh

**Can't resume:** State missing. Start new loop.

## Quick Reference

| Situation | Mode | Command |
|-----------|------|---------|
| Multi-task feature | PRD | `--prd file.json --verify-command "..."` |
| Single task + tests | Verify | `--verify-command "..."` |
| Single task, no tests | Promise | `--completion-promise "DONE"` |
| Exploration | Free run | `--max-iterations N` |
| Continue previous | Resume | `--resume "new instructions"` |

| Flag | Default | Description |
|------|---------|-------------|
| `--prd FILE` | none | PRD file path (enables PRD mode) |
| `--progress FILE` | none | Progress file for cross-session memory |
| `--verify-command CMD` | none | Verification command to run |
| `--completion-promise TEXT` | none | Exit when promise detected |
| `--auto-commit` | ON (PRD mode) | Enable auto-commit |
| `--no-auto-commit` | OFF | Disable auto-commit |
| `--max-iterations N` | 50 | Maximum iterations |
| `--resume` | false | Resume existing session |

**You orchestrate. Ralph executes. Always review.**
