---
name: thinking-sequentially
description: AI agent structures complex reasoning through numbered thought sequences with explicit dependencies. Use when facing multi-step problems, complex debugging, or architectural decisions.
category: methodology
triggers:
  - sequential thinking
  - step by step
  - reasoning chain
  - thought process
  - complex problem
---

# Thinking Sequentially

## Purpose

Structure complex reasoning through numbered thought sequences:

- Decompose multi-step problems into traceable chains
- Track dependencies between insights explicitly
- Create checkpoints for periodic review
- Enable revision when assumptions change
- Support parallel hypothesis exploration

## Quick Start

1. **Define** - State the problem clearly as THOUGHT 1
2. **Decompose** - Break into numbered thoughts with dependencies
3. **Track** - Mark dependencies explicitly: "depends on THOUGHT 2"
4. **Checkpoint** - Summarize every 5-10 thoughts
5. **Revise** - Update thoughts when assumptions change, invalidate dependents
6. **Branch** - Explore parallel hypotheses when needed

## Features

| Feature | Description | Guide |
|---------|-------------|-------|
| Numbered Chains | Explicit THOUGHT 1, 2, 3... with IDs | Number every insight, never skip |
| Dependencies | Mark what each thought relies on | `[THOUGHT 4] <- depends on [2, 3]` |
| Checkpoints | Periodic summaries of findings | Every 5-10 thoughts, list key findings |
| Revisions | Update previous thoughts with new data | Mark original as revised, create new |
| Branching | Explore multiple hypotheses in parallel | Create named branches, select winner |
| Visual Maps | Diagram thought relationships | ASCII diagrams for complex chains |

## Common Patterns

```
# Problem Analysis Pattern
THOUGHT 1: Problem statement
THOUGHT 2: Success criteria
THOUGHT 3: Constraints
THOUGHT 4: Assumptions (validate!)
THOUGHT 5-N: Solution exploration
=== CHECKPOINT ===

# Decision Making Pattern
THOUGHT 1: Decision to make
THOUGHT 2: Options identified
THOUGHT 3-N: Evaluate each option
THOUGHT N+1: Comparison matrix
THOUGHT N+2: Recommendation

# Investigation Pattern
THOUGHT 1: What we observed
THOUGHT 2: What we expected
THOUGHT 3: Gap analysis
THOUGHT 4-N: Hypothesis testing
=== CHECKPOINT: Root cause ===
```

```
# Visual thought mapping
        [THOUGHT 1: Problem]
               |
    +----------+----------+
    |                     |
[THOUGHT 2]         [THOUGHT 3]
    |                     |
    +----------+----------+
               |
        [THOUGHT 4: Synthesis]
```

## Use Cases

- Multi-step debugging requiring traceable reasoning chains
- Architectural decisions with complex trade-off analysis
- Code review requiring systematic examination of multiple concerns
- Root cause analysis with hypothesis tracking
- Complex refactoring requiring dependency-aware planning

## Best Practices

| Do | Avoid |
|----|-------|
| Number every thought explicitly | Skipping numbers for "obvious" thoughts |
| Mark dependencies clearly | Hiding dependencies in prose |
| Summarize at checkpoints | Exceeding 20 thoughts without checkpoint |
| Allow revision with reason | Deleting invalid thoughts (strike through) |
| Time-box each thought | Infinite exploration without bounds |
| Branch for parallel exploration | Branching more than 3 levels deep |
| Validate assumptions early | Proceeding with invalidated dependencies |

## Related Skills

See also these related skill documents for complementary techniques:

- **writing-plans** - Structure plans as numbered thought sequences
- **debugging-systematically** - Apply sequential debugging steps
- **tracing-root-causes** - Use numbered investigation chains
- **brainstorming-ideas** - Generate options in thought steps
