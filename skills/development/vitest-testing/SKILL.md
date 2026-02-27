---
name: vitest-testing
description: Vitest unit testing for TypeScript/JavaScript in Oh My Brand! theme. Test setup, Web Component testing, mocking patterns, and coverage. Use when writing unit tests for frontend code.
metadata:
  author: Wesley Smits
  version: "1.0.0"
---

# Vitest Testing

Unit testing TypeScript/JavaScript code with Vitest for the Oh My Brand! WordPress FSE theme.

---

## When to Use

- Writing unit tests for Web Components
- Testing utility functions (debounce, throttle, etc.)
- Mocking browser APIs (IntersectionObserver, matchMedia)
- Achieving code coverage requirements

---

## Configuration

| File | Template | Purpose |
|------|----------|---------|
| [vitest.config.ts](references/vitest.config.ts) | Vitest configuration | Test settings and coverage |
| [setup.ts](references/setup.ts) | Test setup | Browser API mocks |

---

## Test Templates

| Template | Purpose |
|----------|---------|
| [web-component.test.ts](references/web-component.test.ts) | Web Component tests |
| [debounce.test.ts](references/debounce.test.ts) | Utility function tests |
| [mocking-patterns.ts](references/mocking-patterns.ts) | Mocking examples |

---

## Test Structure

```typescript
import { describe, it, expect, beforeEach, afterEach, vi } from 'vitest';

describe('ComponentName', () => {
    let element: HTMLElement;

    beforeEach(() => {
        document.body.innerHTML = `<my-component></my-component>`;
        element = document.querySelector('my-component')!;
    });

    afterEach(() => {
        document.body.innerHTML = '';
    });

    it('should initialize correctly', () => {
        expect(element).toBeDefined();
    });
});
```

---

## Mock Patterns Quick Reference

### Mock Functions

```typescript
const mockFn = vi.fn();
mockFn.mockReturnValue('value');
mockFn.mockResolvedValue({ data: [] });
expect(mockFn).toHaveBeenCalledWith('arg');
```

### Mock Timers

```typescript
vi.useFakeTimers();
vi.advanceTimersByTime(100);
vi.useRealTimers();
```

### Spy on Methods

```typescript
const spy = vi.spyOn(object, 'method');
spy.mockReturnValue('mocked');
expect(spy).toHaveBeenCalled();
```

See [mocking-patterns.ts](references/mocking-patterns.ts) for complete examples.

---

## Coverage

### Thresholds

| Metric | Threshold |
|--------|-----------|
| Statements | 80% |
| Branches | 80% |
| Functions | 80% |
| Lines | 80% |

### What to Test

**Always test:**
- Public methods and functions
- Edge cases (empty arrays, null values)
- Error handling paths
- User interactions
- Attribute change callbacks

**Don't test:**
- Third-party library internals
- Private implementation details
- Simple getters/setters

---

## Running Tests

| Command | Purpose |
|---------|---------|
| `pnpm test` | Run all tests |
| `pnpm run test:watch` | Watch mode |
| `pnpm run test:coverage` | With coverage |
| `pnpm test -- --testNamePattern="nav"` | Filter by name |
| `pnpm test src/blocks/gallery/` | Specific directory |

---

## Related Skills

- [typescript-standards](../typescript-standards/SKILL.md) - TypeScript conventions
- [web-components](../web-components/SKILL.md) - Web Component patterns
- [phpunit-testing](../phpunit-testing/SKILL.md) - PHP unit testing
- [playwright-testing](../playwright-testing/SKILL.md) - E2E testing

---

## References

- [Vitest Documentation](https://vitest.dev/)
- [Testing Library](https://testing-library.com/)
- [Web Components Testing](https://open-wc.org/docs/testing/testing-package/)
