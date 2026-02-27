---
name: vitest-testing
description: Write and run Vitest unit and integration tests. Use when creating tests, debugging test failures, or checking test coverage.
metadata:
  version: '1.0'
---

# Vitest Testing

## Running Tests

```bash
# Run all tests
npx vitest run

# Run in watch mode
npx vitest

# Run with coverage
npx vitest run --coverage

# Run a specific test file
npx vitest run src/utils/stats.test.ts

# Run tests matching a pattern
npx vitest run -t "rolling average"
```

## Test File Naming

- Unit tests: `<module>.test.ts` co-located with the source file
- Integration tests: `<feature>.integration.test.ts` in a `__tests__/` directory

## Test Structure

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';

describe('ModuleName', () => {
  describe('functionName', () => {
    it('should describe expected behavior', () => {
      // Arrange
      const input = createTestInput();

      // Act
      const result = functionName(input);

      // Assert
      expect(result).toEqual(expectedOutput);
    });

    it('should handle edge case', () => {
      expect(() => functionName(null)).toThrow('Expected valid input');
    });
  });
});
```

## Best Practices

- Test behavior, not implementation. If internal refactoring breaks tests, the tests are too coupled.
- Use descriptive test names: `should calculate rolling AHI average with gaps in data`
- Test edge cases: empty arrays, single elements, NaN values, boundary conditions.
- Mock external dependencies (IndexedDB, OPFS, Web Workers) using `vi.mock()`.
- Use `beforeEach` for setup, not shared mutable state.
- For statistical functions, compare against known values from reference implementations.
- Use `toBeCloseTo()` for floating-point comparisons.
