---
name: wave-planner
version: 1.4.0
description: Transform project issues into execution-ready implementation plans with risk prediction, wave-based organization, specialist agents, and TDD workflow
author: William Smith
triggers:
  keywords:
    - plan the work
    - break this down
    - create waves
    - implementation plan
    - plan the implementation
    - organize into waves
    - sprint planning
    - wave structure
  explicit:
    - /wave-planner
    - /plan
tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Task
  - AskUserQuestion
  - TodoWrite
composes:
  - linear
  - hive-mind
  - governance
---

# Wave Planner

Transform project issues into execution-ready implementation plans with wave-based organization, specialist agent assignments, token estimates, and TDD workflow integration.

---

## Behavioral Classification

**Type**: Guided Decision

This skill asks for your input on key decisions, then executes based on your choices.

**Decision Points**:
1. Which scope? (P0 only, P0+P1, All issues)
2. Architecture decisions requiring clarification
3. Create ADRs for decisions?
4. Risk mitigation strategies

After decisions are made, artifact generation proceeds automatically.

---

## Quick Start

```
# Explicit invocation
/wave-planner Phase 3 Live Data

# Or use natural language (proactive trigger)
"Plan the implementation for the security hardening project"
"Break down the API redesign into waves"
"Create an implementation plan for PROJ-100"
```

---

## What This Skill Does

1. **Checks** for existing artifacts (plans, configs, projects) - MANDATORY
2. **Discovers** issues from your PM tool (Linear, GitHub, Jira)
3. **Surfaces** architecture decisions requiring clarification
4. **Predicts** blockers and fail cases, generates mitigations
5. **Organizes** issues into waves based on shared context
6. **Estimates** tokens using dynamic codebase analysis
7. **Assigns** specialist agents via LLM inference
8. **Generates** execution-ready artifacts (or updates existing)

---

## Pre-Flight Checks (MANDATORY)

Before wave execution, verify the environment. See **[preflight.md](preflight.md)** for the full script.

**Quick check**:

| Check | Command | Expected |
|-------|---------|----------|
| MCP Server | `grep claude-flow .mcp.json` | Found |
| Docker | `docker ps \| grep <project-name>` | Running |
| Git-crypt | `git-crypt status` | Unlocked |
| Linear | `linear issues list --limit 1` | Returns issues |

---

## Artifact Discovery (MANDATORY)

**ALWAYS check for existing artifacts before creating new ones.** Skipping discovery causes duplicate plans, wasted effort, and confusion.

### Discovery Checklist

```bash
# 1. Check for existing implementation plans
ls docs/execution/ | grep -i "<project-name>"

# 2. Check for existing hive-mind configs
ls .claude/hive-mind/ | grep -i "<project-name>"

# 3. Check Linear for existing project
npx tsx ~/.claude/skills/linear/scripts/linear-ops.ts list-initiatives | grep -i "<project-name>"
linear projects list | grep -i "<project-name>"

# 4. Check for related ADRs
ls docs/adr/ | grep -i "<topic>"
```

### When Duplicates Are Found

| Scenario | Action |
|----------|--------|
| Plan exists, not started | Review and update existing |
| Plan exists, partially executed | Continue from current wave |
| Plan exists, completed | Archive and create new if scope changed |
| Config exists, not executed | Use existing config |
| Linear project exists | Reuse, don't create duplicate |

---

## Planning Workflow

### Phase 1: Discovery

The skill detects your PM tool and fetches project issues:

```
Found 12 issues in "Phase 3 Live Data":
- 2 P0-Critical
- 4 P1-High
- 6 P2-Medium

Which scope should we plan?
A) P0-Critical only (2 issues)
B) P0 + P1-High (6 issues)
C) All issues (12 issues)
```

### Phase 2: Architecture Decisions

Decisions are surfaced one at a time to reduce cognitive load:

```
PROJ-101 requires an architecture decision:

**Where should security scan results be stored?**

A) Separate security_scans table
B) JSON column on skills table
C) Dedicated security schema (Recommended)

Should I create an ADR for this decision?
```

### Phase 3: Risk Analysis

The skill analyzes issues for potential blockers and fail cases:

```
Identified Risks:

1. **External Dependency Risk** (PROJ-101)
   - Issue: Feature relies on third-party API
   - Mitigation: Add fallback or cache layer

2. **Breaking Change Risk** (PROJ-102)
   - Issue: Schema changes may break existing clients
   - Mitigation: Version schema, add migration script
```

**Risk Categories**: External Dependency, Breaking Change, Integration, Performance, Security

### Phase 4: Wave Organization

Issues are grouped by shared code context to minimize agent context-switching:

```
Wave 1: Foundation (~81K tokens)
├── PROJ-101: Error handling
├── PROJ-102: Configuration setup
└── Agent: backend-developer

Wave 2: Feature Implementation (~95K tokens)
├── PROJ-103: Core feature
├── PROJ-104: API endpoints
└── Agent: backend-developer
```

### Phase 5: Artifact Generation

| Artifact | Location |
|----------|----------|
| Implementation Plan | `docs/execution/{project}-implementation-plan.md` |
| ADRs | `docs/adr/{number}-{slug}.md` |
| Hive Configs | `.claude/hive-mind/{project}-wave-{n}.yaml` |
| PM Sub-Issues | Created for code review findings |

---

## Execution

After planning, execute waves. See **[execution.md](execution.md)** for full details.

**Quick start**:

```bash
# Execute Wave 1
./claude-flow swarm "Execute Wave 1" \
  --config .claude/hive-mind/{project}-wave-1.yaml \
  --strategy development \
  --mode hierarchical
```

**Wave Completion Checklist** (from execution.md):
1. Run code review: `docker exec <container-name> npm run audit:standards`
2. Fix findings OR create Linear sub-issues
3. Update Linear issue status to Done
4. Commit with Co-Authored-By
5. Create PR if wave is complete

---

## Reference Documentation

For detailed information, see **[reference.md](reference.md)**:

- Output artifact formats (implementation plan, hive configs, ADRs)
- Token estimation calculations and budgets
- Agent assignment logic and triggers
- TDD workflow (red-green-refactor)
- PM tool adapters
- Example planning session

---

## Configuration

```yaml
# ~/.claude/skills/wave-planner/config.yaml
wave-planner:
  defaults:
    max_waves: 6
    max_issues_per_wave: 5
    token_budget_per_wave: 150000

  adapters:
    preferred: auto

  estimation:
    confidence_threshold: medium
    review_cycle_count: 2

  artifacts:
    create_adrs: true
    create_hive_configs: true
    create_sub_issues: true
```

### Environment

```bash
# PM tool credentials (via adapter)
LINEAR_API_KEY=lin_api_xxx
GITHUB_TOKEN=ghp_xxx
```

---

## Skill Composition

This skill orchestrates:

| Skill | Usage |
|-------|-------|
| `linear` | PM operations (fetch, update, create) |
| `hive-mind` | Generate execution configs |
| `governance` | Code review after commits |

If a required skill is unavailable, the skill will prompt for installation.

---

## Troubleshooting

### "Linear skill not available"

Install the Linear skill to `~/.claude/skills/linear/`

### "Cannot detect PM tool"

Specify explicitly:
```
/wave-planner --adapter linear "Project Name"
```

### "Token estimate seems high"

Check confidence level. Low confidence estimates include padding for unknowns. Refine issue descriptions or break into smaller issues.

---

## Related Skills

- [linear](https://github.com/wrsmith108/linear-claude-skill) - PM operations
- hive-mind - Swarm execution *(local skill, available after installation)*
- [governance](https://github.com/wrsmith108/governance-claude-skill) - Code review

---

## Changelog

### v1.4.0 (2026-01-23)
- **Refactor**: Decompose into sub-files for progressive disclosure
- Extract pre-flight checks to preflight.md
- Extract execution details to execution.md
- Extract reference documentation to reference.md
- Add Behavioral Classification section (ADR-025)
- Main SKILL.md reduced from 785 to ~350 lines

### v1.3.0 (2026-01-23)
- **New**: Artifact Discovery phase (MANDATORY before creation)
- Check for existing implementation plans before creating
- Check for existing hive-mind configs before creating
- Check Linear for existing projects/issues
- Discovery workflow with options to review, update, or archive

### v1.2.0 (2026-01-21)
- **New**: Pre-Flight Checks section
- **New**: Automated Code Review Trigger configuration
- **Enhanced**: Wave Completion Checklist with Linear enforcement
- Added "The Deferred Issue Rule" - no deferral without Linear ticket

### v1.1.0 (2026-01-21)
- **New**: Risk Analysis phase
- Predict blockers and fail cases before wave organization
- Auto-generate mitigations for identified risks
- Risk register added to implementation plan output

### v1.0.0 (2026-01-21)
- Initial release
- Linear adapter support
- Dynamic token estimation
- TDD workflow integration
- Hive mind config generation
