---
name: Plan Dependency Graph
description: Build and analyze dependency graphs across plans — identify blocking dependencies, critical paths, parallel work opportunities, and stale plans
category: planning
version: 1.0.0
triggers:
  - plan-created
  - plan-updated
  - milestone-event
globs: ".cursor/plans/**"
---

# Plan Dependency Graph Skill

Analyze all active plans to build a dependency graph, identify the critical path, and detect stale or blocked plans.

## Trigger Conditions
- New plan is created
- Existing plan todos are updated
- Milestone event occurs
- User invokes with "plan triage" or "plan-dependency-graph"

## Input Contract
- **Required:** Path to plans directory (`.cursor/plans/`)
- **Optional:** Specific plan to analyze

## Output Contract
- Complete plan inventory with status summary
- Dependency graph (which plans/todos block others)
- Critical path identification
- Stale plan detection (no updates > 7 days)
- Archive candidates (all todos completed)
- Blocked plan identification (dependencies unresolved)
- Parallel work opportunities (independent plans that can execute simultaneously)

## Tool Permissions
- **Read:** All plan files in `.cursor/plans/`
- **Write:** Plan status updates, archive operations
- **Search:** Grep for plan references, todo statuses

## Execution Steps

1. **Inventory plans**: Scan `.cursor/plans/` for all `.plan.md` files (excluding archive)
2. **Parse status**: For each plan, extract todos with their statuses
3. **Detect dependencies**: Find cross-plan references (one plan's todo depends on another plan)
4. **Build graph**: Create dependency graph of plans and their todos
5. **Find critical path**: Identify the longest chain of blocking dependencies
6. **Detect staleness**: Flag plans with no todo updates in 7+ days
7. **Identify completions**: Plans with all todos completed are archive candidates
8. **Report**: Produce triage report with actionable recommendations

## Plan Health Scores
- **Healthy**: Active progress, no stale todos, no blockers
- **Stale**: No updates in 7+ days
- **Blocked**: Has pending todos with unresolved dependencies
- **Complete**: All todos done — candidate for archival
- **Abandoned**: No updates in 30+ days with pending todos

## References
- `.cursor/rules/130-concern-trigger-registry.mdc`
- `.cursor/agents/planning.md`
