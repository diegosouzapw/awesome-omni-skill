---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
compatibility: opencode
license: MIT
---

# Systematic Debugging

## Overview
Random fixes waste time and mask underlying issues. **Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

## The Iron Law
`NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST`

## When to Use
Use for ANY technical issue (test failures, production bugs, performance).
**ESPECIALLY when:**
- Under time pressure or "quick fix" seems obvious.
- You've already tried multiple fixes or previous fix failed.
- You don't fully understand the issue.

## The Four Phases
Complete each phase before proceeding.

### Phase 1: Root Cause Investigation
1. **Read Error Messages**: Note line numbers, file paths, and stack traces.
2. **Reproduce Consistently**: Identify exact steps to trigger the bug reliably.
3. **Check Recent Changes**: Review git diffs, new dependencies, or config changes.
4. **Gather Evidence**: In multi-component systems, log data at boundaries to isolate the failing layer.
5. **Trace Data Flow**: For deep errors, trace backward to the source (see **Root Cause Tracing** below).

### Phase 2: Pattern Analysis
1. **Find Working Examples**: Locate similar working code.
2. **Compare Against References**: Read reference implementations completely.
3. **Identify Differences**: List every difference between working and broken states.

### Phase 3: Hypothesis and Testing
1. **Form Single Hypothesis**: "I think X is the root cause because Y."
2. **Test Minimally**: Make the SMALLEST change to test the hypothesis.
3. **Verify**: If it fails, form a NEW hypothesis. Don't stack fixes.

### Phase 4: Implementation
1. **Create Failing Test**: Simplest reproduction (use `test-driven-development`).
2. **Implement Single Fix**: Address the root cause identified.
3. **Verify Fix**: Ensure test passes and no regressions.
4. **If 3+ Fixes Fail**: STOP and question the architecture. Discuss with your human partner.

## Red Flags - STOP and Return to Phase 1
- "Quick fix for now" or "Just try changing X."
- Adding multiple changes at once.
- Skipping tests or manual verification only.
- Proposing solutions before tracing data flow.
- Each fix reveals a new problem elsewhere.

## Common Rationalizations
| Excuse | Reality |
|--------|---------|
| "Issue is simple" | Simple bugs have root causes too. |
| "Emergency" | Systematic is FASTER than thrashing. |
| "Test after fix" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes" | Can't isolate what worked. Causes new bugs. |
| "3+ failures" | Architectural problem. Question the pattern. |

## Quick Reference
| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## Supporting Techniques
Use the embedded templates below to apply specific debugging strategies:

### Root Cause Tracing
**Core principle:** Trace backward through the call chain until you find the original trigger, then fix at the source.

```markdown
# Root Cause Tracing

## Overview

Bugs often manifest deep in the call stack (git init in wrong directory, file created in wrong location, database opened with wrong path). Your instinct is to fix where the error appears, but that's treating a symptom.

## The Tracing Process

### 1. Observe the Symptom
```
Error: git init failed in /Users/jesse/project/packages/core
```

### 2. Find Immediate Cause
**What code directly causes this?**
```typescript
await execFileAsync('git', ['init'], { cwd: projectDir });
```

### 3. Ask: What Called This?
```typescript
WorktreeManager.createSessionWorktree(projectDir, sessionId)
  → called by Session.initializeWorkspace()
  → called by Session.initialize()
  → called by test at Project.create()
```

### 4. Keep Tracing Up
**What value was passed?**
- `projectDir = ''` (empty string!)
- Empty string as `cwd` resolves to `process.cwd()`
- That's the source code directory!

### 5. Find Original Trigger
**Where did empty string come from?**
```typescript
const context = setupCoreTest(); // Returns { tempDir: '' }
Project.create('name', context.tempDir); // Accessed before beforeEach!
```

## Adding Stack Traces

When you can't trace manually, add instrumentation:

```typescript
// Before the problematic operation
async function gitInit(directory: string) {
  const stack = new Error().stack;
  console.error('DEBUG git init:', {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  });

  await execFileAsync('git', ['init'], { cwd: directory });
}
```

**Critical:** Use `console.error()` in tests (not logger - may not show)

**Run and capture:**
```bash
npm test 2>&1 | grep 'DEBUG git init'
```

**Analyze stack traces:**
- Look for test file names
- Find the line number triggering the call
- Identify the pattern (same test? same parameter?)

**Finding Which Test Causes Pollution:**
If something appears during tests but you don't know which test is responsible, use the bisection script: `skills/systematic-debugging/find-polluter.sh`. It runs tests one-by-one until it finds the first one that creates the unwanted file or state.
```

### Defense in Depth
**Core principle:** Validate at EVERY layer data passes through. Make the bug structurally impossible.

```markdown
# Defense-in-Depth Validation

## The Four Layers

### Layer 1: Entry Point Validation
**Purpose:** Reject obviously invalid input at API boundary

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  // ... proceed
}
```

### Layer 2: Business Logic Validation
**Purpose:** Ensure data makes sense for this operation

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### Layer 3: Environment Guards
**Purpose:** Prevent dangerous operations in specific contexts

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### Layer 4: Debug Instrumentation
**Purpose:** Capture context for forensics

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```
```

### Condition-Based Waiting
**Core principle:** Wait for the actual condition you care about, not a guess about how long it takes.

```markdown
# Condition-Based Waiting

## Core Pattern

```typescript
// ❌ BEFORE: Guessing at timing
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ AFTER: Waiting for condition
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

## Quick Patterns

| Scenario | Pattern |
|----------|---------|
| Wait for event | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| Wait for state | `waitFor(() => machine.state === 'ready')` |
| Wait for count | `waitFor(() => items.length >= 5)` |
| Wait for file | `waitFor(() => fs.existsSync(path))` |
| Complex condition | `waitFor(() => obj.ready && obj.value > 10)` |

## Implementation

Generic polling function:
```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // Poll every 10ms
  }
}

**Complete Implementation:**
For a production-ready implementation of `waitForEvent`, `waitForEventCount`, and `waitForEventMatch` (useful for complex async systems), see: `skills/systematic-debugging/condition-based-waiting-example.ts`.
```
```

**Related skills:**
- **test-driven-development** - For creating failing test case (Phase 4, Step 1)
- **verification-before-completion** - Verify fix worked before claiming success

## Real-World Impact

From debugging sessions:
- Systematic approach: 15-30 minutes to fix
- Random fixes approach: 2-3 hours of thrashing
- First-time fix rate: 95% vs 40%
- New bugs introduced: Near zero vs common
