---
name: analyze
description: Deep analysis mode - thorough multi-phase investigation with expert consultation for complex problems requiring careful examination
license: MIT
compatibility:
  - runtime:any
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(git:log)
  - Bash(git:blame)
  - Bash(git:show)
metadata:
  author: thoreinstein
  version: 1.1.0
---

# Deep Analysis Mode

Perform a comprehensive analysis using multi-phase investigation and structured synthesis.

## HARD CONSTRAINTS (NON-NEGOTIABLE)

- **READ-ONLY MODE** — This skill is for analysis, not implementation
- **NO CODE WRITING** — Do not write, edit, or modify any source code
- **NO IMPLEMENTATION** — Do not implement features, fix bugs, or make changes
- **OUTPUT IS DOCUMENTATION** — Your deliverable is analysis/plan documents only

If you find yourself wanting to write code, STOP. Analysis produces documents, not software.

## When to Use This Skill

- Before major refactoring or architectural changes
- When evaluating unfamiliar code for risks or technical debt
- During security or performance audits
- When making build-vs-buy or technology decisions
- To produce a documented assessment for stakeholder review
- **As the `/analyze` command** to produce implementation plans for `/implement`

## Workflow

### Phase 1: Reconnaissance

Explore the target area to build context:

1. **Map the structure:**
   - Identify relevant files, modules, and their relationships
   - Understand the dependency graph and data flow

2. **Find patterns:**
   - Look for recurring code patterns (both good and concerning)
   - Identify conventions and deviations from them

3. **Gather context:**
   - Review git history for recent changes and contributors
   - Check for related documentation, comments, or TODOs

### Phase 2: Domain Analysis

Analyze the target across these dimensions:

| Domain           | Focus Areas                                      |
| ---------------- | ------------------------------------------------ |
| **Architecture** | System design, data flow, component dependencies |
| **Security**     | Vulnerabilities, threat model, input validation  |
| **Reliability**  | Scalability, failure modes, error handling       |
| **Performance**  | Bottlenecks, complexity, resource usage          |
| **Code Quality** | Patterns, anti-patterns, maintainability         |

### Phase 3: Deep Dive

Examine comprehensively:

- Edge cases and potential failure modes
- Performance implications under load
- Security attack surface
- Error handling completeness
- Testability and test coverage gaps
- Technical debt and maintenance burden

### Phase 4: Synthesis

Combine findings into a structured report following the template in `references/analysis-report-template.md`.

The report should include:

- **Executive Summary** - Key findings and top recommendation
- **Detailed Analysis** - By domain (architecture, security, performance, code quality)
- **Issues Found** - Prioritized as Critical (P0), High (P1), Medium (P2)
- **Recommendations** - Immediate actions, short-term improvements, long-term considerations
- **Trade-offs** - Analysis of different approaches with pros/cons

## Analysis Focus Areas

| Area                | What to Examine                         |
| ------------------- | --------------------------------------- |
| **Correctness**     | Logic errors, edge cases, assumptions   |
| **Security**        | Input validation, auth, data protection |
| **Performance**     | Complexity, caching, resource usage     |
| **Reliability**     | Error handling, failure modes, recovery |
| **Maintainability** | Readability, coupling, documentation    |
| **Testability**     | Coverage, mocking, isolation            |

## Output Requirements

When used via `/analyze` command:
- **Save plan to:** `working/plans/<ticket-id>-plan.md` in Obsidian vault
- If Obsidian write fails, output full plan in chat and report error

## Constraints

- **Read-only** — analyze code, do not modify it
- **Be thorough but focused** - analyze deeply but stay scoped to the target area
- **Prioritize findings** - not all issues are equal; use P0/P1/P2 consistently
- **Support claims with evidence** - reference specific files, lines, or patterns
- **Provide actionable recommendations** - vague advice is not useful

---

Begin by performing reconnaissance on the target area before conducting domain analysis.

**Remember: Your output is documentation. Do not write code.**
