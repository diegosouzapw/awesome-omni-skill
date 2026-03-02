---
name: Assertion Helper
description: Guide for writing effective test assertions with clear, meaningful error messages across different testing frameworks
---

# Assertion Helper

## Purpose

Help write better assertions that:
- Clearly express expected behavior
- Provide actionable error messages
- Are easy to understand when they fail
- Cover all edge cases
- Are framework-appropriate

## When to Use

Invoke this skill when:
- Writing test assertions
- Debugging failing tests
- Improving test readability
- Creating custom matchers
- Teaching testing best practices

## Instructions

### Step 1: Choose the Right Assertion

Select assertion based on what you're testing:
1. **Equality**: Value comparison
2. **Type**: Data type checking
3. **Truthiness**: Boolean conditions
4. **Existence**: Null/undefined checks
5. **Inclusion**: Array/object membership
6. **Exceptions**: Error throwing
7. **Async**: Promise resolution/rejection

### Step 2: Make It Specific

Use the most specific assertion available:
- `toBe(5)` > `toBeTruthy()`
- `toEqual([1,2,3])` > `toHaveLength(3)`
- `toThrow('Invalid')` > `toThrow()`

### Step 3: Add Context

Provide helpful messages for failures:
```typescript
expect(result, 'User should be authenticated after login').toBe(true);
```

### Step 4: Test Edge Cases

Include assertions for:
- Null/undefined
- Empty values
- Boundaries (min/max)
- Invalid inputs

## Assertion Patterns

### Jest/Vitest

#### Equality Assertions

```typescript
// Primitive values (===)
expect(result).toBe(5);
expect(result).toBe('hello');
expect(result).toBe(true);

// Objects/Arrays (deep equality)
expect(user).toEqual({
  name: 'John',
  age: 30
});

expect(array).toEqual([1, 2, 3]);

// Opposite
expect(result).not.toBe(null);
expect(array).not.toEqual([]);
```

#### Type Assertions

```typescript
// Null/Undefined
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();

// Type checks
expect(typeof result).toBe('string');
expect(Array.isArray(result)).toBe(true);
expect(result instanceof Error).toBe(true);
```

#### Number Assertions

```typescript
// Comparison
expect(age).toBeGreaterThan(18);
expect(age).toBeGreaterThanOrEqual(18);
expect(age).toBeLessThan(100);
expect(age).toBeLessThanOrEqual(100);

// Floating point
expect(result).toBeCloseTo(0.3, 2); // Within 2 decimal places

// NaN check
expect(result).toBeNaN();
```

#### String Assertions

```typescript
// Exact match
expect(text).toBe('Hello World');

// Contains
expect(text).toContain('Hello');

// Regex
expect(email).toMatch(/^[a-z]+@[a-z]+\.[a-z]+$/);

// Case insensitive
expect(text.toLowerCase()).toBe('hello');
```

#### Array/Object Assertions

```typescript
// Array membership
expect(array).toContain(item);
expect(array).toContainEqual({ id: 1 });

// Array length
expect(array).toHaveLength(3);

// Object properties
expect(obj).toHaveProperty('name');
expect(obj).toHaveProperty('address.city', 'NYC');

// Partial object match
expect(user).toMatchObject({
  name: 'John',
  // Other properties ignored
});

// Empty checks
expect(array).toEqual([]);
expect(obj).toEqual({});
```

#### Function/Error Assertions

```typescript
// Function called
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);
expect(mockFn).toHaveBeenCalledWith(arg1, arg2);
expect(mockFn).toHaveBeenLastCalledWith(arg1, arg2);

// Function throws
expect(() => throwError()).toThrow();
expect(() => throwError()).toThrow(Error);
expect(() => throwError()).toThrow('Error message');
expect(() => throwError()).toThrow(/error/i);

// Return value
expect(mockFn).toHaveReturnedWith(value);
```

#### Async Assertions

```typescript
// Promise resolves
await expect(promise).resolves.toBe(value);
await expect(promise).resolves.toEqual({ data: 'value' });

// Promise rejects
await expect(promise).rejects.toThrow();
await expect(promise).rejects.toThrow('Error message');

// Async function
it('should fetch user', async () => {
  const user = await fetchUser(1);
  expect(user).toEqual({ id: 1, name: 'John' });
});
```

---

### Python (pytest)

#### Basic Assertions

```python
# Equality
assert result == 5
assert result != 0

# Identity
assert result is None
assert result is not None

# Boolean
assert is_valid
assert not is_invalid

# Membership
assert item in collection
assert item not in collection

# Type
assert isinstance(result, str)
assert isinstance(result, (int, float))  # Multiple types
```

#### Numeric Assertions

```python
# Comparison
assert age > 18
assert age >= 18
assert age < 100
assert age <= 100

# Approximate (floating point)
import pytest
assert result == pytest.approx(0.3, abs=0.01)

# Math checks
import math
assert math.isnan(result)
assert math.isinf(result)
```

#### String Assertions

```python
# Exact match
assert text == "Hello World"

# Contains
assert "Hello" in text

# Regex
import re
assert re.match(r'^[a-z]+@[a-z]+\.[a-z]+$', email)

# Case insensitive
assert text.lower() == "hello"

# Start/End
assert text.startswith("Hello")
assert text.endswith("World")
```

#### Collection Assertions

```python
# List/Tuple
assert len(collection) == 3
assert collection == [1, 2, 3]
assert collection != []

# Set operations
assert set(collection) == {1, 2, 3}

# Dict
assert 'key' in dictionary
assert dictionary['key'] == 'value'
assert dictionary.get('key') == 'value'
```

#### Exception Assertions

```python
# Exception raised
with pytest.raises(ValueError):
    raise_error()

# Exception message
with pytest.raises(ValueError, match="Invalid input"):
    raise_error()

# Exception details
with pytest.raises(ValueError) as exc_info:
    raise_error()
assert str(exc_info.value) == "Invalid input"
assert exc_info.type == ValueError
```

#### Custom Assertions

```python
def assert_valid_user(user):
    """Custom assertion for user validation"""
    assert user is not None, "User should not be None"
    assert 'id' in user, "User should have an id"
    assert 'name' in user, "User should have a name"
    assert isinstance(user['id'], int), "User id should be int"
    assert len(user['name']) > 0, "User name should not be empty"

# Usage
assert_valid_user(result)
```

---

### Go (testing)

#### Basic Assertions

```go
import "testing"

// Equality
if result != expected {
    t.Errorf("Expected %v, got %v", expected, result)
}

// Boolean
if !isValid {
    t.Error("Expected isValid to be true")
}

// Nil check
if result == nil {
    t.Error("Expected result to not be nil")
}
```

#### Helper Functions

```go
func assertEqual(t *testing.T, expected, actual interface{}) {
    t.Helper()
    if expected != actual {
        t.Errorf("Expected %v, got %v", expected, actual)
    }
}

func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Errorf("Expected no error, got %v", err)
    }
}

func assertError(t *testing.T, err error, message string) {
    t.Helper()
    if err == nil {
        t.Error("Expected error, got nil")
        return
    }
    if !strings.Contains(err.Error(), message) {
        t.Errorf("Expected error containing %q, got %q", message, err.Error())
    }
}

// Usage
func TestFunction(t *testing.T) {
    result, err := Function()
    assertNoError(t, err)
    assertEqual(t, "expected", result)
}
```

---

## Best Practices

### DO:
✅ Use the most specific assertion
✅ Test one thing per assertion
✅ Add descriptive messages
✅ Test both positive and negative cases
✅ Use custom matchers for repeated patterns
✅ Group related assertions

### DON'T:
❌ Use generic assertions (`toBeTruthy()`)
❌ Multiple unrelated assertions in one test
❌ Assertion without context
❌ Test implementation details
❌ Ignore edge cases
❌ Rely on assertion order

## Assertion Patterns

### The Positive/Negative Pattern

```typescript
// Test both that it works AND doesn't work wrong
it('should validate email', () => {
  expect(validate('test@example.com')).toBe(true);
  expect(validate('invalid')).toBe(false);
});
```

### The Boundary Pattern

```typescript
// Test edges of valid ranges
it('should accept ages between 0 and 120', () => {
  expect(isValidAge(0)).toBe(true);
  expect(isValidAge(120)).toBe(true);
  expect(isValidAge(-1)).toBe(false);
  expect(isValidAge(121)).toBe(false);
});
```

### The Null/Undefined Pattern

```typescript
// Always test null/undefined cases
it('should handle null input', () => {
  expect(process(null)).toBeNull();
});

it('should handle undefined input', () => {
  expect(process(undefined)).toBeUndefined();
});
```

### The Error Message Pattern

```typescript
// Verify error messages for debugging
it('should throw descriptive error', () => {
  expect(() => divide(10, 0))
    .toThrow('Cannot divide by zero');
});
```

### The State Verification Pattern

```typescript
// Verify state changes
it('should update state correctly', () => {
  const obj = new Counter();
  expect(obj.count).toBe(0);

  obj.increment();
  expect(obj.count).toBe(1);

  obj.decrement();
  expect(obj.count).toBe(0);
});
```

## Custom Matchers

### Jest Custom Matcher

```typescript
expect.extend({
  toBeValidEmail(received: string) {
    const pass = /^[a-z]+@[a-z]+\.[a-z]+$/.test(received);
    return {
      pass,
      message: () =>
        pass
          ? `Expected ${received} not to be a valid email`
          : `Expected ${received} to be a valid email`
    };
  }
});

// Usage
expect('test@example.com').toBeValidEmail();
```

### Pytest Custom Assertion

```python
def assert_valid_email(email: str) -> None:
    """Assert that email is valid"""
    import re
    pattern = r'^[a-z]+@[a-z]+\.[a-z]+$'
    assert re.match(pattern, email), f"'{email}' is not a valid email"

# Usage
assert_valid_email('test@example.com')
```

## Debugging Failed Assertions

### Add Context

```typescript
// Bad
expect(result).toBe(5);

// Good
expect(result, 'Should calculate correct total after discount').toBe(5);
```

### Use Better Diff

```typescript
// For objects, toEqual gives better diff than toBe
expect(user).toEqual({
  id: 1,
  name: 'John',
  age: 30
});
```

### Break Down Complex Assertions

```typescript
// Bad - hard to debug
expect(result).toEqual({ id: 1, name: 'John', address: { city: 'NYC' } });

// Good - easier to see what failed
expect(result.id).toBe(1);
expect(result.name).toBe('John');
expect(result.address.city).toBe('NYC');
```

## Output Format

When providing assertion guidance:

```
## Assertions for ${TestCase}

**Testing**: ${whatIsBeingTested}

**Recommended Assertions**:
```${language}
${assertions}
```

**Why**:
- ${reason1}
- ${reason2}

**Edge Cases to Test**:
- ${edgeCase1}
- ${edgeCase2}
```

## Related Skills

- `test-pattern-library`: For complete test patterns
- `mock-generator`: For mocking dependencies
- `test-data-factory`: For test data generation
- `error-message-writer`: For better error messages
