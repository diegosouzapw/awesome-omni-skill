---
name: dotnet-testing
description: "Write and run .NET tests following TDD principles. Use when writing tests, implementing TDD workflow, verifying test coverage, or debugging test failures."
---

# .NET Testing Standards

## When to Use This Skill

Use this skill when you need to:

- Write unit tests for new functionality
- Follow TDD (Test-Driven Development) workflow
- Debug failing tests
- Review test quality and coverage

## Prerequisites

**Before writing tests:** Read `.planning/CONVENTIONS.md` for repository-specific test patterns (framework, naming, mocking library). If it doesn't exist, the calling agent should invoke the `Repo Analyser` subagent first.

## Hard Rules

### Must

1. **Write failing test first (TDD)** — Test drives the implementation, not vice versa
2. **One assertion concept per test** — Test one behaviour, multiple asserts on same object is OK
3. **Use repo's existing test framework** — Don't mix xUnit and NUnit in same solution
4. **Tests must be deterministic** — No dependency on time, random, or external state
5. **Follow repo's naming convention** — Check CONVENTIONS.md for pattern
6. **Test behaviour, not implementation** — Tests should survive refactoring

### Must Not

1. **Test private methods directly** — Test through public API; if you need to test private, extract a class
2. **Use Thread.Sleep in tests** — Use async/await, polling with timeout, or test doubles
3. **Share mutable state between tests** — Each test gets fresh state
4. **Mock what you don't own** — Wrap third-party APIs, mock the wrapper
5. **Write tests for trivial code** — Auto-properties, simple DTOs don't need tests
6. **Depend on test execution order** — Tests must run independently and in parallel

## TDD Workflow

1. **RED** — Write a test that expects the behaviour. Run it — it MUST fail. If it passes, the test is wrong or the feature already exists.
2. **GREEN** — Write the MINIMUM code to make the test pass. Hard-coding is acceptable if it satisfies the test.
3. **REFACTOR** — Remove duplication, improve naming, extract methods/classes. Run ALL tests — they MUST still pass.

## Commands Reference

All `dotnet` commands work identically across all shells.

```
dotnet test
dotnet test ./tests/MyProject.UnitTests/
dotnet test --filter "FullyQualifiedName~OrderServiceTests"
dotnet test --filter "Category=Unit"
dotnet test --filter "FullyQualifiedName!~IntegrationTests & FullyQualifiedName!~Integration.Tests"
dotnet test --list-tests
dotnet test --collect:"XPlat Code Coverage"
```

The compound filter excludes all tests from integration test projects — covering both `IntegrationTests` and `Integration.Tests` naming conventions. Use this when integration tests are excluded via the `quality-gates` skill protocol.

## Verification Checklist

Before considering tests complete:

- [ ] Test fails before implementation (TDD RED)
- [ ] Test passes after implementation (TDD GREEN)
- [ ] Refactoring done with tests still passing
- [ ] Follows naming convention from CONVENTIONS.md
- [ ] One behaviour per test
- [ ] No flaky/non-deterministic elements
- [ ] Mocks are for external dependencies only
- [ ] Test class has same structure as others in repo
