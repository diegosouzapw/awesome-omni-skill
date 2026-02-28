---
name: checking-quality
description: Ensures code quality through comprehensive checks including TDD practices, lint/test/build validation, and prevention of common mistakes. Use after completing implementations, fixing bugs, refactoring, or before committing code.
---

# Checking Quality

Comprehensive code quality assurance through systematic validation and common mistake prevention.

## Table of Contents

1. [Quick Start Checklist](#quick-start-checklist)
2. [Quality Gates](#quality-gates)
3. [Test-First Development](#test-first-development)
4. [Common Mistakes](#common-mistakes)
5. [Workflows](#workflows)
6. [Error Recovery](#error-recovery)

## Quick Start Checklist

Before committing any code changes, verify all quality gates pass:

```bash
# Frontend
cd frontend
npm run lint && npm run test && npm run build

# Backend
cd backend
npm run lint && npm run test && npm run build
```

**Commit only when all checks pass:**
- [ ] ✅ Lint: 0 errors, 0 warnings
- [ ] ✅ Tests: All passed
- [ ] ✅ Build: Success

## Quality Gates

### When to Run Quality Checks

Execute complete quality validation after:
- [ ] New feature implementation
- [ ] Bug fixes
- [ ] Refactoring
- [ ] Responding to feedback
- [ ] Before every commit (mandatory)

### Frontend Validation

```bash
cd frontend
npm run lint      # Linting
npm run test      # All tests
npm run build     # Build verification
```

### Backend Validation

```bash
cd backend
npm run lint      # Linting
npm run test      # All tests
npm run build     # Build verification
```

## Test-First Development

**Critical Rule: Always modify tests before implementation.**

### Correct Modification Sequence

When changing implementation:

1. **Modify tests first**
2. Verify tests fail (proving test validity)
3. Update implementation
4. Verify tests pass

### Red-Green-Refactor Cycle

For new features and bug fixes:

1. **RED**: Write failing test
2. **GREEN**: Minimal code to pass test
3. **REFACTOR**: Improve while tests stay green

See [TDD Practices](tdd-practices.md) for detailed guidelines and examples.

## Common Mistakes

### Most Frequent Errors

1. **Forgetting test updates** when modifying implementation
2. **Missing related file updates** after interface changes
3. **Overlooking lint warnings** before committing
4. **Skipping quality checks** when rushed

### Detailed Patterns

For comprehensive coverage of common mistakes with examples, see [Common Mistakes Guide](common-mistakes.md).

### Quick Prevention

- [ ] Change interface → Update all call sites
- [ ] Modify types → Update all usages
- [ ] Add async → Add await to callers
- [ ] Remove unused imports/variables
- [ ] Convert `let` to `const` when not reassigned

## Workflows

### New Feature Implementation

```
1. Write failing test (RED)
2. Run test to confirm failure
3. Write minimal implementation (GREEN)
4. Run test to confirm pass
5. Refactor while keeping tests green
6. Run quality checks (lint/test/build)
7. Commit when all checks pass
```

### Bug Fixing

```
1. Write test reproducing the bug
2. Confirm test fails
3. Fix implementation
4. Confirm test passes
5. Check for related file updates
6. Run quality checks (lint/test/build)
7. Commit when all checks pass
```

### Responding to Feedback

```
1. Understand the feedback
2. Modify tests first (critical!)
3. Update implementation
4. Verify related files
5. Run quality checks (lint/test/build)
6. Commit when all checks pass
```

## Error Recovery

### Lint Errors

1. Read error message carefully
2. **Check tests first** (may have same issue)
3. Fix implementation
4. Re-run `npm run lint`

### Test Failures

1. Identify failing test
2. Determine if implementation or test expectations changed
3. Apply appropriate fix
4. Re-run `npm run test`

### Build Errors

1. Check type errors
2. Verify dependencies
3. Fix issues
4. Re-run `npm run build`

## Best Practices

### Small, Frequent Commits

- Commit after each completed feature
- Break large changes into incremental steps
- Keep each commit focused and self-contained

### Automation

- Configure pre-commit hooks for automatic validation
- Use CI/CD pipelines to enforce quality gates
- Automate repetitive quality checks

### Self-Review

- Re-read code changes before committing
- List modified files and their dependencies
- Verify test coverage for changes

## Integration with Development

Quality checking should be seamless:

1. **During development**: Run specific tests frequently
2. **Before breaks**: Run full test suite
3. **Before committing**: Run complete quality validation
4. **After merging**: Verify integration tests pass

This systematic approach prevents quality issues from reaching the codebase while maintaining development velocity.
