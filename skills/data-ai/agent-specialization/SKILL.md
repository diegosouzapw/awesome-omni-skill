---
name: agent-specialization
description: Guide creation of focused single-purpose agents following the One Agent One Prompt One Purpose principle. Use when designing new agents, refactoring general agents into specialists, or optimizing agent context for a single task.
allowed-tools: Read, Grep, Glob
---

# Agent Specialization Skill

Guide for creating focused, single-purpose agents that maximize effectiveness.

## When to Use

- Designing new agents for workflows
- Refactoring god-mode agents into specialists
- Optimizing agent context usage
- Creating eval-friendly agent architecture

## Core Principle

> "One Agent, One Prompt, One Purpose"

Every agent should:

- Have exactly one purpose
- Run exactly one prompt
- Use the full context window for that purpose
- Be reproducible and improvable

## Design Workflow

### Step 1: Identify the Single Purpose

Ask: "What is the ONE question this agent answers?"

| Good Purpose | Bad Purpose |
| --- | --- |
| "Classify this issue" | "Classify, plan, and implement" |
| "Generate a patch plan" | "Fix all the bugs" |
| "Review against spec" | "Review, test, and document" |

### Step 2: Determine Minimum Required Context

Apply the Minimum Context Principle:

```markdown
## Required Context
- [Specific file or section needed]
- [Pattern or example needed]

## NOT Needed
- [Documentation that's irrelevant]
- [Code that won't be touched]
```

### Step 3: Select Appropriate Tools

Only include tools the agent will actually use:

| Purpose | Tools |
| --- | --- |
| Classification | Read |
| Planning | Read, Write, Glob |
| Implementation | Read, Write, Edit, Bash |
| Review | Read, Bash, Glob |
| Documentation | Read, Write |

### Step 4: Choose Model

Match model to task complexity:

| Model | Best For |
| --- | --- |
| haiku | Classification, simple extraction |
| sonnet | Planning, moderate reasoning |
| opus | Complex implementation, critical decisions |

### Step 5: Design Focused Output Format

Output should be:

- Structured (JSON when appropriate)
- Minimal (only what downstream needs)
- Parseable (for automation)

## Agent Template

```markdown
---
description: [Single sentence describing the ONE purpose]
tools: [Only tools actually needed]
model: [haiku/sonnet/opus based on complexity]
---

# [Agent Name]

You are a [role] agent. Your ONE purpose is to [specific task].

## Your Capabilities

- **[Tool]**: [How it supports the purpose]

## Process

[Focused steps for the single purpose]

## Output Format

[Structured output format]

## Rules

1. [Constraint that maintains focus]
2. [Another constraint]
```

## Anti-Patterns to Avoid

### God Mode Agent

```markdown
# BAD: Does everything
You are an all-purpose assistant. Plan features,
implement code, write tests, review changes, and
create documentation. Handle any request.
```

### Unfocused Output

```markdown
# BAD: Returns everything
Return a detailed analysis including history,
context, alternatives, implications, and
recommendations for all stakeholders.
```

### Kitchen Sink Tools

```markdown
# BAD: All tools enabled
tools: [Read, Write, Edit, Bash, Glob, Grep, WebFetch, Task, ...]
```

## Benefits of Specialization

1. **Full Context Window**: 100% for the task
2. **No Context Confusion**: Single objective
3. **Reproducible**: Same prompt, same behavior
4. **Improvable**: Can optimize independently
5. **Eval-Friendly**: Can A/B test models
6. **Debuggable**: Clear scope of responsibility

## Example: Specialized vs God Mode

### God Mode (Bad)

```markdown
Handle the GitHub issue:
1. Classify it
2. Create a branch
3. Plan the implementation
4. Implement the feature
5. Write tests
6. Run tests
7. Review the implementation
8. Fix any issues
9. Create documentation
10. Create a PR
```

### Specialized (Good)

```text
/classify-issue → Issue Classifier Agent
/generate-branch-name → Branch Namer Agent
/feature → Plan Generator Agent
/implement → Plan Implementer Agent
/test → Test Runner Agent
/review → Spec Reviewer Agent
/patch → Patch Planner Agent
/document → Documentation Generator Agent
/pull-request → PR Creator Agent
```

Each agent does ONE thing well.

## Memory References

- @one-agent-one-purpose.md - Full principle documentation
- @minimum-context-principle.md - Context engineering guidance
- @review-vs-test.md - Example of different purposes

## Version History

- **v1.0.0** (2025-12-26): Initial release

---

## Last Updated

**Date:** 2025-12-26
**Model:** claude-opus-4-5-20251101
