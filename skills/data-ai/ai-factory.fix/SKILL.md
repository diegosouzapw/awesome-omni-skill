---
name: ai-factory.fix
description: Fix a specific bug or problem in the codebase. Analyzes code to find and fix issues without creating plans. Use when user reports a bug, error, or something not working. Always suggests test coverage and adds logging.
argument-hint: <bug description or error message>
allowed-tools: Read Write Edit Glob Grep Bash AskUserQuestion
disable-model-invocation: false
---

# Fix - Quick Bug Fix Workflow

Fix a specific bug or problem by analyzing the codebase directly. No plans, no reports.

## Workflow

### Step 0: Load Project Context & Past Experience

**Read `.ai-factory/DESCRIPTION.md`** if it exists to understand:
- Tech stack (language, framework, database)
- Project architecture
- Coding conventions

**Read all patches from `.ai-factory/patches/`** if the directory exists:
- Use `Glob` to find all `*.md` files in `.ai-factory/patches/`
- Read each patch file to learn from past fixes
- Pay attention to recurring patterns, root causes, and solutions
- If the current problem resembles a past patch — apply the same approach or avoid the same mistakes
- This is your accumulated experience. Use it.

### Step 1: Understand the Problem

From `$ARGUMENTS`, identify:
- Error message or unexpected behavior
- Where it occurs (file, function, endpoint)
- Steps to reproduce (if provided)

If unclear, ask:
```
To fix this effectively, I need more context:

1. What is the expected behavior?
2. What actually happens?
3. Can you share the error message/stack trace?
4. When did this start happening?
```

### Step 2: Investigate the Codebase

**Search for the problem:**
- Find relevant files using Glob/Grep
- Read the code around the issue
- Trace the data flow
- Check for similar patterns elsewhere

**Look for:**
- The root cause (not just symptoms)
- Related code that might be affected
- Existing error handling

### Step 3: Implement the Fix

**Apply the fix with logging:**

```typescript
// ✅ REQUIRED: Add logging around the fix
console.log('[FIX] Processing user input', { userId, input });

try {
  // The actual fix
  const result = fixedLogic(input);
  console.log('[FIX] Success', { userId, result });
  return result;
} catch (error) {
  console.error('[FIX] Error in fixedLogic', {
    userId,
    input,
    error: error.message,
    stack: error.stack
  });
  throw error;
}
```

**Logging is MANDATORY because:**
- User needs to verify the fix works
- If it doesn't work, logs help debug further
- Feedback loop: user provides logs → we iterate

### Step 4: Verify the Fix

- Check the code compiles/runs
- Verify the logic is correct
- Ensure no regressions introduced

### Step 5: Suggest Test Coverage

**ALWAYS suggest covering this case with a test:**

```
## Fix Applied ✅

The issue was: [brief explanation]
Fixed by: [what was changed]

### Logging Added
The fix includes logging with prefix `[FIX]`.
Please test and share any logs if issues persist.

### Recommended: Add a Test

This bug should be covered by a test to prevent regression:

\`\`\`typescript
describe('functionName', () => {
  it('should handle [the edge case that caused the bug]', () => {
    // Arrange
    const input = /* the problematic input */;

    // Act
    const result = functionName(input);

    // Assert
    expect(result).toBe(/* expected */);
  });
});
\`\`\`

Would you like me to create this test?
- [ ] Yes, create the test
- [ ] No, skip for now
```

## Logging Requirements

**All fixes MUST include logging:**

1. **Log prefix**: Use `[FIX]` or `[FIX:<issue-id>]` for easy filtering
2. **Log inputs**: What data was being processed
3. **Log success**: Confirm the fix worked
4. **Log errors**: Full context if something fails
5. **Configurable**: Use LOG_LEVEL if available

```typescript
// Pattern for fixes
const LOG_FIX = process.env.LOG_LEVEL === 'debug' || process.env.DEBUG_FIX;

function fixedFunction(input) {
  if (LOG_FIX) console.log('[FIX] Input:', input);

  // ... fix logic ...

  if (LOG_FIX) console.log('[FIX] Output:', result);
  return result;
}
```

## Examples

### Example 1: Null Reference Error

**User:** `/fix TypeError: Cannot read property 'name' of undefined in UserProfile`

**Actions:**
1. Search for UserProfile component/function
2. Find where `.name` is accessed
3. Add null check with logging
4. Suggest test for null user case

### Example 2: API Returns Wrong Data

**User:** `/fix /api/orders returns empty array for authenticated users`

**Actions:**
1. Find orders API endpoint
2. Trace the query logic
3. Find the bug (e.g., wrong filter)
4. Fix with logging
5. Suggest integration test

### Example 3: Form Validation Not Working

**User:** `/fix email validation accepts invalid emails`

**Actions:**
1. Find email validation logic
2. Check regex or validation library usage
3. Fix the validation
4. Add logging for validation failures
5. Suggest unit test with edge cases

## Important Rules

1. **NO plans** - This is a direct fix, not planned work
2. **NO reports** - Don't create summary documents
3. **ALWAYS log** - Every fix must have logging for feedback
4. **ALWAYS suggest tests** - Help prevent regressions
5. **Root cause** - Fix the actual problem, not symptoms
6. **Minimal changes** - Don't refactor unrelated code
7. **One fix at a time** - Don't scope creep

## After Fixing

```
## Fix Applied ✅

**Issue:** [what was broken]
**Cause:** [why it was broken]
**Fix:** [what was changed]

**Files modified:**
- path/to/file.ts (line X)

**Logging added:** Yes, prefix `[FIX]`
**Test suggested:** Yes

Please test the fix and share logs if any issues.

To add the suggested test:
- [ ] Yes, create test
- [ ] No, skip
```

### Step 6: Create Self-Improvement Patch

**ALWAYS create a patch after every fix.** This builds a knowledge base for future fixes.

**Create the patch:**

1. Create directory if it doesn't exist:
   ```bash
   mkdir -p .ai-factory/patches
   ```

2. Create a patch file with the current timestamp as filename.
   **Format:** `YYYY-MM-DD-HH.mm.md` (e.g., `2026-02-07-14.30.md`)

3. Use this template:

```markdown
# [Brief title describing the fix]

**Date:** YYYY-MM-DD HH:mm
**Files:** list of modified files
**Severity:** low | medium | high | critical

## Problem

What was broken. How it manifested (error message, wrong behavior).
Be specific — include the actual error or symptom.

## Root Cause

WHY the problem occurred. This is the most valuable part.
Not "what was wrong" but "why it was wrong":
- Logic error? Why was the logic incorrect?
- Missing check? Why was it missing?
- Wrong assumption? What was assumed?
- Race condition? What sequence caused it?

## Solution

How the fix was implemented. Key code changes and reasoning.
Include the approach, not just "changed line X".

## Prevention

How to prevent this class of problems in the future:
- What pattern/practice should be followed?
- What should be checked during code review?
- What test would catch this?

## Tags

Space-separated tags for categorization, e.g.:
`#null-check` `#async` `#validation` `#typescript` `#api` `#database`
```

**Example patch:**

```markdown
# Null reference in UserProfile when user has no avatar

**Date:** 2026-02-07 14:30
**Files:** src/components/UserProfile.tsx
**Severity:** medium

## Problem

TypeError: Cannot read property 'url' of undefined when rendering
UserProfile for users without an uploaded avatar.

## Root Cause

The `user.avatar` field is optional in the database schema but the
component accessed `user.avatar.url` without a null check. This was
introduced in commit abc123 when avatar display was added — the
developer tested only with users that had avatars.

## Solution

Added optional chaining: `user.avatar?.url` with a fallback to a
default avatar URL. Also added a null check in the Avatar sub-component.

## Prevention

- Always check if database fields marked as `nullable` / `optional`
  are handled with null checks in the UI layer
- Add test cases for "empty state" — user with minimal data
- Consider a lint rule for accessing nested optional properties

## Tags

`#null-check` `#react` `#optional-field` `#typescript`
```

**This is NOT optional.** Every fix generates a patch. The patch is your learning.

---

**DO NOT:**
- ❌ Create PLAN.md or any plan files
- ❌ Generate reports or summaries (patches are NOT reports — they are learning artifacts)
- ❌ Refactor unrelated code
- ❌ Add features while fixing
- ❌ Skip logging
- ❌ Skip test suggestion
- ❌ Skip patch creation
