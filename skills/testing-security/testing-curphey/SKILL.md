---
name: testing
description: "Test-first development process and test quality review. Use when writing new code, reviewing PRs, diagnosing flaky or failing tests, or improving test coverage. Also use when fixing bugs (write failing test first), adding features (test defines 'done'), or when tests pass but code doesn't work. Essential for TDD, test pyramid strategy, mocking decisions, AAA pattern, table-driven tests, and handling flaky tests."
---

# Testing Skill

## Overview

Tests written after code pass immediately—proving nothing. This skill guides test-first development and systematic test quality review.

**Core principle:** Write the test FIRST. Watch it fail. Then write code to pass. If you didn't see it fail, you don't know it tests the right thing.

## The Test-First Process

### Phase 1: Write Failing Test

**Before writing ANY implementation code:**

1. **Write One Minimal Test**
   - Tests ONE behavior
   - Clear name describing expected outcome
   - See `references/test-pyramid.md` for structure

2. **Watch It Fail**
   - Run the test
   - Confirm it fails for the expected reason (missing feature, not typo)
   - If test passes immediately → test is wrong, fix it

3. **Verify Failure Message**
   - Does the error explain what's wrong?
   - Would you understand this failure in 6 months?

### Phase 2: Write Minimal Code

**Only enough code to pass the test:**

1. **Implement Minimally**
   - Don't add features not tested
   - Don't optimize
   - Don't refactor yet

2. **Watch Test Pass**
   - Run the test
   - Confirm it passes
   - If it fails → fix code, not test

3. **Check Other Tests**
   - All existing tests still pass?
   - If not → fix immediately before continuing

### Phase 3: Refactor

**Only after tests pass:**

1. **Clean Up Code**
   - Remove duplication
   - Improve names
   - Extract helpers

2. **Keep Tests Green**
   - Run tests after every change
   - If tests fail → undo refactor, try again

3. **Don't Add Behavior**
   - Refactoring changes structure, not behavior
   - New behavior requires new test first

### Repeat

Write next failing test. Repeat cycle.

## Red Flags - STOP and Fix

### You're Doing It Wrong If:

```
- Writing code before tests
- Test passes immediately (you're testing existing behavior)
- Adding "just one more feature" without a test
- Debugging why test fails instead of why code is wrong
- Tests require running the whole system
- Can't describe what test is testing in plain English
```

### Test Quality Red Flags:

```
- No assertions (test always passes)
- Assertions on mock behavior instead of real code
- Test logic (if/else in tests)
- Tests depend on each other (order matters)
- Flaky tests (pass sometimes)
- Slow tests (>100ms for unit tests)
```

### Coverage Red Flags:

```
- High coverage, but bugs still found
- Tests only cover happy path
- Tests don't cover error handling
- Tests don't cover edge cases (null, empty, max values)
```

## Common Rationalizations - Don't Accept These

| Excuse | Reality |
|--------|---------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll write tests after" | Tests after pass immediately. Proves nothing. |
| "I already manually tested" | Manual testing isn't repeatable. Automate it. |
| "Tests slow me down" | Debugging slows you down more. TDD is faster. |
| "It's hard to test" | Hard to test = hard to use. Improve the design. |
| "We're moving fast" | Fast now, slow later. Tests enable speed. |
| "I'll refactor later to make it testable" | Do it now. Later never comes. |

## Test Quality Checklist

Before approving any code:

- [ ] **Test First**: Was the test written before the code?
- [ ] **Failure Seen**: Did developer watch test fail first?
- [ ] **One Behavior**: Does each test verify one thing?
- [ ] **Clear Name**: Does test name describe expected behavior?
- [ ] **No Logic**: Are tests simple (no if/else, loops)?
- [ ] **Independent**: Do tests pass in any order?
- [ ] **Fast**: Do unit tests run in <100ms each?
- [ ] **Edge Cases**: Are boundaries and errors covered?

## Test Structure

### The AAA Pattern

```javascript
test('returns user when valid ID provided', async () => {
  // Arrange - set up test data
  const userId = 'user-123';
  await createTestUser({ id: userId, name: 'Test' });

  // Act - execute the code
  const result = await getUser(userId);

  // Assert - verify the outcome
  expect(result.name).toBe('Test');
});
```

### Naming Convention

```
Pattern: [unit] [behavior] when [condition]

Examples:
- "returns empty array when no users exist"
- "throws NotFoundError when user not found"
- "sends welcome email when user signs up"
```

## Test Pyramid

```
         /\
        /  \      E2E (10%) - Critical user journeys only
       /----\
      /      \    Integration (20%) - API, DB, services
     /--------\
    /          \  Unit (70%) - Business logic, pure functions
   /------------\
```

### What to Test at Each Level

| Level | Test | Don't Test |
|-------|------|------------|
| **Unit** | Business logic, calculations, transformations | Frameworks, libraries, I/O |
| **Integration** | API endpoints, database queries, service calls | UI, external services |
| **E2E** | Critical user flows, auth, checkout | Everything (too slow) |

## Quick Commands

```bash
# Run with coverage
npm test -- --coverage
pytest --cov=src
go test -cover ./...

# Run specific test
npm test -- -t "user"
pytest -k "user"
go test -run TestUser

# Watch mode
npm test -- --watch
pytest-watch
```

## References

Detailed patterns and examples in `references/`:
- `test-pyramid.md` - Test strategy and coverage guidelines
- `mocking-guide.md` - When and how to mock
- `flaky-tests.md` - Diagnosing and fixing flaky tests
