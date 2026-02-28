---
name: refactor
description: Analyze code and suggest refactoring opportunities with rationale
---

# Refactoring Analysis

**Current Time:** !`date`
**Go Version:** !`go version`

Analyze code and suggest refactoring opportunities with rationale. Documents findings to Obsidian for tracking and discussion.

## Input

- Target: file path, directory/package, or function/component name
- Optional: specific concern (duplication, complexity, coupling, testability, etc.)

## Investigation Strategy

Launch parallel investigation tracks:

### Track 1: Codebase Exploration (explore agent)

- Map dependencies and call sites for target code
- Identify blast radius of potential changes
- Find related code with similar patterns
- Assess test coverage of affected areas

### Track 2: Code Analysis (inferred agent: go/frontend)

- Infer appropriate agent from target context
- Deep analysis of code smells and patterns
- Identify refactoring opportunities
- Assess complexity metrics

## Refactoring Patterns

### Universal Patterns

- **Extract function/method**: Pull out reusable logic
- **Inline function/variable**: Remove unnecessary indirection
- **Rename**: Improve clarity (with blast radius assessment)
- **Move**: Relocate to better home (file, package, module)
- **Replace conditional with polymorphism**: Simplify branching
- **Introduce parameter object**: Group related parameters
- **Dependency inversion**: Decouple via interfaces/abstractions

### Go-Specific Patterns

- **Extract interface**: Define behavior contracts
- **Consolidate error handling**: Reduce repetitive error checks
- **Replace concrete with interface**: Improve testability
- **Extract middleware**: Separate cross-cutting concerns
- **Table-driven refactor**: Convert repetitive code to data-driven

### Frontend-Specific Patterns

- **Extract component**: Break down large components
- **Extract custom hook**: Reuse stateful logic
- **Lift state up**: Move state to common ancestor
- **Push state down**: Colocate state with usage
- **Extract render function**: Simplify complex JSX
- **Memoization**: Optimize re-renders

## Output

Write to Obsidian via `obsidian_append_content` at:
`$OBSIDIAN_PATH/Refactoring/YYYY-MM-DD-target-name.md`

> **Note**: `$OBSIDIAN_PATH` must be a vault-relative path (e.g., `Projects/myapp`), set per-project via direnv. The `obsidian_append_content` tool expects paths relative to the vault root.

### Document Structure

Use this template for the Obsidian document:

@~/.config/opencode/templates/refactor-analysis.md

## Behavior

1. Parse target to determine scope (file, directory, function, component)
2. Infer appropriate code agent from target context (go vs frontend)
3. Launch explore and inferred code agent in parallel
4. Map dependencies, call sites, and blast radius
5. Identify code smells with specific locations
6. Generate refactoring suggestions with risk assessment
7. Recommend order of operations based on risk and dependencies
8. Write analysis to Obsidian via `obsidian_append_content` with auto-generated filename: `YYYY-MM-DD-target-name.md`

$ARGUMENTS
