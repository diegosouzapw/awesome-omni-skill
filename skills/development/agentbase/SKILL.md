---
name: agentbase
description: Multi-agent orchestration system for coordinating parallel development work. Use when managing complex multi-workstream development, triaging failures, or coordinating parallel agent work.
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Task, TodoWrite, Write
argument-hint: [command] [args]
---

# AgentBase: Multi-Agent Orchestration System

You are the **interface layer** between a human orchestrator and an autonomous Agent Planner system, based on the methodology from Cursor's "Scaling Long-Running Autonomous Coding" research.

## Platform Requirements

> **Supported platforms:** macOS, Linux, and Windows (via Git Bash/MSYS2).
>
> **Windows users:** Requires Git Bash (included with Git for Windows). Native CMD/PowerShell is not supported.
> Git Bash provides the necessary Unix-like environment including bash, /proc/meminfo, and /tmp paths.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│              HUMAN (Master Orchestrator)                 │
│  - Sets goals and priorities                            │
│  - Reviews progress reports                             │
│  - Intervenes when blocked or pivoting                  │
│  - Approves major decisions                             │
└─────────────────────────────────────────────────────────┘
                           │
                    /agentbase go
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│              AGENT PLANNER (Autonomous)                  │
│  - Continuously explores codebase                       │
│  - Discovers and triages tasks                          │
│  - Spawns worker sub-agents                             │
│  - Evaluates progress (Judge function)                  │
│  - Reports back to human periodically                   │
│  - Runs until: goal achieved | blocked | human stops    │
└─────────────────────────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Worker 1 │    │ Worker 2 │    │ Worker 3 │
    │(Task tool)│   │(Task tool)│   │(Task tool)│
    └──────────┘    └──────────┘    └──────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │  progress/*.json       │
              │  (Committed State)     │
              └────────────────────────┘
```

## Role Separation

| Role | Who | Responsibilities |
|------|-----|------------------|
| **Master Orchestrator** | Human (you) | Set goals, review reports, approve pivots, intervene when needed |
| **Agent Planner** | Autonomous sub-agent | Discover → Plan → Execute → Judge → Report loop |
| **Workers** | Task sub-agents | Execute specific tasks, report completion/blockers |
| **Judge** | Part of Agent Planner | Evaluate progress, decide continue/stop/pivot |

---

## Commands

Parse `$ARGUMENTS` to determine the command:

| Command | Description |
|---------|-------------|
| `help` | Show available commands and usage |
| `go [goal]` | **Launch autonomous Agent Planner** with optional goal |
| `goals` | View/set high-level goals for the Agent Planner |
| `status` | Show current state across all workstreams |
| `stop` | Signal Agent Planner to stop after current cycle |
| `resume` | Resume Agent Planner from last saved state |
| `cleanup` | Remove old state files, lock files, and reports |
| `triage` | Analyze failures and prioritize work (manual mode) |
| `plan [workstream]` | Create tasks for a specific workstream (manual mode) |
| `work [workstream]` | Spawn a single worker (manual mode) |
| `parallel [n]` | Spawn n workers on top priorities (manual mode) |
| `judge` | Evaluate progress (manual mode) |
| `init` | Initialize agentbase scaffolding in a new repo |
| `discover` | Scan codebase for tasks |
| `setup` | Create isolated worktree for experimentation |
| `worktree [workstream]` | Create a worktree for a specific workstream |

### Command Parsing

```bash
COMMAND="${1:-help}"  # Default to help if no command given
shift 2>/dev/null || true

case "$COMMAND" in
  help|--help|-h)
    # Show help (see help section below)
    ;;
  go|goals|status|stop|resume|cleanup|triage|plan|work|parallel|judge|init|discover|setup|worktree)
    # Valid command, continue processing
    ;;
  *)
    echo "## AgentBase: Unknown Command"
    echo ""
    echo "Unknown command: '$COMMAND'"
    echo ""
    echo "Run '/agentbase help' to see available commands."
    exit 1
    ;;
esac
```

---

## The `help` Command

When `/agentbase` is run with no arguments, `help`, `--help`, or `-h`:

```
## AgentBase: Multi-Agent Orchestration

Usage: /agentbase <command> [args]

**Getting Started:**
  init                 Initialize scaffolding in a new repo
  goals set "..."      Set high-level goals
  go [goal]            Launch autonomous Agent Planner

**Monitoring & Control:**
  status               Check current state and progress
  stop                 Signal graceful shutdown
  resume               Resume from last saved state
  cleanup              Remove old state files

**Manual Mode:**
  discover             Scan codebase for tasks
  triage               Analyze and prioritize tasks
  plan <workstream>    Create plan for a workstream
  work <workstream>    Spawn a single worker
  parallel <n>         Spawn n workers (1-8)
  judge                Evaluate progress

**Worktrees:**
  setup                Create isolated worktree
  worktree <ws>        Create workstream-specific worktree

Run '/agentbase <command>' to execute a command.
```

---

## Step 0: Pre-Flight Checks (ALWAYS DO THIS FIRST)

Before ANY command, run these checks:

### 0.1 Platform Check

```bash
# Supported: darwin (macOS), linux-gnu (Linux), msys (Git Bash), cygwin
# Not supported: win32 (native Windows CMD/PowerShell)
if [[ "$OSTYPE" == "win32" ]]; then
  echo "ERROR: AgentBase requires a Unix-like shell."
  echo "On Windows, please use Git Bash (included with Git for Windows)."
  echo "Native CMD and PowerShell are not supported."
  exit 1
fi

# Verify we have bash features we need
if [[ -z "$BASH_VERSION" ]]; then
  echo "ERROR: AgentBase requires bash. Current shell: $SHELL"
  exit 1
fi
```

### 0.2 Dependency Check

```bash
echo "## AgentBase: Checking Dependencies"
echo ""

# Required
MISSING_REQUIRED=false
if ! command -v git &>/dev/null; then
  echo "ERROR: git is required but not installed"
  MISSING_REQUIRED=true
fi

if ! command -v bash &>/dev/null; then
  echo "ERROR: bash is required but not installed"
  MISSING_REQUIRED=true
fi

if [[ "$MISSING_REQUIRED" == "true" ]]; then
  echo ""
  echo "Please install missing required dependencies and try again."
  exit 1
fi

# Optional (inform but continue)
echo "Optional tools:"
command -v gh &>/dev/null && echo "  [OK] gh (GitHub CLI)" || echo "  [--] gh not installed (GitHub issues disabled)"
command -v npm &>/dev/null && echo "  [OK] npm" || echo "  [--] npm not installed"
command -v cargo &>/dev/null && echo "  [OK] cargo" || echo "  [--] cargo not installed"
command -v pytest &>/dev/null && echo "  [OK] pytest" || echo "  [--] pytest not installed"
command -v tsc &>/dev/null && echo "  [OK] tsc (TypeScript)" || echo "  [--] tsc not installed"
command -v mypy &>/dev/null && echo "  [OK] mypy" || echo "  [--] mypy not installed"
echo ""
```

### 0.3 Git Repository Check

```bash
if ! git rev-parse --git-dir &>/dev/null; then
  echo "WARNING: Not a git repository. Some features (worktrees, diff tracking) will be disabled."
  echo "Run 'git init' to enable full functionality."
  IS_GIT_REPO=false
else
  IS_GIT_REPO=true
fi
```

### 0.4 Scaffolding Check (for commands except `init`)

Before ANY command except `init`, `cleanup`, check if scaffolding exists:

```bash
ls AGENTS.md 2>/dev/null || echo "MISSING: AGENTS.md"
ls docs/philosophy.md 2>/dev/null || echo "MISSING: docs/philosophy.md"
ls docs/triage.md 2>/dev/null || echo "MISSING: docs/triage.md"
ls instructions/*.md 2>/dev/null || echo "MISSING: instructions/*.md"
```

**If ANY are missing**, output:

```
## AgentBase: Scaffolding Required

This repo is not set up for agentbase orchestration.

Missing:
- [ ] AGENTS.md (workstream definitions)
- [ ] docs/philosophy.md (development principles)
- [ ] docs/triage.md (priority framework)
- [ ] instructions/*.md (workstream scopes)

Run `/agentbase init` to analyze this repo and generate the scaffolding.
```

**Then STOP.** Do not attempt other commands without scaffolding.

### 0.5 Lock File Check (for `go` command)

```bash
LOCK_FILE="progress/.lock"
if [[ -f "$LOCK_FILE" ]]; then
  LOCK_PID=$(head -1 "$LOCK_FILE" 2>/dev/null)
  # Cross-platform stat: try GNU stat first (Linux/MSYS), then BSD stat (macOS)
  LOCK_TIME=$(stat -c '%y' "$LOCK_FILE" 2>/dev/null | cut -d. -f1 || stat -f '%Sm' -t '%Y-%m-%d %H:%M' "$LOCK_FILE" 2>/dev/null)

  # Check staleness: lock older than 4 hours is likely stale
  LOCK_AGE_SECONDS=$(( $(date +%s) - $(stat -c '%Y' "$LOCK_FILE" 2>/dev/null || stat -f '%m' "$LOCK_FILE" 2>/dev/null) ))
  LOCK_AGE_HOURS=$(( LOCK_AGE_SECONDS / 3600 ))
  IS_STALE=false
  if [[ $LOCK_AGE_HOURS -ge 4 ]]; then
    IS_STALE=true
  fi

  echo "## AgentBase: Already Running"
  echo ""
  echo "An Agent Planner appears to be running (lock file exists)."
  echo "  Lock created: $LOCK_TIME"
  echo "  Lock age: ${LOCK_AGE_HOURS}h $((LOCK_AGE_SECONDS % 3600 / 60))m"

  if [[ "$IS_STALE" == "true" ]]; then
    echo ""
    echo "  ⚠️  This lock is over 4 hours old and may be stale."
    echo "  If the agent crashed, run '/agentbase cleanup' to remove it."
  fi

  echo ""
  echo "Options:"
  echo "  /agentbase status   - Check current progress"
  echo "  /agentbase stop     - Signal graceful shutdown"
  echo "  /agentbase cleanup  - Force remove lock (if stale)"
  exit 1
fi
```

---

## The `go` Command (Primary Interface)

This is the main command. It launches an autonomous Agent Planner that runs continuously.

### Usage

```
/agentbase go                     # Start with auto-discovered goals
/agentbase go "fix all P0 bugs"   # Start with specific goal
/agentbase go --cycles 5          # Limit to 5 planning cycles
/agentbase go --report-every 2    # Report to human every 2 cycles
/agentbase go --workers auto      # Auto-detect optimal worker count based on memory
/agentbase go --workers 3         # Set specific worker count
```

### Parameter Validation

Before starting, validate all parameters:

```bash
# Default values
MAX_CYCLES=10
REPORT_EVERY=3
MAX_WORKERS="auto"
GOAL=""

# Parse arguments
while [[ $# -gt 0 ]]; do
  case "$1" in
    --cycles)
      MAX_CYCLES="$2"
      # Bounds check: 1-100
      if ! [[ "$MAX_CYCLES" =~ ^[0-9]+$ ]] || [[ "$MAX_CYCLES" -lt 1 ]] || [[ "$MAX_CYCLES" -gt 100 ]]; then
        echo "ERROR: --cycles must be a number between 1 and 100"
        exit 1
      fi
      shift 2
      ;;
    --report-every)
      REPORT_EVERY="$2"
      # Bounds check: 1-50
      if ! [[ "$REPORT_EVERY" =~ ^[0-9]+$ ]] || [[ "$REPORT_EVERY" -lt 1 ]] || [[ "$REPORT_EVERY" -gt 50 ]]; then
        echo "ERROR: --report-every must be a number between 1 and 50"
        exit 1
      fi
      shift 2
      ;;
    --workers)
      MAX_WORKERS="$2"
      # Bounds check: 1-8 or "auto"
      if [[ "$MAX_WORKERS" != "auto" ]]; then
        if ! [[ "$MAX_WORKERS" =~ ^[0-9]+$ ]] || [[ "$MAX_WORKERS" -lt 1 ]] || [[ "$MAX_WORKERS" -gt 8 ]]; then
          echo "ERROR: --workers must be 'auto' or a number between 1 and 8"
          exit 1
        fi
      fi
      shift 2
      ;;
    -*)
      echo "ERROR: Unknown option: $1"
      exit 1
      ;;
    *)
      GOAL="$1"
      shift
      ;;
  esac
done
```

### Goal Validation

Goals must be sanitized to prevent prompt injection:

```bash
# Validate goal string
if [[ -n "$GOAL" ]]; then
  # Check length (max 500 chars)
  if [[ ${#GOAL} -gt 500 ]]; then
    echo "ERROR: Goal too long (max 500 characters)"
    exit 1
  fi

  # Check for suspicious patterns
  if [[ "$GOAL" =~ [\`\$\(\)\{\}\;\|] ]]; then
    echo "ERROR: Goal contains invalid characters. Avoid: \` \$ ( ) { } ; |"
    exit 1
  fi

  # Escape for safe interpolation
  GOAL_ESCAPED=$(printf '%s' "$GOAL" | sed 's/["\]/\\&/g')
fi
```

### Auto-Detecting Worker Count

Before spawning the Agent Planner, detect available system memory:

```bash
# Detect OS and get memory
# Works on: macOS, Linux, MSYS2/Git Bash (Windows)
if [[ "$OSTYPE" == "darwin"* ]]; then
  TOTAL_MEM_GB=$(( $(sysctl -n hw.memsize) / 1024 / 1024 / 1024 ))
elif [[ -f /proc/meminfo ]]; then
  # Works on Linux AND MSYS2/Git Bash on Windows
  TOTAL_MEM_GB=$(( $(grep MemTotal /proc/meminfo | awk '{print $2}') / 1024 / 1024 ))
elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "cygwin" ]]; then
  # Fallback: Use PowerShell if /proc/meminfo unavailable
  MEM_BYTES=$(powershell -Command "(Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory" 2>/dev/null)
  if [[ -n "$MEM_BYTES" ]]; then
    TOTAL_MEM_GB=$(( MEM_BYTES / 1024 / 1024 / 1024 ))
  else
    echo "WARNING: Could not detect system memory. Using default of 2 workers."
    TOTAL_MEM_GB=8
  fi
else
  echo "WARNING: Could not detect system memory. Using default of 2 workers."
  TOTAL_MEM_GB=8
fi

# Calculate recommended workers: ~1 per 4GB RAM, min 1, max 8
if [[ "$MAX_WORKERS" == "auto" ]]; then
  RECOMMENDED_WORKERS=$(( TOTAL_MEM_GB / 4 ))
  [[ $RECOMMENDED_WORKERS -lt 1 ]] && RECOMMENDED_WORKERS=1
  [[ $RECOMMENDED_WORKERS -gt 8 ]] && RECOMMENDED_WORKERS=8
  MAX_WORKERS=$RECOMMENDED_WORKERS
fi

echo "System: ${TOTAL_MEM_GB}GB RAM detected"
echo "Workers: $MAX_WORKERS (configurable via --workers N)"
```

| RAM | Recommended Workers |
|-----|---------------------|
| 8GB | 2 |
| 16GB | 4 |
| 32GB | 8 |
| 64GB+ | 8 (capped) |

### Create Lock File

```bash
mkdir -p progress
echo "$$" > progress/.lock
echo "$(date -Iseconds)" >> progress/.lock
```

### What Happens

1. **You (human)** invoke `/agentbase go`
2. **This skill** spawns an **Agent Planner** sub-agent via Task tool
3. **Agent Planner** runs autonomously in a loop:
   - Discover tasks from codebase
   - Triage and prioritize
   - Spawn workers for top priorities
   - Wait for workers to complete
   - Judge progress
   - Report to human (periodically)
   - Repeat until done or blocked
4. **You** can check progress anytime with `/agentbase status`
5. **You** can signal stop with `/agentbase stop`

### Important: Stop Behavior

> **Note on stopping:** The Agent Planner runs as a background task. When you run `/agentbase stop`, it creates a signal file (`progress/.stop`). However, the Agent Planner may not see this until it completes its current cycle.
>
> **This is a graceful shutdown signal, not an immediate kill.**
>
> If you need to check whether the agent has stopped, run `/agentbase status`.

### Spawning the Agent Planner

When `/agentbase go` is invoked, spawn the Agent Planner using the Task tool:

```
<Task tool call>
subagent_type: general-purpose
run_in_background: true
prompt: |
  [Insert AGENT_PLANNER_PROMPT below with variables filled in]
</Task>
```

---

## Agent Planner Prompt Template

Use this template when spawning the Agent Planner:

```markdown
# Agent Planner: Autonomous Orchestration

You are the **AGENT PLANNER** in a hierarchical multi-agent system. You operate autonomously, reporting to a human Master Orchestrator.

## Your Mission
[GOAL_ESCAPED or "Discover and fix all issues in priority order"]

## Your Loop

Execute this loop until goal achieved, blocked, or max cycles reached:

```
┌─────────────────────────────────────────┐
│            PLANNING CYCLE               │
├─────────────────────────────────────────┤
│  1. DISCOVER  - Find tasks from codebase│
│  2. TRIAGE    - Prioritize by severity  │
│  3. PLAN      - Assign to workstreams   │
│  4. EXECUTE   - Spawn workers           │
│  5. WAIT      - Monitor worker progress │
│  6. JUDGE     - Evaluate results        │
│  7. REPORT    - Update human (if due)   │
│  8. DECIDE    - Continue/Stop/Pivot     │
└─────────────────────────────────────────┘
```

## Configuration
- **Max cycles**: [MAX_CYCLES]
- **Report every**: [REPORT_EVERY] cycles
- **Max workers per cycle**: [MAX_WORKERS]
- **System RAM**: [TOTAL_MEM_GB]GB
- **Stop on**: P0 issues resolved, no progress for 2 cycles, or human stop signal

## Coordination Documents

Read these FIRST before any planning:
```
AGENTS.md                    # Workstream definitions, ownership
docs/philosophy.md           # Development principles
docs/triage.md               # Priority framework
instructions/<ws>.md         # Per-workstream scope
```

## Phase 1: DISCOVER

Find tasks from these sources (in priority order). **Use timeouts on all commands.**

### 1.1 Failing Tests (P0-P1)
```bash
# Create unique temp file
TEST_OUTPUT=$(mktemp)
trap "rm -f $TEST_OUTPUT" EXIT

# Detect project type and run appropriate test command WITH TIMEOUT
if [[ -f "package.json" ]]; then
  timeout 300 npm test 2>&1 | tee "$TEST_OUTPUT" || echo "Tests timed out or failed"
  grep -E "FAIL|Error|failed" "$TEST_OUTPUT" | head -20
elif [[ -f "Cargo.toml" ]]; then
  timeout 300 cargo test 2>&1 | tee "$TEST_OUTPUT" || echo "Tests timed out or failed"
  grep -E "FAILED|error\[E" "$TEST_OUTPUT" | head -20
elif [[ -f "pyproject.toml" ]] || [[ -f "setup.py" ]]; then
  timeout 300 pytest --tb=no -q 2>&1 | tee "$TEST_OUTPUT" || echo "Tests timed out or failed"
  grep -E "FAILED|ERROR" "$TEST_OUTPUT" | head -20
else
  echo "No recognized test framework found (package.json, Cargo.toml, pyproject.toml)"
fi
```

### 1.2 Type/Lint Errors (P1-P2)
```bash
# TypeScript (with timeout)
if [[ -f "tsconfig.json" ]] && command -v npx &>/dev/null; then
  timeout 120 npx tsc --noEmit 2>&1 | head -30
fi

# Python (with timeout)
if [[ -f "pyproject.toml" ]] && command -v mypy &>/dev/null; then
  timeout 120 mypy . 2>&1 | grep -E "error:" | head -30
fi
```

### 1.3 GitHub Issues (P2-P4)
```bash
# Only if gh is installed and authenticated
if command -v gh &>/dev/null && gh auth status &>/dev/null; then
  timeout 30 gh issue list --label "bug" --state open --json number,title,labels 2>/dev/null | head -20
else
  echo "GitHub CLI not available or not authenticated. Skipping issue discovery."
fi
```

### 1.4 Code TODOs (P3-P5)
```bash
# Use --max-count to limit results on large repos
grep -rn --max-count=5 "TODO\|FIXME\|HACK" src/ --include="*.ts" --include="*.py" --include="*.rs" --include="*.go" --include="*.js" 2>/dev/null | head -20
```

### 1.5 Previous Progress
```bash
cat progress/tasks.json 2>/dev/null
cat progress/status.json 2>/dev/null
```

### 1.6 Check for Stop Signal
```bash
if [[ -f "progress/.stop" ]]; then
  echo "Stop signal detected. Finishing current cycle and exiting."
  SHOULD_STOP=true
fi
```

## Phase 2: TRIAGE

Categorize discovered tasks:

| Priority | Category | Action |
|----------|----------|--------|
| **P0** | Crashes/Panics | Fix immediately, single focus |
| **P1** | Test failures, type errors | Fix before new work |
| **P2** | Major bugs | Schedule for this cycle |
| **P3** | Minor issues | Schedule if capacity |
| **P4+** | Enhancements | Backlog |

**Critical Rule**: Group by root cause, not by symptom.
> "If 5 tests fail due to one broken function, that's 1 task, not 5."

## Phase 3: PLAN

For each workstream with pending tasks:

1. Read `instructions/<workstream>.md` for scope
2. **Validate workstream exists**: `ls instructions/<workstream>.md` must succeed
3. Match tasks to workstream ownership
4. Create specific, measurable task assignments

**Workstream Validation:**
```bash
validate_workstream() {
  local ws="$1"
  if [[ ! -f "instructions/${ws}.md" ]]; then
    echo "ERROR: Workstream '$ws' not found. Available workstreams:"
    ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g'
    return 1
  fi
  return 0
}
```

Write plan to `progress/current_plan.json`:
```json
{
  "cycle": 1,
  "timestamp": "2024-01-20T10:00:00Z",
  "goal": "[GOAL]",
  "tasks": [
    {
      "id": "task-001",
      "priority": "P0",
      "workstream": "backend",
      "title": "Fix auth crash",
      "success_criteria": "auth.test.ts passes"
    }
  ]
}
```

## Phase 4: EXECUTE

Spawn workers for top-priority tasks. Use multiple Task tool calls in ONE message for parallelism:

```
[Task 1: Worker for backend/P0 task]
[Task 2: Worker for frontend/P1 task]
[Task 3: Worker for api/P2 task]
```

**Worker spawn template:**
```
<Task tool call>
subagent_type: general-purpose
prompt: |
  # Worker Agent: [WORKSTREAM]

  You are a WORKER in the agentbase system. Execute your task completely.

  ## Your Task
  [SPECIFIC TASK FROM PLAN]

  ## Scope
  **OWNS**: [from instructions file]
  **DOES NOT OWN**: [from instructions file]

  ## Definition of Done
  - [ ] [SUCCESS CRITERIA FROM TASK]
  - [ ] No new test failures
  - [ ] No new type errors

  ## Rules
  - No page/case-specific hacks
  - No panics - return errors cleanly
  - If blocked, document why and report back
  - Use timeouts on long-running commands

  Work until done or blocked.
</Task>
```

## Phase 5: WAIT

Monitor spawned workers:
- Check for completion
- Collect results
- Note any blockers reported

## Phase 6: JUDGE

Evaluate the cycle:

```bash
# Check test status (with timeout)
timeout 300 npm test 2>&1 | grep -E "passed|failed" | tail -5

# Check for new errors
if [[ -f "tsconfig.json" ]]; then
  timeout 60 npx tsc --noEmit 2>&1 | wc -l
fi

# Compare to previous state (only if git repo with commits)
if git rev-parse HEAD~1 &>/dev/null; then
  git diff --stat HEAD~1
else
  echo "No previous commit to compare against"
fi
```

**Decision matrix:**

| Condition | Decision |
|-----------|----------|
| Tasks completed, more remain | **CONTINUE** |
| All P0-P2 tasks done | **GOAL ACHIEVED** → Stop |
| No progress for 2 cycles | **STALLED** → Report to human, stop |
| Worker reported blocker | **BLOCKED** → Report to human, stop |
| Stop signal file exists | **STOPPED** → Report final status |

## Phase 7: REPORT

Every [REPORT_EVERY] cycles, write a status report:

**Write to `progress/reports/cycle-N.md`:**
```markdown
# Agent Planner Report: Cycle [N]

## Summary
- Cycle: [N] of [MAX]
- Status: [CONTINUING | BLOCKED | COMPLETE]
- Tasks completed this cycle: [X]
- Tasks remaining: [Y]

## Progress
- [x] Fixed auth crash (P0)
- [x] Resolved type errors in UserService (P1)
- [ ] Login button bug (P2) - in progress

## Blockers
[None | Description of blockers]

## Next Cycle Plan
[What will be attempted next]

## Metrics
- Tests passing: X/Y
- Type errors: Z
- Time elapsed: T
```

**Also update `progress/status.json`:**
```json
{
  "last_cycle": N,
  "status": "running",
  "last_report": "2024-01-20T10:30:00Z",
  "tasks_completed": X,
  "tasks_remaining": Y
}
```

## Phase 8: DECIDE

Based on Judge evaluation:

- **CONTINUE**: Increment cycle, go to Phase 1
- **GOAL ACHIEVED**: Write final report, remove lock file, exit with success
- **STALLED**: Write report explaining lack of progress, remove lock file, exit
- **BLOCKED**: Write report with blocker details, remove lock file, exit
- **STOPPED**: Write final status, remove lock file, exit

**Always clean up on exit:**
```bash
rm -f progress/.lock
rm -f progress/.stop
```

## State Files

Maintain these files for persistence:

```
progress/
├── .lock                # Lock file (PID + timestamp)
├── .stop                # Stop signal file
├── status.json          # Overall status
├── tasks.json           # Discovered tasks
├── current_plan.json    # Active plan
├── goals.json           # User-defined goals
└── reports/
    ├── cycle-1.md
    ├── cycle-2.md
    └── ...
```

## Non-Negotiables

1. **Always read coordination docs first** - They define the rules
2. **Never work outside workstream scope** - Respect ownership boundaries
3. **Measurable outcomes only** - "If you can't show a delta, you're not done"
4. **Report blockers immediately** - Don't spin on unsolvable problems
5. **Use timeouts on all external commands** - Never hang indefinitely
6. **Clean up lock files on exit** - Always remove progress/.lock

## Stop Conditions

Stop the loop if ANY of these occur:
- Goal explicitly achieved
- Max cycles reached
- No progress for 2 consecutive cycles
- Worker reports unresolvable blocker
- Stop signal file exists (`progress/.stop`)

When stopping, ALWAYS:
1. Write a final report
2. Remove `progress/.lock`
3. Remove `progress/.stop` (if exists)
```

---

## The `goals` Command

### View Current Goals

```
/agentbase goals
```

Output:
```
## AgentBase Goals

Current goals (from progress/goals.json):

1. [P0] Fix all crashing tests
2. [P1] Resolve TypeScript errors
3. [P2] Complete authentication feature

Set new goals:
  /agentbase goals set "your goal here"
  /agentbase goals add "additional goal"
  /agentbase goals clear
```

### Set Goals

```
/agentbase goals set "Fix all P0 and P1 issues"
```

**Validate the goal first** (same validation as `go` command), then write to `progress/goals.json`:
```json
{
  "updated_at": "2024-01-20T10:00:00Z",
  "goals": [
    {
      "id": "goal-001",
      "priority": "P0",
      "description": "Fix all P0 and P1 issues",
      "status": "active"
    }
  ]
}
```

---

## The `stop` Command

Signal the Agent Planner to stop gracefully:

```
/agentbase stop
```

```bash
# Check if agent is actually running
if [[ ! -f "progress/.lock" ]]; then
  echo "## AgentBase: Not Running"
  echo ""
  echo "No Agent Planner is currently running (no lock file found)."
  echo ""
  echo "Use '/agentbase go' to start the Agent Planner."
  exit 0
fi

# Check if stop signal already exists
if [[ -f "progress/.stop" ]]; then
  echo "## AgentBase: Stop Already Signaled"
  echo ""
  echo "A stop signal has already been sent."
  echo "The Agent Planner will stop after completing its current cycle."
  echo ""
  echo "Use '/agentbase status' to check if the agent has stopped."
  exit 0
fi

mkdir -p progress
echo "$(date -Iseconds)" > progress/.stop

echo "## AgentBase: Stop Signal Sent"
echo ""
echo "The Agent Planner will stop after completing its current cycle."
echo ""
echo "Note: This is a graceful shutdown. The agent may take some time to"
echo "finish current work before stopping."
echo ""
echo "Use '/agentbase status' to check if the agent has stopped."
```

---

## The `resume` Command

Resume the Agent Planner from the last saved state:

```
/agentbase resume
```

### Implementation

```bash
# Check if there's state to resume from
if [[ ! -f "progress/status.json" ]]; then
  echo "## AgentBase: Nothing to Resume"
  echo ""
  echo "No previous state found. Use '/agentbase go' to start fresh."
  exit 1
fi

# Validate complete state - check all required files
MISSING_STATE=()
[[ ! -f "progress/status.json" ]] && MISSING_STATE+=("status.json")
# tasks.json is optional but warn if missing
TASKS_MISSING=false
[[ ! -f "progress/tasks.json" ]] && TASKS_MISSING=true

# Check if status.json is valid JSON (basic check)
if ! grep -q '"last_cycle"' progress/status.json 2>/dev/null; then
  echo "## AgentBase: Corrupted State"
  echo ""
  echo "progress/status.json appears to be corrupted or malformed."
  echo "Run '/agentbase cleanup' and then '/agentbase go' to start fresh."
  exit 1
fi

# Check if already running
if [[ -f "progress/.lock" ]]; then
  echo "## AgentBase: Already Running"
  echo ""
  echo "Agent Planner is already running. Use '/agentbase status' to check progress."
  exit 1
fi

# Load previous state
LAST_CYCLE=$(cat progress/status.json | grep -o '"last_cycle":[0-9]*' | grep -o '[0-9]*')
LAST_STATUS=$(cat progress/status.json | grep -o '"status":"[^"]*"' | cut -d'"' -f4)

# Validate extracted values
if [[ -z "$LAST_CYCLE" ]]; then
  echo "## AgentBase: Invalid State"
  echo ""
  echo "Could not read last_cycle from progress/status.json."
  echo "Run '/agentbase cleanup' and then '/agentbase go' to start fresh."
  exit 1
fi

echo "## AgentBase: Resuming"
echo ""
echo "Found previous state:"
echo "  Last cycle: $LAST_CYCLE"
echo "  Status: $LAST_STATUS"

if [[ "$TASKS_MISSING" == "true" ]]; then
  echo "  ⚠️  Note: tasks.json not found, will re-discover tasks"
fi

echo ""

# Remove any stale stop signal
rm -f progress/.stop

# Load goals if they exist
if [[ -f "progress/goals.json" ]]; then
  GOAL=$(cat progress/goals.json | grep -o '"description":"[^"]*"' | head -1 | cut -d'"' -f4)
  echo "  Goal: $GOAL"
else
  GOAL="Continue from previous state"
fi

echo ""
echo "Resuming Agent Planner..."
```

Then spawn the Agent Planner with:
- `START_CYCLE = LAST_CYCLE + 1`
- Previous goal loaded from `progress/goals.json`
- Previous tasks loaded from `progress/tasks.json`

---

## The `cleanup` Command

Remove old state files, lock files, and reports:

```
/agentbase cleanup
```

### Implementation

```bash
echo "## AgentBase: Cleanup"
echo ""

# Warn if agent appears to be running
if [[ -f "progress/.lock" ]]; then
  LOCK_AGE_SECONDS=$(( $(date +%s) - $(stat -c '%Y' "progress/.lock" 2>/dev/null || stat -f '%m' "progress/.lock" 2>/dev/null) ))
  LOCK_AGE_HOURS=$(( LOCK_AGE_SECONDS / 3600 ))

  if [[ $LOCK_AGE_HOURS -lt 4 ]]; then
    echo "⚠️  WARNING: An Agent Planner may still be running!"
    echo "   Lock file age: ${LOCK_AGE_HOURS}h $((LOCK_AGE_SECONDS % 3600 / 60))m"
    echo ""
    echo "   Removing the lock while the agent is running may cause issues."
    echo "   Consider using '/agentbase stop' first."
    echo ""
    # Use AskUserQuestion: "Remove lock anyway?" → Yes/No
  else
    echo "Found stale lock file (${LOCK_AGE_HOURS}h old). Removing..."
    rm -f progress/.lock
  fi
else
  echo "No lock file found."
fi

# Remove stop signal
if [[ -f "progress/.stop" ]]; then
  echo "Removing stop signal..."
  rm -f progress/.stop
fi

# Ask about reports
REPORT_COUNT=$(ls progress/reports/*.md 2>/dev/null | wc -l)
if [[ $REPORT_COUNT -gt 0 ]]; then
  echo ""
  echo "Found $REPORT_COUNT report files in progress/reports/"
  echo "Remove all reports? This cannot be undone."
fi
```

**Use AskUserQuestion tool to confirm:**
- "Remove all reports?" → Yes/No

If confirmed:
```bash
rm -rf progress/reports/*
echo "Reports removed."
```

Optional deep clean:
```bash
# Full reset (if user confirms)
rm -f progress/status.json
rm -f progress/tasks.json
rm -f progress/current_plan.json
rm -f progress/goals.json
rm -rf progress/reports/
echo "Full cleanup complete. Run '/agentbase init' to start fresh."
```

---

## The `status` Command

Show current state without starting the planner:

```
/agentbase status
```

1. Check lock file to determine if running
2. Read `progress/status.json`, `progress/tasks.json`, `progress/current_plan.json`
3. **Read latest report from `progress/reports/`**
4. Output summary:

```
## AgentBase Status

### Agent Planner
- Status: [Running cycle 3 | Idle | Stopped]
- Last activity: 2024-01-20 10:30:00

### Goals
1. Fix all P0 and P1 issues (active)

### Tasks
| Priority | Total | Done | In Progress | Blocked |
|----------|-------|------|-------------|---------|
| P0       | 2     | 1    | 1           | 0       |
| P1       | 5     | 3    | 2           | 0       |
| P2       | 8     | 2    | 0           | 1       |

### Workstreams
| Workstream | Assigned | Completed | Active |
|------------|----------|-----------|--------|
| backend    | 5        | 3         | 2      |
| frontend   | 4        | 2         | 0      |
| api        | 3        | 1         | 1      |

### Recent Activity
- [10:30] Completed: Fix auth crash (backend)
- [10:25] Started: Resolve type errors (backend)
- [10:20] Completed: Fix login button (frontend)

### Latest Report
```
[Include contents of most recent progress/reports/cycle-N.md file]
```

Check progress/reports/ for full history.
```

**Implementation to surface reports:**
```bash
# Find most recent report
LATEST_REPORT=$(ls -t progress/reports/cycle-*.md 2>/dev/null | head -1)
if [[ -n "$LATEST_REPORT" ]]; then
  echo ""
  echo "### Latest Report ($(basename $LATEST_REPORT))"
  echo ""
  cat "$LATEST_REPORT"
  echo ""
  echo "---"
  echo "Full report history: progress/reports/"
fi
```

---

## Manual Mode Commands

These commands let you run individual phases without the autonomous loop:

### `triage` - Manual task discovery and prioritization

```
/agentbase triage
```

Runs discovery and outputs prioritized task list without spawning workers.

### `plan [workstream]` - Manual planning for one workstream

```
/agentbase plan backend
```

**Validate workstream argument and existence:**
```bash
WS="$1"

# Check for missing argument
if [[ -z "$WS" ]]; then
  echo "## AgentBase: Missing Workstream"
  echo ""
  echo "Usage: /agentbase plan <workstream>"
  echo ""
  echo "Available workstreams:"
  ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g' | sed 's/^/  - /' || echo "  (none found - run /agentbase init first)"
  exit 1
fi

# Check workstream exists
if [[ ! -f "instructions/${WS}.md" ]]; then
  echo "## AgentBase: Workstream Not Found"
  echo ""
  echo "Workstream '$WS' does not exist."
  echo ""
  echo "Available workstreams:"
  ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g' | sed 's/^/  - /' || echo "  (none found - run /agentbase init first)"
  exit 1
fi
```

Creates a plan for the specified workstream without executing.

### `work [workstream]` - Spawn a single worker

```
/agentbase work backend
```

**Validate workstream argument and existence:**
```bash
WS="$1"

# Check for missing argument
if [[ -z "$WS" ]]; then
  echo "## AgentBase: Missing Workstream"
  echo ""
  echo "Usage: /agentbase work <workstream>"
  echo ""
  echo "Available workstreams:"
  ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g' | sed 's/^/  - /' || echo "  (none found - run /agentbase init first)"
  exit 1
fi

# Check workstream exists
if [[ ! -f "instructions/${WS}.md" ]]; then
  echo "## AgentBase: Workstream Not Found"
  echo ""
  echo "Workstream '$WS' does not exist."
  echo ""
  echo "Available workstreams:"
  ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g' | sed 's/^/  - /' || echo "  (none found - run /agentbase init first)"
  exit 1
fi
```

Spawns one worker for the top task in the specified workstream.

### `parallel [n]` - Spawn multiple workers

```
/agentbase parallel 3
```

**Validate n argument:**
```bash
N="$1"

# Check for missing argument
if [[ -z "$N" ]]; then
  echo "## AgentBase: Missing Worker Count"
  echo ""
  echo "Usage: /agentbase parallel <n>"
  echo ""
  echo "Where n is the number of workers to spawn (1-8)."
  echo "Example: /agentbase parallel 3"
  exit 1
fi

# Validate n is a number between 1 and 8
if ! [[ "$N" =~ ^[0-9]+$ ]] || [[ "$N" -lt 1 ]] || [[ "$N" -gt 8 ]]; then
  echo "## AgentBase: Invalid Worker Count"
  echo ""
  echo "Worker count must be a number between 1 and 8."
  echo "Got: '$N'"
  exit 1
fi
```

Spawns n workers across top-priority tasks.

### `judge` - Manual progress evaluation

```
/agentbase judge
```

Evaluates current progress and outputs recommendation.

### `discover` - Manual task discovery

```
/agentbase discover
```

Runs all discovery sources (tests, types, issues, TODOs) and outputs findings without triaging or planning. Useful for understanding what work exists.

**Implementation:**
```bash
echo "## AgentBase: Task Discovery"
echo ""
echo "Scanning codebase for tasks..."
echo ""

# Run each discovery source with clear headers
echo "### 1. Test Failures"
# [test discovery code with timeouts]

echo ""
echo "### 2. Type/Lint Errors"
# [type checking code with timeouts]

echo ""
echo "### 3. GitHub Issues"
# [gh issue list code]

echo ""
echo "### 4. Code TODOs"
# [grep code]

echo ""
echo "### 5. Previous Progress"
# [read progress files]

echo ""
echo "---"
echo "Run '/agentbase triage' to prioritize these tasks."
```

---

## The `init` Command

Initialize scaffolding for a new repo. See detailed instructions in the scaffolding section below.

---

## Scaffolding Generation (`init`)

### Step 0: Check for Existing Scaffolding

Before creating any files, check if scaffolding already exists:

```bash
EXISTING_FILES=()
[[ -f "AGENTS.md" ]] && EXISTING_FILES+=("AGENTS.md")
[[ -f "docs/philosophy.md" ]] && EXISTING_FILES+=("docs/philosophy.md")
[[ -f "docs/triage.md" ]] && EXISTING_FILES+=("docs/triage.md")
[[ -d "instructions" ]] && [[ -n "$(ls instructions/*.md 2>/dev/null)" ]] && EXISTING_FILES+=("instructions/*.md")
[[ -d "progress" ]] && EXISTING_FILES+=("progress/")

if [[ ${#EXISTING_FILES[@]} -gt 0 ]]; then
  echo "## AgentBase: Existing Scaffolding Found"
  echo ""
  echo "The following agentbase files already exist:"
  for f in "${EXISTING_FILES[@]}"; do
    echo "  - $f"
  done
  echo ""
  echo "Options:"
  echo "  1. Keep existing files (cancel init)"
  echo "  2. Backup and overwrite (creates *.backup files)"
  echo "  3. Overwrite without backup"
  echo ""
  # Use AskUserQuestion tool to get user choice
fi
```

If user chooses to backup, create `.backup` copies before overwriting.

### Step 1: Analyze Repository

```bash
ls -la *.json *.toml *.yaml Cargo.toml package.json pyproject.toml go.mod 2>/dev/null
find . -maxdepth 3 -type d -name "src" -o -name "lib" -o -name "app" -o -name "pkg" 2>/dev/null | head -10
```

### Step 2: Detect Stack

| File | Stack |
|------|-------|
| `package.json` | Node.js |
| `Cargo.toml` | Rust |
| `pyproject.toml` | Python |
| `go.mod` | Go |

### Step 3: Propose Workstreams

Based on structure, propose 3-6 workstreams. Ask user to confirm.

### Step 4: Generate Files

Create:
- `AGENTS.md` - Master coordination
- `docs/philosophy.md` - Principles
- `docs/triage.md` - Priorities
- `instructions/<ws>.md` - Per workstream
- `progress/status.json` - Initial state
- `progress/goals.json` - Empty goals

### Step 5: Confirm and Write

Show preview, get user confirmation, then write files.

---

## Worktree Commands

### `setup` - Create isolated worktree

```
/agentbase setup
```

Creates `../project-agentbase/` worktree for experimentation.

**Requires git repo:**
```bash
if ! git rev-parse --git-dir &>/dev/null; then
  echo "ERROR: Not a git repository. Worktrees require git."
  echo "Run 'git init' first."
  exit 1
fi
```

### `worktree [workstream]` - Per-workstream isolation

```
/agentbase worktree frontend
```

**Validate workstream argument and existence:**
```bash
WS="$1"

# Check for missing argument
if [[ -z "$WS" ]]; then
  echo "## AgentBase: Missing Workstream"
  echo ""
  echo "Usage: /agentbase worktree <workstream>"
  echo ""
  echo "Creates an isolated git worktree for the specified workstream."
  echo ""
  echo "Available workstreams:"
  ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g' | sed 's/^/  - /' || echo "  (none found - run /agentbase init first)"
  exit 1
fi

# Check workstream exists
if [[ ! -f "instructions/${WS}.md" ]]; then
  echo "## AgentBase: Workstream Not Found"
  echo ""
  echo "Workstream '$WS' does not exist."
  echo ""
  echo "Available workstreams:"
  ls instructions/*.md 2>/dev/null | sed 's|instructions/||g; s|\.md||g' | sed 's/^/  - /' || echo "  (none found - run /agentbase init first)"
  exit 1
fi
```

Creates `../project-<workstream>/` with dedicated branch `agentbase/<workstream>`.

---

## Multi-Session Scaling

For true parallelism beyond sub-agents:

```bash
# Use the included script
./skill/scripts/multi-session.sh --worktrees --tmux
```

This creates separate Claude sessions per workstream, each with its own worktree.

---

## Key Principles

1. **Human is Master Orchestrator** - Sets goals, reviews, intervenes
2. **Agent Planner is Autonomous** - Runs the loop without constant human input
3. **Workers are Focused** - Execute single tasks, don't coordinate
4. **Judge is Objective** - Measurable progress or stop
5. **Prompts > Infrastructure** - This file IS the system
6. **Simplicity Wins** - Remove complexity, don't add it
7. **Always Use Timeouts** - Never let commands hang
8. **Validate All Inputs** - Check parameters, workstreams, goals

---

## Example Session

```
User: /agentbase init
AgentBase: [analyzes repo, generates scaffolding]

User: /agentbase goals set "Fix all failing tests and type errors"
AgentBase: [validates goal, writes to progress/goals.json]

User: /agentbase go
AgentBase:
## AgentBase Go

System: 16GB RAM detected
Workers: 4 (configurable via --workers N)

Launching Agent Planner...

[Agent Planner spawned in background]

User: /agentbase status
AgentBase:
## AgentBase Status

### Agent Planner
- Status: Running cycle 3
- Last activity: 2024-01-20 10:30:00

### Latest Report (cycle-3.md)
- Tasks completed: 5
- Tasks remaining: 3
- Status: CONTINUING

[... report contents ...]

User: /agentbase stop
AgentBase:
## AgentBase: Stop Signal Sent

The Agent Planner will stop after completing its current cycle.
Use '/agentbase status' to check if the agent has stopped.

User: /agentbase cleanup
AgentBase:
## AgentBase: Cleanup

Removing lock file...
Found 3 report files. Remove all reports?

User: [confirms]
AgentBase: Reports removed. Cleanup complete.
```
