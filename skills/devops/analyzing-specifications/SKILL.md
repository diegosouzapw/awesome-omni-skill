---
name: analyzing-specifications
description: Use when analyzing requirements or project specifications - guides shannon analyze command, explains 8D complexity output, caching behavior, context-aware mode with --project flag
---

# Analyzing Specifications

## Overview

shannon analyze invokes Shannon Framework spec-analysis skill to perform 8D complexity analysis with automatic caching.

## Basic Usage

```bash
shannon analyze project_spec.md
shannon analyze spec.md --json > analysis.json
shannon analyze spec.md --project myapp  # Context-aware
```

## Output Interpretation

**8D Complexity Score** (0.00-1.00):
- 0.00-0.30: Simple (hours-1 day, 1-2 agents)
- 0.30-0.50: Moderate (1-2 days, 2-3 agents)
- 0.50-0.70: Complex (2-4 days, 3-7 agents)
- 0.70-0.85: High (1-2 weeks, 8-15 agents)
- 0.85-1.00: Critical (2+ weeks, 15-25 agents)

**Domain Distribution**: Frontend, Backend, Database, Mobile, DevOps percentages

**Phase Plan**: 5 phases with timelines and tasks

## Caching

Automatic:
- First run: API call (~$0.02, 30-60s)
- Second run: Cache hit (<500ms, $0.00)
- TTL: 7 days
- Check: shannon cache stats

## Context-Aware Mode

For existing projects:
```bash
# 1. Onboard project first
shannon onboard /path/to/project --project-id myapp

# 2. Analyze with context
shannon analyze new_feature.md --project myapp
```

With context: Mentions existing modules, patterns, tech stack.
Without context: Generic recommendations.

## Options

- `--json`: JSON output for automation
- `--project ID`: Context-aware analysis
- `--no-cache`: Skip cache (force fresh analysis)
- `--session-id ID`: Custom session tracking
