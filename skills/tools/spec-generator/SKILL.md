---
name: spec-generator
argument-hint: [instructions]
allowed-tools: AskUserQuestion, Write
description: Interview user in-depth to create a detailed spec
---

# /spec-generator — Interview-driven spec generation

Interview the user in-depth using AskUserQuestion to produce a comprehensive, detailed specification file.

## Workflow

1. Read the user's instructions (passed as arguments)
2. Begin an in-depth interview using AskUserQuestion
3. Ask probing, non-obvious questions across all dimensions
4. Continue interviewing until thorough coverage is achieved
5. Write the final spec to a file

## Interview Strategy

### Phase 1: Core Vision

- What problem does this solve and for whom?
- What does success look like? What are the measurable outcomes?
- What existing solutions have been considered and why are they insufficient?

### Phase 2: Technical Deep-Dive

- Architecture: monolith vs microservices, data flow, state management
- Data model: entities, relationships, constraints, migrations
- API design: endpoints, auth, rate limiting, versioning
- Infrastructure: hosting, scaling, caching, CDN, CI/CD
- Dependencies: third-party services, libraries, licensing concerns

### Phase 3: UI & UX

- User flows: entry points, happy paths, error states
- Interaction patterns: real-time vs batch, optimistic updates
- Accessibility: screen readers, keyboard nav, color contrast
- Responsive behavior: breakpoints, mobile-first considerations
- Loading states, empty states, skeleton screens

### Phase 4: Edge Cases & Constraints

- Concurrency: race conditions, optimistic locking, conflict resolution
- Failure modes: network errors, partial failures, retry strategies
- Security: auth model, data encryption, input validation, CSRF, XSS
- Performance budgets: load times, bundle sizes, database query limits
- Data volume: pagination, infinite scroll, search indexing

### Phase 5: Tradeoffs & Decisions

- Build vs buy decisions
- Consistency vs availability tradeoffs
- Speed-to-market vs long-term maintainability
- Scope: what is explicitly out of scope for v1?
- Migration path: how to get from current state to target state

### Phase 6: Operational Concerns

- Monitoring and alerting requirements
- Logging and debugging strategy
- Rollback and deployment strategy
- Testing strategy: unit, integration, e2e, visual regression
- Documentation needs: API docs, runbooks, onboarding

## Interview Rules

- Ask ONE focused question at a time (never batch multiple questions)
- Do NOT ask obvious or surface-level questions
- Follow up on vague answers — push for specifics
- Challenge assumptions respectfully when they seem risky
- Adapt questions based on previous answers (skip irrelevant phases)
- If the user says "that's it" or "done", wrap up immediately
- Aim for 10-20 questions depending on project complexity

## Output Format

Write the spec to `SPEC.md` (or a filename specified by the user) with this structure:

```markdown
# [Project Name] — Specification

## Overview
[1-2 paragraph summary of what this is and why it exists]

## Goals & Success Criteria
- [Measurable outcome 1]
- [Measurable outcome 2]

## User Stories
- As a [role], I want [capability] so that [benefit]

## Technical Architecture
### System Design
[Architecture decisions, data flow diagrams in text]

### Data Model
[Entities, relationships, constraints]

### API Design
[Endpoints, auth, contracts]

## UI/UX Specification
### User Flows
[Step-by-step flows for key interactions]

### Wireframe Descriptions
[Text descriptions of key screens/components]

## Edge Cases & Error Handling
[Documented edge cases and how they're handled]

## Security Considerations
[Auth, encryption, validation, threat model]

## Performance Requirements
[Budgets, benchmarks, scaling expectations]

## Tradeoffs & Decisions
[Key decisions made during the interview with rationale]

## Out of Scope (v1)
[Explicitly excluded items]

## Open Questions
[Unresolved items that need further discussion]
```

## Prompt

Follow the user instructions and interview me in detail using the AskUserQuestion tool about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. Make sure the questions are not obvious — be very in-depth and continue interviewing me continually until it's complete. Then, write the spec to a file.

## Guidelines

- Never skip the interview — always ask questions, even if instructions seem complete
- Prefer depth over breadth — 3 deep questions on architecture beats 10 shallow ones
- The spec should be detailed enough that another developer can implement from it alone
- Include direct quotes from user answers when they capture intent well
- Flag any contradictions discovered during the interview
