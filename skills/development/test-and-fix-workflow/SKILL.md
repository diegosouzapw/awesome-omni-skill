---
name: test-and-fix-workflow
description: Automated workflow for running tests and fixing failures systematically. Use when implementing the mandatory test workflow or fixing code quality issues. Keywords - testing, debugging, workflow, failures, systematic fixes.
compatibility: Requires yarn, embedded-components workspace
metadata:
  version: "2.0.0"
  author: jpmorgan-payments
  lastUpdated: "2025-12-24"
  priority: high
---

# Test and Fix Workflow

## Overview

Systematic workflow for running tests and fixing failures. This is the implementation of the mandatory code quality workflow.

## Workflow Steps

### 1. Run Test Suite

Execute all tests in the project:

```powershell
cd embedded-components
yarn test
```

This runs:
- Type checking (TypeScript)
- Format checking (Prettier)
- Linting (ESLint)
- Unit tests (Vitest)

### 2. Analyze Failures

Categorize failures by type:

**TypeScript Errors:**
- Type mismatches
- Missing type definitions
- Import errors
- Interface violations

**Formatting Errors:**
- Indentation issues
- Quote style violations
- Line break problems
- Trailing comma issues

**Linting Errors:**
- Unused variables
- Missing dependencies
- Code style violations
- Best practice violations

**Test Failures:**
- Assertion failures
- Mock issues
- Timeout errors
- Component rendering failures

### 3. Prioritize Fixes

Fix in this order:

1. **TypeScript errors** - Must be fixed first (blocking)
2. **Formatting errors** - Auto-fixable
3. **Linting errors** - Most auto-fixable
4. **Test failures** - Requires analysis

### 4. Fix Issues Systematically

**For TypeScript Errors:**

```powershell
# Check types
cd embedded-components
yarn typecheck

# Fix type issues in code
# Then re-check
yarn typecheck
```

**For Formatting Errors:**

```powershell
# Auto-fix all formatting
cd embedded-components
yarn format

# Verify
yarn format:check
```

**For Linting Errors:**

```powershell
# Auto-fix linting
cd embedded-components
yarn lint:fix

# Verify
yarn lint
```

**For Test Failures:**

```powershell
# Run specific test file
cd embedded-components
yarn test ComponentName.test.tsx

# Run in watch mode for debugging
yarn test:watch ComponentName.test.tsx

# Fix test or implementation
# Re-run tests
yarn test
```

### 5. Verify All Fixes

Re-run complete test suite:

```powershell
cd embedded-components
yarn test
```

### 6. Iterate Until Green

Repeat steps 3-5 until all tests pass.

## Common Failure Patterns

### TypeScript: Missing Type

```typescript
// ❌ Error: Parameter 'value' implicitly has an 'any' type
const formatValue = (value) => value.toString();

// ✅ Fix: Add type annotation
const formatValue = (value: number): string => value.toString();
```

### TypeScript: Type Mismatch

```typescript
// ❌ Error: Type 'string' is not assignable to type 'number'
const age: number = "25";

// ✅ Fix: Use correct type or convert
const age: number = 25;
// or
const age: number = parseInt("25", 10);
```

### Formatting: Inconsistent Quotes

```typescript
// ❌ Error: Replace `"` with `'`
const name = "John";

// ✅ Fix: Auto-fixed by yarn format
const name = 'John';
```

### Linting: Unused Variable

```typescript
// ❌ Error: 'unused' is assigned but never used
const unused = "value";
const used = "other";

// ✅ Fix: Remove or use it
const used = "other";
// or
const unused = "value";
console.log(unused);
```

### Linting: Missing useEffect Dependencies

```typescript
// ❌ Error: React Hook useEffect has a missing dependency: 'userId'
useEffect(() => {
  fetchUser(userId);
}, []);

// ✅ Fix: Add dependency
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

### Test: Assertion Failure

```typescript
// ❌ Failure: expected 5 to equal 4
test("adds numbers", () => {
  expect(add(2, 2)).toBe(5);
});

// ✅ Fix: Correct assertion or implementation
test("adds numbers", () => {
  expect(add(2, 2)).toBe(4);
});
```

### Test: Async Timeout

```typescript
// ❌ Error: Timeout - Async callback was not invoked within timeout
test("loads data", () => {
  renderComponent();
  expect(screen.getByText("Data")).toBeInTheDocument();
});

// ✅ Fix: Use waitFor for async operations
test("loads data", async () => {
  renderComponent();
  await waitFor(() => {
    expect(screen.getByText("Data")).toBeInTheDocument();
  });
});
```

### Test: Mock Not Reset

```typescript
// ❌ Error: Mock from previous test affecting current test
test("test 1", () => {
  server.use(http.get("/api/data", () => HttpResponse.json({ id: 1 })));
  // test assertions
});

test("test 2", () => {
  // Still using mock from test 1
  // test assertions
});

// ✅ Fix: Reset handlers in renderComponent or beforeEach
beforeEach(() => {
  server.resetHandlers();
});
```

## Debugging Strategies

### Isolate the Issue

```powershell
# Run single test file
cd embedded-components
yarn test ComponentName.test.tsx

# Run single test
yarn test ComponentName.test.tsx -t "specific test name"

# Run type check only
yarn typecheck

# Run linter only
yarn lint
```

### Use Watch Mode

```powershell
# Watch mode for rapid iteration
cd embedded-components
yarn test:watch ComponentName.test.tsx
```

### Verbose Output

```powershell
# More detailed output
cd embedded-components
yarn test --verbose
```

### Coverage Report

```powershell
# Generate coverage report
cd embedded-components
yarn test:coverage

# View report
start coverage/index.html
```

## Quick Fix Script

Create a PowerShell script to automate fixes:

```powershell
# fix-issues.ps1
cd embedded-components

Write-Host "Fixing formatting..." -ForegroundColor Yellow
yarn format

Write-Host "Fixing linting..." -ForegroundColor Yellow
yarn lint:fix

Write-Host "Running tests..." -ForegroundColor Yellow
yarn test

if ($LASTEXITCODE -eq 0) {
    Write-Host "All tests passed!" -ForegroundColor Green
} else {
    Write-Host "Some tests failed. Review output above." -ForegroundColor Red
}
```

## Best Practices

1. **Fix one issue at a time** - Don't batch unrelated fixes
2. **Verify each fix** - Run tests after each fix
3. **Commit working code** - Commit after all tests pass
4. **Don't skip failures** - Address all issues before committing
5. **Use auto-fix first** - Let tools fix what they can
6. **Document complex fixes** - Add comments for non-obvious fixes
7. **Update tests if needed** - Ensure tests reflect requirements

## Time-Saving Tips

```powershell
# Chain auto-fixes
cd embedded-components
yarn format; yarn lint:fix

# Check specific file quickly
yarn typecheck src/components/ComponentName.tsx

# Run related tests only
yarn test ComponentName

# Keep tests running in watch mode
yarn test:watch
```

## Integration with Git

```powershell
# Complete workflow before commit
cd embedded-components
yarn format; yarn lint:fix; yarn test

# If all pass, commit
cd ..
git add embedded-components
git commit -m "fix: resolve test failures"
git push
```

## Troubleshooting

### Cache Issues

```powershell
cd embedded-components
yarn cache clean
Remove-Item -Recurse -Force node_modules
yarn install
```

### Port Conflicts

```powershell
# Find process using port
netstat -ano | findstr :3000

# Kill process
taskkill /PID <PID> /F
```

### Memory Issues

```powershell
$env:NODE_OPTIONS="--max-old-space-size=4096"
cd embedded-components
yarn test
```

## References

- See `.github/copilot/skills/code-quality-workflow/` for quality gates
- See `.github/copilot/skills/component-testing/` for testing patterns
- See `.github/copilot/prompts/run-tests-and-fix.md` for detailed prompt
