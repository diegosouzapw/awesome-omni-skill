---
name: review-architecture
description: Verify DDD patterns, Clean Architecture boundaries, and bITdevKit-specific conventions in modular monolith projects
---

# Architecture Review Skill (DDD + Clean Architecture)

Specialized architectural review for modular monoliths using **Domain-Driven Design (DDD)**, **Clean/Onion Architecture**, and **bITdevKit patterns**. This skill verifies layer boundaries, domain purity, CQRS patterns, and proper use of Result<T> error handling.

## When to Apply This Skill

Use this skill when:

- **Conducting architecture reviews** of new features or modules
- **Reviewing pull requests** that add/modify domain models, commands, queries, or endpoints
- **Refactoring** existing code to align with DDD/Clean Architecture
- **Onboarding** new developers to the architectural patterns
- **Validating** that layer boundaries remain intact after changes
- **Auditing** cross-module dependencies in a modular monolith

## Architecture Overview

### Layer Structure (Onion/Clean Architecture)

Dependencies flow **inward only** from outer layers to inner layers:

```
┌─────────────────────────────────────────┐
│      Presentation (Outermost)           │  Minimal API endpoints, DTOs
│  ┌───────────────────────────────────┐  │
│  │      Infrastructure               │  │  EF Core, Repositories, Jobs
│  │  ┌─────────────────────────────┐  │  │
│  │  │     Application             │  │  │  Commands, Queries, Handlers
│  │  │  ┌───────────────────────┐  │  │  │
│  │  │  │   Domain (Innermost)  │  │  │  │  Aggregates, Entities, Value Objects
│  │  │  │   Pure Business Logic │  │  │  │  Domain Events, Enumerations
│  │  │  │   ZERO Dependencies   │  │  │  │
│  │  │  └───────────────────────┘  │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### Module Organization

Each module under `src/Modules/<ModuleName>` follows this structure:

```
CoreModule/
├── CoreModule.Domain/         # Pure business logic (NO external dependencies)
│   └── Model/
│       ├── CustomerAggregate/
│       │   ├── Customer.cs           (Aggregate Root)
│       │   └── Events/
│       ├── EmailAddress.cs           (Value Object)
│       └── CustomerStatus.cs         (Enumeration)
├── CoreModule.Application/    # Use case orchestration (references Domain only)
│   ├── Commands/
│   │   ├── CustomerCreateCommand.cs
│   │   └── CustomerCreateCommandHandler.cs
│   └── Queries/
├── CoreModule.Infrastructure/ # Technical implementation (references Domain + Application)
│   ├── EntityFramework/
│   │   ├── CoreModuleDbContext.cs
│   │   └── Configurations/
│   └── Repositories/
└── CoreModule.Presentation/   # Endpoints & DTOs (references Application via IRequester)
    └── Web/
        └── Endpoints/
            └── CustomerEndpoints.cs
```

## Review Priorities

### 🔴 CRITICAL Issues (Must Fix Before Merge)

These violations break architectural boundaries or introduce serious design flaws:

- **Layer boundary violations**: Dependency flows outward (e.g., Domain → Application, Application → Infrastructure)
- **Cross-module direct references**: Modules directly referencing each other (breaks modular isolation)
- **Domain impurity**: Domain layer has external dependencies (EF Core, Application, Infrastructure)
- **DbContext in Application/Domain**: Direct DbContext usage outside Infrastructure layer
- **Public setters on aggregates/entities**: Breaks encapsulation; use change methods instead
- **Exceptions for business rules**: Using exceptions instead of Result<T> for expected failures
- **Test independence violations**: Tests sharing mutable state or depending on execution order
- **Resource leaks**: IDisposable not properly disposed (missing using statements)
- **Endpoints with business logic**: Business rules implemented in Presentation layer

### 🟡 IMPORTANT Issues (Should Fix Soon)

These issues affect maintainability, consistency, or future extensibility:

- **CQRS naming violations**: Commands/queries not following `[Entity][Action]Command/Query` pattern
- **Repository pattern violations**: Not using IGenericRepository<T> in Application layer
- **Strongly-typed ID inconsistency**: Missing `[TypedEntityId<Guid>]` attributes
- **Specification pattern ignored**: Complex queries inline instead of using specifications
- **IRequester not used**: Endpoints directly instantiating handlers instead of using IRequester.SendAsync
- **N+1 query problems**: Missing .Include() causing excessive database round-trips
- **Async/await violations**: Blocking on async code (.Result, .Wait())
- **CancellationToken missing**: Async methods not accepting CancellationToken parameters
- **Domain event naming**: Not in past tense or missing "DomainEvent" suffix

### 🟢 SUGGESTIONS (Nice to Have)

These improve code quality but are not blocking:

- **XML documentation**: Missing on public APIs
- **Mapster configuration**: Ad-hoc mapping instead of MapperRegister
- **OpenAPI metadata**: Endpoints missing .WithName, .WithSummary, .Produces<T>
- **Modern C# features**: Opportunities to use pattern matching, expression-bodied members, etc.
- **Value object naming**: Generic names (Email) instead of descriptive (EmailAddress)

## Quick Reference to Detailed Guidance

This skill includes comprehensive checklists, examples, and templates:

### Checklists (6 files)

1. **[Layer Boundaries](checklists/01-layer-boundaries.md)**: Verify dependency flow, detect circular references
2. **[Domain Patterns](checklists/02-domain-patterns.md)**: Validate aggregates, entities, value objects, domain events
3. **[CQRS Patterns](checklists/03-cqrs-patterns.md)**: Check commands, queries, handlers, validators, IRequester usage
4. **[Repository & Data Access](checklists/04-repository-data-access.md)**: Ensure proper abstraction, specification pattern, N+1 detection
5. **[Presentation Endpoints](checklists/05-presentation-endpoints.md)**: Validate thin adapters, IRequester delegation, OpenAPI docs
6. **[Result & Error Handling](checklists/06-result-error-handling.md)**: Enforce Result<T> pattern, HTTP mapping, error clarity

### Examples (5 files)

1. **[Aggregate Patterns](examples/aggregate-patterns.md)**: WRONG vs CORRECT aggregate implementations
2. **[Value Object Patterns](examples/value-object-patterns.md)**: WRONG vs CORRECT value object implementations
3. **[CQRS Examples](examples/cqrs-examples.md)**: Command/query structure, handler delegation, IRequester usage
4. **[Result Pattern Examples](examples/result-pattern-examples.md)**: Using Result<T> instead of exceptions
5. **[Layer Violations](examples/layer-violations.md)**: Common boundary violations and how to fix them

### Documentation (2 files)

1. **[ADR Quick Reference](docs/adr-quick-reference.md)**: One-paragraph summaries of all 20 ADRs with quick lookup
2. **[bITdevKit Patterns](docs/bitdevkit-patterns.md)**: IRequester/INotifier, repository behaviors, module registration

### Templates (2 files)

1. **[Architecture Review Template](templates/architecture-review-template.md)**: Comment format for violations
2. **[Review Summary Template](templates/review-summary-template.md)**: Summary format with ADR references

## Example Workflow: Conducting an Architecture Review

### Step 1: Identify Scope

Determine which modules and layers are affected:

```bash
# Check which files changed in a PR
git diff main...feature-branch --name-only | grep "src/Modules/"

# Example output:
# src/Modules/CoreModule/CoreModule.Domain/Model/CustomerAggregate/Customer.cs
# src/Modules/CoreModule/CoreModule.Application/Commands/CustomerCreateCommand.cs
# src/Modules/CoreModule/CoreModule.Presentation/Web/Endpoints/CustomerEndpoints.cs
```

**Layers affected**: Domain, Application, Presentation

### Step 2: Check Layer Boundaries (🔴 CRITICAL)

**Use**: [checklists/01-layer-boundaries.md](checklists/01-layer-boundaries.md)

Verify dependencies flow inward only:

- ✅ **Domain**: No `using` statements referencing Application, Infrastructure, or Presentation
- ✅ **Application**: Only `using` statements referencing Domain
- ✅ **Presentation**: Uses `IRequester.SendAsync()` to call Application layer

**Reference**: ADR-0001 (Clean/Onion Architecture)

### Step 3: Validate Domain Patterns (🔴 CRITICAL)

**Use**: [checklists/02-domain-patterns.md](checklists/02-domain-patterns.md)

Check aggregate `Customer.cs`:

- ✅ Private setters on all properties
- ✅ Factory method `Customer.Create()` returns `Result<Customer>`
- ✅ Change methods (`ChangeName`, `ChangeEmail`) return `Result<Customer>`
- ✅ Domain events registered (`CustomerCreatedDomainEvent`)
- ✅ `[TypedEntityId<Guid>]` attribute on `CustomerId`
- ✅ Collection exposed as `IReadOnlyCollection<Address>`

**Reference**: ADR-0012 (Domain Logic in Domain Layer), ADR-0008 (Typed Entity IDs)

### Step 4: Review CQRS Implementation (🟡 IMPORTANT)

**Use**: [checklists/03-cqrs-patterns.md](checklists/03-cqrs-patterns.md)

Check `CustomerCreateCommand.cs`:

- ✅ Named `CustomerCreateCommand` (follows `[Entity][Action]Command` pattern)
- ✅ Nested `Validator` class using `AbstractValidator<T>`
- ✅ Handler uses `IGenericRepository<Customer>`, NOT DbContext
- ✅ Handler delegates to `Customer.Create()` (domain), not business logic in handler
- ✅ Returns `Result<CustomerId>`

**Reference**: ADR-0011 (Application Logic in Commands/Queries), ADR-0009 (FluentValidation)

### Step 5: Validate Repository Usage (🔴 CRITICAL)

**Use**: [checklists/04-repository-data-access.md](checklists/04-repository-data-access.md)

Check handler:

- ✅ Injects `IGenericRepository<Customer>` (abstraction)
- ❌ Injects `CoreModuleDbContext` directly → **WRONG (ADR-0004 violation)**

**Fix**: Replace DbContext with repository abstraction.

**Reference**: ADR-0004 (Repository Pattern with Decorator Behaviors)

### Step 6: Review Endpoints (🟡 IMPORTANT)

**Use**: [checklists/05-presentation-endpoints.md](checklists/05-presentation-endpoints.md)

Check `CustomerEndpoints.cs`:

- ✅ Derives from `EndpointsBase`
- ✅ Uses `IRequester.SendAsync(command, ct)` to delegate to Application
- ✅ Uses `.MapHttpCreated()` to map `Result<CustomerId>` to HTTP 201
- ✅ Includes `CancellationToken ct` parameter
- ⚠️ Missing `.WithName("CreateCustomer")` → **SUGGESTION (🟢)**

**Reference**: ADR-0014 (Minimal API Endpoints), ADR-0005 (Requester/Notifier)

### Step 7: Check Result<T> Error Handling (🔴 CRITICAL)

**Use**: [checklists/06-result-error-handling.md](checklists/06-result-error-handling.md)

Check error handling:

- ✅ `Customer.Create()` returns `Result<Customer>`
- ✅ Validation failures return `Result.Failure("error message")`
- ❌ `throw new ValidationException()` in domain method → **WRONG (ADR-0002 violation)**

**Fix**: Replace exception with `Result<T>`.

**Reference**: ADR-0002 (Result Pattern for Error Handling)

### Step 8: Generate Review Summary

**Use**: [templates/review-summary-template.md](templates/review-summary-template.md)

Create summary with:

- **Issues Found**: 🔴 2, 🟡 1, 🟢 1
- **Top 3 Priorities**: DbContext in Application (ADR-0004), Exception for business rule (ADR-0002), Missing endpoint name (🟢)
- **Architecture Compliance**: ⚠️ (critical issues present)
- **ADRs Referenced**: ADR-0001, ADR-0002, ADR-0004, ADR-0005, ADR-0008, ADR-0009, ADR-0011, ADR-0012, ADR-0014

## Integration with Architectural Decision Records (ADRs)

This skill references **20 Architectural Decision Records (ADRs)** located in `docs/ADR/`. Each ADR documents a key architectural decision with context, rationale, and consequences.

### How to Use ADRs in Reviews

1. **Identify the pattern**: Determine which architectural pattern is involved (e.g., layer boundaries, CQRS, Result<T>)
2. **Find the ADR**: Use [docs/adr-quick-reference.md](docs/adr-quick-reference.md) to locate the relevant ADR
3. **Reference in feedback**: Cite the ADR number and title in review comments (e.g., "ADR-0001: Clean/Onion Architecture")
4. **Explain the impact**: Use the ADR's rationale to explain why the violation matters

### Key ADRs by Category

**Core Architecture**:

- ADR-0001: Clean/Onion Architecture with Strict Layer Boundaries
- ADR-0003: Modular Monolith Architecture

**Domain & Data**:

- ADR-0012: Domain Logic Encapsulation in Domain Layer
- ADR-0008: Typed Entity IDs using Source Generators
- ADR-0007: Entity Framework Core with Code-First Migrations

**Application Layer**:

- ADR-0011: Application Logic in Commands & Queries
- ADR-0009: FluentValidation Strategy
- ADR-0010: Mapster for Object Mapping

**Error Handling**:

- ADR-0002: Result Pattern for Error Handling

**Presentation & API**:

- ADR-0014: Minimal API Endpoints with DTO Exposure
- ADR-0005: Requester/Notifier (Mediator) Pattern

**Infrastructure**:

- ADR-0004: Repository Pattern with Decorator Behaviors
- ADR-0019: Specification Pattern for Repository Queries

See [docs/adr-quick-reference.md](docs/adr-quick-reference.md) for complete list with one-paragraph summaries.

## Common Violation Patterns

### 🔴 CRITICAL: Domain → Application Dependency

**Symptom**: Domain layer references Application types (commands, queries, handlers)

```csharp
// WRONG: Domain references Application
namespace MyApp.Domain.CustomerAggregate;

using MyApp.Application.Commands; // ❌ Domain → Application dependency

public class Customer : AggregateRoot<CustomerId>
{
    public CustomerCreatedCommand ToCommand() // ❌ Domain knows about Application
    {
        return new CustomerCreatedCommand(this.FirstName, this.LastName);
    }
}
```

**Why Critical**: Violates ADR-0001 (Clean/Onion Architecture). Domain must be pure business logic with ZERO external dependencies.

**Fix**: Remove Application reference. Application layer creates commands from domain entities, not vice versa.

**Reference**: [examples/layer-violations.md](examples/layer-violations.md)

### 🔴 CRITICAL: Application → Infrastructure (DbContext)

**Symptom**: Application handlers inject DbContext directly

```csharp
// WRONG: Application uses DbContext
namespace MyApp.Application.Commands;

using MyApp.Infrastructure.EntityFramework; // ❌ Application → Infrastructure dependency

public class CustomerCreateCommandHandler
{
    private readonly CoreModuleDbContext context; // ❌ Direct DbContext usage

    public async Task<Result<CustomerId>> Handle(CustomerCreateCommand request, CancellationToken ct)
    {
        var customer = Customer.Create(...);
        this.context.Customers.Add(customer); // ❌ Application knows about EF Core
        await this.context.SaveChangesAsync(ct);
    }
}
```

**Why Critical**: Violates ADR-0001 (layer boundaries) and ADR-0004 (repository pattern). Application layer cannot reference Infrastructure.

**Fix**: Use `IGenericRepository<Customer>` abstraction.

**Reference**: [examples/layer-violations.md](examples/layer-violations.md), [checklists/04-repository-data-access.md](checklists/04-repository-data-access.md)

### 🔴 CRITICAL: Exceptions for Business Rules

**Symptom**: Domain methods throw exceptions for validation failures

```csharp
// WRONG: Exception for business rule
public static Customer Create(string name, string email)
{
    if (string.IsNullOrWhiteSpace(name))
    {
        throw new ValidationException("Name is required"); // ❌ Exception for expected failure
    }

    return new Customer(name, email);
}
```

**Why Critical**: Violates ADR-0002 (Result Pattern). Exceptions should only be used for truly exceptional cases, not expected failures.

**Fix**: Return `Result<Customer>` instead.

**Reference**: [examples/result-pattern-examples.md](examples/result-pattern-examples.md)

### 🟡 IMPORTANT: CQRS Naming Violations

**Symptom**: Commands/queries not following naming conventions

```csharp
// WRONG: Poor naming
public sealed record CreateCustomerRequest(...) : IRequest<Result<CustomerId>>; // ❌ "Request" suffix
public sealed record GetCustomer(...) : IRequest<Result<CustomerModel>>; // ❌ Missing "Query" suffix
```

**Why Important**: Violates ADR-0011 (CQRS patterns). Inconsistent naming makes codebase harder to navigate.

**Fix**: Use `[Entity][Action]Command` and `[Entity][Action]Query` patterns.

**Reference**: [checklists/03-cqrs-patterns.md](checklists/03-cqrs-patterns.md)

### 🟡 IMPORTANT: N+1 Query Problem

**Symptom**: Missing .Include() causes multiple database round-trips

```csharp
// WRONG: N+1 query problem
var customers = await repository.FindAllAsync(cancellationToken: ct);
foreach (var customer in customers)
{
    // Each iteration triggers a separate query for addresses!
    var addresses = customer.Addresses.ToList();
}
```

**Why Important**: Performance issue. Can cause significant slowdowns with large datasets.

**Fix**: Use eager loading with specifications.

**Reference**: [checklists/04-repository-data-access.md](checklists/04-repository-data-access.md)

## Success Criteria for Architecture Reviews

Code passes architectural review when:

- ✅ **No layer boundary violations**: Dependencies flow inward only
- ✅ **Domain layer is pure**: ZERO external dependencies (only bITdevKit domain abstractions)
- ✅ **No circular module references**: Modules are self-contained
- ✅ **Repository abstractions used**: Application uses `IGenericRepository<T>`, not DbContext
- ✅ **Result<T> pattern enforced**: Domain methods return Result for failures, not exceptions
- ✅ **Aggregates properly encapsulated**: Private setters, change methods, factory methods
- ✅ **CQRS naming consistent**: Commands/queries follow `[Entity][Action]Command/Query` pattern
- ✅ **Endpoints are thin adapters**: No business logic in Presentation layer
- ✅ **IRequester pattern used**: Endpoints delegate to Application via `IRequester.SendAsync()`
- ✅ **No N+1 query problems**: Proper eager loading with .Include() or specifications
- ✅ **CancellationToken propagated**: All async methods accept and pass CancellationToken
- ✅ **Test independence**: No shared mutable state between tests

## Tips for Effective Architecture Reviews

### Do

- **Start with layer boundaries**: Verify dependencies flow inward before checking patterns
- **Reference ADRs**: Always cite the relevant ADR number and title in feedback
- **Provide examples**: Show WRONG vs CORRECT code, not just abstract explanations
- **Explain impact**: Don't just say "this violates ADR-0001", explain *why* it matters
- **Use checklists systematically**: Work through checklists sequentially to avoid missing issues
- **Distinguish CRITICAL vs IMPORTANT**: Use emoji indicators (🔴🟡🟢) to prioritize feedback
- **Test architecture rules**: Suggest NetArchTest rules to prevent future violations

### Don't

- **Mix code quality and architecture**: Focus on architectural concerns; use review-code skill for code quality
- **Overwhelm with minor issues**: Prioritize critical and important issues over suggestions
- **Accept "it works" as justification**: Architecture violations accumulate technical debt
- **Skip positive feedback**: Acknowledge good patterns when you see them
- **Review without ADR context**: Always consult ADRs to understand the rationale behind patterns

## Related Skills

- **review-code**: Use for code quality, security, testing, performance, and documentation reviews
- **domain-add-aggregate**: Use to scaffold new domain aggregates following DDD patterns
- **adr-writer**: Use to create new ADRs for architectural decisions

## References

- [AGENTS.md](../../../AGENTS.md): Project-specific architecture patterns and conventions
- [.github/copilot-instructions.md](../../copilot-instructions.md): Detailed coding guidelines
- [docs/ADR/](../../../docs/ADR/): All 20 Architectural Decision Records
- [bITdevKit Documentation](https://github.com/BridgingIT-GmbH/bITdevKit/tree/main/docs): Official bITdevKit patterns and features

---

**Version**: 1.0
**Last Updated**: 2026-01-14
**Maintainer**: bITdevKit Team
