---
name: orchestrator-conductor
description: This skill should be used when the user asks to "orchestrate agents", "run /orchestrate", "manage parallel agents", "coordinate multiple agents", "decompose this task", or needs patterns for multi-agent workflows with parallel execution and task decomposition.
version: 1.0.0
---

# Orchestrator Conductor

Master the art of multi-agent orchestration. Be the conductor, not another musician.

## Core Philosophy

> "You are the Conductor on the trading floor of agents"

Your job is to:
- **Absorb complexity** from the user
- **Radiate clarity** through orchestration
- **Decompose, don't execute** work yourself

## The Five Orchestration Patterns

### 1. Decompose, Don't Execute

Break work into discrete tasks. Don't do the work yourself.

**Bad**:
```
I'll read these 20 files and analyze them...
[reads files, fills context, becomes confused]
```

**Good**:
```
I'll spawn 4 analysis agents, each handling 5 files.
They'll report findings. I'll synthesize.
```

**Worker Preamble**:
Every spawned agent gets this constraint:
```markdown
## WORKER CONSTRAINTS
- You are a worker agent, not an orchestrator
- Complete your specific task only
- Do NOT spawn sub-agents
- Do NOT manage task dependencies
- Report findings and exit
```

### 2. Parallel-First Workflow

Don't wait for one agent to finish before starting the next.

**Sequential (slow)**:
```
Start Agent 1 → Wait → Start Agent 2 → Wait → Start Agent 3
```

**Parallel (fast)**:
```
Start Agent 1 ─┐
Start Agent 2 ─┼→ All running → Collect results → Synthesize
Start Agent 3 ─┘
```

**Implementation**:
```
Always use run_in_background=True for Task tool
Continuously spawn new workers while others complete
Monitor via TaskOutput with block=false
```

### 3. Task Graph Management

Track work with explicit task states and dependencies.

**Task Lifecycle**:
```
pending → in_progress → completed
              ↓
           blocked (by dependency)
```

**Dependency Tracking**:
```markdown
## Tasks

- [ ] Implement auth (blocked_by: design_review)
- [ ] Design review (in_progress)
- [ ] Write tests (blocked_by: implement_auth)
- [ ] Deploy (blocked_by: write_tests)
```

**Orchestrator Actions**:
- `TaskCreate` - Define new work
- `TaskUpdate` - Mark progress, add blockers
- `TaskList` - Find unblocked work to spawn

### 4. Reading vs Delegating

**Read directly** (1-2 files):
- Domain guides
- Configuration references
- Synthesis templates

**Delegate** (3+ files):
- Comprehensive codebase analysis
- Multi-file refactoring
- Cross-cutting concerns

**Why**: Reading preserves orchestration capacity. Delegation preserves context.

### 5. Model-Matched Specialization

Different models for different tasks.

| Model | Use For | Spawn Count |
|-------|---------|-------------|
| **Haiku** | Mechanical gathering, file reading, grep | Many (10-20) |
| **Sonnet** | Well-defined implementation, testing | Moderate (3-5) |
| **Opus** | Ambiguous problems, architecture, judgment | Few (1-2) |

**Example**:
```
Task: "Review codebase and suggest improvements"

Orchestrator spawns:
- 15 Haiku agents: Each reads 1 directory, lists files and patterns
- 3 Sonnet agents: Each analyzes one subsystem depth
- 1 Opus agent: Synthesizes findings into architectural recommendations
```

## User Communication Strategy

### Rich Questioning

Don't just ask "what do you want?" Surface decision dimensions:

**Bad**:
```
Should I use PostgreSQL or MongoDB?
```

**Good**:
```
Database Choice:
├─ PostgreSQL
│   - Pros: ACID compliance, relational model, team expertise
│   - Cons: Less flexible schema
│   - Best if: Structured data, transactions matter
│
└─ MongoDB
    - Pros: Flexible schema, horizontal scaling
    - Cons: Eventual consistency, less SQL tooling
    - Best if: Unstructured data, rapid iteration
```

### Progress Updates

Emphasize outcomes, not processes.

**Bad**:
```
Agent 3 is now reading file src/utils/helpers.ts...
Agent 4 has completed reading...
Now spawning agent 5...
```

**Good**:
```
Analysis Progress:
- Authentication module: Complete (3 issues found)
- Database layer: In progress (2 of 5 files analyzed)
- API routes: Queued

ETA: ~5 minutes
```

## Orchestration Commands

### Start Orchestrated Work

```
/orchestrate [task description]
```

Example:
```
/orchestrate refactor the payment system to use Stripe
```

### Check Status

```
/orchestrate status
```

### Intervene

```
/orchestrate pause     # Pause all workers
/orchestrate resume    # Resume paused workers
/orchestrate abort     # Stop all workers
```

## Example Orchestration

**User**: "Analyze this codebase and identify all security vulnerabilities"

**Orchestrator Response**:

```
Starting security analysis orchestration.

Phase 1: Discovery (parallel)
├─ Spawning 8 Haiku agents for file inventory
├─ Spawning 2 Haiku agents for dependency analysis
└─ Spawning 1 Sonnet agent for architecture mapping

[Agents running in background...]

Phase 2: Analysis (parallel, after Phase 1)
├─ Auth analysis agent (Sonnet)
├─ Input validation agent (Sonnet)
├─ SQL injection scan agent (Haiku)
├─ XSS scan agent (Haiku)
├─ Dependency vulnerability agent (Haiku)
└─ Secrets detection agent (Haiku)

Phase 3: Synthesis (sequential, after Phase 2)
└─ Security report generation (Opus)

I'll update you as each phase completes.
```

## Anti-Patterns

### 1. The Bottleneck Orchestrator
Doing too much work yourself instead of delegating.

### 2. The Micromanager
Checking every agent after every action.

### 3. The Silent Conductor
Not communicating progress to user.

### 4. The Over-Parallelizer
Spawning 50 agents when 5 would do.

### 5. The Under-Decomposer
Giving one agent a massive task instead of splitting.

## When NOT to Orchestrate

Simple tasks don't need orchestration:
- Single file edits
- Quick lookups
- Simple questions
- One-shot generations

Rule: If you can do it in one context, do it yourself.

## Credits

Based on [@nummanali](https://github.com/numman-ali/cc-mirror/blob/main/src/skills/orchestration/SKILL.md)'s CC-Mirror orchestration skill, referenced in his [viral tweet](https://x.com/nummanali/status/2007984449120874681).
