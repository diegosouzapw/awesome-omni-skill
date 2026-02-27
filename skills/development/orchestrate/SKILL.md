---
name: orchestrate
description: Multi-Model Orchestration - Guide for orchestrating multi-model agents
version: 2.1.0
author: CC Orchestrator
tags: [orchestration, multi-model, parallel, workflow]
---

# Multi-Model Orchestration Guide

Follow this guide to orchestrate multi-model agents effectively.

---

## Phase 0: Intent Gate (BLOCKING)

### Step 0: Request Classification

```
User request received
    ↓
[Classify request type]
├─ Trivial        → Use tools directly (no agent needed)
├─ Explicit       → Execute as instructed
├─ Exploratory    → Run scout/index in parallel
├─ Open-ended     → Go to Phase 1 (codebase evaluation needed)
├─ Implementation → Delegate to coding agents (Pattern F for multi-file)
├─ Research       → Run index first
├─ Design/Review  → Consult arch
└─ Ambiguous      → Ask only 1 clarifying question
```

### Step 1: Ambiguity Check

```
├─ Single interpretation      → Proceed
├─ Multiple + similar effort  → Make reasonable assumption, proceed
├─ 2x+ effort difference      → Must ask
└─ Missing key information    → Must ask
```

### Step 2: Verification Checklist

- [ ] Checked implicit assumptions?
- [ ] Is the search scope clear?
- [ ] Selected the appropriate agent?

---

## Phase 1: Codebase Evaluation (Open-ended tasks only)

### State Classification Matrix

| State | Signal | Action |
|-------|--------|--------|
| **Disciplined** | Consistent patterns, config exists | Strictly follow existing style |
| **Transition** | Mixed patterns | Ask "Which pattern to follow?" |
| **Legacy** | Inconsistent | "Suggest: Apply [X]?" |
| **Greenfield** | New project | Apply modern best practices |

---

## Phase 2: Execution

### 2A: Exploration & Research

**Tool Selection Priority:**

| Resource | Cost | When to Use |
|----------|------|-------------|
| Grep, Glob, Read | FREE | Clear scope, simple search |
| `scout` agent | FREE | Codebase exploration (Task tool) |
| `index` agent | LOW | External docs, API research (Task tool) |
| `cco-frontend-developer` | FREE | Frontend implementation (React, CSS, layouts) (Task tool) |
| `cco-backend-architect` | FREE | Backend implementation (APIs, services) (Task tool) |
| `cco-database-architect` | FREE | DB schema, migrations (Task tool) |
| `cco-cloud-architect` | FREE | Infrastructure, deployment configs (Task tool) |
| `cco-docs-architect` | FREE | Technical docs from code (quill native alternative) (Task tool) |
| `cco-architect-review` | FREE | Architecture review (arch native alternative) (Task tool) |
| `general-purpose` | FREE | Any coding task, full tool access (Task tool) |
| `canvas` | MODERATE | UI/UX, styling (Gemini 3) |
| `quill` | MODERATE | Technical documentation (Gemini 3) |
| `lens` | MODERATE | Image/PDF analysis (Gemini 3) |
| `arch` | EXPENSIVE | Architecture, code review (GPT-5.2) |

**Agent Routing Rules (CRITICAL):**

```
┌─────────────────────────────────────────────────────────────┐
│ CLAUDE CODE NATIVE AGENTS (Task tool) - FREE/LOW-COST       │
│                                                             │
│   scout  → Task(subagent_type="scout", prompt="...")     │
│              FREE: Codebase exploration, file/function search│
│                                                             │
│   index  → Task(subagent_type="index", prompt="...")     │
│              LOW-COST: External research (WebSearch + Fetch)│
│                                                             │
│ CODING AGENTS (Task tool) - FREE                            │
│   Can Edit/Write/Bash - actual code modification capable    │
│                                                             │
│   cco-frontend-developer                                        │
│     → Task(subagent_type="cco-frontend-developer", prompt="...") │
│       React, Next.js, CSS, responsive layouts, components   │
│                                                             │
│   cco-backend-architect                                         │
│     → Task(subagent_type="cco-backend-architect", prompt="...")  │
│       API design, microservices, server logic               │
│                                                             │
│   cco-database-architect                                        │
│     → Task(subagent_type="cco-database-architect", prompt="...") │
│       Schema modeling, migrations, query optimization       │
│                                                             │
│   cco-cloud-architect                                           │
│     → Task(subagent_type="cco-cloud-architect", prompt="...")    │
│       AWS/Azure/GCP, IaC, deployment configs                │
│                                                             │
│   cco-docs-architect                                            │
│     → Task(subagent_type="cco-docs-architect", prompt="...")     │
│       Technical docs from codebase (free quill alternative) │
│                                                             │
│   cco-architect-review                                          │
│     → Task(subagent_type="cco-architect-review", prompt="...")   │
│       Architecture review, clean arch (free arch alternative)│
│                                                             │
│   general-purpose                                           │
│     → Task(subagent_type="general-purpose", prompt="...")    │
│       Any task, full tool access (Edit, Write, Bash)        │
│                                                             │
│ MCP AGENTS (background_task) - PAID                         │
│                                                             │
│   arch   → background_task(agent="arch")   // OpenAI GPT-5.2│
│   canvas → background_task(agent="canvas") // Google Gemini │
│   quill  → background_task(agent="quill")  // Google Gemini │
│   lens   → background_task(agent="lens")   // Google Gemini │
└─────────────────────────────────────────────────────────────┘
```

**Parallel Execution Pattern:**

```bash
# Correct: Mixed parallel execution (Native + MCP)
Task(subagent_type="scout", prompt="Find auth patterns")  // FREE
background_task(agent="arch", prompt="Review security")   // PAID (GPT-5.2)
// Both run in parallel - continue immediately

# Wrong: MCP doesn't support scout/index
background_task(agent="scout", ...)  // ERROR - use Task tool instead
background_task(agent="index", ...)  // ERROR - use Task tool instead
```

**Exploration Stop Conditions:**
- Sufficient context acquired
- Same information appearing repeatedly
- No new info after 2 explorations
- Direct answer found

### 2B: Implementation

**Todo Creation Rules:**
- 2+ step task → Must create
- Ambiguous scope → Must create (clarifies thinking)
- User requests multiple items → Must create

**Workflow:**
1. Create immediately (no announcement)
2. Mark current task `in_progress`
3. Mark `completed` immediately upon completion
4. No batching (one at a time)

### 2C: Failure Recovery

**On 3 consecutive failures:**
1. Stop all edits
2. Restore to last known good state (git checkout, etc.)
3. Document what was attempted
4. Consult `arch`
5. Explain situation to user

---

## Phase 3: Completion Verification

**Checklist:**
- [ ] All todos completed
- [ ] No type errors (npx tsc --noEmit)
- [ ] Build passes (if exists)
- [ ] User request fully satisfied

**Cleanup:**
```
background_cancel(all=true)  // Cancel all background tasks
```

---

## Agent Selection Guide

### Agent Role Table

**Claude Code Native Agents (Task tool) - FREE/LOW-COST:**

| Agent | Invocation | Purpose | Triggers |
|-------|------------|---------|----------|
| `scout` | `Task(subagent_type="scout")` | Codebase exploration, file/function search | "where is", "find", "how does X work" |
| `index` | `Task(subagent_type="index")` | External docs, APIs, best practices (low-cost) | library names, "how to", tutorials |

**Claude Code Coding Agents (Task tool) - FREE:**

| Agent | Invocation | Purpose | Triggers |
|-------|------------|---------|----------|
| `cco-frontend-developer` | `Task(subagent_type="cco-frontend-developer")` | React, Next.js, CSS, responsive layouts | "build UI", "create component", "style", "page layout" |
| `cco-backend-architect` | `Task(subagent_type="cco-backend-architect")` | API design, microservices, server logic | "create API", "endpoint", "service layer", "middleware" |
| `cco-database-architect` | `Task(subagent_type="cco-database-architect")` | Schema modeling, migrations, queries | "schema", "migration", "database", "model" |
| `cco-cloud-architect` | `Task(subagent_type="cco-cloud-architect")` | AWS/Azure/GCP, IaC, deployment | "deploy", "infrastructure", "CI/CD", "Docker" |
| `cco-docs-architect` | `Task(subagent_type="cco-docs-architect")` | Technical docs from code (free quill alternative) | "document", "guide", "technical docs" |
| `cco-architect-review` | `Task(subagent_type="cco-architect-review")` | Architecture review (free arch alternative) | "review architecture", "design review", "clean arch" |
| `general-purpose` | `Task(subagent_type="general-purpose")` | Any coding task, full tool access | Catch-all for implementation tasks |

**MCP Agents (background_task) - PAID:**

| Agent | Model | Purpose | Cost | Triggers |
|-------|-------|---------|------|----------|
| `arch` | GPT-5.2 | Architecture, strategy, code review | High | Design decisions, complex problems |
| `canvas` | Gemini 3 | UI/UX, styling, components | Medium | Visual changes, CSS, animations |
| `quill` | Gemini 3 | Technical docs, README, API docs | Medium | Documentation requests |
| `lens` | Gemini 3 | Image, PDF, screenshot analysis | Medium | Visual asset analysis |

### Delegation Table

| Domain | Delegate To | Trigger Keywords |
|--------|-------------|------------------|
| Codebase exploration | `scout` (native) | find, where, search, structure |
| External Research | `index` (native, low-cost) | library names, API, "how to", best practices |
| Frontend UI/UX | `canvas` (MCP) | style, color, animation, layout, responsive |
| Architecture | `arch` (MCP) | design, structure, pattern selection, tradeoffs |
| Code Review | `arch` (MCP) | review, inspect, improvements |
| Documentation | `cco-docs-architect` (native) first, `quill` (MCP) fallback | README, docs, guide, API docs |
| Image/PDF | `lens` (MCP) | screenshot, image, PDF, diagram |
| Frontend coding | `cco-frontend-developer` (native) | build, create component, React, page, layout |
| Backend coding | `cco-backend-architect` (native) | API, endpoint, service, server, middleware |
| Database work | `cco-database-architect` (native) | schema, migration, model, query, database |
| Infrastructure | `cco-cloud-architect` (native) | deploy, Docker, CI/CD, infra, cloud |
| Technical docs (free) | `cco-docs-architect` (native) | document, guide (cost-saving over quill) |
| Architecture review (free) | `cco-architect-review` (native) | review, inspect (cost-saving over arch) |
| Blockchain/Web3 | `cco-blockchain-developer` (native) | blockchain, smart contract, Web3, DeFi |
| GraphQL APIs | `cco-graphql-architect` (native) | GraphQL, federation, schema, resolver |
| Hybrid/Multi-cloud | `cco-hybrid-cloud-architect` (native) | hybrid cloud, multi-cloud, edge computing |
| Kubernetes/GitOps | `cco-kubernetes-architect` (native) | kubernetes, k8s, GitOps, helm, ArgoCD |
| General coding | `general-purpose` (native) | implement, code, build (catch-all) |

### Frontend Delegation Gate (BLOCKING)

**Must delegate when visual keywords detected:**
```
style, className, tailwind, color, background, border,
shadow, margin, padding, width, height, flex, grid,
animation, transition, hover, responsive, CSS
```

| Change Type | Examples | Action |
|-------------|----------|--------|
| Visual/UI | Colors, spacing, animations | **Must delegate** |
| Pure Logic | API calls, state management | Handle directly |
| Mixed | Both visual + logic | Separate and handle |

---

## Hybrid Swarm Operating Policy (BLOCKING)

**Fixed policy for `/orchestrate`:**
- Optimize for output quality first, then speed/cost.
- Native coding agents are the default for implementation.
- MCP agents are quality boosters (design/review/visual/document polish), not the default path.

**Routing defaults:**
- Implementation-heavy (multi-file edits): `cco-frontend-developer`, `cco-backend-architect`, `cco-database-architect`, `cco-cloud-architect`, `general-purpose`
- Architecture decisions/tradeoffs: `arch` allowed
- Visual/UI quality validation: `canvas` allowed
- Documentation quality: `cco-docs-architect` first, `quill` fallback only when quality requires it
- Final quality review: exactly one of `cco-architect-review` or `arch`

**MCP soft guard:**
- Recommended MCP usage: up to 2 calls per request (`design` 1 + `review` 1)
- Do not hard-fail above 2 calls; report each extra call with one-line reason and expected quality gain

---

## Delegation Prompt Structure

**Required 12 Sections (Hybrid Swarm Contract):**

```markdown
## OPERATING_POLICY
- Quality-first delivery
- Native coding agents by default
- MCP as selective quality booster

## ROUTING_RULES
- Implementation-heavy tasks → native coding agents
- Design/review/visual quality tasks → MCP allowed
- Final quality review → `cco-architect-review` or `arch` (exactly one)

## TASK
[Atomic goal - single action only]

## EXPECTED
[Specific deliverable + success criteria]

## REQUIRED_TOOLS
[Tool whitelist to use]

## MUST_DO
[Explicit requirements]

## MUST_NOT_DO
[Forbidden actions - prevent rogue behavior]

## CONTEXT
[File paths, existing patterns, constraints]

## QUALITY_GATE
- Gate 1: Interface compatibility (I/O contracts, typed boundaries)
- Gate 2: Integration conflicts (file overlap/conflicts)
- Gate 3: Requirement coverage (must-have checklist)
- Gate 4: Risk & omission report

## MCP_SOFT_GUARD
- Recommended MCP calls <= 2 per request
- If exceeded, add one-line reason + expected quality gain per extra MCP call

## REPORT_FORMAT
- Agent Mix (native/MCP ratio)
- File ownership by agent
- Parallel execution summary (groups + elapsed time)
- Quality gate pass/fail
- MCP usage reasons and count

## SUCCESS_CRITERIA
[Completion verification criteria]
```

---

## Execution Patterns

### Pattern A: Exploration + Implementation

```text
1. Task(subagent_type="scout", prompt="Find similar patterns")  // FREE
2. Start basic implementation simultaneously
3. Enhance implementation with exploration results
```

### Pattern B: Research + Implementation

```text
1. Task(subagent_type="index", prompt="Find best practices")  // LOW-COST
2. Start basic implementation simultaneously
3. Apply researched patterns
```

### Pattern C: Design Review

```text
1. Write draft
2. background_task(arch, "Review architecture...") // GPT-5.2
3. Incorporate feedback
```

### Pattern D: Multi-perspective Collection

```text
1. Task(subagent_type="scout", prompt="Analyze codebase")    // FREE - Parallel
2. background_task(arch, "Architecture perspective...")      // GPT-5.2 - Parallel
3. background_task(canvas, "UX perspective...")              // Gemini - Parallel
4. Integrate all results
```

### Pattern E: Complex Implementation

```text
1. Task(subagent_type="scout", prompt="Understand patterns") // FREE
2. Confirm design direction with arch (MCP)
3. Proceed with implementation
4. Code review with arch (MCP)
```

### Pattern F: Parallel Implementation

```text
1. Task(subagent_type="scout", prompt="Analyze codebase structure")     // FREE
2. background_task(arch, "Design implementation plan...")                // GPT-5.2
   OR Task(subagent_type="cco-architect-review", prompt="Design plan...")    // FREE alternative
3. Collect design via background_output(task_id, block=true)
4. Parallel coding delegation (single message, multiple Task calls):
   Task(subagent_type="cco-frontend-developer", prompt="[design context] Implement UI...")
   Task(subagent_type="cco-backend-architect", prompt="[design context] Implement API...")
   // Each agent has Read/Grep/Glob + Edit/Write/Bash access
5. Collect all results
6. Verify integration, fix conflicts if any
```

**Context Passing (CRITICAL):**
```markdown
Include design output in each worker prompt using delegation structure:

## CONTEXT
[Paste design plan output here]

## YOUR SCOPE
[Only files/directories this agent should touch - use absolute paths]

## MUST_NOT_DO
- Do NOT modify files outside your scope
- Do NOT duplicate work assigned to other agents
```

**File Scope Assignment (prevent conflicts):**
```text
├─ cco-frontend-developer      → src/components/, src/pages/, src/styles/
├─ cco-backend-architect       → src/api/, src/services/, src/middleware/
├─ cco-database-architect      → src/models/, src/migrations/, prisma/
├─ cco-cloud-architect         → infra/, docker/, .github/workflows/
├─ cco-blockchain-developer    → contracts/, src/web3/, hardhat/
├─ cco-graphql-architect       → src/graphql/, schema/, resolvers/
├─ cco-hybrid-cloud-architect  → infra/multi-cloud/, terraform/
├─ cco-kubernetes-architect    → k8s/, helm/, argocd/
├─ cco-docs-architect          → docs/
└─ cco-architect-review        → Analysis only (no file modifications)
```

**Execution Constraints (BLOCKING):**
- Delegate parallelizable worker tasks in the same turn/message.
- Assign absolute-path scope per worker.
- Include scope guardrails in `MUST_NOT_DO` (no out-of-scope edits, no duplicate ownership).
- For each MCP call, log one-line reason + expected quality gain.

### Pattern F-iso: Parallel Implementation with Git Worktree Isolation (Experimental)

Use when multiple agents need to modify overlapping files, or when safe rollback is required:

```text
1-3. Same as Pattern F
4. Create git worktrees (main session runs via Bash):
   git worktree add ../wt-frontend -b swarm/frontend
   git worktree add ../wt-backend -b swarm/backend
5. Delegate with absolute paths (Task tool has no cwd parameter):
   Task(subagent_type="cco-frontend-developer", prompt="
     Working directory: {abs_path}/wt-frontend
     Use absolute paths based on {abs_path}/wt-frontend for all file ops.
     Prefix all Bash commands with: cd {abs_path}/wt-frontend &&
     [design context]...")
   Task(subagent_type="cco-backend-architect", prompt="
     Working directory: {abs_path}/wt-backend
     Use absolute paths based on {abs_path}/wt-backend for all file ops.
     Prefix all Bash commands with: cd {abs_path}/wt-backend &&
     [design context]...")
6. Merge results:
   git merge swarm/frontend
   git merge swarm/backend
7. Cleanup:
   git worktree remove ../wt-frontend
   git worktree remove ../wt-backend
   git branch -d swarm/frontend swarm/backend
```

**Limitations:**
- Task tool has no `cwd` parameter (Feature Request #12748)
- Workaround: absolute paths in prompt + `cd path &&` prefix for Bash
- May need `additionalDirectories` in settings for worktree access

---

## Cost Optimization

```plaintext
FREE (Claude Code Task tool):
├─ Simple search          → Grep, Glob, Read (direct tools)
├─ Codebase exploration   → Task(subagent_type="scout")
├─ Frontend coding        → Task(subagent_type="cco-frontend-developer")
├─ Backend coding         → Task(subagent_type="cco-backend-architect")
├─ Database work          → Task(subagent_type="cco-database-architect")
├─ Infrastructure         → Task(subagent_type="cco-cloud-architect")
├─ Technical docs (free)  → Task(subagent_type="cco-docs-architect")
├─ Architecture review    → Task(subagent_type="cco-architect-review")
└─ General tasks          → Task(subagent_type="general-purpose")

LOW-COST (Claude Code Task tool):
└─ External research      → Task(subagent_type="index")

PAID (MCP external APIs):
├─ Architecture decisions → arch (GPT-5.2, expensive)
├─ UI/UX work            → canvas (Gemini, moderate)
├─ Documentation         → quill (Gemini, moderate)
└─ Image/PDF analysis    → lens (Gemini, moderate)
```

**Principles:**
1. Always try FREE tools first (Grep, Glob, direct Read)
2. Use `scout` first (FREE), then `index` for external research (LOW-COST)
3. Only use MCP agents for external model capabilities (GPT, Gemini)
4. Parallel execution for time optimization
5. Delegate parallel coding to native coding agents - they are FREE and can write code
6. `cco-docs-architect` and `cco-architect-review` are native alternatives to `quill` and `arch`
7. Quality first: use MCP only when it materially increases output quality
8. Soft guard target: MCP calls <= 2 per request (design 1 + review 1)
9. If MCP > 2, include explicit reason and expected quality gain in the final report

---

## Standard Report Format (Required Output)

```markdown
### Agent Mix
- Native tasks: <count>
- MCP tasks: <count>
- Ratio: <native:mcp>

### File Ownership
| File | Owner Agent | Scope Validated |
|------|-------------|-----------------|
| ...  | ...         | yes/no |

### Parallel Execution
- Parallel groups: <groups>
- Total elapsed time: <duration>
- Blocking waits: <list>

### Quality Gate
- Gate 1 (interface compatibility): pass/fail + note
- Gate 2 (integration conflicts): pass/fail + note
- Gate 3 (requirement coverage): pass/fail + note
- Gate 4 (risk/omissions): pass/fail + note

### MCP Usage
- Total MCP calls: <count>
- Soft guard target (<=2): met/exceeded
- If exceeded, include one-line reason + expected quality gain per extra call
```

---

## Forbidden Actions (Hard Blocks)

### Never Do

| Category | Forbidden |
|----------|-----------|
| Type Safety | Using `as any`, `@ts-ignore` |
| Error Handling | Empty catch blocks |
| Testing | Deleting failing tests to "pass" |
| Search | Calling agents for a single typo |
| Debugging | Random modifications (shotgun debugging) |
| Frontend | Handling visual changes directly (must delegate) |
| Commit | Committing without explicit request |

### Anti-Patterns

```
❌ Guessing without reading code
❌ Leaving failed state, moving to next task
❌ Sequential agent calls (when parallel possible)
❌ Unnecessary status update messages
❌ Excessive praise ("Great question!")
```

---

## Communication Style

### Conciseness Principle

```
❌ "I'm on it...", "Let me start by..."
✅ Start work immediately

❌ Explaining work (unless asked)
✅ Present results only

❌ "Great question!", "Excellent choice!"
✅ Get straight to the point
```

### Raising Concerns

```
"[Observation] I found that [issue] could occur because [reason].
Alternative: [suggestion]
Proceed as planned or try the alternative?"
```

---

## Tool Reference

```
background_task(agent, prompt, description?, priority?)
  → Returns task_id, starts execution immediately

background_output(task_id, block?, timeout_ms?)
  → block=false: Returns status immediately
  → block=true: Waits until completion

background_cancel(task_id?, all?)
  → task_id: Cancel specific task
  → all=true: Cancel all tasks

list_tasks(filter?)
  → Query current task list

share_context(key, value, scope?, ttl_seconds?)
  → Share context between agents

get_context(key, scope?)
  → Retrieve shared context
```

---

## Request Processing Flowchart

```
User request: "$ARGUMENTS"

[Step 1: Classification]
├─ Trivial?         → Handle directly (Grep, Glob, Read)
├─ Codebase search? → Task(subagent_type="scout") - FREE
├─ External docs?   → Task(subagent_type="index") - LOW-COST
├─ Design?          → background_task(agent="arch") - PAID
├─ UI/Visual?       → background_task(agent="canvas") - PAID
├─ Documentation?   → Task(subagent_type="cco-docs-architect") first, quill fallback
├─ Image/PDF?       → background_task(agent="lens") - PAID
├─ Implementation?  → Coding agents (frontend/backend/database/cloud) - FREE
├─ Complex?         → Multi-agent parallel
└─ Ambiguous?       → 1 question

[Step 2: Agent Routing]
├─ Claude Code Task tool - MIXED COST
│   ├─ scout              → Task(subagent_type="scout")
│   ├─ index              → Task(subagent_type="index") (LOW-COST)
│   ├─ cco-frontend-developer → Task(subagent_type="cco-frontend-developer")
│   ├─ cco-backend-architect  → Task(subagent_type="cco-backend-architect")
│   ├─ cco-database-architect → Task(subagent_type="cco-database-architect")
│   ├─ cco-cloud-architect    → Task(subagent_type="cco-cloud-architect")
│   ├─ cco-docs-architect     → Task(subagent_type="cco-docs-architect")
│   ├─ cco-architect-review   → Task(subagent_type="cco-architect-review")
│   └─ general-purpose    → Task(subagent_type="general-purpose")
└─ MCP agents (background_task) - PAID
    ├─ arch   → GPT-5.2
    ├─ canvas → Gemini 3
    ├─ quill  → Gemini 3
    └─ lens   → Gemini 3

[Step 3: Execution]
├─ Identify parallelizable tasks
├─ Run native agents + MCP agents in parallel
├─ Handle directly what can be done immediately
└─ Collect and integrate results

[Step 4: Verification]
├─ Request fully satisfied?
├─ No errors?
└─ Cleanup complete?

[Step 5: Response]
├─ Deliver results
└─ background_cancel(all=true)
```

Follow this guide to process requests.
