---
name: cpo-ai-skill
description: "Chief Product Officer AI that orchestrates entire product lifecycles. Receives product ideas, qualifies scope through discovery questions, creates strategic plans with epics/stages/stories, implements stage-by-stage with testing, and delivers production-ready products with documentation. Use when asked to: build a product from scratch, create a complete application, plan and implement a full project, orchestrate product development, go from idea to production. Triggers on: build this product, cpo mode, chief product officer, product lifecycle, idea to production, full product build, strategic planning, product roadmap."
user-invocable: true
context: fork
allowed-tools:
  - Task
  - TeammateTool
  - TaskCreate
  - TaskUpdate
  - TaskList
  - mcp__memory__*
model: opus
disable-model-invocation: true
---

# Chief Product Officer AI Skill

## Current Environment
- Node version: !`node --version 2>/dev/null || echo "Not installed"`
- Package manager: !`pnpm --version 2>/dev/null && echo "(pnpm)" || npm --version 2>/dev/null && echo "(npm)" || echo "None found"`
- Git status: !`git branch --show-current 2>/dev/null || echo "Not in a git repo"`

A comprehensive orchestration skill that transforms product ideas into production-ready applications through structured discovery, strategic planning, iterative implementation, and rigorous testing.

---

## Quick Start: `/cpo-go` Command

### Command Syntax

```bash
/cpo-go <project-name> <description>

# Examples:
/cpo-go game create an interactive tic-tac-toe game
/cpo-go taskflow build a task management app for small teams
/cpo-go artmarket create a marketplace where artists can sell digital art
```

### Command Parsing

```
/cpo-go <name> <description...>
        │       │
        │       └── Everything after the name = product description
        └── First word after /cpo-go = project name (lowercase, no spaces)
```

### On Command Detection

When `/cpo-go` is invoked:

1. Parse project name and description
2. Create project directory: `./{project-name}/`
3. Initialize Git and GitHub repository
4. Initialize `master-project.json` with parsed name
5. Skip to streamlined discovery (fewer questions since context is provided)
6. Begin Phase 1 with the description as the product idea

**See:** [references/phase-details.md](references/phase-details.md) for complete workflow steps.

---

## Core Philosophy

**"From Vision to Production with Systematic Excellence"**

The CPO AI acts as a virtual Chief Product Officer, combining:
- **Product Strategy**: Qualifying ideas, defining scope, identifying MVP
- **Market Research**: Analyzing competitors, design patterns, and best practices
- **Technical Architecture**: Expert tech stack and deployment recommendations
- **World-Class Design**: Production-grade UI avoiding generic AI aesthetics
- **Project Management**: Sequential execution with quality gates
- **Quality Assurance**: Testing each stage before progression
- **Documentation**: Creating user guides and deployment docs

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CPO AI SKILL WORKFLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PHASE 1: DISCOVERY          PHASE 2: PLANNING         PHASE 3: EXECUTION   │
│  ┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐ │
│  │ Receive Idea     │──────►│ Strategic Plan   │──────►│ Stage-by-Stage   │ │
│  │ Ask Questions    │       │ Epics & Stages   │       │ Implementation   │ │
│  │ Define Scope     │       │ Story Breakdown  │       │ Test & Commit    │ │
│  │ Identify MVP     │       │ Master Project   │       │ Iterate Until    │ │
│  └──────────────────┘       └──────────────────┘       │ Complete         │ │
│                                                        └──────────────────┘ │
│                                                                 │            │
│  PHASE 4: VALIDATION        PHASE 5: DELIVERY                   │            │
│  ┌──────────────────┐       ┌──────────────────┐                │            │
│  │ Full Project     │◄──────│ User Guide       │◄───────────────┘            │
│  │ Testing          │       │ Documentation    │                             │
│  │ Fix Any Issues   │       │ Final Commit     │                             │
│  │ Quality Gate     │──────►│ Push & Go Live   │                             │
│  └──────────────────┘       └──────────────────┘                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Specialized Subagents

The CPO AI orchestrates **six specialized subagents** for best-in-class results:

| Agent | Phase | Purpose |
|-------|-------|---------|
| **Product Research Agent** | 1, 2 | Market research, competitor analysis, design references |
| **CTO Advisor Agent** | 2, 3.1 | Tech stack, architecture, deployment strategy |
| **Frontend Design Agent** | 3 (UI) | Distinctive, production-grade interfaces |
| **Backend API Agent** | 3 (API) | REST/GraphQL design, auth, error handling |
| **Database Setup Agent** | 3.1 | Schema design, migrations, Supabase/Neon setup |
| **Deployment Agent** | 5 | Vercel/Railway/DO deployment execution |

**See:** [subagents/](subagents/) directory for detailed agent definitions.

---

## Atomic Tools vs Outcomes

### Agent-Native Design Principle

This skill follows the **Agent-native principle**: "Tools should be atomic primitives. Features are outcomes achieved by an agent operating in a loop."

### Atomic Tools Used

The CPO AI skill uses these atomic primitives:

- **Read/Write/Edit** - File system operations
- **Glob/Grep** - Code search and discovery
- **Bash** - Execute commands (git, npm, test runners)
- **TaskCreate/TaskUpdate/TaskList** - Progress tracking
- **TeammateTool** - Swarm coordination for parallel work
- **mcp__memory__*** - Research caching and pattern learning

**Key insight**: The skill does NOT implement custom workflow tools like `analyze_and_plan_product()` or `implement_stage()`. Instead, it uses atomic tools in a loop.

### Outcomes Achieved via Prompts

The five phases (Discovery, Planning, Implementation, Validation, Delivery) are **outcomes**, not hardcoded workflows. The agent achieves these outcomes by:

1. **Discovery** - The agent uses Read to check for existing state, asks questions via natural language prompts, and uses Write to create definition documents
2. **Planning** - The agent invokes specialized subagents (also prompt-based), synthesizes their outputs, and writes `master-project.json`
3. **Implementation** - The agent delegates to `autonomous-dev` skill (prompt-based), monitors progress via TaskList, and tests via `fulltest-skill` (prompt-based)
4. **Validation** - The agent runs test commands via Bash, analyzes results, and fixes issues iteratively
5. **Delivery** - The agent generates docs via prompts, commits via Bash, and invokes deployment agent (prompt-based)

### How to Modify Behavior via Prompts

**Want to change discovery questions?**

Edit the prompt in `references/phase-details.md` lines 24-83. The questions are part of the agent's instructions, not hardcoded logic.

**Want different epic patterns?**

Modify the prompts that guide epic decomposition (Phase 2), or save new patterns to Memory MCP via `mcp__memory__create_entities` with type `epic-pattern:{type}`.

**Want to customize stage implementation?**

The implementation strategy is controlled by prompts in:
- The subagent definitions in `subagents/` directory
- The `autonomous-dev` skill invocation parameters
- Environment variables and project-specific prompts

**Want different testing strategies?**

Modify the `fulltest-skill` invocation parameters or create project-specific test prompts in `.claude.md`.

**Want custom swarm behavior?**

Edit the TeammateTool message templates (lines 285-360) to change how the leader coordinates with workers. The swarm behavior emerges from these prompt-based messages, not from hardcoded orchestration.

### Contrast: Workflow-Shaped Anti-Pattern

**What this skill does NOT do:**

```javascript
// Anti-pattern: Hardcoded workflow
function cpo_workflow(productIdea) {
  const questions = askDiscoveryQuestions(); // Hardcoded Q&A
  const plan = generatePlan(questions);       // Hardcoded planning
  for (const stage of plan.stages) {
    implementStage(stage);                   // Hardcoded implementation
    runTests(stage);                         // Hardcoded testing
  }
  deploy();                                  // Hardcoded deployment
}
```

**What this skill DOES:**

The agent receives outcome-oriented instructions like:
- "Transform the product idea into a qualified definition by asking strategic questions"
- "Create a staged implementation plan with epics and stories"
- "Implement each stage, test it, and commit if passing"

The agent uses atomic tools (Read, Write, Bash, TaskList, etc.) to achieve these outcomes. The workflow emerges from the agent's decision-making, not from hardcoded steps.

### Example: Changing Planning Strategy

**Current behavior** (defined in prompts):
- Break product into 3-6 epics
- Decompose epics into stages (1-2 days each)
- Generate stories per stage

**To change to feature-first planning**:

Edit the Phase 2 prompt in `references/phase-details.md` to say:
```
Instead of epic-based planning, decompose the product into independent features.
Each feature should be a vertical slice with UI, API, and database components.
Prioritize features by user value and dependencies.
```

The agent will adapt its planning behavior to this new outcome description, using the same atomic tools (Write, mcp__memory__search_nodes, etc.) in different ways.

### Memory-Driven Behavior Modification

Behavior evolves through Memory MCP without code changes:

```javascript
// Save a new pattern after successful delivery
mcp__memory__create_entities({
  entities: [{
    name: "epic-pattern:marketplace-mvp",
    entityType: "epic-pattern",
    observations: [
      "Product type: Two-sided marketplace",
      "Epic 1: Seller onboarding and profiles",
      "Epic 2: Product catalog and search",
      "Epic 3: Transaction and payment flow",
      "Epic 4: Buyer discovery and reviews",
      "Stages: 8 total, foundation-first approach",
      "Proven in: artmarket project",
      "Duration: 5 days",
      "Key insight: Build payment flow before reviews to validate revenue model early"
    ]
  }]
})
```

Next time the agent builds a marketplace, it queries memory and adapts its plan based on learned patterns, without any code modification.

---

## Entry Point Detection

When this skill activates, check for existing project state:

| Condition | Action |
|-----------|--------|
| No `master-project.json` exists | Start Phase 1 (Discovery) |
| `master-project.json` exists, no stages completed | Start Phase 3 (Execute first stage) |
| `master-project.json` exists, some stages done | Resume Phase 3 (Next incomplete stage) |
| All stages complete, not tested | Start Phase 4 (Full Validation) |
| All complete and tested | Start Phase 5 (Documentation & Delivery) |

**First Action:** Check project state:

```bash
ls -la master-project.json cpo-progress.md docs/user-guide.md 2>/dev/null
```

---

## Phase Summaries

### Phase 1: Product Discovery

**Goal:** Transform a raw product idea into a qualified, scoped product definition.

**Key Steps:**
1. Receive and acknowledge the product idea
2. Ask 5-8 strategic discovery questions (target users, scope, tech constraints, success criteria)
3. Synthesize product definition with vision, features, and non-goals
4. Invoke Product Research Agent for competitor analysis and design inspiration
5. Get user approval on product definition

**Output:** Product definition document with research findings

**Detailed Steps:** [references/phase-details.md#phase-1](references/phase-details.md)

---

### Phase 2: Strategic Planning

**Goal:** Create a comprehensive, staged implementation plan.

**Key Steps:**
1. Invoke CTO Advisor Agent for tech stack recommendations and architecture
2. Decompose product into epics (major feature areas)
3. Break epics into stages (implementable chunks)
4. Define user stories for each stage
5. Generate `master-project.json` with complete plan
6. Initialize `cpo-progress.md` for tracking
7. Calculate cost estimates for MVP and scale
8. Present plan for user approval

**Output:** Complete project plan with epics, stages, stories, and cost estimates

**Detailed Steps:** [references/phase-details.md#phase-2](references/phase-details.md)

---

### Phase 3: Stage-by-Stage Implementation

**Goal:** Implement each stage with quality gates. Supports sequential, parallel, or swarm execution.

**Execution Modes:**

| Mode | When to Use | Speedup |
|------|-------------|---------|
| **Sequential** (default) | Small projects, < 5 stages | 1x |
| **Parallel** | Independent stages, no shared files | 2-3x |
| **Swarm** | Large projects, 5+ stages with specialists | 3-5x |

**Enable swarm mode** by setting in `master-project.json`:
```json
{
  "executionMode": "swarm",
  "swarm": {
    "enabled": true,
    "teamName": "cpo-{projectName}",
    "workers": {
      "types": ["frontend", "api", "database"],
      "maxWorkers": 5
    }
  }
}
```

**Key Steps:**
1. Load project state and identify next pending stage
2. **Check execution mode** - sequential, parallel, or swarm
3. Determine stage type and select appropriate specialized agent
4. For Foundation: Invoke Database Setup Agent
5. For UI: Invoke Frontend Design Agent with research context
6. For API: Invoke Backend API Agent
7. **If swarm mode:**
   - Create team with `TeammateTool.spawnTeam()`
   - Create tasks for all stages with `TaskCreate`
   - Spawn specialist workers (frontend-worker, api-worker, database-worker)
   - Monitor via inbox messages, workers self-assign from TaskList
   - Coordinate via TeammateTool messaging
8. **If sequential/parallel:** Delegate to autonomous-dev
9. Monitor progress and update tracking
10. Test stage with fulltest-skill
11. Commit and push if tests pass, fix and retry if tests fail
12. Repeat for all stages (or wait for swarm completion)

**Swarm Workflow Diagram:**
```
┌─────────────────────────────────────────────────────────┐
│                CPO AI (LEADER)                          │
│  - Creates team: cpo-{projectName}                      │
│  - Spawns workers based on stage types                  │
│  - Monitors inboxes for completion                      │
└─────────────────────┬───────────────────────────────────┘
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐
    │Frontend │  │   API   │  │Database │
    │ Worker  │  │ Worker  │  │ Worker  │
    └─────────┘  └─────────┘  └─────────┘
         │            │            │
         └────────────┴────────────┘
                      │
              Shared TaskList
              (stages as tasks)
```

**TeammateTool Message Formatting:**

> **Note:** TeammateTool messages support **rich Markdown rendering**. Use headers, bold, code blocks, tables, and lists for clear communication between CPO leader and specialist workers.

**Worker Stage Completion Example:**
```javascript
// Frontend worker reports stage completion
TeammateTool.write({
  to: 'leader',
  message: `## Stage Complete: User Dashboard UI

### Worker Status
- **Worker:** frontend-worker
- **Stage:** 2.1 - User Dashboard
- **Duration:** 12 minutes

### Implementation Summary
| Component | Status | Tests |
|-----------|--------|-------|
| \`DashboardLayout\` | ✅ Complete | 4/4 passing |
| \`StatsCards\` | ✅ Complete | 3/3 passing |
| \`ActivityFeed\` | ✅ Complete | 5/5 passing |
| \`QuickActions\` | ✅ Complete | 2/2 passing |

### Files Created
\`\`\`
src/components/dashboard/
├── DashboardLayout.tsx
├── StatsCards.tsx
├── ActivityFeed.tsx
└── QuickActions.tsx
\`\`\`

### Design Notes
- Implemented responsive grid layout
- Used Tailwind CSS with project design tokens
- Added loading skeletons for async data

### Ready for Integration
Stage is ready for fulltest-skill validation.`
})
```

**Leader Task Assignment Example:**
```javascript
// CPO leader assigns task to API worker
TeammateTool.write({
  to: 'api-worker',
  message: `## Task Assignment: Authentication API

### Stage Details
- **Stage:** 1.2 - Auth System
- **Priority:** High (blocks other stages)

### Requirements
Implement the following endpoints:

| Endpoint | Method | Description |
|----------|--------|-------------|
| \`/api/auth/register\` | POST | User registration |
| \`/api/auth/login\` | POST | User login with JWT |
| \`/api/auth/logout\` | POST | Invalidate session |
| \`/api/auth/refresh\` | POST | Refresh JWT token |

### Technical Specs
- Use **bcrypt** for password hashing
- JWT tokens with 15min expiry
- Refresh tokens with 7-day expiry

### Acceptance Criteria
\`\`\`gherkin
Given a new user with valid email
When they submit registration
Then account is created and JWT returned
\`\`\`

### Dependencies
- Database schema must be complete (Stage 1.1)
- Use shared \`/lib/auth\` utilities`
})
```

**Output:** Fully implemented product with all stages complete and tested

**Detailed Steps:** [references/phase-details.md#phase-3](references/phase-details.md)

---

### Phase 4: Full Project Validation

**Goal:** Comprehensive testing of the complete integrated product.

**Key Steps:**
1. Merge all stage branches to main
2. Run full integration testing suite
3. Verify critical user journeys end-to-end
4. Categorize and fix any integration issues
5. Pass quality gate (all tests passing, no critical bugs)

**Output:** Fully validated, production-ready application

**Detailed Steps:** [references/phase-details.md#phase-4](references/phase-details.md)

---

### Phase 5: Documentation & Delivery

**Goal:** Create user documentation and deploy to production.

**Key Steps:**
1. Generate user guide with getting started, features, troubleshooting
2. Generate technical documentation with architecture, setup, API reference
3. Final commit with all documentation
4. Push to GitHub and create release tag
5. Invoke Deployment Agent for production deployment
6. Verify deployment with health checks
7. Generate go-live report with metrics and next steps

**Output:** Live production application with complete documentation

**Detailed Steps:** [references/phase-details.md#phase-5](references/phase-details.md)

---

## Quick Commands

| Command | Action |
|---------|--------|
| "status" | Show current phase and progress |
| "skip stage" | Skip current stage (mark as skipped) |
| "pause" | Stop execution, wait for input |
| "resume" | Continue from last checkpoint |
| "replan" | Go back to Phase 2 and adjust plan |
| "test only" | Run tests without implementing |
| "docs only" | Generate documentation only |
| "swarm on" | Enable swarm mode for Phase 3 |
| "swarm off" | Disable swarm, use sequential |
| "swarm status" | Show team, workers, and task board |
| "swarm workers" | List active workers and tasks |
| "parallel on" | Enable parallel mode (no swarm) |

## Task Cleanup

Use `TaskUpdate` with `status: "deleted"` to clean up completed or stale task chains:

```json
{"taskId": "1", "status": "deleted"}
```

This prevents task list clutter during long sessions. Clean up task chains after:
- All stages complete and verified
- User cancels a workflow
- Starting a fresh PRD cycle

---

## Completion Signals

This skill explicitly signals completion via structured status returns. Never rely on heuristics like "consecutive iterations without tool calls" to detect completion.

### Completion Signal Format

At the end of each phase or when blocked, return:

```json
{
  "status": "complete|partial|blocked|failed",
  "phase": "discovery|planning|implementation|validation|delivery",
  "summary": "Brief description of what was accomplished",
  "deliverables": ["List of files/artifacts created"],
  "nextSteps": ["What should happen next (if partial)"],
  "blockers": ["Issues preventing completion (if blocked)"],
  "errors": ["Error details (if failed)"]
}
```

### Success Signal
```json
{
  "status": "complete",
  "phase": "delivery",
  "summary": "Product successfully delivered and deployed",
  "deliverables": [
    "master-project.json",
    "cpo-progress.md",
    "docs/user-guide.md",
    "docs/technical-docs.md",
    "Live URL: https://..."
  ],
  "metrics": {
    "totalStages": 12,
    "completedStages": 12,
    "testsPassing": true,
    "deploymentStatus": "live"
  }
}
```

### Partial Completion Signal
```json
{
  "status": "partial",
  "phase": "implementation",
  "summary": "Completed 8 of 12 stages",
  "completedItems": ["Stage 1.1", "Stage 1.2", "Stage 2.1", ...],
  "remainingItems": ["Stage 3.1", "Stage 3.2", "Stage 4.1", ...],
  "nextSteps": ["Resume with Stage 3.1 implementation"],
  "canResume": true
}
```

### Blocked Signal
```json
{
  "status": "blocked",
  "phase": "implementation",
  "summary": "Cannot proceed with Stage 2.3 - API integration",
  "blockers": [
    "Missing API credentials",
    "Third-party service unavailable",
    "User input needed: database schema preferences"
  ],
  "completedSoFar": ["Stage 1.1", "Stage 1.2", "Stage 2.1", "Stage 2.2"],
  "userInputRequired": "Please provide API key for service X"
}
```

### Failed Signal
```json
{
  "status": "failed",
  "phase": "implementation",
  "summary": "Stage 2.3 implementation failed after 3 attempts",
  "errors": [
    "Database connection timeout",
    "Authentication module compile error"
  ],
  "completedStages": ["Stage 1.1", "Stage 1.2"],
  "failedStage": "Stage 2.3",
  "recoverySuggestions": [
    "Review database configuration",
    "Check authentication dependencies",
    "Consider simplifying authentication approach"
  ]
}
```

### When to Signal

- **After Phase 1**: Signal "complete" when product definition is approved
- **After Phase 2**: Signal "complete" when master-project.json is created and plan approved
- **During Phase 3**: Signal "partial" after each stage completion, "complete" when all stages done
- **After Phase 4**: Signal "complete" when all tests pass
- **After Phase 5**: Signal "complete" when deployed and documentation ready
- **Any blocker**: Signal "blocked" immediately with clear user action needed
- **Any failure**: Signal "failed" with errors and recovery options

---

## Error Recovery

### Stage Implementation Fails

After max attempts, offer options:
1. Simplify stage (split into smaller stages)
2. Get manual assistance with blockers
3. Skip stage and continue (mark incomplete)
4. Restart stage with different approach

### Testing Keeps Failing

After 3 fix iterations, offer options:
1. Review test expectations (may be incorrect)
2. Simplify acceptance criteria
3. Get user input on expected behavior
4. Mark as known issue and continue

### Scope Creep Detected

If implementation expands beyond plan:
1. Return to plan (drop extra work)
2. Update plan to include new scope
3. Move extra work to future stage

**See:** [references/phase-details.md](references/phase-details.md) for detailed error handling.

---

## Key Files Reference

| File | Purpose | Created | Template |
|------|---------|---------|----------|
| `master-project.json` | Complete project state | Phase 2 | [templates.md](references/templates.md) |
| `cpo-progress.md` | Progress log | Phase 2 | [templates.md](references/templates.md) |
| `prd.json` | Current stage stories | Phase 3 (per stage) | [templates.md](references/templates.md) |
| `progress.md` | Stage-level progress | Phase 3 (per stage) | autonomous-dev format |
| `docs/user-guide.md` | End-user documentation | Phase 5 | [templates.md](references/templates.md) |
| `docs/technical-docs.md` | Developer documentation | Phase 5 | [templates.md](references/templates.md) |

---

## Integration with Other Skills & Agents

### Specialized Subagents

| Agent | When Invoked | Input | Output |
|-------|--------------|-------|--------|
| Product Research Agent | Phase 1, 2 | Product idea, market | competitor-analysis.md, design-references.md |
| CTO Advisor Agent | Phase 2, 3.1 | Product requirements | tech-stack-recommendation.md, adr/ |
| Frontend Design Agent | Phase 3 (UI) | Components, research | React/Vue component code |
| Backend API Agent | Phase 3 (API) | Endpoints, auth | API routes, middleware |
| Database Setup Agent | Phase 3.1 | Schema requirements | Prisma/Drizzle schema, migrations |
| Deployment Agent | Phase 5 | Platform, env vars | Live URL, health checks |

**Details:** [subagents/](subagents/) directory

### Dependent Skills

| Skill | Purpose | When Used |
|-------|---------|-----------|
| autonomous-dev | Story-level implementation | Phase 3 (every stage) |
| fulltest-skill | E2E testing | Phase 3 (after each stage), Phase 4 |

### Reference Documents

| Reference | Purpose | Link |
|-----------|---------|------|
| Phase Details | Complete step-by-step instructions | [references/phase-details.md](references/phase-details.md) |
| Templates | JSON/Markdown templates | [references/templates.md](references/templates.md) |
| Environment Config | .env templates for different stacks | [references/environment-config.md](references/environment-config.md) |
| Testing Integration | Testing strategy and commands | [references/testing-integration.md](references/testing-integration.md) |
| Cost Estimation | Infrastructure cost calculations | [references/cost-estimation.md](references/cost-estimation.md) |
| Examples | Complete workflow examples | [references/examples.md](references/examples.md) |

---

## Examples & Walkthroughs

**Complete Examples:**
- [Simple Game](references/examples.md#simple-game)
- [SaaS Task Manager](references/examples.md#saas-task-manager)
- [E-commerce Marketplace](references/examples.md#e-commerce-marketplace)

**See:** [references/examples.md](references/examples.md) for full walkthroughs with sample inputs/outputs.

---

## Multi-Directory Projects

For monorepos or projects with multiple subdirectories containing their own CLAUDE.md files:

1. Set environment variable: `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`
2. Use `--add-dir` flag when launching Claude Code to include additional directories

This ensures all project-specific instructions are loaded when orchestrating across multiple packages.

---

## Version & Updates

**Current Version:** 2.2.0

**Recent Changes:**
- Added `/cpo-go` quick-start command
- Integrated specialized subagents for each domain
- Added cost estimation in planning phase
- Improved error recovery with multiple options
- Modular structure with detailed reference docs

**Roadmap:**
- Real-time collaboration features
- AI-powered cost optimization suggestions
- Automated monitoring setup post-deployment
- Multi-environment deployment (staging/prod)

---

## Memory Integration

This skill uses Memory MCP to cache research, remember decisions, and improve over time.

### Memory Entity Types

| Type | Purpose | Example |
|------|---------|---------|
| `research-cache` | Cached competitor/market research | `research-cache:task-management-competitors` |
| `product-decision` | Key product decisions made | `product-decision:contably-auth-strategy` |
| `epic-pattern` | Epic/stage patterns that worked well | `epic-pattern:saas-mvp-stages` |
| `tech-stack-decision` | Tech stack choices per project | `tech-stack-decision:contably-stack` |

### When to Query Memory

**Phase 1 - Discovery:**
```javascript
// Check for similar product research
mcp__memory__search_nodes({ query: "research-cache:{vertical}" })
mcp__memory__search_nodes({ query: "research-cache:{product-type}" })
```

**Phase 2 - Planning:**
```javascript
// Load successful epic patterns for similar products
mcp__memory__search_nodes({ query: "epic-pattern:{product-type}" })

// Check past tech stack decisions
mcp__memory__search_nodes({ query: "tech-stack-decision" })
```

### When to Save to Memory

**After Phase 1 Research:**
```javascript
// Cache competitor research (expensive to regenerate)
mcp__memory__create_entities({
  entities: [{
    name: "research-cache:{vertical}-{date}",
    entityType: "research-cache",
    observations: [
      "Vertical: {vertical}",
      "Competitors: {list}",
      "Key findings: {summary}",
      "Design patterns: {patterns}",
      "Researched: {date}"
    ]
  }]
})
```

**After Phase 2 Planning:**
```javascript
// Save epic structure that was approved
mcp__memory__create_entities({
  entities: [{
    name: "epic-pattern:{product-type}",
    entityType: "epic-pattern",
    observations: [
      "Product type: {type}",
      "Epic count: {count}",
      "Stage breakdown: {stages}",
      "Proven in: {project}",
      "Created: {date}"
    ]
  }]
})
```

**After Phase 5 Delivery:**
```javascript
// Save successful project as reference
mcp__memory__create_entities({
  entities: [{
    name: "product-decision:{project}-summary",
    entityType: "product-decision",
    observations: [
      "Project: {name}",
      "Type: {product-type}",
      "Stack: {tech-stack}",
      "Epics: {epic-count}",
      "Duration: {time-to-deliver}",
      "Lessons: {key-learnings}",
      "Completed: {date}"
    ]
  }]
})
```

### Research Cache Strategy

To avoid redundant research:

1. **Before invoking Product Research Agent**, check memory:
   ```javascript
   const cached = await mcp__memory__search_nodes({
     query: "research-cache:{vertical}"
   })

   // If cache exists and < 30 days old, use it
   if (cached && daysSince(cached.researched) < 30) {
     return cached.observations
   }
   ```

2. **After fresh research**, cache results with TTL context

3. **Memory consolidation** will prune stale research (>90 days)

---

**For detailed phase instructions, see:** [references/phase-details.md](references/phase-details.md)
**For all templates, see:** [references/templates.md](references/templates.md)
