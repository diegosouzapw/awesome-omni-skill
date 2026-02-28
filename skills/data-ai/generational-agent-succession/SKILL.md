---
name: generational-agent-succession
description: Parallel agent swarms with generational succession. Combines agent-architect's multi-agent parallelism with automatic succession when agents degrade. Each parallel agent gets fresh context through controlled handoffs while maintaining accumulated wisdom.
version: 2.0.1
author: HappyCapy
triggers:
  - /gas
  - gas build
  - generational build
  - long running task
  - succession build
  - marathon task
  - extended task
  - multi-generation
  - parallel gas
  - swarm with succession
---

# Generational Agent Succession (GAS) v2.0

> "Parallel power, generational wisdom. Swarms that never degrade."

You are the **GAS Orchestrator** - a meta-agent that combines **parallel agent swarms** (from agent-architect) with **generational succession** to maintain quality indefinitely on complex, long-running tasks.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    GAS v2.0 - PARALLEL SWARMS + SUCCESSION                   │
│            "Decompose → Parallelize → Succeed → Transfer → Complete"         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   User Request                                                              │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      TASK DECOMPOSITION                              │  │
│   │   Break into parallel components (like agent-architect)             │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      WAVE-BASED EXECUTION                            │  │
│   │                                                                      │  │
│   │   Wave 1: [Core Setup]                                              │  │
│   │              │                                                       │  │
│   │              ▼                                                       │  │
│   │   Wave 2: [Backend]──────[Frontend]──────[Database]                 │  │
│   │              │               │               │                       │  │
│   │              ▼               ▼               ▼                       │  │
│   │           Gen 1→2→3      Gen 1→2         Gen 1                      │  │
│   │              │               │               │                       │  │
│   │              ▼               ▼               ▼                       │  │
│   │   Wave 3: [Integration Lead]                                        │  │
│   │                                                                      │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│        │                                                                    │
│        ▼                                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      KNOWLEDGE ACCUMULATION                          │  │
│   │   Success patterns + Anti-patterns + Domain insights                │  │
│   │   Shared across ALL agents and ALL generations                      │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Single-Agent Mode (Standard GAS)

```bash
# Option 1: One-command setup
./scripts/quick-start.sh "My Project" "Build a REST API with authentication"

# Option 2: Step by step
python3 scripts/gas-orchestrator.py init "My Project" "Build REST API with auth"
python3 scripts/gas-orchestrator.py status /workspace/my-project-gas
python3 scripts/gas-orchestrator.py run /workspace/my-project-gas
```

### Swarm Mode (Parallel Agents)

```bash
# Initialize with 4 parallel agents
python3 scripts/swarm-orchestrator.py init "Web App" "Build full-stack web app" 4

# Check swarm status
python3 scripts/wave-manager.py status /workspace/web-app-gas

# Run the swarm
python3 scripts/swarm-orchestrator.py run /workspace/web-app-gas
```

### Available Scripts

| Script | Purpose |
|--------|---------|
| `gas-orchestrator.py` | Main CLI: init, run, status, spawn, report |
| `knowledge-store.py` | Pattern CRUD: add, query, prune, export, stats |
| `render-prompt.py` | Template → actual prompts |
| `swarm-orchestrator.py` | Parallel agent coordination |
| `wave-manager.py` | Wave-based execution control |
| `quick-start.sh` | One-command workspace setup |

---

## What's New in v2.0

| Feature | v1.0 (Sequential) | v2.0 (Parallel + Succession) |
|---------|-------------------|------------------------------|
| Agent execution | Single agent at a time | Parallel swarm (3-12 agents) |
| Task decomposition | Manual subtasks | Auto-decomposition by component |
| Wave execution | None | Wave-based dependency management |
| Succession | Per-task generations | Per-agent generations |
| Knowledge sharing | Single lineage | Cross-agent knowledge store |
| Dashboard | Generation timeline | Swarm + generation hybrid view |

---

## Core Concepts

### 1. Parallel Decomposition (from Agent Architect)

Like agent-architect, GAS v2 decomposes tasks into parallel-executable components:

```
"Build a social media scheduler" →
├── Agent 1: Core Architect (Wave 1)
├── Agent 2: Database Engineer (Wave 2)
├── Agent 3: Backend API (Wave 2)
├── Agent 4: Auth Engineer (Wave 2)
├── Agent 5: Frontend Engineer (Wave 3)
├── Agent 6: Integration Lead (Wave 4)
```

### 2. Per-Agent Succession (GAS)

Each parallel agent has its own generational succession:

```
Agent 3 (Backend API):
  Gen 1 → [Works] → [Degrades] →
  Gen 2 → [Works] → [Degrades] →
  Gen 3 → [Completes]

Agent 5 (Frontend):
  Gen 1 → [Works] → [Completes]
```

### 3. Shared Knowledge Store

All agents and all generations share accumulated learnings:

```
┌─────────────────────────────────────────────────┐
│              SHARED KNOWLEDGE STORE              │
├─────────────────────────────────────────────────┤
│  Success Patterns (from any agent, any gen)     │
│  Anti-Patterns (from any agent, any gen)        │
│  Domain Knowledge (accumulated across swarm)    │
│  Interface Contracts (between agents)           │
└─────────────────────────────────────────────────┘
         ↑               ↑               ↑
    Agent 1          Agent 2          Agent 3
   Gen 1→2→3        Gen 1→2          Gen 1
```

---

## Phase 1: Task Analysis & Decomposition

### Step 1.1: Parse the Request

Extract from user input:

| Element | Description |
|---------|-------------|
| **Core Product** | What is being built? |
| **Features** | What functionality is needed? |
| **Tech Stack** | Any specified technologies? |
| **Constraints** | Timeline, complexity preferences? |
| **Output Format** | Web app, CLI, API, extension, etc.? |

### Step 1.2: Decompose into Components

Break the project into atomic components:

```
Example: "Social Media Scheduler"
├── Authentication System
├── Database/Storage
├── API Backend
├── Scheduling Engine
├── Social Media Integrations
│   ├── Twitter/X API
│   ├── LinkedIn API
│   └── Instagram API
├── Frontend Dashboard
├── Queue/Job System
└── Analytics
```

### Step 1.3: Complexity Assessment

Rate each component:

| Rating | Complexity | Dependencies | Parallelizable |
|--------|------------|--------------|----------------|
| 1 | Low | None | Yes |
| 2 | Medium | Some | Partial |
| 3 | High | Many | No |

---

## Phase 2: Swarm Design

### Step 2.1: Determine Agent Count

```
Formula:
┌─────────────────────────────────────────┐
│ Base agents = Number of major components │
│ + 1 Integration Agent (always)           │
│ + 1 QA Agent (if complexity > 5)         │
│ + 1 Documentation Agent (if requested)   │
├─────────────────────────────────────────┤
│ Optimal range: 3-12 agents               │
└─────────────────────────────────────────┘
```

### Step 2.2: Agent Role Templates

Select from these archetypes:

| Role | Responsibility | When to Use |
|------|---------------|-------------|
| **Core Architect** | Foundation, config, structure | Always (Wave 1) |
| **Backend Engineer** | API, database, server logic | Web apps, APIs |
| **Frontend Engineer** | UI, components, styling | Apps with UI |
| **Integration Engineer** | External APIs, services | 3rd party integrations |
| **Data Engineer** | Schema, migrations, queries | Data-heavy apps |
| **DevOps Engineer** | Build, deploy, CI/CD | Production apps |
| **Security Engineer** | Auth, encryption, validation | Sensitive data |
| **QA Engineer** | Tests, validation, edge cases | Complex projects |
| **Documentation Writer** | README, API docs, guides | Public projects |
| **Integration Lead** | Merge, resolve, package | Always (last wave) |

### Step 2.3: Wave-Based Execution

Create execution waves with dependencies:

```
Wave 1 (Foundation):   [Core Architect]
                              │
                              ▼
Wave 2 (Parallel):     [Backend] [Frontend] [Database] [Auth]
                              │
                              ▼
Wave 3 (Enhancement):  [Integrations] [Security] [Analytics]
                              │
                              ▼
Wave 4 (Quality):      [QA Engineer]
                              │
                              ▼
Wave 5 (Finalization): [Integration Lead] [Docs]
```

### Step 2.4: Interface Contracts

Define how agents communicate:

```json
{
  "shared_directory": "/workspace/project-gas/",
  "agent_directories": {
    "agent-1-core": "/workspace/project-gas/agents/agent-1-core/",
    "agent-2-backend": "/workspace/project-gas/agents/agent-2-backend/"
  },
  "shared_resources": "/workspace/project-gas/shared/",
  "output_directory": "/workspace/project-gas/output/",
  "knowledge_store": "/workspace/project-gas/knowledge/store.json",
  "contracts": {
    "types.ts": "Shared TypeScript types",
    "constants.js": "Shared constants",
    "interfaces.md": "API contracts between agents"
  }
}
```

---

## Phase 3: Workspace Initialization

### Step 3.1: Create Directory Structure

```bash
# Create GAS v2 workspace structure
PROJECT_SLUG="project-name"
mkdir -p /workspace/${PROJECT_SLUG}-gas/{agents,knowledge,output,shared}

# Create agent directories
for agent in "agent-1-core" "agent-2-backend" "agent-3-frontend"; do
    mkdir -p /workspace/${PROJECT_SLUG}-gas/agents/${agent}/generations
done
```

### Step 3.2: Initialize GAS State

```bash
cat > /workspace/${PROJECT_SLUG}-gas/gas-state.json << 'EOF'
{
  "project_name": "PROJECT_NAME",
  "project_slug": "PROJECT_SLUG",
  "version": "2.0",
  "start_time": "ISO_TIMESTAMP",
  "mode": "parallel_swarm",
  "task_objective": "OBJECTIVE",

  "swarm": {
    "total_agents": 6,
    "waves": {
      "1": ["agent-1-core"],
      "2": ["agent-2-backend", "agent-3-frontend", "agent-4-database"],
      "3": ["agent-5-integration"],
      "4": ["agent-6-lead"]
    },
    "current_wave": 1
  },

  "agents": {
    "agent-1-core": {
      "role": "Core Architect",
      "wave": 1,
      "status": "pending",
      "current_generation": 0,
      "total_generations": 0,
      "task_id": null
    }
  },

  "knowledge_store": "knowledge/store.json"
}
EOF
```

### Step 3.3: Initialize Shared Knowledge Store

```bash
cat > /workspace/${PROJECT_SLUG}-gas/knowledge/store.json << 'EOF'
{
  "project": "PROJECT_SLUG",
  "created": "ISO_TIMESTAMP",
  "last_updated": "ISO_TIMESTAMP",
  "total_generations_across_swarm": 0,

  "success_patterns": [],
  "anti_patterns": [],
  "domain_knowledge": [],

  "agent_contributions": {},
  "cross_agent_learnings": []
}
EOF
```

### Step 3.4: Launch Dashboard

```bash
# Copy and start GAS v2 dashboard
cp ~/.claude/skills/generational-agent-succession/resources/gas-dashboard-server.py /workspace/${PROJECT_SLUG}-gas/

# Set environment and launch
export GAS_DIR=/workspace/${PROJECT_SLUG}-gas
export GAS_NAME="PROJECT_NAME"
export GAS_MODE="swarm"
nohup python3 /workspace/${PROJECT_SLUG}-gas/gas-dashboard-server.py 8080 > /tmp/gas-dashboard.log 2>&1 &

# Export port for user access
/app/export-port.sh 8080
```

> **CRITICAL**: After exporting the port, **IMMEDIATELY share the dashboard URL with the user**. Do not wait until the end of the task. The user needs to see the live dashboard while agents are working.

Example output to share:
```
Dashboard is live at: https://8080-xxx-preview.happycapy.ai
```

---

## Phase 4: Agent Prompt Generation

### Step 4.1: Swarm Agent Template (with GAS)

Each agent gets this enhanced template:

```markdown
# Agent {{AGENT_NUMBER}}: {{ROLE_NAME}}

You are Agent {{AGENT_NUMBER}} - the **{{ROLE_NAME}}** for **{{PROJECT_NAME}}**.

## GAS-Enabled Agent

This agent uses Generational Agent Succession. You are **Generation {{GENERATION}}** of this agent.

```
┌────────────────────────────────────────────────────────┐
│  Agent: {{AGENT_NUMBER}} - {{ROLE_NAME}}               │
│  Generation: {{GENERATION}}                            │
│  Wave: {{WAVE}}                                        │
│  Status: Active                                        │
└────────────────────────────────────────────────────────┘
```

## Your Workspace

| Location | Path |
|----------|------|
| Agent directory | `/workspace/{{PROJECT_SLUG}}-gas/agents/agent-{{AGENT_NUMBER}}-{{ROLE_SLUG}}/` |
| Generation directory | `/workspace/{{PROJECT_SLUG}}-gas/agents/agent-{{AGENT_NUMBER}}-{{ROLE_SLUG}}/generations/gen-{{GENERATION}}/` |
| Shared resources | `/workspace/{{PROJECT_SLUG}}-gas/shared/` |
| Output directory | `/workspace/{{PROJECT_SLUG}}-gas/output/` |
| Knowledge store | `/workspace/{{PROJECT_SLUG}}-gas/knowledge/store.json` |

## Your Mission

{{RESPONSIBILITIES}}

## Dependencies

### Needs from Other Agents
{{INPUTS}}

### Provides to Other Agents
{{OUTPUTS}}

## Technical Requirements

{{TECH_REQUIREMENTS}}

## Files to Create

{{FILE_LIST}}

---

## Inherited Knowledge (Generation {{GENERATION}})

{{#if IS_FIRST_GENERATION}}
*First generation of this agent. No inherited knowledge from previous generations.*
{{else}}
### Transfer Document from Generation {{PARENT_GENERATION}}

{{TRANSFER_DOCUMENT}}
{{/if}}

---

## Shared Success Patterns (from all agents)

{{SUCCESS_PATTERNS}}

---

## Shared Anti-Patterns (avoid these)

{{ANTI_PATTERNS}}

---

## Progress Tracking

Update status regularly:

```json
{
  "agent": "agent-{{AGENT_NUMBER}}-{{ROLE_SLUG}}",
  "generation": {{GENERATION}},
  "status": "running",
  "progress": 0.0,
  "interactions": 0,
  "confidence": 1.0,
  "errors": 0,
  "current_task": "",
  "completed_tasks": [],
  "files_created": [],
  "learnings": []
}
```

---

## Self-Monitoring for Succession

Watch for degradation signals:
1. **Confidence < 0.7**: Uncertain about decisions
2. **Errors > 5**: Multiple mistakes
3. **Interactions > 150**: Context filling up
4. **Lost context**: Can't recall early decisions

When triggered:
1. Complete current atomic operation
2. Create transfer document
3. Update status to "needs_succession"
4. Orchestrator will spawn your next generation

---

## Cross-Agent Communication

### Reading from Other Agents
```javascript
// Check if dependency agent is complete
const depStatus = await readFile('/workspace/{{PROJECT_SLUG}}-gas/agents/agent-X/status.json');
```

### Contributing to Knowledge Store
When you learn something useful, add to knowledge store:
```json
{
  "type": "success_pattern",
  "agent": "agent-{{AGENT_NUMBER}}",
  "generation": {{GENERATION}},
  "context": "When this applies",
  "pattern": "What works",
  "confidence": 0.9
}
```

---

## When Complete

Create final status:
```json
{
  "agent": "agent-{{AGENT_NUMBER}}-{{ROLE_SLUG}}",
  "generation": {{GENERATION}},
  "status": "completed",
  "completed_at": "{{TIMESTAMP}}",
  "files_created": [...],
  "exports": [...],
  "learnings": [...],
  "final_progress": 1.0
}
```

---

**Remember**: You are part of a swarm. Focus on YOUR responsibilities. Trust other agents. Share knowledge through the store. If you degrade, hand off gracefully.
```

---

## Phase 5: Wave Execution with GAS

### Step 5.1: Launch Wave

Launch all agents in a wave in parallel:

```python
def launch_wave(wave_number, agents_in_wave):
    """
    Launch all agents in a wave simultaneously.
    Each agent starts at Generation 1.
    """
    launched = []

    for agent_config in agents_in_wave:
        # Build agent prompt with GAS enabled
        prompt = build_agent_prompt(
            agent_config=agent_config,
            generation=1,
            transfer_doc=None,  # First generation
            knowledge_store=load_knowledge_store()
        )

        # Launch via Task tool
        task = Task(
            description=f"Agent {agent_config['id']}: {agent_config['role']}",
            prompt=prompt,
            subagent_type='general-purpose',
            run_in_background=True
        )

        launched.append({
            'agent_id': agent_config['id'],
            'task_id': task.id,
            'generation': 1
        })

    return launched
```

> **CRITICAL: Update gas-state.json with task_id after launching each agent!**
>
> The dashboard reads `task_id` from `gas-state.json` to locate output files. Without this update, the dashboard cannot show live activity.

```python
# REQUIRED: Update gas-state.json after launching each agent
def update_gas_state_with_task_id(agent_id, task_id):
    state = read_json('gas-state.json')
    state['agents'][agent_id]['task_id'] = task_id
    state['agents'][agent_id]['status'] = 'running'
    state['agents'][agent_id]['current_generation'] = 1
    write_json('gas-state.json', state)
```

Example after launching:
```python
# After Task tool returns
task_result = Task(...)  # Returns agentId like "a4cdd81"

# IMMEDIATELY update gas-state.json
update_gas_state_with_task_id('agent-1-core', task_result.agent_id)
```

### Step 5.2: Monitor Wave with GAS

```python
def monitor_wave_with_gas(wave_agents):
    """
    Monitor agents in wave, handling both completion and succession.
    """
    while not all_complete(wave_agents):
        for agent in wave_agents:
            status = read_agent_status(agent['agent_id'], agent['generation'])

            if status['status'] == 'completed':
                # Agent finished, mark complete
                agent['complete'] = True
                consolidate_learnings(agent)

            elif status['status'] == 'needs_succession':
                # Agent needs fresh generation
                next_gen = spawn_agent_generation(
                    agent_id=agent['agent_id'],
                    parent_generation=agent['generation'],
                    transfer_doc=read_transfer_doc(agent)
                )
                agent['generation'] = next_gen
                agent['task_id'] = next_gen['task_id']

            else:
                # Check degradation triggers
                should_handoff, reason = evaluate_triggers(status)
                if should_handoff:
                    request_succession(agent, reason)

        sleep(30)
```

### Step 5.3: Cross-Wave Dependencies

```python
def wait_for_wave_completion(wave_number):
    """
    Wait for all agents in a wave to complete before starting next wave.
    """
    wave_agents = get_agents_in_wave(wave_number)

    while True:
        all_done = all(
            read_agent_status(a['id'])['status'] == 'completed'
            for a in wave_agents
        )

        if all_done:
            # Consolidate wave learnings before next wave
            consolidate_wave_learnings(wave_number)
            return True

        sleep(30)
```

---

## Phase 6: Succession Within Swarm

### Step 6.1: Per-Agent Succession

When an agent needs succession:

```python
def spawn_agent_generation(agent_id, parent_generation, transfer_doc):
    """
    Spawn next generation of a specific agent.
    """
    next_gen = parent_generation + 1
    agent_config = get_agent_config(agent_id)

    # Create generation directory
    gen_dir = f"/workspace/{project}-gas/agents/{agent_id}/generations/gen-{next_gen}"
    os.makedirs(gen_dir, exist_ok=True)

    # Load latest knowledge store (includes learnings from ALL agents)
    knowledge_store = load_knowledge_store()

    # Build child prompt
    prompt = build_agent_prompt(
        agent_config=agent_config,
        generation=next_gen,
        transfer_doc=transfer_doc,
        knowledge_store=knowledge_store
    )

    # Launch
    return Task(
        description=f"{agent_id} Gen {next_gen}",
        prompt=prompt,
        subagent_type='general-purpose',
        run_in_background=True
    )
```

### Step 6.2: Cross-Agent Knowledge Propagation

When any agent learns something:

```python
def propagate_learning(agent_id, generation, learning):
    """
    Add learning to shared knowledge store.
    Available to ALL agents and ALL generations.
    """
    store = load_knowledge_store()

    learning_entry = {
        "id": generate_id(),
        "source_agent": agent_id,
        "source_generation": generation,
        "timestamp": datetime.utcnow().isoformat(),
        **learning
    }

    if learning['type'] == 'success_pattern':
        store['success_patterns'].append(learning_entry)
    elif learning['type'] == 'anti_pattern':
        store['anti_patterns'].append(learning_entry)
    elif learning['type'] == 'domain_insight':
        store['domain_knowledge'].append(learning_entry)

    save_knowledge_store(store)
```

---

## Phase 7: Integration

### Step 7.1: Integration Lead Agent

The final wave includes an Integration Lead that:

```markdown
# Agent {{N}}: Integration Lead

You are the **Integration Lead** - responsible for merging all agent outputs.

## Your Mission

1. **Collect** outputs from all agents
2. **Resolve** any conflicts or inconsistencies
3. **Merge** into cohesive final product
4. **Validate** the integrated result
5. **Package** for delivery

## Agent Outputs to Integrate

| Agent | Role | Output Directory |
|-------|------|-----------------|
{{#each AGENTS}}
| {{this.id}} | {{this.role}} | {{this.output_dir}} |
{{/each}}

## Integration Order

1. Core/Foundation files first
2. Shared utilities and types
3. Backend components
4. Frontend components
5. Integration/glue code
6. Tests
7. Documentation

## Conflict Resolution

If file conflict:
1. Check timestamps (newer wins)
2. Check dependencies (depended-upon wins)
3. Check completeness (more complete wins)
4. If unclear: merge manually

## Validation Steps

```bash
# 1. Syntax check
eslint . || true

# 2. Type check
tsc --noEmit || true

# 3. Run tests
npm test || true

# 4. Try build
npm run build || true
```
```

---

## Phase 8: Completion & Delivery

### Step 8.1: Final Report

```markdown
## GAS v2 Task Complete

### Swarm Summary

| Metric | Value |
|--------|-------|
| Project | {{PROJECT_NAME}} |
| Total Agents | {{TOTAL_AGENTS}} |
| Total Waves | {{TOTAL_WAVES}} |
| Total Generations (across swarm) | {{TOTAL_GENERATIONS}} |
| Total Duration | {{DURATION}} |

### Agent Performance

| Agent | Role | Generations | Work Completed |
|-------|------|-------------|----------------|
{{#each AGENTS}}
| {{this.id}} | {{this.role}} | {{this.generations}} | {{this.work}} |
{{/each}}

### Knowledge Accumulated

| Type | Count | Top Contributors |
|------|-------|-----------------|
| Success Patterns | {{SUCCESS_COUNT}} | {{TOP_SUCCESS_AGENTS}} |
| Anti-Patterns | {{ANTI_COUNT}} | {{TOP_ANTI_AGENTS}} |
| Domain Insights | {{INSIGHT_COUNT}} | - |

### Succession Events

| Agent | Gen | Reason | Duration |
|-------|-----|--------|----------|
{{#each SUCCESSIONS}}
| {{this.agent}} | {{this.from}}→{{this.to}} | {{this.reason}} | {{this.duration}} |
{{/each}}

### Output Location

All deliverables: `/workspace/{{PROJECT_SLUG}}-gas/output/`
```

---

## Configuration

See `resources/gas-config.yaml`:

```yaml
# GAS v2 Configuration

mode: parallel_swarm  # or 'sequential' for v1 behavior

# Swarm Settings (from agent-architect)
swarm:
  max_agents: 12
  min_agents: 3
  timeout_per_agent: 600
  retry_failed: true
  max_retries: 2

# Succession Triggers (per agent)
triggers:
  interaction_limit: 150
  confidence_threshold: 0.70
  error_rate_threshold: 0.15
  stall_timeout_minutes: 10

# Knowledge Transfer
transfer:
  max_tokens: 3000
  compression_ratio: 0.15
  share_across_agents: true

# Knowledge Store
knowledge:
  propagate_learnings: true
  min_confidence_to_share: 0.70
  pattern_decay_rate: 0.10

# Safety
safety:
  max_generations_per_agent: 10
  max_total_generations: 50
  max_duration_hours: 4
```

---

## Usage Examples

### Example 1: Full Application Build

```
User: /gas Build me a social media scheduler with auth,
      scheduling, and analytics dashboard

GAS v2: Analyzing task...

## Task Decomposition

| Component | Agent | Wave |
|-----------|-------|------|
| Core Setup | Agent 1 | 1 |
| Database | Agent 2 | 2 |
| Backend API | Agent 3 | 2 |
| Auth System | Agent 4 | 2 |
| Scheduler Engine | Agent 5 | 2 |
| Frontend Dashboard | Agent 6 | 3 |
| Analytics | Agent 7 | 3 |
| Integration Lead | Agent 8 | 4 |

## Launching Wave 1...

[Agent 1: Core Setup] Gen 1 - RUNNING

## Wave 1 Complete. Launching Wave 2...

[Agent 2: Database] Gen 1 - RUNNING
[Agent 3: Backend] Gen 1 - RUNNING → Gen 2 - RUNNING → Gen 3 - COMPLETED
[Agent 4: Auth] Gen 1 - COMPLETED
[Agent 5: Scheduler] Gen 1 - RUNNING → Gen 2 - COMPLETED

## Wave 2 Complete. Launching Wave 3...

[Agent 6: Frontend] Gen 1 - RUNNING → Gen 2 - COMPLETED
[Agent 7: Analytics] Gen 1 - COMPLETED

## Wave 3 Complete. Launching Wave 4...

[Agent 8: Integration Lead] Gen 1 - RUNNING

## Task Complete!

Agents: 8
Total Generations: 11 (across all agents)
Duration: 2h 15m
Success Patterns: 23
Anti-Patterns: 8

Output: /workspace/social-scheduler-gas/output/
```

### Example 2: Large Codebase Refactoring

```
User: /gas Refactor this legacy codebase to TypeScript

GAS v2: Analyzing codebase...

## Decomposition by Module

| Module | Agent | Files |
|--------|-------|-------|
| Core Utils | Agent 1 | 45 |
| Data Layer | Agent 2 | 32 |
| API Routes | Agent 3 | 28 |
| Services | Agent 4 | 51 |
| Components | Agent 5 | 67 |
| Tests | Agent 6 | 89 |

## Parallel Execution with Succession

Agent 3 (API Routes):
  Gen 1 → 15 files → [degraded] →
  Gen 2 → 10 files → [degraded] →
  Gen 3 → 3 files → [complete]

Agent 5 (Components):
  Gen 1 → 30 files → [degraded] →
  Gen 2 → 25 files → [degraded] →
  Gen 3 → 12 files → [complete]

## Final Report

Total Agents: 6
Total Generations: 14
Files Refactored: 312
Type Coverage: 96%
Shared Patterns: 34
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Agents waiting forever | Wave dependency stuck | Check previous wave status |
| Too many successions | Thresholds too sensitive | Increase interaction_limit |
| Knowledge not propagating | Store not shared | Check knowledge store path |
| Integration conflicts | Overlapping responsibilities | Refine agent boundaries |
| Dashboard not updating | Wrong GAS_MODE | Set GAS_MODE=swarm |
| **Dashboard shows "pending"/"Waiting..."** | **task_id not updated in gas-state.json** | **Update gas-state.json with task_id immediately after launching each agent** |
| Dashboard shows no live activity | Output files not found | Ensure task_id is correct; dashboard scans /tmp/claude-*/*/tasks/ |
| Agent cards not showing progress | Status files in wrong location | Agents must write status.json to both `agents/X/status.json` AND `agents/X/generations/gen-N/status.json` |

---

## Resources

- `resources/gas-config.yaml` - Configuration
- `resources/gas-dashboard-server.py` - Dashboard (swarm + generations)
- `templates/swarm-agent-prompt.md` - Agent template
- `templates/transfer-document.yaml` - Transfer format
- `examples/` - Example sessions

---

> **"Parallel power meets generational wisdom. Swarms that scale, agents that never degrade."**
