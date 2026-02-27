---
name: architecture-decision-records
description: This skill should be used when documenting significant technical decisions, reviewing past architectural choices, or establishing decision processes. It provides ADR templates and best practices.
allowed-tools: Read
---

# Architecture Decision Records

Capture the context and rationale behind significant technical decisions using structured ADR formats.

## When to Use This Skill

| Write ADR                  | Skip ADR               |
| -------------------------- | ---------------------- |
| New framework adoption     | Minor version upgrades |
| Database technology choice | Bug fixes              |
| API design patterns        | Implementation details |
| Security architecture      | Routine maintenance    |
| Integration patterns       | Configuration changes  |

## Quick Start

1. Copy the template: `cp docs/adr/template.md docs/adr/NNNN-your-title.md`
2. Fill in: Context, Decision Drivers, Options, Decision, Consequences
3. PR for review (2+ senior engineers)
4. Update `docs/adr/README.md` index after merge

## Core Concepts

An Architecture Decision Record captures:

- **Context**: Why we needed to make a decision
- **Decision**: What we decided
- **Consequences**: What happens as a result

### ADR Lifecycle

```
Proposed --> Accepted --> Deprecated --> Superseded
                |
             Rejected
```

Read `reference/adr-lifecycle.md` for status transitions, deprecation patterns, and review checklists.

## Process

### 1. Choose a Template

Pick the format that fits the decision's complexity:

| Decision Complexity | Template |
|---------------------|----------|
| Simple tech selection | Y-Statement (one paragraph) |
| Medium decision | Lightweight ADR (0.5-1 page) |
| Significant architecture change | Standard MADR (1-2 pages) |
| Retiring a decision | Deprecation ADR |
| Major cross-team proposal | RFC Style (2-4 pages) |

Read `reference/adr-templates.md` for all template formats ready to copy-paste.

### 2. Write the ADR

- Start with context -- explain the problem before the solution
- List 2-3 real alternatives with honest pros/cons
- State the decision clearly
- Document both positive and negative consequences with specifics

### 3. Review and Approve

- Submit as PR with 2+ senior engineer reviewers
- Consult affected teams
- Assess security, cost, and reversibility implications

### 4. Maintain

- Update ADR index after acceptance
- Create implementation tickets
- Never edit accepted ADRs -- write new ones to supersede

Read `reference/adr-examples.md` for complete worked examples (PostgreSQL selection, TypeScript adoption, MongoDB deprecation, event sourcing RFC).

## Minimal Template (Copy-Paste Starter)

```markdown
# ADR-NNNN: [Title]

## Status
Proposed | Accepted | Deprecated | Superseded by ADR-XXXX

## Context
[Why do we need to decide this? What's the problem?]

## Decision
We will [decision].

## Consequences
- **Good**: [benefits]
- **Bad**: [drawbacks]
- **Mitigations**: [how we'll address the bad]
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Writing ADR after implementation | Write during design phase |
| Listing only one option | Always include 2-3 real alternatives |
| Vague consequences | Be specific: "Adds ~200ms latency to checkout" |
| Editing accepted ADRs | Write new ADR that supersedes |
| No decision drivers | List explicit criteria with priorities |
| Missing rejected ADRs | Document rejected options too |

## Directory Structure

```
docs/adr/
  README.md           # Index of all ADRs
  template.md         # Team's ADR template
  0001-use-postgresql.md
  0002-caching-strategy.md
```

## Reference Files

| File | Contents |
|------|----------|
| `reference/adr-templates.md` | All formats: MADR, lightweight, Y-statement, deprecation, RFC |
| `reference/adr-examples.md` | Complete worked examples for each format |
| `reference/adr-lifecycle.md` | Status transitions, review checklists, automation with adr-tools |

## Resources

- [MADR Template](https://adr.github.io/madr/)
- [ADR GitHub Organization](https://adr.github.io/)
- [adr-tools](https://github.com/npryce/adr-tools)

## Error Handling

**Conflicting ADRs**: When a new decision contradicts an existing ADR, create a superseding ADR that explicitly references and deprecates the old one.

**Missing context**: If the decision rationale is unclear or incomplete, flag it and request clarification before recording.
