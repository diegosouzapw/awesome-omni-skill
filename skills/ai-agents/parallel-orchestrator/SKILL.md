---
name: parallel-orchestrator
description: Orchestrate parallel agent workflows using HtmlGraph's ParallelWorkflow. Activate when planning multi-agent work, using Task tool for sub-agents, or coordinating concurrent feature implementation.
---

# Parallel Orchestrator Skill

## When to Activate This Skill

**Trigger keywords:**
- "parallel", "concurrent", "simultaneously"
- "multiple agents", "spawn agents", "Task tool"
- "work in parallel", "parallelize"
- "speed up", "faster completion"

**Trigger situations:**
- Planning work that could run concurrently
- Multiple independent features ready to implement
- Using Claude Code's Task tool for sub-agents
- Coordinating 2+ agents on different tasks

---

## Core Principle: 6-Phase Parallel Workflow

HtmlGraph provides `ParallelWorkflow` for optimal parallel agent execution:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. ANALYZE     →  Check dependencies, assess risks            │
│  2. PREPARE     →  Cache shared context, isolate tasks         │
│  3. DISPATCH    →  Generate prompts, spawn agents              │
│  4. MONITOR     →  Track health, detect anti-patterns          │
│  5. AGGREGATE   →  Collect results, check conflicts            │
│  6. VALIDATE    →  Verify outputs, update dependencies         │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Pre-Flight Analysis (REQUIRED)

**Always run analysis before dispatching agents!**

```python
from htmlgraph import SDK

sdk = SDK(agent="orchestrator")

# Method 1: Quick check
parallel = sdk.get_parallel_work(max_agents=5)
print(f"Max parallelism: {parallel['max_parallelism']}")
print(f"Ready now: {parallel['ready_now']}")

# Method 2: Full analysis with ParallelWorkflow
plan = sdk.plan_parallel_work(max_agents=3)

if plan["can_parallelize"]:
    print(f"✅ Parallelize {plan['task_count']} tasks")
    print(f"   Speedup: {plan['speedup_factor']:.1f}x")
    print(f"   Ready: {plan['ready_tasks']}")
else:
    print(f"⚠️ {plan['recommendation']}")
```

### Decision Criteria

| Condition | Action |
|-----------|--------|
| `max_parallelism >= 2` | Can parallelize |
| `len(ready_tasks) < 2` | Work sequentially |
| Shared file edits | Partition or sequence |
| `speedup < 1.5x` | May not be worth cost |

---

## Phase 2: Context Preparation

**Reduce redundant file reads by pre-caching shared context:**

```python
# Identify files ALL agents need
shared_files = [
    "src/models.py",      # Data models
    "src/config.py",      # Configuration
    "tests/conftest.py",  # Test fixtures
]

# Plan with shared context
plan = sdk.plan_parallel_work(
    max_agents=3,
    shared_files=shared_files
)
```

### What Preparation Does

1. **Reads shared files once** (not per-agent)
2. **Generates summaries** for agent context
3. **Identifies file conflicts** before dispatch
4. **Creates isolation rules** (which files each agent owns)

---

## Phase 3: Dispatch with Task Tool

**CRITICAL: Send ALL Task calls in a SINGLE message for true parallelism!**

```python
# Get ready-to-use prompts
prompts = plan["prompts"]

# CORRECT: All in one message (parallel)
for p in prompts:
    Task(
        subagent_type="general-purpose",
        prompt=p["prompt"],
        description=p["description"]
    )

# WRONG: Sequential messages (not parallel)
# result1 = Task(...)  # Wait for completion
# result2 = Task(...)  # Then next one
```

### Prompt Structure (Auto-Generated)

Each prompt includes:
```markdown
## Task: {feature_id}
Title: {title}
Priority: {priority}

## Your Assignment
{specific_instructions}

## Pre-Cached Context (DO NOT re-read these)
- models.py: Contains User, Session, Feature classes
- config.py: DATABASE_URL, API_KEY settings

## Files to AVOID (other agents editing)
- {files_assigned_to_other_agents}

## Efficiency Guidelines
- Use Grep before Read (search then read)
- Batch Edit operations
- Mark feature complete when done
```

---

## Phase 4: Monitor (During Execution)

Agents track their own health via transcript analytics:

### Health Metrics Tracked

| Metric | Healthy | Warning |
|--------|---------|---------|
| Retry rate | < 30% | > 30% |
| Context rebuilds | < 5 | > 5 |
| Tool diversity | > 30% | < 30% |
| Overall health | > 70% | < 70% |

### Anti-Patterns Detected

```python
# These patterns trigger warnings:
("Read", "Read", "Read")     # Cache instead
("Edit", "Edit", "Edit")     # Batch edits
("Bash", "Bash", "Bash", "Bash")  # Check errors
("Grep", "Grep", "Grep")     # Read results first
```

---

## Phase 5: Aggregate Results

**After all agents complete:**

```python
# Collect agent IDs from Task tool responses
agent_ids = ["agent-abc123", "agent-def456", "agent-ghi789"]

# Aggregate with SDK
results = sdk.aggregate_parallel_results(agent_ids)

print(f"Successful: {results['successful']}/{results['total_agents']}")
print(f"Health: {results['avg_health_score']:.0%}")
print(f"Speedup: {results['parallel_speedup']:.1f}x")
print(f"Conflicts: {results['conflicts']}")
print(f"Anti-patterns: {results['total_anti_patterns']}")
```

### Result Structure

```python
{
    "total_agents": 3,
    "successful": 3,
    "failed": 0,
    "total_duration_seconds": 450.0,
    "parallel_speedup": 2.3,
    "avg_health_score": 0.80,
    "total_anti_patterns": 4,
    "files_modified": ["auth.py", "api.py", "tests/..."],
    "conflicts": [],  # Empty = good!
    "recommendations": [...],
    "validation": {
        "no_conflicts": True,
        "all_successful": True,
        "healthy_execution": True,
    },
    "all_passed": True
}
```

---

## Phase 6: Validate

```python
if results["all_passed"]:
    print("✅ Parallel execution validated!")
    # Commit all changes together
else:
    # Handle issues
    for rec in results["recommendations"]:
        print(f"⚠️ {rec}")
```

---

## Optimal Tool Patterns

### DO (Efficient)

| Pattern | Why |
|---------|-----|
| `Grep → Read` | Search before reading |
| `Read → Edit → Bash` | Read, modify, test |
| `Glob → Read` | Find files first |
| Single Task message | True parallelism |

### DON'T (Anti-Patterns)

| Pattern | Problem | Fix |
|---------|---------|-----|
| `Read → Read → Read` | Redundant reads | Cache content |
| `Edit → Edit → Edit` | Unbatched | Combine edits |
| Sequential Task calls | No parallelism | Single message |
| Overlapping files | Conflicts | Isolate scope |

---

## When NOT to Parallelize

| Situation | Reason | Alternative |
|-----------|--------|-------------|
| Shared dependencies | Conflicts | Sequential + handoff |
| Tasks < 1 minute | Overhead not worth it | Sequential |
| Overlapping files | Merge conflicts | Partition files |
| Complex coordination | Risk of errors | Plan agent |

---

## Integration with Other Skills

### With `htmlgraph-tracker`
- Parallel agents each track their own session
- Activities attributed to features automatically
- Drift detection per agent

### With `strategic-planning`
- Use `find_bottlenecks()` before parallel dispatch
- Prioritize work that unlocks most downstream tasks
- Assess risks before large parallel batches

---

## Quick Reference

```python
from htmlgraph import SDK

sdk = SDK(agent="orchestrator")

# 1. Plan
plan = sdk.plan_parallel_work(max_agents=3)

# 2. Check
if plan["can_parallelize"]:
    # 3. Dispatch (all at once!)
    for p in plan["prompts"]:
        Task(prompt=p["prompt"], ...)

    # 4. Aggregate (after completion)
    results = sdk.aggregate_parallel_results(agent_ids)

    # 5. Validate
    if results["all_passed"]:
        print("✅ Success!")
```

---

## Troubleshooting

### "Not enough independent tasks"
- Check dependency graph: `sdk.get_parallel_work()`
- Resolve bottlenecks first: `sdk.find_bottlenecks()`

### High anti-pattern count
- Add shared file caching
- Review agent prompts for efficiency guidelines

### File conflicts detected
- Improve task isolation in Phase 2
- Consider sequential execution for overlapping work

### Low health scores
- Check retry rates (consecutive same-tool usage)
- Add more context to agent prompts
- Use Grep before Read pattern
