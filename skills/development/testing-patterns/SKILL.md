---
name: testing-patterns
description: TDD and unit testing guidance for Crispy CRM. Use when writing tests, implementing TDD, debugging test failures, or setting up test infrastructure. Covers Vitest patterns, React Admin component testing, Zod schema validation testing, Supabase mocking, E2E with Playwright, and manual E2E testing with Claude Chrome. Integrates with verification-before-completion for test verification.
---

# Testing Patterns for Crispy CRM

## Overview

Comprehensive testing guidance for the Crispy CRM codebase. Covers unit testing with Vitest, component testing with React Admin context, validation testing with Zod schemas, and E2E testing with Playwright.

**Philosophy:** Tests should verify behavior, not implementation. Focus on what the code does, not how it does it.

## When to Use

Use this skill when:
- Writing new tests (unit, integration, E2E)
- Implementing TDD (Test-Driven Development)
- Debugging failing tests
- Setting up test infrastructure
- Mocking Supabase or React Admin
- Understanding test coverage requirements
- Writing Claude Chrome manual E2E prompts

## Test Stack

| Layer | Tool | Location |
|-------|------|----------|
| **Unit Tests** | Vitest | `src/**/__tests__/*.test.ts` |
| **Component Tests** | Vitest + React Testing Library | `src/**/__tests__/*.test.tsx` |
| **Validation Tests** | Vitest + Zod | `src/atomic-crm/validation/__tests__/` |
| **E2E Tests** | Playwright | `tests/e2e/` |
| **Manual E2E** | Claude Chrome | `docs/tests/e2e/` |
| **Database Tests** | pgTAP | `supabase/tests/` |

## TDD Workflow

### The Red-Green-Refactor Cycle

1. **RED:** Write a failing test that describes expected behavior. Run it to confirm failure.
2. **GREEN:** Write minimal code to make the test pass. Speed over elegance.
3. **REFACTOR:** Clean up while tests still pass. This step is NOT optional.

### Arrange-Act-Assert Pattern

Structure every test with three clear sections:

```typescript
it('calculates total with tax', () => {
  // Arrange: Set up test data
  const items = [{ price: 100 }, { price: 50 }];
  const taxRate = 0.1;

  // Act: Execute the code
  const total = calculateTotalWithTax(items, taxRate);

  // Assert: Verify the outcome
  expect(total).toBe(165);
});
```

### Where TDD Thrives

| Domain | TDD Effectiveness | Why |
|--------|-------------------|-----|
| **Validation schemas** | Excellent | Well-defined inputs/outputs |
| **Pure functions** | Excellent | No side effects |
| **Data providers** | Very Good | Clear API contracts |
| **Hooks** | Very Good | Isolated logic |
| **Components** | Good | Need user interaction testing |
| **UI layout** | Fair | Visual verification better |

### TDD in Watch Mode

```bash
just test:watch
# 1. Write failing test -> See RED in terminal
# 2. Write code -> See GREEN
# 3. Refactor -> Confirm still GREEN
# 4. Commit -> Repeat
```

### Common TDD Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Skipping refactor | Technical debt accumulates | Refactor after EVERY green |
| Testing implementation | Brittle tests break on refactor | Test inputs -> outputs only |
| Writing tests after code | Confirmation bias | Discipline: test FIRST |
| Chasing 100% coverage | Meaningless tests | Focus on behavior coverage |
| Giant test steps | Hard to debug failures | Small increments |

## Automatic Activation

This skill activates automatically for implementation tasks. When you see:
- "implement feature", "create component", "add handler", "new schema"
- Any file modification in `src/atomic-crm/**/*.ts` or `src/atomic-crm/**/*.tsx`

**The skill will remind you:**
1. Write test FIRST (Red phase)
2. Run test to confirm it fails
3. Implement minimal code (Green phase)
4. Refactor while tests pass
5. Verify with `just test` before claiming complete

### Integration with verification-before-completion

Testing is now enforced at completion time:
- Cannot claim "done" without test evidence
- Cannot claim "feature complete" without passing tests
- UI changes prompt for Manual E2E via Claude Chrome

## Quick Reference

### Running Tests

```bash
just test              # All unit tests
just test:watch        # Watch mode for TDD
just test:coverage     # Generate coverage report
just test:ui           # Vitest UI for debugging
npx playwright test    # E2E tests
npx supabase test db   # Database tests
```

### Coverage Requirements

| Type | Minimum | Target |
|------|---------|--------|
| **Validation schemas** | 90% | 100% |
| **Data providers** | 80% | 90% |
| **Components** | 70% | 80% |
| **Hooks** | 80% | 90% |
| **E2E critical paths** | 100% | 100% |

## Integration with Other Skills

| Skill | Integration |
|-------|-------------|
| `verification-before-completion` | Run tests before claiming "done" |
| `fail-fast-debugging` | Use test failures to trace root cause |
| `enforcing-principles` | Tests verify schema validation at API boundary |

## Decision Tree

```
Need to write tests?
|
+- New feature?
|  +- Start with TDD -> Write failing test first
|
+- Bug fix?
|  +- Write test that reproduces bug -> Fix -> Verify passes
|
+- Validation logic?
|  +- Test Zod schemas -> Valid/invalid inputs, coercion, defaults
|
+- React component?
|  +- Use renderWithAdminContext -> Test user interactions
|
+- Database logic?
|  +- pgTAP tests -> RLS policies, constraints, triggers
|
+- Full user journey?
   +- Playwright E2E -> Critical path scenarios
```

## Red Flags - STOP and Review

If you find yourself:
- Writing tests after code is "done" -> Consider TDD next time
- Testing internal state -> Test behavior instead
- Skipping async waitFor -> Race condition risk
- Using raw `render()` for React Admin components -> Use `renderWithAdminContext`
- No assertions in test -> Test is meaningless
- Massive snapshots -> Use specific assertions

**Remember:** Tests are documentation. They should clearly express what the code does.

## Resources

For detailed implementation patterns, see the reference files below:

- [vitest-patterns.md](references/vitest-patterns.md) - Test file structure, hook testing, anti-patterns, ESLint enforcement, CLI commands
- [mock-patterns.md](references/mock-patterns.md) - Supabase mocking, per-test overrides, typed mock factories
- [react-admin-testing.md](references/react-admin-testing.md) - renderWithAdminContext, form/error/accessibility testing, E2E with Playwright, Claude Chrome manual E2E, pgTAP
- [zod-testing.md](references/zod-testing.md) - Schema validation testing, coercion, defaults, strict vs passthrough

<!-- @resource references/vitest-patterns.md "Vitest configuration and test patterns" -->
<!-- @resource references/mock-patterns.md "Mock setup and typed factories" -->
<!-- @resource references/react-admin-testing.md "React Admin component testing, E2E, and database testing" -->
<!-- @resource references/zod-testing.md "Zod schema validation testing patterns" -->
