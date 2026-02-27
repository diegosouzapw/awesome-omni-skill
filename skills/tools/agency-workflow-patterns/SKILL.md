---
name: agency-workflow-patterns
description: Master orchestration patterns, multi-agent coordination, and effective workflow composition using the Agency plugin's 51+ specialized agents. Activate when planning complex implementations, coordinating multiple agents, or optimizing development workflows.
triggers:
  - "agency patterns"
  - "workflow orchestration"
  - "using agency"
  - "agent coordination"
  - "how to use orchestrator"
  - "multi-agent workflow"
  - "parallel execution"
  - "agency best practices"
---

# Agency Workflow Patterns

Master the art of orchestrating 51+ specialized agents to automate your complete software development lifecycle. This skill teaches you when to use the orchestrator, how to coordinate multiple agents, and patterns for maximum efficiency.

## Core Orchestration Principles

### When to Use the Orchestrator

**Use orchestrator for**:
- Multi-step workflows spanning multiple domains
- Complex tasks requiring different specialist expertise
- Tasks where parallel execution provides value
- Workflows needing strategic planning before execution

**Use specialist agents directly for**:
- Single-domain tasks with clear scope
- Quick focused work (< 30 minutes)
- Tasks where you know exactly which specialist to use

### The Orchestration Flow

```
User Command → Orchestrator → Planning → Specialist Selection → Parallel Execution → Synthesis
```

1. **Intake**: Orchestrator classifies request type
2. **Planning**: Decomposes into discrete, delegatable tasks
3. **Selection**: Matches tasks to optimal specialists
4. **Execution**: Spawns agents (parallel when possible)
5. **Synthesis**: Collects outputs and presents unified results

## Common Workflow Patterns

### Pattern 1: Investigation & Research

**Use Case**: Understanding existing code or systems

**Flow**:
```
User Question → Orchestrator → Explore Agent → Summary
```

**Example**:
- User: "How does authentication work in this codebase?"
- Orchestrator spawns `Explore` agent
- Explore agent finds relevant files, traces flow
- Returns summary with key file references

**Key Insight**: Use `Explore` agent to keep main context clean

### Pattern 2: Issue Implementation (Full Cycle)

**Use Case**: GitHub/Jira issue from start to finish

**Flow**:
```
Issue → Fetch Details → Plan → Review Plan → Implement → Test → Review → PR
```

**Agents Involved**:
- Orchestrator (coordination)
- Plan agent (planning phase)
- Specialist reviewer (architecture review)
- Specialist coder (implementation)
- Reality Checker (quality verification)

**Example Command**: `/agency:work 123`

**Checkpoints**:
1. After planning (confirm approach)
2. After implementation (verify changes)
3. Before PR creation (final approval)

See [Full Issue Implementation Flow](references/issue-implementation-flow.md) for details.

### Pattern 3: Parallel Implementation

**Use Case**: Multiple independent features or bug fixes

**Flow**:
```
Identify Parallel Tasks → Spawn Specialists Simultaneously → Integrate Results
```

**Example**:
- User: "Implement user dashboard with charts, filters, and export"
- Orchestrator identifies 3 independent tracks:
  - Track A: Frontend Developer → Chart components
  - Track B: Frontend Developer → Filter components
  - Track C: Backend Architect → Export API
- All spawn simultaneously
- Integrate when complete

**Benefits**:
- 3x faster than sequential
- Specialists work in isolation
- Clean context separation

See [Parallel Execution Strategies](references/parallel-execution.md) for details.

### Pattern 4: Quality Review Pipeline

**Use Case**: Comprehensive code review before release

**Flow**:
```
Code → Security Review → Quality Review → Performance Review → Test Review → Synthesis
```

**Agents in Parallel**:
- Legal Compliance Checker (security)
- Reality Checker (quality & bugs)
- Performance Benchmarker (performance)
- Test Results Analyzer (test coverage)

**Example Command**: `/agency:review`

**Output**: Unified report with all findings

### Pattern 5: Sprint Automation

**Use Case**: Implement entire sprint

**Flow**:
```
Fetch Sprint Issues → Prioritize → Find Parallel Work → Execute in Batches → Monitor Progress
```

**Example Command**: `/agency:gh-sprint`

**Orchestrator Actions**:
1. Fetch all sprint issues
2. Analyze dependencies
3. Group into parallel batches
4. Execute batches sequentially
5. Report progress and blockers

See [Sprint Automation Workflow](references/sprint-automation.md) for details.

## Agent Selection Guidelines

### Engineering Agents

| Agent | Best For | Avoid For |
|-------|----------|-----------|
| **Frontend Developer** | React, Vue, UI components | Backend logic, databases |
| **Backend Architect** | APIs, databases, system design | UI implementation |
| **AI Engineer** | ML pipelines, model integration | General web development |
| **DevOps Automator** | CI/CD, Docker, infrastructure | Application code |
| **Senior Developer** | Complex logic, difficult problems | Simple CRUD |
| **Rapid Prototyper** | MVPs, proof-of-concepts | Production systems |

### Design Agents

| Agent | Best For | Avoid For |
|-------|----------|-----------|
| **UI Designer** | Visual design, component libraries | User research |
| **UX Researcher** | User needs, usability testing | Visual design |
| **UX Architect** | Information architecture, flows | Implementation |

### Testing Agents

| Agent | Best For | Avoid For |
|-------|----------|-----------|
| **Reality Checker** | Bug finding, edge cases | Performance testing |
| **API Tester** | API contract validation | UI testing |
| **Performance Benchmarker** | Load testing, optimization | Functional testing |

See [Complete Agent Roster](references/agent-roster.md) for all 51+ agents.

## Context Management

### Keep Orchestrator Context Lean

**Retain**:
- Task status (pending/in-progress/complete)
- Output artifact locations (file paths)
- Key decisions made
- Blockers requiring user input

**Discard**:
- Full file contents from agents
- Implementation details (live in agent outputs)
- Verbose logs and debug output

**Target Context Sizes**:
- Investigation: < 2K tokens
- Quick Fix: < 1K tokens
- Feature Plan: < 4K tokens
- Full Implementation: < 6K tokens

### Why This Matters

**Benefits of Lean Context**:
- Faster responses
- Lower costs
- Better focus
- Can handle more complex workflows

**How to Achieve**:
- Use `Explore` agent for research
- Agents return summaries, not full outputs
- File paths instead of contents
- Key decisions, not reasoning traces

## Parallel vs Sequential Execution

### When to Execute in Parallel

**✅ Parallelize When**:
- Tasks are independent (no shared state)
- Different domains/specialists
- No sequential dependencies
- Time-sensitive work

**Example**: UI components + API endpoints + Database schema

### When to Execute Sequentially

**⛔ Sequential When**:
- Tasks have dependencies (A must finish before B)
- Shared state mutations
- Need output from previous task
- Risk of conflicts

**Example**: Plan → Review Plan → Implement → Test

See [Execution Strategies](references/execution-strategies.md) for decision trees.

## Checkpoint Strategy

### User Approval Points

**Always pause for approval**:
- After planning phase (before implementation)
- Before destructive operations (database migrations)
- When ambiguity detected
- When user explicitly requested ("plan only")

**Skip approval for**:
- Simple fixes (< 20 LOC)
- User said "just do it" or "auto-approve"
- Similar patterns previously approved

**Present concisely**:
```
📋 Plan: Dark Mode Implementation

I'll use 4 agents across 3 phases:

Phase 1: Research
└── Explore → Find existing theme patterns

Phase 2: Implementation (Parallel)
├── Frontend Developer → Theme provider + toggle
├── Frontend Developer → Component style updates
└── Backend Architect → User preference API

Phase 3: Verification
└── Reality Checker → Visual regression tests

Checkpoints: After Phase 1 (confirm approach), After Phase 3 (final review)

Approve? (y/auto/modify)
```

## Advanced Patterns

### Pattern: Iterative Refinement

**Use Case**: Complex feature needing multiple refinement cycles

**Flow**:
```
Initial Plan → Review → Refine Plan → Implement → Test → Refine → Test → Ship
```

**When to Use**: Novel features, unclear requirements, high uncertainty

### Pattern: Research-First Implementation

**Use Case**: Working with unfamiliar codebase or technology

**Flow**:
```
Research → Document Findings → Plan → Implement
```

**Research Phase**:
- Explore agent finds relevant code
- Documents patterns and conventions
- Identifies dependencies
- Creates knowledge base

**Then**: Implementation uses research as foundation

### Pattern: Specialist Cascade

**Use Case**: Progressive deepening of expertise

**Flow**:
```
Senior Developer (initial) → Specialist (refinement) → Another Specialist (integration)
```

**Example**:
1. Senior Developer: Initial implementation
2. Performance Benchmarker: Optimization pass
3. Security Compliance: Security hardening

See [Advanced Orchestration](references/advanced-patterns.md) for more patterns.

## Troubleshooting

### Issue: Too Many Context Switches

**Symptom**: Orchestrator slow, losing track
**Solution**: Reduce parallel agents, increase batch size

### Issue: Wrong Agent Selected

**Symptom**: Agent struggles with task
**Solution**: Review agent selection criteria, may need different specialist

### Issue: Workflow Stalled

**Symptom**: Agent waiting for input, unclear blocking
**Solution**: Check for missing user approval, clarify requirements

See [Troubleshooting Guide](references/troubleshooting.md) for common issues.

## Best Practices

1. **Start with Planning**: Always create plan before implementation
2. **Use Orchestrator for Complexity**: Let orchestrator handle multi-step workflows
3. **Parallelize Aggressively**: Identify independent work
4. **Keep Context Lean**: Use agents to isolate work
5. **Checkpoint Strategically**: Pause for approval at key points
6. **Match Agent to Task**: Use right specialist for the job
7. **Document Decisions**: Capture key choices in artifacts
8. **Iterate on Feedback**: Refine based on results

## Quick Reference

**Investigation**: `Explore` agent → summary
**Issue Implementation**: `/agency:work {number}`
**Sprint Automation**: `/agency:gh-sprint`
**Code Review**: `/agency:review`
**Parallel Work**: `/agency:gh-parallel`
**Planning Only**: `/agency:plan`

## Related Skills

- `github-workflow-best-practices` - GitHub-specific workflows
- `code-review-standards` - Review criteria and quality gates
- `testing-strategy` - Testing approaches and standards

---

**Remember**: The orchestrator is a strategic coordinator, not an executor. Let specialists do the work while orchestrator manages the big picture.
