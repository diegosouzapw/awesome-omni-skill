---
name: testing
description: Testing standards and best practices
---

# Testing Standards

## Philosophy

Tests are **verification infrastructure**, not checkboxes. They protect against regressions, document expected behavior, and surface real issues for human evaluation.

**Comprehensive, not exhaustive.** Cover meaningful scenarios without combinatorial explosion.

---

## Pre-Flight Checklist

Before writing any test, answer:

1. What is the **contract**? (inputs → outputs, side effects, errors)
2. What are the **boundaries**? (empty, nil, zero, max, negative)
3. What can go **wrong**? (every error return path)
4. What **state** does it depend on? (filesystem, env vars, time)
5. Is there **concurrency**? (goroutines, shared state)

**Write tests for #2 and #3 FIRST.** Edge cases and errors catch real bugs.

---

## Go Test Essentials

```go
func TestFoo(t *testing.T) {
    t.Parallel()           // Unless shared state
    dir := t.TempDir()     // Auto-cleanup, not os.MkdirTemp
    t.Setenv("KEY", "val") // Auto-restore, prevents t.Parallel() misuse
}

func helper(t *testing.T) {
    t.Helper()  // REQUIRED - fixes line numbers in failures
}
```

**Error assertions:**
```go
if !errors.Is(err, ErrNotFound) { ... }      // Sentinel errors
if !errors.As(err, &pathErr) { ... }         // Wrapped errors
if !strings.Contains(err.Error(), "x") {...} // Message matching
```

---

## Table-Driven Tests

```go
func TestCharge(t *testing.T) {
    tests := []struct {
        name    string
        card    Card   // GIVEN
        amount  int64  // GIVEN
        wantErr error  // THEN
    }{
        {"expired card", Card{Expiry: past}, 100, ErrExpired},
        {"valid card", Card{Expiry: future}, 100, nil},
        {"negative amount", Card{}, -1, ErrInvalidAmount},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := svc.Charge(tt.card, tt.amount)  // WHEN
            if !errors.Is(err, tt.wantErr) {
                t.Errorf("got %v, want %v", err, tt.wantErr)
            }
        })
    }
}
```

---

## Coverage Requirements

| Path Type | Required | Purpose |
|-----------|----------|---------|
| Happy path | Always | Confirms expected behavior |
| Error paths | Always | Every error return tested |
| Edge cases | When meaningful | nil, empty, zero, boundaries |

**If a function returns an error, test that it returns the right error.**

---

## Real-World Test Data

Tests must reflect production reality:

| Aspect | Bad | Good |
|--------|-----|------|
| Data | `User{Name: "test"}` | `User{Name: "María García-López"}` |
| Scale | Single item | Batch of 100+ |
| State | Empty/zero only | Realistic pre-existing data |

**Mocks are a last resort.** Prefer interfaces with test implementations or recorded fixtures.

---

## Test Integrity Rules

```
❌ FORBIDDEN: Changing assertions to match broken behavior
❌ FORBIDDEN: Deleting tests that "don't work anymore"
❌ FORBIDDEN: Adding t.Skip() without justification
✅ REQUIRED: Escalate unexpected failures for evaluation
```

Tests may fix implementation **only** when:
1. Test reveals a **legitimate bug** (doesn't match documented contract)
2. Fix is **isolated** to the defect (no scope creep)

**Escalate when:** Multiple tests fail, architectural issue suspected, or fix requires API changes.

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---------|---------|-----|
| Testing unexported details | Brittle | Test exported behavior |
| One giant test function | Hard to diagnose | Table-driven subtests |
| Shared mutable state | Race conditions | Fresh setup per test |
| `time.Sleep` | Slow, flaky | Channels, contexts, polling |
| `os.Chdir` in tests | Breaks parallel | Pass paths explicitly |
| Missing `t.Helper()` | Wrong line numbers | Add to all helpers |
| `os.MkdirTemp` without cleanup | Leaks | Use `t.TempDir()` |

---

## Organization

```
pkg/service/
├── service.go
├── service_test.go       # Unit tests
└── service_int_test.go   # //go:build integration
internal/testutil/        # Shared helpers
```

**Always run:** `go test -race ./...`
