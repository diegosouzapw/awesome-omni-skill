---
name: testing
description: Testing patterns and best practices for unit, integration, and E2E testing.
globs: ["**/*.test.js", "**/*.spec.js", "**/*.test.ts", "**/*.spec.ts", "**/test/**", "**/tests/**", "**/__tests__/**", "**/jest.config.*", "**/vitest.config.*", "**/playwright.config.*", "**/cypress.config.*"]
priority: 85
tags: ["workflow"]
---

# Testing Best Practices

## Test Structure (AAA Pattern)
- Arrange: Set up test data and conditions
- Act: Execute the code under test
- Assert: Verify the expected outcome
- Use descriptive test names: "should X when Y"

## Unit Tests
- Test one behavior per test
- Mock external dependencies
- Test edge cases and error paths
- Keep tests fast (< 100ms each)
- Use factories for test data
- Aim for 80%+ coverage

## Integration Tests
- Test component interactions
- Use test database/containers
- Clean up after tests
- Test happy and error paths
- Use realistic data

## E2E Tests
- Test critical user flows only
- Keep tests independent
- Use stable selectors (data-testid)
- Handle async properly
- Run in CI with retries

## Mocking
- Mock at boundaries (APIs, DB)
- Use factories for test data
- Don't over-mock (test real behavior)
- Reset mocks between tests
- Prefer fakes over mocks when possible

## Tools
- Jest/Vitest for unit tests
- Playwright/Cypress for E2E
- MSW for API mocking
- Faker for test data
- Testing Library for component tests
