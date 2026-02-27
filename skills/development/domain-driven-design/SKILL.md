---
name: domain-driven-design
description: Domain-Driven Design system for software development. Use when designing new systems with DDD principles, refactoring existing codebases toward DDD, generating code scaffolding (entities, aggregates, repositories, domain events), facilitating Event Storming sessions, creating bounded context maps, or performing code reviews with a DDD lens. Covers both strategic design (bounded contexts, subdomains, context maps, ubiquitous language) and tactical design (entities, value objects, aggregates, domain services, repositories). Supports all major architecture patterns (Hexagonal/Ports & Adapters, CQRS, Event Sourcing, Clean Architecture) with language-agnostic guidance and concrete examples in Python and TypeScript.
---

# Domain-Driven Design Skill

Apply DDD to build software that reflects deep understanding of the business domain.

## Quick Reference

| Task | Reference |
|------|-----------|
| Bounded contexts, subdomains, context maps | [strategic-design.md](references/strategic-design.md) |
| Entities, value objects, aggregates, repositories | [tactical-design.md](references/tactical-design.md) |
| Hexagonal, CQRS, Event Sourcing, Clean Architecture | [architecture-patterns.md](references/architecture-patterns.md) |
| Event Storming facilitation & documentation | [event-storming.md](references/event-storming.md) |
| Python implementations (Pydantic, SQLAlchemy, FastAPI) | [python-patterns.md](references/python-patterns.md) |
| TypeScript implementations (NestJS, TypeORM, Prisma) | [typescript-patterns.md](references/typescript-patterns.md) |
| DDD code review criteria | [code-review.md](references/code-review.md) |

## Core Workflow

### 1. Identify the Task Type

**Designing new system?** → Start with strategic design, then tactical
**Refactoring existing code?** → Assess current state, identify bounded contexts, refactor incrementally
**Generating scaffolding?** → Determine patterns needed, generate code
**Event Storming?** → Follow facilitation guide
**Code review?** → Apply DDD checklist

### 2. Strategic Before Tactical

Always establish strategic design first:
1. Identify **subdomains** (Core, Supporting, Generic)
2. Define **bounded contexts** and their boundaries
3. Map **context relationships** (upstream/downstream, conformist, ACL, etc.)
4. Establish **ubiquitous language** per context

### 3. Select Architecture Pattern

Choose based on domain complexity and requirements:

| Pattern | When to Use |
|---------|-------------|
| **Layered** | Simple CRUD, low complexity |
| **Hexagonal** | Need to isolate domain from infrastructure |
| **Clean Architecture** | Complex business rules, multiple delivery mechanisms |
| **CQRS** | Different read/write models, complex queries |
| **Event Sourcing** | Audit trail required, temporal queries, event-driven |

Patterns can be combined (e.g., Hexagonal + CQRS + Event Sourcing).

### 4. Apply Tactical Patterns

Select tactical building blocks based on needs:

| Building Block | Purpose |
|----------------|---------|
| **Entity** | Identity matters, mutable, lifecycle |
| **Value Object** | Defined by attributes, immutable, no identity |
| **Aggregate** | Consistency boundary, transactional unit |
| **Domain Service** | Stateless operations spanning multiple aggregates |
| **Repository** | Collection-like interface for aggregate persistence |
| **Domain Event** | Record of something significant that happened |
| **Factory** | Complex object creation logic |
| **Specification** | Encapsulated business rules for querying/validation |

### 5. Implementation Guidelines

**General principles:**
- Domain layer has ZERO infrastructure dependencies
- Depend on abstractions (interfaces/protocols), not concretions
- One aggregate = one repository = one transaction
- Aggregates reference other aggregates by ID only
- Validate invariants within aggregate boundaries
- Use domain events for cross-aggregate communication

**Language selection:**
- Read [python-patterns.md](references/python-patterns.md) for Python with Pydantic, SQLAlchemy, FastAPI
- Read [typescript-patterns.md](references/typescript-patterns.md) for TypeScript with NestJS, TypeORM, Prisma

## Project Structure Template

```
src/
├── domain/                    # Pure domain logic (no dependencies)
│   ├── model/                 # Entities, Value Objects, Aggregates
│   ├── service/               # Domain Services
│   ├── event/                 # Domain Events
│   ├── repository/            # Repository interfaces (ports)
│   └── specification/         # Business rule specifications
├── application/               # Use cases, orchestration
│   ├── command/               # Command handlers (write)
│   ├── query/                 # Query handlers (read)
│   ├── dto/                   # Data transfer objects
│   └── service/               # Application services
├── infrastructure/            # External concerns
│   ├── persistence/           # Repository implementations
│   ├── messaging/             # Event bus, message queue
│   └── external/              # Third-party integrations
└── interface/                 # Delivery mechanisms
    ├── api/                   # REST/GraphQL controllers
    ├── cli/                   # Command-line interface
    └── event/                 # Event consumers
```

## Anti-Patterns to Avoid

- **Anemic Domain Model**: Entities with only getters/setters, logic in services
- **God Aggregate**: Too many entities in one aggregate
- **Shared Kernel Abuse**: Overusing shared code between contexts
- **Infrastructure Leak**: Database concerns in domain layer
- **Missing Ubiquitous Language**: Technical terms instead of domain terms
- **Aggregate Reference by Object**: Should reference by ID only
- **Transaction Across Aggregates**: Violates consistency boundaries

## When NOT to Use DDD

DDD adds complexity. Avoid for:
- Simple CRUD applications
- Technical/infrastructure projects without complex business logic
- Prototypes or throwaway code
- Teams unfamiliar with the domain (learn domain first)

Use DDD when:
- Complex, evolving business logic
- Long-lived systems requiring maintainability
- Multiple teams working on related domains
- Domain experts available for collaboration