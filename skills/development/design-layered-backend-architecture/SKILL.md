---
name: design-layered-backend-architecture
description: Design or review a layered backend architecture (routes, controllers, services, repositories) for HTTP APIs and microservices.
---

# Design Layered Backend Architecture

## Purpose
Define a repeatable backend service structure that keeps HTTP concerns, business logic, and data access cleanly separated.

## When to use
Use this skill when you are:
- Starting a new backend service or module
- Refactoring an existing codebase toward clearer boundaries
- Reviewing architecture for maintainability and testability
- Defining team-wide conventions for “where code goes”

## Inputs
- Primary service responsibilities and domain boundaries
- The external interfaces (HTTP endpoints, jobs, events)
- Persistence model and integration points
- Existing constraints (framework, ORM/query layer, deployment model)

## Outputs
- A layering blueprint with clear responsibilities
- A directory/module layout that supports the blueprint
- A request lifecycle map (how a request flows through layers)
- A list of architectural invariants to enforce in code review

## Core model
A typical layered service uses four layers:

1. **Routes**
   - Define paths/methods
   - Register middleware
   - Delegate to controllers
   - MUST NOT implement business logic

2. **Controllers**
   - Validate and normalize inputs
   - Translate domain results to HTTP responses
   - Map errors to status codes and error shapes
   - SHOULD be thin and focused on HTTP

3. **Services**
   - Implement business rules and orchestration
   - Call repositories and other services
   - MUST NOT depend on HTTP objects

4. **Repositories**
   - Encapsulate persistence logic
   - Implement queries, transactions, and mappings
   - SHOULD hide ORM/query details from services

## Architectural invariants
- Each layer MUST have a single primary responsibility.
- Dependencies MUST flow inward:
  - routes → controllers → services → repositories
- Services MUST be unit-testable without HTTP or database.
- Data access SHOULD be testable via repository tests or integration tests.

## Decision checklist
Use these decisions to avoid ambiguous boundaries.

- **Do we need a repository layer?**
  - MAY be skipped for simple CRUD.
  - SHOULD be added once queries become non-trivial, transactional, or reused.

- **Where do we validate input?**
  - SHOULD be at the controller boundary (closest to untrusted input).

- **Where do we implement authorization?**
  - If the rule depends on HTTP context, keep enforcement in middleware/controller.
  - If the rule is domain-specific, enforce in services as well.

## Steps
1. Identify the public interfaces (endpoints/jobs/events).
2. Define domain services and responsibilities.
3. Define repository boundaries (per aggregate or per module).
4. Define controller responsibilities and error mapping contract.
5. Define route layout and middleware order.
6. Document the request lifecycle and verification checkpoints.
7. Add templates to standardize new modules.

## Verification

- [ ] New endpoints follow the dependency direction (routes → controllers → services → repositories)
- [ ] Controllers contain no data access queries
- [ ] Routes contain no business logic
- [ ] Services contain no HTTP concepts (`req`, `res`)
- [ ] At least one service is unit-tested in isolation
- [ ] Build passes after applying the architecture

## Boundaries

- MUST NOT allow dependencies to flow outward (repositories calling controllers)
- MUST NOT embed business logic in routes or controllers
- MUST NOT introduce circular dependencies between layers
- SHOULD NOT skip repository layer for non-trivial queries
- SHOULD NOT mix multiple responsibilities in a single layer
- SHOULD NOT bypass architectural invariants without documented justification

## Included assets
- Examples: see `./examples/` for request lifecycle and module layout blueprints.
