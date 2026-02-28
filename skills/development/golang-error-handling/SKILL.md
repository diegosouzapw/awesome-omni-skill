---
name: golang-error-handling
description: Go error handling review. Use when checking error wrapping, context propagation, or error checking patterns. Ensures proper error chains, context usage, and nil checks.
license: MIT
metadata:
  author: saifoelloh
  version: "2.0.0"
  parent_skill: golang-best-practices
  sources:
    - "Learning Go: An Idiomatic Approach (Jon Bodner)"
    - "Concurrency in Go (Katherine Cox-Buday)"
  last_updated: "2026-01-22"
---

# Golang Error Handling

Expert-level error handling review for Go applications. Ensures proper error wrapping, context propagation, and error checking that preserves error chains and enables proper timeout/cancellation handling.

## When to Apply

Use this skill when:
- Reviewing error propagation patterns
- Checking context usage and cancellation
- Auditing error wrapping with %w vs %v
- Investigating timeout or cancellation issues
- Verifying nil checks for interfaces
- Ensuring errors.Is/As usage

## Rule Categories by Priority

| Priority | Count | Focus |
|----------|-------|-------|
| Critical | 3 | Prevents error chain loss, context leaks |
| High | 3 | Correctness and reliability |
| Medium | 1 | Code quality |

## Rules Covered (7 total)

### Critical Issues (3)

- `critical-error-wrapping` - Use %w for error wrapping, not %v
- `critical-error-shadow` - Don't shadow err variable in nested scopes
- `critical-context-leak` - Always defer cancel() after context creation

### High-Impact Patterns (3)

- `high-context-propagation` - Propagate context through call chain
- `high-error-is-as` - Use errors.Is/As instead of ==
- `high-interface-nil` - Check interface nil correctly

### Medium Improvements (1)

- `medium-sentinel-error-usage` - Use sentinel errors for stable categories

## How to Use

### For Error Handling Review

1. Scan code for error handling patterns
2. Check against rules in priority order (Critical first)
3. Verify error wrapping preserves error chains
4. Ensure context propagation for timeouts/cancellation
5. Check for proper nil interface handling

### Common Patterns

**Error chain review**:
```
Check this code for proper error wrapping
```

**Context propagation audit**:
```
Verify context is propagated correctly
```

**Error checking patterns**:
```
Review error handling for wrapped errors
```

## Trigger Phrases

This skill activates when you say:
- "Review error handling"
- "Check error wrapping"
- "Verify context propagation"
- "Check for context leaks"
- "Review error chains"
- "Audit error patterns"

## Output Format

```
## Critical Error Handling Issues: X

### [Rule Name] (Line Y)
**Issue**: Brief description
**Impact**: Lost error chain / Context leak / Wrong error check
**Fix**: Suggested correction
**Example**:
```go
// Corrected code
```

## Related Skills

- [golang-concurrency-safety](../concurrency-safety/SKILL.md) - For goroutine context patterns
- [golang-clean-architecture](../clean-architecture/SKILL.md) - For error handling across layers

## Philosophy

Based on Go best practices:

- **Error chains are valuable** - Use %w to preserve error context
- **Context is essential** - Always propagate for cancellation/timeout
- **Errors are values** - Use errors.Is/As for wrapped error checking
- **Nil interfaces are tricky** - Check correctly to avoid bugs

## Notes

- Focus on error handling correctness
- All examples show proper error wrapping patterns
- Context propagation is critical for production systems
- Related to concurrency but focused on error semantics
