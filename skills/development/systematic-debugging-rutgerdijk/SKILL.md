---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

**Announce at start:** "I'm using the systematic-debugging skill to investigate this issue."

## The Iron Laws

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

```
NO SHORTCUTS. NO WORKAROUNDS. FIX THE ROOT CAUSE.
```

**Violating the letter of this rule is violating the spirit of debugging.**

### Forbidden Shortcuts

These are NEVER acceptable fixes:

**TypeScript/JavaScript:**
| Shortcut | Why It's Wrong |
|----------|----------------|
| `// @ts-ignore` or `// @ts-expect-error` | Suppresses type safety, bug waiting to happen |
| `// eslint-disable` or `// eslint-disable-next-line` | Hides code quality issues |
| `any` type | Bypasses type checking, defeats the purpose |
| `as` type casting to force types | Bypasses type safety |
| Adding `!` (non-null assertion) | Lies to the compiler |
| Empty catch blocks `catch (e) {}` | Swallows errors, makes debugging impossible |

**.NET/C#:**
| Shortcut | Why It's Wrong |
|----------|----------------|
| `#pragma warning disable` | Hides compiler warnings |
| `[SuppressMessage]` attributes | Hides analyzer findings |
| `catch (Exception) { }` empty catch | Swallows all errors silently |
| `catch { }` without exception variable | Can't even log what went wrong |
| `object` instead of proper type | Bypasses type safety |
| `dynamic` keyword | Bypasses compile-time checking |
| `!` null-forgiving operator | Lies to the compiler about nullability |
| `default!` to silence nullable warnings | Hiding null issues, not fixing them |
| `[Obsolete]` on new code to silence warnings | Abuse of the attribute |
| `.Result` or `.Wait()` on async | Deadlock risk, hides async issues |

**Both:**
| Shortcut | Why It's Wrong |
|----------|----------------|
| Commenting out code | Hides the problem, breaks functionality |
| Wrapping in try/catch without handling | Masks the real error |
| Disabling tests that fail | Tests exist for a reason |
| Deleting tests that fail | You're deleting the safety net |

**If you find yourself reaching for any of these, STOP.**

Return to Phase 1. Find the root cause. Fix it properly.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully**
   - Don't skip past errors or warnings
   - Read stack traces completely
   - Note line numbers, file paths, error codes

2. **Reproduce Consistently**
   - Can you trigger it reliably?
   - What are the exact steps?
   - If not reproducible → gather more data, don't guess

3. **Check Recent Changes**
   - What changed that could cause this?
   - Git diff, recent commits
   - New dependencies, config changes

4. **Gather Evidence in Multi-Component Systems**

   For .NET/React stack, check each layer:
   ```bash
   # Frontend
   echo "=== Browser Console ==="
   # Check DevTools console for errors

   # API Layer
   echo "=== API Response ==="
   curl -v http://localhost:5000/api/endpoint

   # Backend Logs
   echo "=== .NET Logs ==="
   # Check Serilog output

   # Database
   echo "=== Database State ==="
   # Check PostgreSQL for data issues
   ```

5. **Trace Data Flow**
   - Where does bad value originate?
   - Trace backward through call stack
   - Fix at source, not at symptom

### Phase 2: Pattern Analysis

1. **Find Working Examples**
   - Locate similar working code in same codebase
   - What works that's similar to what's broken?

2. **Compare Against References**
   - Check instruction files in `.github/instructions/` for common mistakes
   - Read the relevant instruction file completely

3. **Identify Differences**
   - What's different between working and broken?
   - List every difference, however small

### Phase 3: Hypothesis and Testing

1. **Form Single Hypothesis**
   - State clearly: "I think X is the root cause because Y"
   - Be specific, not vague

2. **Test Minimally**
   - Make the SMALLEST possible change to test hypothesis
   - One variable at a time
   - Don't fix multiple things at once

3. **Verify Before Continuing**
   - Did it work? Yes → Phase 4
   - Didn't work? Form NEW hypothesis
   - DON'T add more fixes on top

4. **When You Don't Know**
   - Say "I don't understand X"
   - Don't pretend to know
   - Don't guess and hope
   - Ask for help or research more

### Phase 4: Implementation

1. **Create Failing Test Case**
   - Follow TDD: write test BEFORE fix
   - Test must fail without the fix
   - Commit: `test(<scope>): add test for <issue>`

2. **Implement Single Fix**
   - Address the root cause identified
   - ONE change at a time
   - Commit: `fix(<scope>): <description>`

**Scopes:** `backend`, `frontend`, `api`, `db`, `auth`, `tests`, `docs`

3. **Verify Fix**
   - Test passes now?
   - No other tests broken?
   - Issue actually resolved?

4. **If 3+ Fixes Failed: Question Architecture**

   Pattern indicating architectural problem:
   - Each fix reveals new shared state/coupling
   - Fixes require "massive refactoring"
   - Each fix creates new symptoms elsewhere

   **STOP and discuss with user before attempting more fixes**

## Red Flags - STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "Let me just comment this out for now"
- "I'll add @ts-ignore to make it compile"
- "I'll disable this lint rule"
- "I'll use `any` type here"
- Proposing solutions before tracing data flow
- Reaching for suppressions instead of fixes

**ALL of these mean: STOP. Return to Phase 1.**

## User Signals You're Doing It Wrong

Watch for these redirections from the user:
- "Is that not happening?" → You assumed without verifying
- "Will it show us...?" → You should have added evidence gathering
- "Stop guessing" → You're proposing fixes without understanding
- "Go deeper" → You're treating symptoms, not root cause
- "Why did you comment that out?" → You took a shortcut
- "That's a workaround, not a fix" → Return to Phase 1

**When you see these: STOP. Return to Phase 1.**

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question pattern, don't fix again. |
| "I'll just comment this out temporarily" | Temporary becomes permanent. Fix it properly now. |
| "The linter rule is wrong here" | The linter found a real issue. Fix the code, not the linter. |
| "@ts-ignore is fine for this one case" | One case becomes many. Fix the type error properly. |
| "Using `any` saves time" | `any` hides bugs that will cost MORE time later. |
| "I'll suppress the warning for now" | Warnings exist for a reason. Understand and fix them. |
| "The error doesn't affect functionality" | If it doesn't matter, why suppress it? If it matters, fix it. |
| "I don't understand the error, so ignore it" | Not understanding is the problem. Learn, then fix. |

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## Supporting Techniques

These techniques are part of systematic debugging and available in this directory:

| Technique | When to Use | File |
|-----------|-------------|------|
| **Root Cause Tracing** | Bug appears deep in call stack, need to trace backward | `root-cause-tracing.md` |
| **Defense-in-Depth** | After fixing, add validation at every layer | `defense-in-depth.md` |
| **Condition-Based Waiting** | Flaky tests with arbitrary delays | `condition-based-waiting.md` |

### Root Cause Tracing

When error is deep in call stack:
1. Find immediate cause
2. Ask: "What called this?"
3. Keep tracing up
4. Fix at the SOURCE, not the symptom

See `root-cause-tracing.md` for detailed process with .NET/React examples.

### Defense-in-Depth

After finding root cause, add validation at EVERY layer:
1. Entry point (Controller/Form)
2. Business logic (Service)
3. Environment guards
4. Debug instrumentation

See `defense-in-depth.md` for the 4-layer pattern.

### Condition-Based Waiting

For flaky tests, replace arbitrary delays with condition polling:

```csharp
// ❌ await Task.Delay(500);
// ✅ await WaitForAsync(() => GetResult(), r => r != null, "result");
```

See `condition-based-waiting.md` for implementation patterns.

## Integration with Other Skills

**After debugging:**
- `verification` — Verify fix before claiming success
- `layered-review` — Review changes before finalizing

**Related skills:**
- `test-driven-development` — Write failing test before fix (Phase 4)
- `executing-plans` — TDD enforcement during implementation
- `pressure-test-scenarios` — Scenarios 1, 10, 11, 12 test debugging discipline

**Instruction files to consult:**
- `dotnet.instructions.md` — Common .NET debugging patterns
- `react.instructions.md` — Common React debugging patterns
- `testing-dotnet.instructions.md` — xUnit/FluentAssertions patterns
- `testing-playwright.instructions.md` — E2E test debugging
