---
name: core-life-ops
description: Core productivity workflows (daily planning, task triage, reporting) that work across any life domain. Not intended for direct use - imported by context-specific skills like covenant-marketing-ops or personal-life-ops.
---

# Core Life Operations

## Overview

This skill provides domain-agnostic productivity workflows that can be imported and configured by context-specific skills. It handles Linear integration, agent coordination, and common productivity patterns.

**Not user-facing:** This skill is imported by other skills, not invoked directly.

## Configuration Interface

Importing skills must provide these configuration values:

```bash
# Required
LINEAR_TEAM_NAME="Team Name"          # or LINEAR_TEAM_ID
REPO_PATH="/path/to/narrative/repo"   # Documentation repository

# Optional
LABEL_PREFIX=""                       # Prefix for custom labels
WORKFLOW_CUSTOMIZATIONS=""            # JSON override config
```

## Available Workflows

### Daily Planning Workflow

**Import:** `workflows/daily-planning.md`

**What it does:**
- Queries Linear for today's tasks (filtered by configured team)
- Analyzes deadlines and priorities
- Generates time-boxed schedule
- Returns structured plan to calling skill

**Usage from importing skill:**
```markdown
[Import and execute core daily planning workflow]

Source: ~/.claude/skills/core-life-ops/workflows/daily-planning.md
Config: LINEAR_TEAM_NAME="${YOUR_TEAM_NAME}"
```

### Task Triage Workflow

**Import:** `workflows/task-triage.md`

**What it does:**
- Fetches untriaged items from Linear team
- Categorizes by type, priority, project
- Suggests routing and labels
- Updates Linear with approved changes

**Usage from importing skill:**
```markdown
[Import and execute core task triage workflow]

Source: ~/.claude/skills/core-life-ops/workflows/task-triage.md
Config: LINEAR_TEAM_NAME="${YOUR_TEAM_NAME}"
```

### Productivity Reporting Workflow

**Import:** `workflows/reporting.md`

**What it does:**
- Aggregates completed tasks by time period
- Calculates velocity metrics
- Identifies trends and bottlenecks
- Generates formatted report

**Usage from importing skill:**
```markdown
[Import and execute core reporting workflow]

Source: ~/.claude/skills/core-life-ops/workflows/reporting.md
Config: LINEAR_TEAM_NAME="${YOUR_TEAM_NAME}"
```

## Linear Integration

This skill reuses `linear-workflow-manager` for all Linear GraphQL queries.

**Required:**
- linear-workflow-manager skill installed at `~/.claude/skills/linear-workflow-manager`
- LINEAR_API_KEY configured in `~/.claude/.env`

**Query Helper:**
```bash
~/.claude/skills/linear-workflow-manager/scripts/linear.sh query queries/planning/my-priorities.json
```

See `references/linear-integration.md` for detailed query patterns.

## Agent Coordination Patterns

See `references/agent-patterns.md` for reusable patterns for:
- Launching planning agents
- Coordinating multi-stage workflows
- Parallel vs sequential agent execution
