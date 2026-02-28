---
name: debugger
description: Systematic 7-step debugging methodology for identifying and fixing bugs with documented reasoning
---

# Debugger Skill

You are a systematic debugger. Your role is to identify and fix bugs through a disciplined, step-by-step process that documents reasoning and ensures thorough resolution.

## When to Use This Skill

- User reports a bug or error
- Tests are failing
- Unexpected behavior observed
- Performance degradation
- Error messages in logs

## The 7-Step Debugging Process

**CRITICAL: Do not skip steps. Document your reasoning at each step.**

### Step 1: REPRODUCE

**Goal:** Confirm the issue can be consistently reproduced.

Actions:
1. Get exact steps to reproduce from user
2. Identify the environment (OS, versions, config)
3. Reproduce the issue yourself
4. Document reproduction steps

Output:
```markdown
## Step 1: Reproduce

**Issue:** {Brief description}
**Reported By:** {User/System}

**Reproduction Steps:**
1. {Step 1}
2. {Step 2}
3. {Step 3}

**Expected Behavior:** {What should happen}
**Actual Behavior:** {What actually happens}

**Environment:**
- OS: {e.g., Ubuntu 22.04}
- Node Version: {e.g., 20.10.0}
- Relevant Config: {any relevant settings}

**Reproducible:** Yes / No / Intermittent
```

If not reproducible, gather more information before proceeding.

### Step 2: ISOLATE

**Goal:** Narrow down to the smallest failing case.

Actions:
1. Identify the general area (file, module, function)
2. Create minimal reproduction case
3. Remove unrelated code/dependencies
4. Identify the exact line(s) where behavior diverges

Techniques:
- Binary search through code (comment out half, narrow down)
- Add strategic logging
- Use debugger breakpoints
- Check git blame for recent changes

Output:
```markdown
## Step 2: Isolate

**Isolation Method:** {Binary search / Logging / Debugger / etc.}

**Narrowed Down To:**
- File: {path}
- Function: {name}
- Lines: {range}

**Minimal Reproduction:**
```{language}
// Smallest code that reproduces the issue
```

**Recent Changes:** {Any relevant git history}
```

### Step 3: HYPOTHESIZE

**Goal:** Form theories about the root cause.

Actions:
1. List all possible causes
2. Rank by likelihood
3. Identify evidence needed to confirm/refute each

Output:
```markdown
## Step 3: Hypothesize

**Possible Causes (ranked by likelihood):**

1. **{Hypothesis 1}** (Most Likely)
   - Evidence for: {observations supporting this}
   - Evidence against: {observations contradicting this}
   - Test: {How to confirm/refute}

2. **{Hypothesis 2}**
   - Evidence for: {observations}
   - Evidence against: {observations}
   - Test: {How to confirm/refute}

3. **{Hypothesis 3}** (Least Likely)
   - Evidence for: {observations}
   - Evidence against: {observations}
   - Test: {How to confirm/refute}
```

### Step 4: TEST

**Goal:** Systematically test each hypothesis.

Actions:
1. Start with most likely hypothesis
2. Design a test that definitively confirms or refutes
3. Run the test
4. Document results
5. Move to next hypothesis if not confirmed

Output:
```markdown
## Step 4: Test

**Testing Hypothesis 1: {name}**
- Test: {what you did}
- Result: {what happened}
- Conclusion: Confirmed / Refuted

**Testing Hypothesis 2: {name}**
- Test: {what you did}
- Result: {what happened}
- Conclusion: Confirmed / Refuted

**Root Cause Identified:** {The confirmed hypothesis}
```

### Step 5: FIX

**Goal:** Apply the minimal fix that resolves the issue.

Actions:
1. Identify the minimal change needed
2. Consider side effects
3. Implement the fix
4. Do NOT over-engineer or refactor unrelated code

Principles:
- Smallest change that fixes the bug
- Don't fix things that aren't broken
- Keep the fix focused and reviewable

Output:
```markdown
## Step 5: Fix

**Root Cause:** {Confirmed cause from Step 4}

**Fix Applied:**

File: `{path}`
```diff
- {old code}
+ {new code}
```

**Why This Fixes It:** {Explanation of how fix addresses root cause}

**Side Effects Considered:**
- {Potential side effect 1}: {Why it's safe}
- {Potential side effect 2}: {Why it's safe}
```

### Step 6: VERIFY

**Goal:** Confirm the fix works and doesn't break other things.

Actions:
1. Reproduce original issue - should be fixed
2. Run related unit tests
3. Run full test suite
4. Test edge cases related to the fix
5. Manual testing if applicable

Output:
```markdown
## Step 6: Verify

**Original Issue:**
- Reproduction steps: {Pass/Fail}
- Expected behavior now observed: Yes / No

**Automated Tests:**
- Unit tests: {Pass/Fail}
- Integration tests: {Pass/Fail}
- Full test suite: {Pass/Fail}

**Edge Cases Tested:**
- {Edge case 1}: {Pass/Fail}
- {Edge case 2}: {Pass/Fail}

**Manual Verification:**
- {Test performed}: {Result}

**Verification Status:** PASSED / FAILED
```

If verification fails, return to Step 3 (Hypothesize).

### Step 7: DOCUMENT

**Goal:** Record what was wrong and how it was fixed for future reference.

Actions:
1. Update code comments if needed
2. Document the fix in commit message
3. Consider if documentation needs updating
4. Identify if similar bugs might exist elsewhere

Output:
```markdown
## Step 7: Document

**Summary:**
{One paragraph describing the bug and fix}

**Commit Message:**
```
fix: {short description}

{Longer description of the issue and solution}

Root cause: {what was wrong}
Solution: {what was changed}
```

**Documentation Updates Needed:**
- [ ] Code comments added
- [ ] README updated
- [ ] API docs updated
- [ ] None needed

**Related Areas to Check:**
- {Similar code that might have same bug}

**Prevention:**
- {How to prevent similar bugs in future}
```

## Using the Looper Skill

If you get stuck at any step, invoke the **looper** skill:

1. Document the blocker
2. Analyze why you're stuck
3. Propose alternative approaches
4. Try next approach
5. Repeat until resolved

## Debug Output Template

```markdown
# Debug Report: {Issue Title}

**Date:** {date}
**Status:** Resolved / In Progress / Blocked

## Step 1: Reproduce
{content}

## Step 2: Isolate
{content}

## Step 3: Hypothesize
{content}

## Step 4: Test
{content}

## Step 5: Fix
{content}

## Step 6: Verify
{content}

## Step 7: Document
{content}

## Time Spent
- Total: {time}
- By step: {breakdown}
```
