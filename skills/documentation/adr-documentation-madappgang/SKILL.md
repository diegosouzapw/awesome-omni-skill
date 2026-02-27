---
name: adr-documentation
description: Architecture Decision Records (ADR) documentation practice. Use when documenting architectural decisions, recording technical trade-offs, creating decision logs, or establishing architectural patterns. Trigger keywords - "ADR", "architecture decision", "decision record", "trade-offs", "architectural decision", "decision log".
version: 0.1.0
tags: [dev, adr, architecture, decisions, documentation, trade-offs]
keywords: [adr, architecture, decision, record, trade-offs, documentation, rationale, alternatives]
plugin: dev
updated: 2026-01-28
---

# ADR Documentation

## Overview

### What are ADRs?

Architecture Decision Records (ADRs) are lightweight documents that capture important architectural decisions along with their context and consequences. They serve as a historical record of why certain choices were made, helping teams avoid revisiting settled debates and providing context for future developers.

**Key characteristics:**
- **Immutable**: Once accepted, ADRs are rarely modified (supersede instead)
- **Context-rich**: Capture the environment and constraints at decision time
- **Outcome-focused**: Document what was decided and why
- **Alternative-aware**: Show what options were considered and rejected

### Why Document Decisions?

**Context Preservation:**
- Future team members understand *why* things are the way they are
- Avoid "we've always done it this way" without knowing the reason
- Preserve institutional knowledge across team changes

**Onboarding:**
- New developers can read ADR history to understand system evolution
- Reduces repetitive explanations of architectural choices
- Provides learning path through project decisions

**Avoiding Repeat Mistakes:**
- Document why certain approaches were rejected
- Prevent re-proposing failed solutions
- Learn from past trade-offs

**Alignment:**
- Create shared understanding across team
- Resolve disagreements with documented rationale
- Enable asynchronous decision-making

### When to Write an ADR

Write an ADR when:
- **Significant architectural choices**: Microservices vs monolith, event-driven vs request/response
- **Technology selections**: Database choice, framework adoption, language decisions
- **Pattern decisions**: State management approach, authentication strategy, error handling patterns
- **Breaking changes**: API redesigns, data model changes, migration strategies
- **Performance-critical decisions**: Caching strategies, optimization approaches, scalability patterns
- **Security-related choices**: Authentication methods, encryption strategies, access control models
- **Infrastructure decisions**: Deployment strategy, hosting platform, CI/CD approach
- **Dependency selections**: Major library choices, third-party service integrations

**Don't write an ADR for:**
- Minor implementation details
- Temporary workarounds
- Personal coding style preferences
- Reversible low-impact choices

## ADR Structure

### Standard Format

```markdown
# ADR-XXXX: [Short Title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-YYYY]

## Date
YYYY-MM-DD

## Context
[Problem statement and environment]

## Decision
[What was decided]

## Consequences
[What results from this decision]

## Alternatives Considered
[Other options evaluated]

## Related ADRs
[Links to related decisions]

## References
[External links and documentation]
```

### Section Guidelines

**Title:**
- Format: `ADR-XXXX: Short Decision Title`
- Be specific: "ADR-0012: Use PostgreSQL for Primary Database"
- Not generic: "ADR-0012: Database Choice"

**Status:**
- **Proposed**: Under discussion, not yet accepted
- **Accepted**: Decision is final and being implemented
- **Deprecated**: No longer recommended but not replaced
- **Superseded by ADR-YYYY**: Replaced by a newer decision

**Date:**
- Use ISO format: YYYY-MM-DD
- Date of status change, not original proposal

**Context:**
- Describe the forces at play
- Technical constraints
- Business requirements
- Team capabilities
- Timeline pressures
- Budget considerations
- Example: "We need to store structured data with complex relationships. Team has limited database expertise. Must support 100k+ users within 6 months."

**Decision:**
- State clearly what was decided
- Include scope and boundaries
- Specify what is NOT included
- Example: "We will use PostgreSQL as our primary database for all structured data. NoSQL solutions may be used for specific use cases (caching, session storage) but PostgreSQL is the default."

**Consequences:**
- **Positive**: What benefits we gain
- **Negative**: What drawbacks we accept
- **Neutral**: What trade-offs we're making

Example:
```markdown
### Positive
- Strong ACID guarantees for transactions
- Rich query capabilities with SQL
- Mature ecosystem and tooling

### Negative
- Requires careful index management for performance
- Scaling horizontally is more complex than NoSQL
- Team needs PostgreSQL training

### Neutral
- Need to establish backup/restore procedures
- Must define connection pooling strategy
```

**Alternatives Considered:**
- List 2-4 alternatives seriously evaluated
- For each: Pros, Cons, Why rejected
- Be fair to alternatives (avoid strawman arguments)

**Related ADRs:**
- Link to decisions that influenced this one
- Link to decisions this one influences
- Create a decision graph over time

**References:**
- Documentation links
- Blog posts or articles that influenced decision
- Benchmark results
- Proof of concepts

## ADR Numbering and Naming

### Sequential Numbering

Use zero-padded sequential numbers:
- `ADR-0001`, `ADR-0002`, ..., `ADR-0010`, ..., `ADR-0100`
- Four digits handles up to 9,999 ADRs (enough for most projects)
- Don't skip numbers when superseding (new ADR gets next number)

### File Naming

Format: `ADR-XXXX-short-title.md`

Examples:
- `ADR-0001-use-postgresql-database.md`
- `ADR-0002-adopt-react-19-frontend.md`
- `ADR-0003-implement-jwt-authentication.md`

**Naming guidelines:**
- Use lowercase
- Use hyphens (not underscores or spaces)
- Keep title short but descriptive
- Include key technology/pattern name

### Location

Recommended locations:
- **Option 1**: `ai-docs/decisions/` (alongside other AI-focused docs)
- **Option 2**: `docs/adr/` (standard documentation location)
- **Option 3**: Root `adr/` directory (if ADRs are primary docs)

**Choose consistently across project.**

For Claude Code plugins:
- Use `ai-docs/decisions/` to co-locate with CLAUDE.md and other AI docs
- Makes ADRs easily discoverable by Claude

## When to Write an ADR

### Technology Adoption

**Write an ADR when:**
- Adopting a new framework or major library
- Choosing between competing technologies
- Upgrading to a new major version with breaking changes

**Examples:**
- ADR-0015: Migrate from Webpack to Vite
- ADR-0023: Adopt TypeScript for Backend Services
- ADR-0031: Use Bun Runtime Instead of Node.js

### Architectural Pattern Selection

**Write an ADR when:**
- Choosing architectural style (monolith, microservices, serverless)
- Selecting state management approach
- Defining API design patterns
- Establishing error handling strategies

**Examples:**
- ADR-0008: Use Event-Driven Architecture for Order Processing
- ADR-0012: Implement Repository Pattern for Data Access
- ADR-0019: Adopt REST over GraphQL for Public API

### Breaking Changes

**Write an ADR when:**
- Making changes that break backward compatibility
- Migrating data models
- Redesigning core APIs
- Changing authentication mechanisms

**Examples:**
- ADR-0025: Remove Legacy API v1 Endpoints
- ADR-0033: Migrate from Sessions to JWT Tokens
- ADR-0041: Change User ID from Integer to UUID

### Performance-Critical Decisions

**Write an ADR when:**
- Implementing caching strategies
- Making database optimization choices
- Selecting algorithms for hot paths
- Deciding on async vs sync processing

**Examples:**
- ADR-0017: Implement Redis for Session Caching
- ADR-0022: Use WebSockets for Real-Time Updates
- ADR-0028: Adopt Background Job Queue for Heavy Processing

### Security-Related Choices

**Write an ADR when:**
- Choosing authentication methods
- Implementing authorization strategies
- Selecting encryption approaches
- Defining security policies

**Examples:**
- ADR-0010: Use OAuth 2.0 with PKCE for Mobile Auth
- ADR-0014: Implement Row-Level Security in PostgreSQL
- ADR-0020: Adopt OWASP Guidelines for API Security

## ADR Lifecycle

### Proposal Phase

**Creating a Proposal:**
1. Draft ADR with status "Proposed"
2. Share with team for feedback
3. Discuss in team meeting or async comments
4. Iterate on Context and Alternatives sections

**Review Process:**
- Ensure all alternatives are fairly represented
- Verify consequences are realistic
- Check that context captures current constraints
- Validate decision aligns with project goals

### Acceptance

**When to Accept:**
- Team consensus reached (or decision maker approves)
- Decision is being implemented
- No major unresolved concerns

**Accepting an ADR:**
1. Change status to "Accepted"
2. Update date to acceptance date
3. Commit to repository
4. Communicate to team

**Post-Acceptance:**
- ADR becomes immutable (don't edit decision or alternatives)
- May add notes or references sections if new info emerges
- Use comments to clarify, don't rewrite history

### Deprecation Process

**When to Deprecate:**
- Decision is no longer recommended but not replaced
- Technology or pattern is obsolete
- Security vulnerability makes approach unsafe

**Deprecating an ADR:**
1. Change status to "Deprecated"
2. Add "Deprecation Note" section explaining why
3. Update date to deprecation date
4. Don't remove ADR from repository

Example deprecation note:
```markdown
## Deprecation Note (2026-01-15)

This decision has been deprecated due to security vulnerabilities
discovered in the XYZ library. See ADR-0045 for the replacement
approach using the ABC library.
```

### Superseding Old ADRs

**When to Supersede:**
- Making a decision that replaces an old one
- Reversing a previous decision
- Evolving a decision with significant changes

**Superseding Process:**
1. Create new ADR with next sequential number
2. Reference old ADR in "Context" section
3. Update old ADR status to "Superseded by ADR-XXXX"
4. Explain why old decision is being replaced

**Example:**
Old ADR-0012:
```markdown
## Status
Superseded by ADR-0034
```

New ADR-0034:
```markdown
## Context
ADR-0012 chose MongoDB for our primary database. After 2 years
of production use, we've encountered scaling issues with our
relational data model. This ADR supersedes ADR-0012.
```

## Integration with Claude Code

### Guiding Agent Behavior

ADRs help Claude understand project conventions:

**In CLAUDE.md:**
```markdown
## Architectural Decisions

Key decisions documented in ai-docs/decisions/:
- ADR-0001: Use PostgreSQL for all structured data
- ADR-0005: Adopt React 19 with TypeScript strict mode
- ADR-0012: Implement repository pattern for data access

Claude should follow these patterns in all code generation.
```

### Referencing ADRs

**In skill files:**
```markdown
## Data Access Patterns

Follow the repository pattern (ADR-0012). All database queries
must go through repository classes in the `repositories/` directory.
```

**In agent instructions:**
```markdown
When generating database code:
1. Use PostgreSQL (ADR-0001)
2. Implement via repository pattern (ADR-0012)
3. Use TypeORM for ORM layer (ADR-0018)
```

### ADR Compliance Checking

Create a validation agent to check ADR compliance:

**Example agent task:**
```markdown
Review the code changes and check compliance with ADRs:

- ADR-0001: All database access uses PostgreSQL
- ADR-0012: All queries go through repository classes
- ADR-0022: Real-time features use WebSockets

Report any violations.
```

## Best Practices

### Do

**✓ Write ADRs for significant decisions**
- Technology choices with long-term impact
- Architectural patterns affecting multiple components
- Security or performance-critical decisions

**✓ Document alternatives fairly**
- Present real alternatives, not strawmen
- Show pros/cons objectively
- Explain rejection reasons clearly

**✓ Keep ADRs immutable after acceptance**
- Don't rewrite history
- Use new ADRs to supersede old ones
- Preserve original context and rationale

**✓ Link related ADRs**
- Show decision dependencies
- Create decision graph over time
- Help readers understand evolution

**✓ Update status promptly**
- Accept when implementation begins
- Deprecate when no longer valid
- Supersede when replaced

### Don't

**✗ Don't over-document**
- Skip ADRs for trivial choices
- Don't write ADRs for personal style preferences
- Avoid ADRs for easily reversible decisions

**✗ Don't write vague context**
- Bad: "We needed a database"
- Good: "We need ACID transactions for financial data, team has SQL experience, must scale to 1M users"

**✗ Don't ignore alternatives**
- Always document what else was considered
- Explain why alternatives were rejected
- Be honest about trade-offs

**✗ Don't let ADRs get stale**
- Update status when decisions change
- Deprecate obsolete ADRs
- Don't leave "Proposed" ADRs open indefinitely

**✗ Don't make ADRs too long**
- Keep to 1-2 pages maximum
- Link to detailed docs instead of including them
- Focus on decision and rationale, not implementation details

### Writing Good Context

**Include:**
- Technical constraints (performance, security, scalability)
- Business requirements (timeline, budget, features)
- Team capabilities (skills, experience, size)
- Environmental factors (existing systems, integrations)

**Example - Poor Context:**
```markdown
## Context
We need a database for our application.
```

**Example - Good Context:**
```markdown
## Context
Our e-commerce platform needs to store product catalogs, orders,
and user data. Key requirements:
- ACID transactions for order processing (financial accuracy)
- Support for complex queries (product search, filtering)
- Scale to 100k daily active users within 6 months
- Team has 3 years SQL experience, no NoSQL experience
- Must integrate with existing PostgreSQL warehouse system
- Budget allows managed database hosting

Performance requirements:
- 95th percentile query time < 100ms
- Support 1000 concurrent connections
- 99.9% uptime SLA
```

### Documenting Alternatives Fairly

**Structure for each alternative:**
```markdown
### Alternative X: [Technology/Pattern Name]

**Description**: [Brief overview]

**Pros**:
- [Benefit 1]
- [Benefit 2]
- [Benefit 3]

**Cons**:
- [Drawback 1]
- [Drawback 2]

**Why Rejected**: [Honest explanation of why this wasn't chosen]
```

**Example:**
```markdown
### Alternative 2: MongoDB

**Description**: Document-oriented NoSQL database

**Pros**:
- Flexible schema allows rapid iteration
- Built-in horizontal scaling
- Good for unstructured data

**Cons**:
- No ACID transactions across collections (deal-breaker for orders)
- Team has no MongoDB experience (6-month learning curve)
- Query language less powerful for complex joins

**Why Rejected**: Lack of multi-document transactions is critical
for our order processing workflow where we need to atomically
update inventory, orders, and user balances. The 6-month learning
curve also conflicts with our launch timeline.
```

## Examples

### Example 1: Database Selection

```markdown
# ADR-0001: Use PostgreSQL for Primary Database

## Status
Accepted

## Date
2026-01-28

## Context

Our SaaS application needs a database for user accounts, subscriptions,
and billing data. Requirements:

**Functional:**
- Store structured relational data (users, plans, invoices)
- Support complex queries (reporting, analytics)
- ACID transactions for billing operations

**Non-Functional:**
- Scale to 500k users in year one
- 99.9% uptime requirement
- Response time < 100ms for 95th percentile
- Point-in-time recovery for data safety

**Team & Environment:**
- Team has 5+ years PostgreSQL experience
- Existing monitoring tools support PostgreSQL
- Running on AWS with RDS available
- $500/month database budget

## Decision

We will use PostgreSQL (version 15+) as our primary database for
all structured data. Deployment via AWS RDS for managed service.

**Scope:**
- All user data, subscriptions, billing
- Application-level caching with Redis
- Logs stored in CloudWatch, not database

## Consequences

### Positive
- Strong ACID guarantees protect billing integrity
- Rich SQL query support for complex reports
- Mature ecosystem (ORMs, monitoring, tools)
- Team expertise minimizes ramp-up time
- AWS RDS provides automated backups and high availability

### Negative
- Horizontal scaling requires careful sharding strategy
- More expensive than serverless options at low volume
- Requires connection pool management

### Neutral
- Need to establish index management process
- Must define migration strategy for schema changes
- Need to set up automated backup testing

## Alternatives Considered

### Alternative 1: MongoDB

**Pros**:
- Flexible schema for rapid iteration
- Built-in horizontal scaling (sharding)
- JSON-native data model

**Cons**:
- No multi-document ACID transactions (critical for billing)
- Team has no MongoDB experience
- Less powerful query language for analytics

**Why Rejected**: Lack of strong ACID transactions across documents
is a deal-breaker for our billing workflow.

### Alternative 2: MySQL

**Pros**:
- Similar feature set to PostgreSQL
- Team has some MySQL experience
- Slightly better write performance at scale

**Cons**:
- Less feature-rich than PostgreSQL (no JSONB, less robust CTEs)
- Replication setup more complex
- Team primarily PostgreSQL-experienced

**Why Rejected**: PostgreSQL's advanced features (JSONB, window
functions, CTEs) provide better developer experience. Team expertise
is stronger with PostgreSQL.

### Alternative 3: DynamoDB

**Pros**:
- Serverless, no infrastructure management
- Auto-scaling included
- Very cost-effective at low volume

**Cons**:
- No ACID transactions across partition keys
- Limited query capabilities (no joins)
- Steep learning curve for team
- Vendor lock-in to AWS

**Why Rejected**: Limited query capabilities would severely hamper
our reporting and analytics requirements. Team would need 6+ months
to become productive.

## Related ADRs
- None (first ADR)

## References
- PostgreSQL 15 documentation: https://www.postgresql.org/docs/15/
- AWS RDS PostgreSQL pricing: https://aws.amazon.com/rds/postgresql/pricing/
- Internal benchmark results: docs/benchmarks/database-comparison-2026.pdf
```

### Example 2: Framework Adoption

```markdown
# ADR-0005: Adopt React 19 for Frontend Framework

## Status
Accepted

## Date
2026-01-28

## Context

Starting a new web application requiring:
- Rich interactive UI with complex state management
- Real-time updates (WebSocket integration)
- Mobile-responsive design
- Developer productivity (fast iteration)

**Team:**
- 2 developers with React experience
- 1 developer with Vue experience
- All have JavaScript/TypeScript experience

**Timeline:**
- MVP in 3 months
- Need to hire 2 more frontend developers

## Decision

Use React 19 with TypeScript for all frontend development.

**Stack:**
- React 19 (with new hooks and compiler)
- TypeScript strict mode
- Vite for build tooling
- React Router for routing
- TanStack Query for data fetching

## Consequences

### Positive
- Large ecosystem of libraries and components
- Strong TypeScript support
- Easy to hire React developers
- Team has existing React experience
- React 19 compiler improves performance automatically

### Negative
- React 19 is new (potential early adopter issues)
- Steeper learning curve than Vue for new hires
- Need to choose state management approach (not built-in)

### Neutral
- Need to establish component architecture patterns
- Must define folder structure conventions
- Need to configure testing infrastructure

## Alternatives Considered

### Alternative 1: Vue 3

**Pros**:
- Simpler API, easier to learn
- Built-in state management (Pinia)
- Excellent documentation

**Cons**:
- Smaller ecosystem than React
- Harder to hire Vue developers
- Only 1 team member has experience

**Why Rejected**: Hiring pipeline and ecosystem size favor React.
Team majority has React experience.

### Alternative 2: Svelte

**Pros**:
- Excellent performance (compile-time framework)
- Simple, elegant API
- Less boilerplate than React

**Cons**:
- Much smaller ecosystem
- Very difficult to hire Svelte developers
- No team experience
- Newer framework (less mature)

**Why Rejected**: Hiring risk is too high. Need to scale team quickly.

### Alternative 3: Next.js (React Framework)

**Pros**:
- Server-side rendering built-in
- File-based routing
- API routes included

**Cons**:
- We're building a SPA, not needing SSR
- Adds complexity we don't need
- Opinionated structure may limit flexibility

**Why Rejected**: We don't need SSR for our use case. Prefer flexibility
of React + Vite without framework constraints.

## Related ADRs
- ADR-0006: Use TanStack Query for Data Fetching (future)
- ADR-0007: Implement Zustand for Global State (future)

## References
- React 19 release notes: https://react.dev/blog/2024/12/05/react-19
- TypeScript configuration: docs/typescript-setup.md
```

### Example 3: Plugin Architecture

```markdown
# ADR-0010: Implement Hook-Based Plugin System

## Status
Accepted

## Date
2026-01-28

## Context

Claude Code needs an extensible plugin system allowing third-party
developers to add agents, commands, and skills.

**Requirements:**
- Plugins must be installable via git URLs
- Support hot-reloading during development
- Isolate plugin failures from core system
- Allow marketplace distribution
- Support per-project and global plugins

**Constraints:**
- Must work with existing Claude Code architecture
- Can't require core code changes for new plugins
- Need to maintain security boundaries

## Decision

Implement a hook-based plugin system where:

1. Plugins declare components (agents, commands, skills) in plugin.json
2. Core loads plugins at startup from settings.json
3. Hooks allow plugins to extend core functionality
4. Each plugin runs in isolated context
5. Marketplace serves plugin metadata via marketplace.json

**Architecture:**
```
.claude/
  settings.json (enabled plugins)
  plugins/
    {marketplace}/
      {plugin-name}/
        plugin.json
        agents/
        commands/
        skills/
```

## Consequences

### Positive
- Third-party developers can extend Claude Code
- Plugins are easy to install and remove
- Hot-reloading speeds development
- Marketplace enables discovery
- Isolation prevents plugin failures affecting core

### Negative
- Plugin API is a contract we must maintain
- Breaking changes require migration path
- Plugin quality varies (marketplace curation needed)
- Additional complexity in core loader

### Neutral
- Need to document plugin development guide
- Must establish plugin review process for marketplace
- Need to create example plugins

## Alternatives Considered

### Alternative 1: Script-Based Plugins

**Description**: Plugins as standalone scripts invoked by core

**Pros**:
- Very simple implementation
- Complete isolation (separate processes)
- Language-agnostic (any executable)

**Cons**:
- Poor performance (process overhead)
- Difficult to share state or context
- Hard to debug across process boundaries
- No TypeScript types for plugin API

**Why Rejected**: Performance and developer experience are too poor.

### Alternative 2: npm Package Plugins

**Description**: Plugins as npm packages imported at runtime

**Pros**:
- Leverages existing npm ecosystem
- Version management via package.json
- TypeScript types work seamlessly

**Cons**:
- Requires publishing to npm (friction for internal plugins)
- Can't install from git URLs easily
- Difficult to support local development plugins
- Security concerns with npm supply chain

**Why Rejected**: Git-based installation is critical requirement.
npm adds unnecessary friction for private/local plugins.

### Alternative 3: WebAssembly Plugins

**Description**: Plugins compiled to WASM for sandboxing

**Pros**:
- Strong isolation guarantees
- Language-agnostic plugin development
- Near-native performance

**Cons**:
- Requires WASM compilation (complex toolchain)
- Limited access to Node.js APIs
- Much higher barrier to entry for plugin developers
- Difficult to debug

**Why Rejected**: Too complex for our use case. Most plugins need
Node.js APIs (file system, process spawning). Developer experience
is much worse than JavaScript/TypeScript plugins.

## Related ADRs
- ADR-0011: Use marketplace.json for Plugin Discovery (future)
- ADR-0012: Implement Plugin Versioning Strategy (future)

## References
- Plugin API design: ai-docs/PLUGIN_API.md
- Example plugin: plugins/example-plugin/
- Marketplace spec: .claude-plugin/MARKETPLACE_SPEC.md
```

## Summary

ADRs are a lightweight but powerful tool for documenting architectural decisions. They:
- Preserve context and rationale for future teams
- Prevent revisiting settled debates
- Enable better onboarding and knowledge transfer
- Create accountability for architectural choices

**Key principles:**
- Write ADRs for significant, hard-to-reverse decisions
- Document alternatives fairly and honestly
- Keep ADRs immutable after acceptance
- Use superseding for evolving decisions
- Integrate ADRs into Claude Code workflows

**For Claude Code specifically:**
- Place ADRs in `ai-docs/decisions/`
- Reference ADRs in CLAUDE.md to guide agent behavior
- Use ADRs to ensure consistency across agent-generated code
- Create ADR compliance checking agents

Start writing ADRs today for your next significant architectural decision!
