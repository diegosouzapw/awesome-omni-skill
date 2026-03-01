---
name: debug
description: This skill should be used when the user asks to "debug this", "find the bug", "fix this error", "why isn't this working", "troubleshoot", or when code changes need verification through browser testing.
version: 1.0.0
---

# Debug Workflow Skill

This skill implements a systematic debug loop with **runtime logs access**, modeled after Cursor's Debug Mode.

## Core Principle: Evidence-Based Debugging

**DO NOT immediately write a fix.** Like Cursor, this workflow requires:
1. Generate hypotheses first
2. Instrument code to collect runtime data
3. Fix based on actual evidence
4. Verify the fix works
5. Clean up all instrumentation

## Task Classification

Before starting any work, classify the request:

### Level 1: Minor Tweak (No debug loop, no commit)
**Examples:** Font size change, color adjustment, padding/margin tweak, text update
**Criteria:**
- Single property change in CSS or config
- No logic changes
- Purely visual/cosmetic
- User says "just", "quick", "small", "tweak"

**Action:** Make the change, brief confirmation, done. No debug loop, no commit.
```
---
**Minor Tweak**
Changed [what] from [old] to [new] in [file]
---
```

### Level 2: Small Change (Quick verify, no commit unless requested)
**Examples:** Add a CSS class, rename a variable, adjust a threshold, fix a typo in logic
**Criteria:**
- Few lines of code
- Low risk of breaking anything
- Related to current work (part of previous task)

**Action:** Make change, quick browser check for errors, report. No full debug loop.
```
---
**Small Change**
Updated [what] in [file]
Verified: No console errors
---
```

### Level 3: Task (Full debug loop if issues, commit when done)
**Examples:** New feature, bug fix, refactor, behavior change
**Criteria:**
- Multiple files or significant logic
- Could break existing functionality
- User explicitly asks to debug/fix
- Errors are present

**Action:** Full debug workflow with instrumentation, verification, cleanup, and commit.

### Classification Rules
- Default to **lower level** when uncertain
- Escalate if errors appear during quick verify
- User can override: "just make the change" = Level 1, "debug this" = Level 3
- Accumulate Level 1/2 changes; commit as batch when user says "commit" or at end of session

## When Full Debug Loop Activates

- User reports a bug or error
- Code changes need browser verification AND errors are found
- Console errors or unexpected behavior observed
- User explicitly asks to "debug", "fix", "troubleshoot", or "find the bug"
- A Level 2 change reveals unexpected errors

---

## The Debug Loop

### Phase 1: Hypothesize

Before ANY code changes, generate 2-3 hypotheses:

**Output format:**
```
---
**Debug Session Started**

**Reported Issue:** [user's description]

**Hypotheses:**
1. [Most likely cause] - [why this might be it]
2. [Alternative cause] - [why this might be it]
3. [Edge case] - [why this might be it]

**Investigation Plan:** [which hypothesis to test first, what logs needed]
---
```

### Phase 2: Instrument for Runtime Logs

**This is the key step that matches Cursor's approach.**

Add logging to collect: variable states, execution paths, timing information.

#### Method 1: Source Code Instrumentation (Primary - like Cursor)

Edit the actual source files to add debug logging:

```javascript
// BEFORE
function processData(data) {
  const result = transform(data);
  return result;
}

// AFTER (instrumented)
function processData(data) {
  console.log('[DEBUG:processData] input:', JSON.stringify(data));
  console.log('[DEBUG:processData] timestamp:', Date.now());
  const result = transform(data);
  console.log('[DEBUG:processData] output:', JSON.stringify(result));
  return result;
}
```

**Instrumentation rules:**
- Prefix ALL debug logs with `[DEBUG:functionName]`
- Log function inputs AND outputs
- Log timestamps for async operations
- Log variable states at decision points
- Track the instrumented files for cleanup

#### Method 2: Runtime Injection (Supplementary)

For quick checks without modifying source, use browser injection:

```javascript
mcp__chrome__evaluate_script({
  function: `() => {
    // Wrap a function to trace calls
    const original = window.someFunction;
    window.someFunction = function(...args) {
      console.log('[DEBUG:runtime] someFunction called:', args);
      const result = original.apply(this, args);
      console.log('[DEBUG:runtime] someFunction returned:', result);
      return result;
    };
    return 'Instrumentation installed';
  }`
})
```

### Phase 3: Reproduce & Collect Logs

1. **Reload the extension** (if source was modified):
   - Navigate to `chrome://extensions`
   - Click reload button

2. **Ask user to reproduce:**
   ```
   ---
   **Action Required:** Please reproduce the issue by [specific steps].
   I've added instrumentation to collect runtime data.
   ---
   ```

3. **Collect the runtime logs:**
   ```
   mcp__chrome__list_console_messages({ types: ["log", "error", "warn"] })
   ```

4. **Parse the evidence:**
   - Filter for `[DEBUG:` prefixed messages
   - Extract variable states, execution paths, timing
   - Identify where behavior diverges from expected

**Output format:**
```
---
**Runtime Logs Collected**

**Execution Path:** [what functions were called in what order]
**Variable States:** [key values observed]
**Timing:** [any delays or race conditions]
**Anomaly Found:** [where things went wrong]
---
```

### Phase 4: Fix Based on Evidence

Now—and only now—implement the fix:

1. **Identify root cause** from the collected logs
2. **Implement minimal fix** - target 2-3 lines when possible
3. **Keep instrumentation in place** for verification

**Output format:**
```
---
**Debug Loop [#]**
- Evidence: [what the logs showed]
- Root Cause: [identified problem]
- Fix: [description of change]
- Next: Verifying the fix
---
```

### Phase 5: Verify

1. **Reload extension** (if modified)
2. **Ask user to reproduce again:**
   ```
   ---
   **Verification Required:** Please try to reproduce the bug again.
   The fix is in place. If the bug is gone, I'll clean up the debug code.
   ---
   ```
3. **Check logs for success:**
   - No errors in console
   - Expected execution path
   - Correct variable states

4. **Evaluate:**
   - **Fixed:** Proceed to cleanup
   - **Not fixed:** Return to Phase 2 with more instrumentation

### Phase 6: Cleanup

**Critical: Remove ALL instrumentation.**

1. **Remove all `[DEBUG:` logging statements** from source files
2. **Revert any runtime injection** (page reload clears these)
3. **Verify only the minimal fix remains**
4. **Final reload and test** to confirm clean state

**Output format:**
```
---
**Debug Complete**

- Issue: [original problem]
- Cause: [root cause identified via runtime logs]
- Fix: [what was changed - should be minimal]
- Verified: [how it was tested]
- Cleanup: [instrumentation removed from X files]
---
```

### Phase 7: Commit (After User Verification)

**When the user confirms the fix is complete**, automatically commit the changes:

1. **Stage the fixed files** (only the files with actual fixes, not any leftover debug code)
2. **Create a commit** with a descriptive message:
   ```
   git commit -m "Fix: [brief description of the bug fix]

   - Root cause: [what was wrong]
   - Solution: [what was changed]

   Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"
   ```

3. **Report the commit:**
   ```
   ---
   **Committed**
   - Hash: [commit hash]
   - Files: [list of committed files]
   - Message: [commit message summary]
   ---
   ```

**Note:** Only commit after user explicitly confirms the fix works. Do not commit automatically after verification—wait for user confirmation.

---

## Iteration Rules

- **Max iterations:** 5 before escalating to user
- **Progress reports:** After each iteration, output status (don't pause)
- **Track instrumented files:** Maintain a list for cleanup
- **Escalate when:**
  - Same error after 3 different fix attempts
  - Need credentials or external access
  - Multiple valid approaches require user decision
  - Issue is outside the codebase

---

## Instrumentation Tracking

Maintain a mental list of instrumented code:

```
**Instrumented Files:**
- extension/modules/capacity-view.js (lines 45, 67, 89)
- extension/background.js (lines 120, 145)
```

This ensures complete cleanup.

---

## Screenshot Policy

**Do NOT take screenshots** unless the issue can ONLY be verified visually (layout bugs, visual rendering issues, CSS problems).

For all other cases, use:
- Console logs (`mcp__chrome__list_console_messages`)
- Network requests (`mcp__chrome__list_network_requests`)
- DOM snapshots (`mcp__chrome__take_snapshot`)
- Script evaluation (`mcp__chrome__evaluate_script`)

The user can see the browser directly—screenshots waste context and add no value for functional issues.

---

## References

- `references/instrumentation.md` - Detailed instrumentation patterns
- `references/browser-tools.md` - Chrome MCP tool reference
- `references/strategies.md` - Common debugging strategies
