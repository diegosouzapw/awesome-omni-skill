---
name: team-frontend
description: Unified team skill for frontend development team. All roles invoke this skill with --role arg. Built-in ui-ux-pro-max design intelligence. Triggers on "team frontend".
allowed-tools: TeamCreate(*), TeamDelete(*), SendMessage(*), TaskCreate(*), TaskUpdate(*), TaskList(*), TaskGet(*), Task(*), AskUserQuestion(*), TodoWrite(*), Read(*), Write(*), Edit(*), Bash(*), Glob(*), Grep(*), WebFetch(*), WebSearch(*)
---

# Team Frontend Development

Unified team skill: frontend development with built-in ui-ux-pro-max design intelligence. Covers requirement analysis, design system generation, frontend implementation, and quality assurance. All team members invoke with `--role=xxx` to route to role-specific execution.

## Architecture

```
┌──────────────────────────────────────────────────────┐
│  Skill(skill="team-frontend")                         │
│  args="<task-description>" or args="--role=xxx"       │
└──────────────────────────┬───────────────────────────┘
                           │ Role Router
                ┌──── --role present? ────┐
                │ NO                      │ YES
                ↓                         ↓
         Orchestration Mode         Role Dispatch
         (auto -> coordinator)     (route to role.md)
                │
           ┌────┴────┬───────────┬───────────┬───────────┐
           ↓         ↓           ↓           ↓           ↓
      ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
      │ coord  │ │analyst │ │architect│ │developer│ │  qa   │
      │        │ │ANALYZE-*│ │ARCH-*  │ │DEV-*   │ │QA-*   │
      └────────┘ └────────┘ └────────┘ └────────┘ └────────┘
```

## Command Architecture

Each role is organized as a folder with a `role.md` orchestrator and optional `commands/` for delegation:

```
roles/
├── coordinator/
│   ├── role.md
│   └── commands/
├── analyst/
│   ├── role.md
│   └── commands/
│       └── design-intelligence.md
├── architect/
│   ├── role.md
│   └── commands/
├── developer/
│   ├── role.md
│   └── commands/
└── qa/
    ├── role.md
    └── commands/
        └── pre-delivery-checklist.md
```

## Role Router

### Input Parsing

Parse `$ARGUMENTS` to extract `--role`. If absent -> Orchestration Mode (auto route to coordinator).

### Role Registry

| Role | File | Task Prefix | Type | Compact |
|------|------|-------------|------|---------|
| coordinator | [roles/coordinator/role.md](roles/coordinator/role.md) | (none) | orchestrator | **compressed -> must re-read** |
| analyst | [roles/analyst/role.md](roles/analyst/role.md) | ANALYZE-* | pipeline | compressed -> must re-read |
| architect | [roles/architect/role.md](roles/architect/role.md) | ARCH-* | pipeline | compressed -> must re-read |
| developer | [roles/developer/role.md](roles/developer/role.md) | DEV-* | pipeline | compressed -> must re-read |
| qa | [roles/qa/role.md](roles/qa/role.md) | QA-* | pipeline | compressed -> must re-read |

> **COMPACT PROTECTION**: Role files are execution documents, not reference material. When context compression occurs and role instructions are reduced to summaries, **you MUST immediately `Read` the corresponding role.md to reload before continuing execution**. Do not execute any Phase based on summaries.

### Dispatch

1. Extract `--role` from arguments
2. If no `--role` -> route to coordinator (Orchestration Mode)
3. Look up role in registry -> Read the role file -> Execute its phases

### Orchestration Mode

When invoked without `--role`, coordinator auto-starts. User just provides task description.

**Invocation**: `Skill(skill="team-frontend", args="<task-description>")`

**Lifecycle**:
```
User provides task description
  -> coordinator Phase 1-3: Requirement clarification + industry identification -> TeamCreate -> Create task chain
  -> coordinator Phase 4: spawn first batch workers (background) -> STOP
  -> Worker executes -> SendMessage callback -> coordinator advances next step
  -> Loop until pipeline complete -> Phase 5 report
```

**User Commands** (wake paused coordinator):

| Command | Action |
|---------|--------|
| `check` / `status` | Output execution status graph, no advancement |
| `resume` / `continue` | Check worker states, advance next step |

---

## Shared Infrastructure

The following templates apply to all worker roles. Each role.md only needs to write **Phase 2-4** role-specific logic.

### Worker Phase 1: Task Discovery (shared by all workers)

Every worker executes the same task discovery flow on startup:

1. Call `TaskList()` to get all tasks
2. Filter: subject matches this role's prefix + owner is this role + status is pending + blockedBy is empty
3. No tasks -> idle wait
4. Has tasks -> `TaskGet` for details -> `TaskUpdate` mark in_progress

**Resume Artifact Check** (prevent duplicate output after resume):
- Check whether this task's output artifact already exists
- Artifact complete -> skip to Phase 5 report completion
- Artifact incomplete or missing -> normal Phase 2-4 execution

### Worker Phase 5: Report (shared by all workers)

Standard reporting flow after task completion:

1. **Message Bus**: Call `mcp__ccw-tools__team_msg` to log message
   - Parameters: operation="log", team=**<session-id>**, from=<role>, to="coordinator", type=<message-type>, summary="[<role>] <summary>", ref=<artifact-path>
   - **CLI fallback**: When MCP unavailable -> `ccw team log --team <session-id> --from <role> --to coordinator --type <type> --summary "[<role>] ..." --json`
   - **Note**: `team` must be session ID (e.g., `FES-xxx-date`), NOT team name. Extract from `Session:` field in task description.
2. **SendMessage**: Send result to coordinator (content and summary both prefixed with `[<role>]`)
3. **TaskUpdate**: Mark task completed
4. **Loop**: Return to Phase 1 to check next task

### Wisdom Accumulation (all roles)

Cross-task knowledge accumulation. Coordinator creates `wisdom/` directory at session initialization.

**Directory**:
```
<session-folder>/wisdom/
├── learnings.md      # Patterns and insights
├── decisions.md      # Architecture and design decisions
├── conventions.md    # Codebase conventions
└── issues.md         # Known risks and issues
```

**Worker Load** (Phase 2): Extract `Session: <path>` from task description, read wisdom directory files.
**Worker Contribute** (Phase 4/5): Write this task's discoveries to corresponding wisdom files.

### Role Isolation Rules

#### Output Tagging

All outputs must carry `[role_name]` prefix.

#### Coordinator Isolation

| Allowed | Forbidden |
|---------|-----------|
| Requirement clarification (AskUserQuestion) | Direct code writing/modification |
| Create task chain (TaskCreate) | Calling implementation subagents |
| Dispatch tasks to workers | Direct analysis/testing/review |
| Monitor progress (message bus) | Bypassing workers |
| Report results to user | Modifying source code |

#### Worker Isolation

| Allowed | Forbidden |
|---------|-----------|
| Process tasks with own prefix | Process tasks with other role prefixes |
| SendMessage to coordinator | Communicate directly with other workers |
| Use tools declared in Toolbox | Create tasks for other roles (TaskCreate) |
| Delegate to commands/ files | Modify resources outside own responsibility |

### Message Bus (All Roles)

Every SendMessage **before**, must call `mcp__ccw-tools__team_msg` to log.

**Message types by role**:

| Role | Types |
|------|-------|
| coordinator | `task_unblocked`, `sync_checkpoint`, `fix_required`, `error`, `shutdown` |
| analyst | `analyze_ready`, `analyze_progress`, `error` |
| architect | `arch_ready`, `arch_revision`, `arch_progress`, `error` |
| developer | `dev_complete`, `dev_progress`, `error` |
| qa | `qa_passed`, `qa_result`, `fix_required`, `error` |

### Shared Memory

Cross-role accumulated knowledge stored in `shared-memory.json`:

| Field | Owner | Content |
|-------|-------|---------|
| `design_intelligence` | analyst | ui-ux-pro-max output |
| `design_token_registry` | architect | colors, typography, spacing, shadows |
| `component_inventory` | architect | Component specs |
| `style_decisions` | architect | Design system decisions |
| `qa_history` | qa | QA audit results |
| `industry_context` | analyst | Industry-specific rules |

Each role reads in Phase 2, writes own fields in Phase 5.

---

## Pipeline Architecture

### Three Pipeline Modes

```
page (single page - linear):
  ANALYZE-001 -> ARCH-001 -> DEV-001 -> QA-001

feature (multi-component feature - with architecture review):
  ANALYZE-001 -> ARCH-001(tokens+structure) -> QA-001(architecture-review)
  -> DEV-001(components) -> QA-002(code-review)

system (full frontend system - dual-track parallel):
  ANALYZE-001 -> ARCH-001(tokens) -> QA-001(token-review)
  -> [ARCH-002(components) || DEV-001(tokens)](parallel, blockedBy QA-001)
  -> QA-002(component-review) -> DEV-002(components) -> QA-003(final)
```

### Generator-Critic Loop (developer <-> qa)

Developer and qa iterate to ensure code quality and design compliance:

```
┌──────────┐     DEV artifact        ┌──────────┐
│ developer│ ─────────────────────>  │    qa    │
│(Generator)│                        │ (Critic) │
│          │  <───────────────────── │          │
└──────────┘   QA feedback           └──────────┘
               (max 2 rounds)

Convergence: qa.score >= 8 && qa.critical_count === 0
```

### Consulting Pattern (developer -> analyst)

Developer can request design decision consultation via coordinator:

```
developer -> coordinator: "Need design decision consultation"
coordinator -> analyst: Create ANALYZE-consult task
analyst -> coordinator: Design recommendation
coordinator -> developer: Forward recommendation
```

### Cadence Control

**Beat model**: Event-driven, each beat = coordinator wake -> process -> spawn -> STOP.

```
Beat Cycle (single beat)
═══════════════════════════════════════════════════════════
  Event                   Coordinator              Workers
───────────────────────────────────────────────────────────
  callback/resume ──> ┌─ handleCallback ─┐
                      │  mark completed   │
                      │  check pipeline   │
                      ├─ handleSpawnNext ─┤
                      │  find ready tasks │
                      │  spawn workers ───┼──> [Worker A] Phase 1-5
                      │  (parallel OK)  ──┼──> [Worker B] Phase 1-5
                      └─ STOP (idle) ─────┘         │
                                                     │
  callback <─────────────────────────────────────────┘
  (next beat)              SendMessage + TaskUpdate(completed)
═══════════════════════════════════════════════════════════
```

**Pipeline beat view**:

```
Page mode (4 beats, strictly serial)
──────────────────────────────────────────────────────────
Beat  1         2         3         4
      │         │         │         │
      ANALYZE -> ARCH -> DEV -> QA
      ▲                            ▲
   pipeline                     pipeline
    start                        done

A=ANALYZE  ARCH=architect  D=DEV  Q=QA

Feature mode (5 beats, with architecture review gate)
──────────────────────────────────────────────────────────
Beat  1         2         3         4         5
      │         │         │         │         │
      ANALYZE -> ARCH -> QA-1 -> DEV -> QA-2
                          ▲               ▲
                   arch review      code review

System mode (7 beats, dual-track parallel)
──────────────────────────────────────────────────────────
Beat  1         2       3       4              5       6       7
      │         │       │  ┌────┴────┐         │       │       │
      ANALYZE -> ARCH-1 -> QA-1 -> ARCH-2 || DEV-1 -> QA-2 -> DEV-2 -> QA-3
                                   ▲                              ▲
                              parallel window                 final check
```

**Checkpoints**:

| Trigger | Location | Behavior |
|---------|----------|----------|
| Architecture review gate | QA-001 (arch review) complete | Pause if critical issues, wait for architect revision |
| GC loop limit | developer <-> qa max 2 rounds | Exceed rounds -> stop iteration, report current state |
| Pipeline stall | No ready + no running | Check missing tasks, report to user |

**Stall Detection** (coordinator `handleCheck` executes):

| Check | Condition | Resolution |
|-------|-----------|------------|
| Worker no response | in_progress task no callback | Report waiting task list, suggest user `resume` |
| Pipeline deadlock | no ready + no running + has pending | Check blockedBy dependency chain, report blocking point |
| GC loop exceeded | DEV/QA iteration > max_rounds | Terminate loop, output latest QA report |

### Task Metadata Registry

| Task ID | Role | Phase | Dependencies | Description |
|---------|------|-------|-------------|-------------|
| ANALYZE-001 | analyst | analysis | (none) | Requirement analysis + design intelligence via ui-ux-pro-max |
| ARCH-001 | architect | design | ANALYZE-001 | Design token system + component architecture |
| ARCH-002 | architect | design | QA-001 (system mode) | Component specs refinement |
| DEV-001 | developer | impl | ARCH-001 or QA-001 | Frontend component/page implementation |
| DEV-002 | developer | impl | QA-002 (system mode) | Component implementation from refined specs |
| QA-001 | qa | review | ARCH-001 or DEV-001 | Architecture review or code review |
| QA-002 | qa | review | DEV-001 | Code review (feature/system mode) |
| QA-003 | qa | review | DEV-002 (system mode) | Final quality check |

---

## Coordinator Spawn Template

When coordinator spawns workers, use background mode (Spawn-and-Stop):

```
Task({
  subagent_type: "general-purpose",
  description: "Spawn <role> worker",
  team_name: <team-name>,
  name: "<role>",
  run_in_background: true,
  prompt: `You are team "<team-name>" <ROLE>.

## Primary Directive
All your work must be executed through Skill to load role definition:
Skill(skill="team-frontend", args="--role=<role>")

Current requirement: <task-description>
Session: <session-folder>

## Role Guidelines
- Only process <PREFIX>-* tasks, do not execute other role work
- All output prefixed with [<role>] identifier
- Only communicate with coordinator
- Do not use TaskCreate for other roles
- Call mcp__ccw-tools__team_msg before every SendMessage

## Workflow
1. Call Skill -> load role definition and execution logic
2. Follow role.md 5-Phase flow
3. team_msg + SendMessage results to coordinator
4. TaskUpdate completed -> check next task`
})
```

## ui-ux-pro-max Integration

### Design Intelligence Engine

Analyst role invokes ui-ux-pro-max via Skill to obtain industry design intelligence:

| Action | Invocation |
|--------|------------|
| Full design system recommendation | `Skill(skill="ui-ux-pro-max", args="<industry> <keywords> --design-system")` |
| Domain search (UX, typography, color) | `Skill(skill="ui-ux-pro-max", args="<query> --domain <domain>")` |
| Tech stack guidance | `Skill(skill="ui-ux-pro-max", args="<query> --stack <stack>")` |
| Persist design system (cross-session) | `Skill(skill="ui-ux-pro-max", args="<query> --design-system --persist -p <projectName>")` |

**Supported Domains**: product, style, typography, color, landing, chart, ux, web
**Supported Stacks**: html-tailwind, react, nextjs, vue, svelte, shadcn, swiftui, react-native, flutter

**Fallback**: If ui-ux-pro-max skill not installed, degrade to LLM general design knowledge. Suggest installation: `/plugin install ui-ux-pro-max@ui-ux-pro-max-skill`

## Session Directory

```
.workflow/.team/FE-<slug>-<YYYY-MM-DD>/
├── team-session.json           # Session state
├── shared-memory.json          # Cross-role accumulated knowledge
├── wisdom/                     # Cross-task knowledge
│   ├── learnings.md
│   ├── decisions.md
│   ├── conventions.md
│   └── issues.md
├── analysis/                   # Analyst output
│   ├── design-intelligence.json
│   └── requirements.md
├── architecture/               # Architect output
│   ├── design-tokens.json
│   ├── component-specs/
│   │   └── <component-name>.md
│   └── project-structure.md
├── qa/                         # QA output
│   └── audit-<NNN>.md
└── build/                      # Developer output
    ├── token-files/
    └── component-files/
```

## Error Handling

| Scenario | Resolution |
|----------|------------|
| Unknown --role value | Error with available role list |
| Missing --role arg | Orchestration Mode -> auto route to coordinator |
| Role file not found | Error with expected path (roles/<name>/role.md) |
| QA score < 6 over 2 GC rounds | Coordinator reports to user |
| Dual-track sync failure | Fallback to single-track sequential execution |
| ui-ux-pro-max skill not installed | Degrade to LLM general design knowledge, show install command |
| DEV cannot find design files | Wait for sync point or escalate to coordinator |
