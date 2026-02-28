---
name: test-pyramid-design
description: Design optimal test pyramids with unit/integration/E2E ratios. Identify anti-patterns and recommend architecture-specific testing strategies.
allowed-tools: Read, Write, Glob, Grep, Task, WebSearch, WebFetch
---

# Test Pyramid Design

## When to Use This Skill

Use this skill when:

- **Test Pyramid Design tasks** - Working on design optimal test pyramids with unit/integration/e2e ratios. identify anti-patterns and recommend architecture-specific testing strategies
- **Planning or design** - Need guidance on Test Pyramid Design approaches
- **Best practices** - Want to follow established patterns and standards

## Overview

The Test Pyramid, introduced by Mike Cohn, visualizes the ideal distribution of tests across different levels. More granular tests at the bottom (fast, cheap) and fewer broad tests at the top (slow, expensive).

## The Classic Test Pyramid

```text
                    ┌───────┐
                   /  E2E    \         ~10%
                  /   Tests   \        (UI, Acceptance)
                 /─────────────\
                /  Integration  \      ~20%
               /     Tests       \     (API, Component)
              /───────────────────\
             /      Unit Tests     \   ~70%
            /       (Fast, Cheap)   \  (Methods, Classes)
           └─────────────────────────┘
```

## Test Level Characteristics

| Level | Speed | Cost | Scope | Confidence | Maintenance |
|-------|-------|------|-------|------------|-------------|
| Unit | Fastest (ms) | Lowest | Single unit | Low | Low |
| Integration | Medium (s) | Medium | Components | Medium | Medium |
| E2E | Slowest (min) | Highest | Full system | High | High |

## Common Pyramid Shapes

### Healthy Pyramid ✅

```text
     /\          Unit: 70%+
    /  \         Integration: 20%
   /    \        E2E: 10%
  /      \
 /________\      Fast feedback, low maintenance
```

**Characteristics**:

- Fast CI/CD pipeline
- Quick feedback loop
- Low flakiness
- Easy maintenance

### Ice Cream Cone ❌

```text
   ██████████     E2E: 60%
    ████████      Integration: 30%
      ████        Unit: 10%
```

**Problems**:

- Slow pipelines (hours)
- Flaky tests
- Expensive maintenance
- Late feedback

**Fix**: Extract unit tests, reduce E2E scope

### Cupcake ❌

```text
   ██████████     E2E: 40%
   ██████████     Manual: 50%
      ████        Unit: 10%
```

**Problems**:

- High manual testing cost
- Inconsistent coverage
- Slow release cycles

**Fix**: Automate critical paths, add integration layer

### Hourglass ❌

```text
      ██          E2E: 10%
   ██████████     Integration: 70%
      ██          Unit: 20%
```

**Problems**:

- Slow integration tests
- Brittle test fixtures
- Missing unit test coverage

**Fix**: Push coverage down to unit level

## Architecture-Specific Pyramids

### Monolithic Application

```text
     /\          Unit: 70%
    /  \         Integration: 20%
   /    \        E2E: 10%
  /______\
```

Focus on unit tests for business logic.

### Microservices

```text
     /\          E2E/Contract: 10%
    /  \         Integration: 30%
   /    \        Unit: 60%
  /______\
```

More integration tests for service boundaries.
Add contract tests between services.

### Event-Driven Architecture

```text
     /\          E2E: 10%
    /  \         Integration: 35%
   /    \        Unit: 55%
  /______\
```

Integration tests for event handlers.
Unit tests for event processing logic.

### Frontend-Heavy (SPA)

```text
     /\          E2E: 15%
    /  \         Integration: 25%
   /    \        Unit/Component: 60%
  /______\
```

Component tests replace some unit tests.
Visual regression testing at integration level.

## Testing Trophy (Kent C. Dodds)

Alternative for frontend applications:

```text
                 ┌─────┐
                 │ E2E │              ~5%
              ┌──┴─────┴──┐
              │Integration│           ~50%
           ┌──┴───────────┴──┐
           │    Static        │       ~10%
        ┌──┴─────────────────┴──┐
        │        Unit            │    ~35%
        └────────────────────────┘
```

## Designing Your Pyramid

### Step 1: Assess Current State

Count tests by level:

```text
Level        | Count | % Total | Time    | Pass Rate
-------------|-------|---------|---------|----------
E2E          | 150   | 45%     | 30 min  | 85%
Integration  | 100   | 30%     | 10 min  | 95%
Unit         | 80    | 25%     | 30 sec  | 99%
```

### Step 2: Identify Anti-Pattern

| If You Have... | Shape | Priority Fix |
|----------------|-------|--------------|
| E2E > 30% | Ice Cream Cone | Extract to lower levels |
| Unit < 50% | Inverted | Add unit tests for logic |
| Integration > 50% | Hourglass | Push to unit level |
| Manual > 20% | Cupcake | Automate critical paths |

### Step 3: Set Target Ratios

Based on architecture:

| Architecture | Unit | Integration | E2E |
|--------------|------|-------------|-----|
| Monolith | 70% | 20% | 10% |
| Microservices | 60% | 30% | 10% |
| Serverless | 50% | 40% | 10% |
| Frontend SPA | 40% | 45% | 15% |

### Step 4: Migration Plan

Prioritize by ROI:

1. **Quick wins**: Convert flaky E2E to integration
2. **Coverage gaps**: Add unit tests for critical logic
3. **Contract tests**: Add for service boundaries
4. **Remove duplicates**: Consolidate redundant E2E tests

## Test Level Guidelines

### Unit Tests

**Should test**:

- Business logic
- Algorithms
- Data transformations
- Validation rules
- Edge cases

**Should NOT test**:

- External services
- Database queries
- File system operations
- Third-party libraries

### Integration Tests

**Should test**:

- Database repository operations
- API endpoints
- Message handlers
- External service adapters
- Component interactions

**Should NOT test**:

- Pure business logic (use unit tests)
- Full user journeys (use E2E)

### E2E Tests

**Should test**:

- Critical user journeys
- Happy paths
- Smoke tests
- Cross-system workflows

**Should NOT test**:

- Edge cases (use unit tests)
- API contracts (use integration tests)
- Everything (be selective)

## .NET Example: Test Project Structure

```text
src/
  MyApp.Domain/           # Business logic
  MyApp.Application/      # Use cases
  MyApp.Infrastructure/   # Data access
  MyApp.Api/              # HTTP endpoints

tests/
  MyApp.Domain.Tests/           # Unit tests (70%)
    Features/
      Orders/
        CreateOrderTests.cs
        OrderValidationTests.cs

  MyApp.Application.Tests/      # Integration tests (20%)
    Features/
      Orders/
        CreateOrderHandlerTests.cs

  MyApp.Api.Tests/              # E2E tests (10%)
    Features/
      Orders/
        OrdersEndpointTests.cs
```

### Test Distribution Verification

```csharp
// Build script or CI check
public class TestDistributionTests
{
    [Fact]
    public void TestPyramidRatiosAreHealthy()
    {
        var unitTests = CountTestsInAssembly("MyApp.Domain.Tests");
        var integrationTests = CountTestsInAssembly("MyApp.Application.Tests");
        var e2eTests = CountTestsInAssembly("MyApp.Api.Tests");

        var total = unitTests + integrationTests + e2eTests;

        Assert.True(unitTests / total >= 0.60, "Unit tests should be ≥60%");
        Assert.True(e2eTests / total <= 0.15, "E2E tests should be ≤15%");
    }
}
```

## Metrics and Monitoring

### Pipeline Health

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Unit test time | < 1 min | 1-5 min | > 5 min |
| Integration time | < 5 min | 5-15 min | > 15 min |
| E2E test time | < 10 min | 10-30 min | > 30 min |
| Flaky test rate | < 1% | 1-5% | > 5% |

### Coverage Balance

Track coverage by pyramid level:

- Unit coverage of business logic: ≥90%
- Integration coverage of APIs: ≥80%
- E2E coverage of critical paths: 100%

## Integration Points

**Inputs from**:

- Architecture documents → Pyramid shape
- Risk assessment → E2E scope

**Outputs to**:

- `test-strategy-planning` skill → Strategy document
- CI/CD configuration → Pipeline stages
- `automation-strategy` skill → Automation scope
