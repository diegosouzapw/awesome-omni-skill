---
name: fix-ci
description: Diagnose and fix GitHub Actions CI failures. Use when CI pipeline is failing.
argument-hint: "[run-id|--learn]"
category: orchestration
context: fork
---

# /fix-ci

## Usage

```bash
/fix-ci              # Fix latest failure
/fix-ci 12345678     # Fix specific run
/fix-ci --learn      # Show historical fix patterns
```

## Description

Two-phase CI failure resolution: diagnose with debugger agents, then fix with domain-specialized agents.

## Architecture

### Phase 1: Diagnosis (Parallel Debuggers)

Deploy debugger agents in parallel to investigate each failure. Each debugger returns:

- **Root cause**: What actually failed and why
- **Domain**: Classification for agent routing (see matrix below)
- **Files**: Specific files that need changes
- **Fix approach**: Recommended solution

### Phase 2: Fix (Specialized Agents)

Route fixes to domain experts based on diagnosis:

| Domain | Fix Agent | Examples |
|--------|-----------|----------|
| test | test-engineer | Test failures, missing mocks, assertion errors |
| security | security-auditor | Auth issues, credential problems, vulnerability fixes |
| frontend | frontend-engineer | React/Vue errors, CSS issues, client-side bugs |
| backend | backend-engineer | API errors, server logic, microservice issues |
| data | data-engineer | Database errors, migration issues, query problems |
| pipeline | devops | Workflow syntax, CI config, deployment issues |
| architecture | architect | Design issues, unclear domains, cross-cutting concerns |

## Workflow

```text
┌─────────────────────────────────────────────────────────────────┐
│ 1. FETCH                                                        │
│    gh run view <run-id> --json jobs                            │
│    → Get failure details from GitHub Actions API                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. TEAM SETUP                                                   │
│    TeamCreate → fix-ci-{run-id}                                │
│    Create diagnosis tasks in shared task list                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. DIAGNOSE (Parallel Teammates)                                │
│    Spawn diagnoser-1..N teammates (one per failure)            │
│    Each returns: { root_cause, domain, files, fix_approach }    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. FIX (Parallel Teammates)                                     │
│    Spawn fixer-{domain} teammates based on classification       │
│    Each teammate fixes issues in their domain                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. VERIFY                                                       │
│    Commit fixes, push to remote                                 │
│    Monitor CI run until complete                                │
│    If still failing → iterate from step 2                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. CLEANUP (Always runs, even on failure)                       │
│    SendMessage shutdown_request to all teammates                │
│    TeamDelete                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Execution Steps

### Step 1: Create Task Plan

```text
TaskCreate: "Fetch CI failure details" (no blockers)
TaskCreate: "Set up diagnosis team" (blockedBy: fetch)
TaskCreate: "Diagnose failures" (blockedBy: team setup)
TaskCreate: "Fix failures" (blockedBy: diagnose)
TaskCreate: "Verify CI passes" (blockedBy: fix)
TaskCreate: "Cleanup team" (blockedBy: verify)
```

### Step 2: Fetch CI Failures

```text
TaskUpdate: "Fetch CI failure details" → in_progress
```

```bash
# Get latest failed run (or use provided run-id)
gh run list --status failure --limit 1 --json databaseId,conclusion,event
gh run view <run-id> --json jobs,conclusion
```

Extract: job names, failure messages, log URLs

```text
TaskUpdate: "Fetch CI failure details" → completed
```

### Step 3: Create Team and Diagnose (Parallel Teammates)

```text
TaskUpdate: "Set up diagnosis team" → in_progress
```

```text
# Create the team
TeamCreate:
  team_name: "fix-ci-{run-id}"
  description: "CI failure resolution for run {run-id}"

# Create a diagnosis task for each failure
TaskCreate: "Diagnose: {job-1-name}" (team task)
TaskCreate: "Diagnose: {job-2-name}" (team task)
...
```

```text
TaskUpdate: "Set up diagnosis team" → completed
TaskUpdate: "Diagnose failures" → in_progress
```

Spawn one diagnoser teammate per failure **in a SINGLE message with multiple Task tool calls**:

```text
Task tool call 1:
  subagent_type: "general-purpose"
  name: "diagnoser-1"
  team_name: "fix-ci-{run-id}"
  model: "sonnet"
  prompt: |
    You are an expert debugging and performance specialist. Your capabilities:

    **Bug Investigation:**
    - Intermittent bug investigation: Race conditions, timing issues, heisenbug tracking
    - Production forensics: Log analysis, distributed tracing, failure cascade investigation
    - Memory leak detection: Heap analysis, garbage collection patterns, allocation tracking
    - Root cause analysis: Systematic investigation, evidence correlation, failure timeline

    **Performance Engineering:**
    - Performance profiling: CPU, memory, I/O profiling and bottleneck identification
    - Optimization strategies: Algorithm optimization, caching, query optimization

    ## Your Task

    Investigate CI failure in job '<job-1-name>':
    - Error output: <paste relevant log lines>
    - Job URL: <url>

    Analyze the failure, read relevant source files, and determine root cause.

    Write your diagnosis to .tmp/diagnosis-{job-1-name}.json:
    {
      "root_cause": "Brief description of what failed",
      "domain": "test|security|frontend|backend|data|pipeline|architecture",
      "files": ["list", "of", "files", "to", "fix"],
      "fix_approach": "How to fix this issue"
    }

    Then mark your assigned task as completed.

Task tool call 2:
  subagent_type: "general-purpose"
  name: "diagnoser-2"
  team_name: "fix-ci-{run-id}"
  model: "sonnet"
  prompt: |
    [Same identity preamble as above]

    ## Your Task

    Investigate CI failure in job '<job-2-name>':
    ...
```

Wait for all diagnoser teammates to complete their tasks. Read diagnosis JSON files.

```text
TaskUpdate: "Diagnose failures" → completed
```

### Step 4: Classify and Fix (Parallel Teammates)

```text
TaskUpdate: "Fix failures" → in_progress
```

Group diagnosis results by domain. Create a fix task for each domain group. Spawn one
fixer teammate per domain **in a SINGLE message with multiple Task tool calls**:

| Diagnosis Domain | Teammate Name | Prompt Specialization |
|------------------|---------------|----------------------|
| test | fixer-test | Test patterns, mock strategies, assertion fixes |
| security | fixer-security | Auth fixes, credential handling, vulnerability remediation |
| frontend | fixer-frontend | React/Vue patterns, CSS fixes, client-side debugging |
| backend | fixer-backend | API logic, server patterns, microservice fixes |
| data | fixer-data | Database queries, migration fixes, data integrity |
| pipeline | fixer-pipeline | Workflow syntax, CI config, deployment fixes |
| architecture | fixer-architecture | Design patterns, cross-cutting concerns |

```text
Task tool call:
  subagent_type: "general-purpose"
  name: "fixer-{domain}"
  team_name: "fix-ci-{run-id}"
  model: "sonnet"
  prompt: |
    You are a {domain} specialist. Fix the following CI failure(s):

    Failure 1:
    - Root cause: <from diagnosis>
    - Files to modify: <from diagnosis>
    - Approach: <from diagnosis>

    Implement the fix. Do not make unrelated changes.
    Then mark your assigned task as completed.
```

Wait for all fixer teammates to complete their tasks.

```text
TaskUpdate: "Fix failures" → completed
```

### Step 5: Commit and Verify

```text
TaskUpdate: "Verify CI passes" → in_progress
```

```bash
# Stage and commit fixes (use explicit file list from diagnosis, never git add -A)
git add <files from diagnosis JSONs>
git commit -m "fix(ci): <summary of fixes>"

# Push and monitor
git push
gh run watch
```

```text
TaskUpdate: "Verify CI passes" → completed
```

### Step 6: Cleanup (Always Runs)

**This step runs even if earlier steps fail.** Clean up the team regardless of outcome.

```text
TaskUpdate: "Cleanup team" → in_progress
```

```text
# Shutdown all teammates
SendMessage:
  type: "shutdown_request"
  recipient: "diagnoser-1"
  content: "Workflow complete, shutting down"

SendMessage:
  type: "shutdown_request"
  recipient: "diagnoser-2"
  content: "Workflow complete, shutting down"

# ... repeat for all active teammates (diagnosers + fixers)

# Delete the team
TeamDelete
```

```text
TaskUpdate: "Cleanup team" → completed
```

### Step 7: Iterate if Needed

If CI still fails after fix:

1. Return to Step 3 (create new team with incremented attempt)
2. Re-diagnose (may be different issues)
3. Deploy appropriate fix teammates
4. Continue until green

```text
TaskList: show final status of all phases
```

## Expected Output

```text
User: /fix-ci

🔍 Fetching CI failures from run #987654...
📊 Found 3 failures: lint, test:unit, build

🏗️ Creating team: fix-ci-987654

🔬 Phase 1: Diagnosis
   Spawning 3 diagnoser teammates...
   [tmux panes show diagnoser-1, diagnoser-2, diagnoser-3]

   diagnoser-1 (lint):
   └─ Domain: frontend
   └─ Cause: ESLint error in auth.ts - unused variable
   └─ Files: src/auth.ts

   diagnoser-2 (test:unit):
   └─ Domain: test
   └─ Cause: Mock outdated for new API response shape
   └─ Files: tests/api.test.ts

   diagnoser-3 (build):
   └─ Domain: pipeline
   └─ Cause: Missing dependency declaration
   └─ Files: package.json

🔧 Phase 2: Fix
   Spawning 3 fixer teammates:
   └─ fixer-frontend → src/auth.ts
   └─ fixer-test → tests/api.test.ts
   └─ fixer-pipeline → package.json

   ✓ fixer-frontend: Removed unused variable
   ✓ fixer-test: Updated mock to match new API shape
   ✓ fixer-pipeline: Added missing dependency

💾 Committed and pushed...

📊 Monitoring CI run #987655...
⏳ Running... (2 min)

✅ All CI checks passed!
🧹 Shutting down team fix-ci-987654...
🎉 CI fixed in 1 iteration
```

### Learn Mode

```text
User: /fix-ci --learn

📊 Historical Fix Patterns (last 30 days):

By Domain:
  test        │ ████████████████ │ 42% (21 fixes)
  frontend    │ ████████         │ 22% (11 fixes)
  pipeline    │ ██████           │ 16% (8 fixes)
  backend     │ ████             │ 10% (5 fixes)
  security    │ ██               │  6% (3 fixes)
  data        │ ██               │  4% (2 fixes)

Success Rate by Agent:
  test-engineer      │ 95% (20/21)
  frontend-engineer  │ 91% (10/11)
  devops             │ 88% (7/8)
  backend-engineer   │ 80% (4/5)

Common Root Causes:
  1. Outdated test mocks (18 occurrences)
  2. Lint violations (12 occurrences)
  3. Missing dependencies (6 occurrences)
```

## Notes

- Two-phase architecture separates diagnosis from fixing
- Uses TeamCreate for tmux visibility and shared task coordination
- All teammates spawned with `model: "sonnet"` to match custom agent cost/behavior
- Fixer teammates for simple domains (docs, lint, config) can use `model: "haiku"` for cost savings
- Debugger identity and capabilities embedded in diagnoser spawn prompts (prompt-based specialization)
- Domain-specific context embedded in fixer spawn prompts
- Cleanup step (shutdown + TeamDelete) always runs, even on failure
- Manual cleanup if needed: `rm -rf ~/.claude/teams/fix-ci-* ~/.claude/tasks/fix-ci-*`
- When [#24316][tc] lands, replace `subagent_type: "general-purpose"` with custom agent types
- Thinking level gap: teammates use default thinking, not ultrathink — a real limitation until #24316
- Iterates until GitHub shows all checks green

[tc]: https://github.com/anthropics/claude-code/issues/24316
