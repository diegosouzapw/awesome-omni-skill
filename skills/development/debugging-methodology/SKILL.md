---
name: debugging-methodology
description: Systematic debugging methodology following reproduce → locate → hypothesize → verify → fix → regression workflow. Use when debugging issues, investigating bugs, troubleshooting problems, or when asked about debugging process, problem-solving, or issue investigation.
---

# Debugging Methodology

Systematic approach to debugging: Reproduce → Locate → Hypothesize → Verify → Fix → Regression

## Quick Start

### Core Debugging Flow

```
1. Reproduce  - Confirm problem can be reliably reproduced
2. Locate     - Use binary search to narrow down scope
3. Hypothesize - Form hypothesis based on evidence
4. Verify     - Add logs/breakpoints to verify
5. Fix        - Minimal modification
6. Regression - Confirm no new issues
```

### Golden Rules

**MUST DO**:
- Reproduce before fixing
- Document debugging process

**NEVER DO**:
- Blindly modify code
- Expand fix scope
- Ignore root cause

## Step 1: Reproduce

### Goal
Confirm the problem can be reliably reproduced.

### Questions to Answer

1. **What is the problem?**
   - What is the expected behavior?
   - What is the actual behavior?
   - What is the difference?

2. **When does it occur?**
   - Always?
   - Under specific conditions?
   - Intermittently?

3. **How to reproduce?**
   - Exact steps to reproduce
   - Required environment/data
   - Success rate (100%? 50%?)

### Reproduction Template

```markdown
## Bug Report

### Expected Behavior
User should be redirected to dashboard after login

### Actual Behavior
User stays on login page, no error message

### Steps to Reproduce
1. Navigate to /login
2. Enter email: test@example.com
3. Enter password: password123
4. Click "Login" button
5. Observe: Page doesn't redirect

### Environment
- Browser: Chrome 120
- OS: Windows 11
- User role: Regular user
- Data: Fresh database

### Reproduction Rate
100% (happens every time)
```

### If Cannot Reproduce

- Gather more information from reporter
- Check environment differences
- Review recent changes
- Check logs for patterns

## Step 2: Locate

### Goal
Narrow down the problem scope using binary search.

### Binary Search Strategy

```
1. Identify the boundaries (start and end of flow)
2. Check the middle point
3. Determine which half contains the problem
4. Repeat until problem is isolated
```

### Example: Login Flow

```
Full Flow:
[UI Click] → [Form Validation] → [API Call] → [Auth Service] → [Database] → [Response] → [Redirect]

Binary Search:
1. Check: Does API call succeed? → YES
2. Check: Does auth service return token? → YES
3. Check: Does response reach frontend? → YES
4. Check: Does redirect function get called? → NO ← Problem found!

Narrowed to: Redirect logic in frontend
```

### Locating Tools

**Console Logs**:
```typescript
console.log('1. Before API call')
console.log('2. API response:', response)
console.log('3. Before redirect')
console.log('4. After redirect')
```

**Debugger Breakpoints**:
```typescript
debugger; // Execution pauses here
```

**Network Tab**:
- Check API requests/responses
- Verify status codes
- Inspect headers and payloads

**React DevTools**:
- Check component state
- Verify props
- Inspect component tree

## Step 3: Hypothesize

### Goal
Form evidence-based hypothesis about the root cause.

### Hypothesis Framework

```markdown
## Hypothesis

### Observation
Redirect function is not being called after successful login

### Possible Causes
1. Conditional logic preventing redirect
2. Error thrown before redirect
3. Async timing issue
4. State not updated correctly

### Most Likely Cause
Based on code review, there's a conditional check for `user.isVerified`
that might be failing

### Evidence
- User object exists
- Token is valid
- But `user.isVerified` is undefined (not true/false)
```

### Common Root Causes

**Logic Errors**:
- Incorrect conditional logic
- Off-by-one errors
- Wrong operator (== vs ===)

**State Issues**:
- Stale state
- Race conditions
- Missing state updates

**Async Problems**:
- Not awaiting promises
- Callback hell
- Race conditions

**Type Issues**:
- Undefined vs null
- Type coercion
- Missing properties

## Step 4: Verify

### Goal
Confirm hypothesis through testing.

### Verification Methods

**Add Logging**:
```typescript
console.log('user.isVerified:', user.isVerified)
console.log('typeof user.isVerified:', typeof user.isVerified)
console.log('user.isVerified === true:', user.isVerified === true)
```

**Add Breakpoints**:
```typescript
if (user.isVerified) {
  debugger; // Does this execute?
  navigate('/dashboard')
}
```

**Write Test**:
```typescript
test('should redirect when user is verified', () => {
  const user = { isVerified: undefined }
  const result = shouldRedirect(user)
  expect(result).toBe(false) // Confirms hypothesis
})
```

### Hypothesis Validation

✅ **Hypothesis Confirmed**:
- Proceed to fix

❌ **Hypothesis Rejected**:
- Form new hypothesis
- Repeat verification

## Step 5: Fix

### Goal
Apply minimal fix that addresses root cause.

### Minimal Fix Principle

```typescript
// ❌ Bad: Over-fixing
function handleLogin(user) {
  // Added unnecessary validation
  if (!user) return
  if (!user.email) return
  if (!user.id) return
  
  // Fixed the actual issue
  if (user.isVerified === true) {
    navigate('/dashboard')
  }
  
  // Added unnecessary features
  trackLoginEvent(user)
  updateLastLogin(user)
}

// ✅ Good: Minimal fix
function handleLogin(user) {
  // Only fix the identified issue
  if (user.isVerified === true) {  // Changed from: if (user.isVerified)
    navigate('/dashboard')
  }
}
```

### Fix Documentation

```markdown
## Fix Applied

### Root Cause
`user.isVerified` can be undefined, but code only checked truthiness

### Solution
Changed condition from `if (user.isVerified)` to `if (user.isVerified === true)`

### Why This Works
Explicitly checks for true value, handles undefined case correctly

### Files Changed
- src/auth/LoginHandler.ts (line 45)

### Tests Added
- Should not redirect when isVerified is undefined
- Should redirect when isVerified is true
```

## Step 6: Regression

### Goal
Ensure fix doesn't introduce new problems.

### Regression Checklist

- [ ] Original issue is fixed
- [ ] No new errors in console
- [ ] All existing tests still pass
- [ ] Manual testing of related features
- [ ] Performance not degraded

### Regression Testing

```bash
# Run full test suite
npm run test

# Run specific feature tests
npm run test -- auth

# Manual testing
# 1. Test original bug scenario
# 2. Test edge cases
# 3. Test related features
```

### If Regression Found

1. **Document the regression**
2. **Revert the fix**
3. **Form new hypothesis**
4. **Try alternative fix**

## Advanced Debugging Techniques

### Time-Travel Debugging

Use Redux DevTools or similar:
- Replay actions
- Inspect state at any point
- Identify when state became incorrect

### Performance Debugging

```typescript
// Measure execution time
console.time('expensive-operation')
expensiveOperation()
console.timeEnd('expensive-operation')

// Profile with React DevTools
// Use Profiler tab to identify slow renders
```

### Memory Leak Debugging

```typescript
// Check for memory leaks
// 1. Take heap snapshot before
// 2. Perform action
// 3. Take heap snapshot after
// 4. Compare snapshots
```

### Network Debugging

```bash
# Use browser DevTools Network tab
# Check:
- Request/response timing
- Status codes
- Headers
- Payloads
- CORS issues
```

## Common Debugging Patterns

### Pattern 1: Intermittent Bugs

**Characteristics**: Bug doesn't always occur

**Approach**:
1. Identify reproduction rate
2. Look for race conditions
3. Check async operations
4. Add extensive logging
5. Test with different timing

### Pattern 2: Environment-Specific Bugs

**Characteristics**: Works in dev, fails in production

**Approach**:
1. Compare environment configurations
2. Check environment variables
3. Verify build process
4. Test with production data
5. Check browser/OS differences

### Pattern 3: Integration Bugs

**Characteristics**: Components work individually, fail together

**Approach**:
1. Test integration points
2. Verify data contracts
3. Check timing/ordering
4. Add integration tests
5. Use mocks to isolate

## Debugging Tools

### Browser DevTools

**Console**:
- `console.log()` - Basic logging
- `console.table()` - Tabular data
- `console.trace()` - Stack trace
- `console.time()` - Performance timing

**Debugger**:
- Breakpoints
- Step through code
- Watch expressions
- Call stack inspection

**Network**:
- Request/response inspection
- Timing analysis
- Throttling simulation

**React DevTools**:
- Component tree
- Props/state inspection
- Profiler

### IDE Debugging

**VS Code**:
- Breakpoints
- Variable inspection
- Debug console
- Call stack

**Configuration**:
```json
{
  "type": "chrome",
  "request": "launch",
  "name": "Debug Chrome",
  "url": "http://localhost:3000",
  "webRoot": "${workspaceFolder}/src"
}
```

## Debugging Checklist

### Before Starting
- [ ] Can you reproduce the issue?
- [ ] Do you have reproduction steps?
- [ ] Have you checked recent changes?
- [ ] Have you reviewed error logs?

### During Debugging
- [ ] Are you using binary search?
- [ ] Are you documenting findings?
- [ ] Are you testing hypotheses?
- [ ] Are you avoiding scope creep?

### Before Committing Fix
- [ ] Is the root cause identified?
- [ ] Is the fix minimal?
- [ ] Are tests added?
- [ ] Is regression testing done?
- [ ] Is the fix documented?

## Best Practices

1. **Reproduce First** - Never fix what you can't reproduce
2. **Binary Search** - Narrow scope systematically
3. **Evidence-Based** - Form hypotheses based on evidence
4. **Minimal Fix** - Fix only what's broken
5. **Document Process** - Help future debugging
6. **Test Thoroughly** - Prevent regressions
7. **Learn from Bugs** - Improve code to prevent similar issues

## Anti-Patterns

❌ **Shotgun Debugging**
- Randomly changing code hoping it fixes the issue
- No systematic approach
- Often makes things worse

❌ **Cargo Cult Debugging**
- Copying solutions without understanding
- "It worked before, so I'll try it again"
- Doesn't address root cause

❌ **Debugging by Coincidence**
- "It works now, but I don't know why"
- Didn't identify root cause
- Bug will likely return

❌ **Scope Creep**
- Fixing unrelated issues while debugging
- Refactoring during bug fix
- Makes it hard to verify the fix

## Related Documentation

- **Testing**: See testing steering for test-driven debugging
- **Code Review**: See code-quality power for systematic review
- **Workflow**: See workflow steering for debugging phase integration

---

*Remember: Systematic debugging is faster than random changes. Follow the process, document your findings, and learn from each bug.*
