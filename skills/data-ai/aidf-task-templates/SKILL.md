---
name: aidf-task-templates
description: Task template definitions for AIDF. Provides structured templates for component, refactor, test, docs, architecture, and bugfix task types.
version: 1.0.0
author: AIDF
tags: templates, tasks, project-management, workflow
globs: .ai/tasks/**, .ai/templates/**
---

# AIDF Task Templates

Structured templates for AI-assisted development tasks. Each task type has a specific template with goal, scope, requirements, and definition of done.

## Available Task Types

### Component
New feature or component implementation. Includes scope definition, implementation requirements, and quality gates.

### Refactor
Code restructuring without changing behavior. Requires before/after comparison and all existing tests must pass.

### Test
Adding or improving test coverage. Defines coverage targets, test categories, and quality thresholds.

### Docs
Documentation writing or improvement. Accuracy verification, working examples, and audience targeting.

### Architecture
System design and planning. Trade-off analysis, component diagrams, and migration paths.

### Bugfix
Bug reproduction and fix. Requires root cause analysis, fix verification, and regression test.

## Task Structure

Every task follows this structure:

```markdown
# TASK: [Name]

## Goal
[What needs to be accomplished]

## Task Type
[component|refactor|test|docs|architecture|bugfix]

## Suggested Roles
- [role names]

## Scope

### Allowed
- [file patterns]

### Forbidden
- [file patterns]

## Requirements
[Detailed requirements]

## Definition of Done
- [ ] [Criteria 1]
- [ ] [Criteria 2]
```
