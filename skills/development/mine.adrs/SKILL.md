---
name: mine.adrs
description: Create and maintain Architecture Decision Records (ADRs) for significant project decisions. Tracks context, choices, consequences, and alternatives.
user-invokable: true
---

# Architecture Decision Records

Lightweight system for recording significant architectural and technical decisions.

## When to Activate

- Making architecture or design pattern choices
- Selecting technologies, libraries, or frameworks
- Planning major refactors that change system structure
- Changing infrastructure or deployment strategies
- Introducing breaking changes to APIs or data models

## ADR Template

```markdown
# ADR-NNNN: Title

## Status
Accepted

## Context
What is the issue or situation that requires a decision? Include relevant
constraints, requirements, and forces at play.

## Decision
What is the change that we're proposing and/or doing? State the decision
clearly and concisely.

## Consequences
What becomes easier or harder as a result of this decision? Include both
positive and negative consequences.

## Alternatives Considered
What other options were evaluated? Why were they rejected? Keep it brief
but include the key trade-offs.
```

## Directory Convention

The recommended location is `design/adrs/`:

```
design/adrs/
├── 0001-use-sqlite-for-persistence.md
├── 0002-adopt-command-pattern.md
└── ...
```

ADRs are numbered sequentially and append-only — superseded decisions get a `Superseded by: ADR-NNN` status, not deleted.

## Relationship with Research and Audits

ADRs sit in a workflow with research briefs and audits:

1. **Research** (`/mine.research`) explores the problem space and evaluates options
2. **ADR** records the chosen option and why
3. Research prereqs then guide the implementation of that decision

A research brief without a corresponding ADR means the work is still exploratory. An ADR without a research brief means the decision was straightforward enough not to need one.

**Audits** (`/mine.audit`) are backward-looking assessments that may surface issues requiring architectural decisions — those decisions get recorded as ADRs.

## Workflow

### Creating a New ADR

1. Check existing ADRs: `ls design/adrs/` (or `adrs/`) to find the next sequence number
2. Create the directory if it doesn't exist
3. Name the file: `NNNN-kebab-case-title.md` (zero-padded, e.g., `0001`)
4. Fill in all sections — Context and Alternatives are the most valuable parts
5. Set Status to `Accepted`

### Updating an Existing ADR

- **Superseded**: When a new ADR replaces an old one, update the old ADR's Status to `Superseded by ADR-XXXX` and link to the new one
- **Deprecated**: When a decision is reversed without replacement, set Status to `Deprecated` and add a note explaining why

### What Makes a Good ADR

**Good titles** describe the decision, not the problem:
- "Use PostgreSQL for persistence" (not "Database choice")
- "Adopt event sourcing for order processing" (not "Order system design")
- "Switch from REST to GraphQL for client API" (not "API refactor")

**Good context** explains the forces and constraints:
- What problem are we solving?
- What constraints exist (team size, timeline, existing tech)?
- What quality attributes matter most (performance, maintainability)?

**Good consequences** are honest about trade-offs:
- What do we gain?
- What do we lose or accept as a cost?
- What new risks or complexities does this introduce?

## Status Values

| Status | Meaning |
|--------|---------|
| `Accepted` | Decision is active and in effect |
| `Superseded by ADR-XXXX` | Replaced by a newer decision |
| `Deprecated` | Decision reversed, no longer applies |

## Example

```markdown
# ADR-0001: Use SQLite for Local Development Database

## Status
Accepted

## Context
The application needs a local database for development and testing.
Developers use different operating systems and we want minimal setup
friction. Production uses PostgreSQL.

## Decision
Use SQLite for local development and testing environments. Production
will continue to use PostgreSQL. We will use an ORM abstraction layer
to minimize database-specific code.

## Consequences
- Easier onboarding: no database server installation needed
- Faster test execution: in-memory SQLite for unit tests
- Risk of subtle behavior differences between SQLite and PostgreSQL
- Must avoid PostgreSQL-specific features in application code

## Alternatives Considered
- **Docker PostgreSQL**: Closer to production but adds Docker dependency
  and slower test startup. Rejected for development friction.
- **In-memory data structures**: Simplest but no SQL testing. Rejected
  because we need to test query logic.
```
