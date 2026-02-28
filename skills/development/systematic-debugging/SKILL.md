---
name: systematic-debugging
description: "Use for ANY technical issue - bugs, test failures, unexpected behavior, performance problems. Find root cause BEFORE attempting fixes."
---

# Systematic Debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes.

## When to Use

Use for ANY technical issue:
- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:**
- Issue seems simple (simple bugs have root causes too)
- You're in a hurry (rushing guarantees rework)

## The Four Phases

You MUST complete each phase before proceeding to the next.

### Phase 1: Root Cause Investigation

**Goal:** Understand WHAT is happening and WHERE.

1. **Reproduce the issue** - Can you make it happen consistently?
2. **Gather evidence** - Logs, stack traces, error messages
3. **Isolate the scope** - Which component/module/function?
4. **Timeline** - When did it start? What changed?

**Output:** "The issue is [X] happening in [Y] because [Z]."

### Phase 2: Pattern Analysis

**Goal:** Understand WHY it's happening.

1. **Check recent changes** - Git log, deployment history
2. **Look for patterns** - Does it happen under specific conditions?
3. **Review assumptions** - What did we assume that might be wrong?
4. **Check dependencies** - External services, libraries, data

**Output:** "Root cause is [specific mechanism]."

### Phase 3: Hypothesis and Testing

**Goal:** Confirm your understanding.

1. **Form hypothesis** - "If X is the cause, then Y should happen"
2. **Design experiment** - How to test the hypothesis?
3. **Run experiment** - Execute and observe
4. **Validate** - Did results match prediction?

**Output:** "Confirmed: [root cause] causes [symptom] because [mechanism]."

### Phase 4: Implementation

**Goal:** Fix the root cause, not the symptom.

1. **Write failing test** - Reproduces the bug (TDD skill)
2. **Implement fix** - Address root cause
3. **Verify fix** - Test passes
4. **Regression check** - No new failures

**Output:** Committed fix with test.

## Red Flags - STOP and Follow Process

| You're Doing This | Reality |
|-------------------|---------|
| "Let me try this quick fix" | You're guessing. Stop. Investigate. |
| "I think I know what's wrong" | Thinking ≠ knowing. Gather evidence. |
| "This worked last time" | Past ≠ present. Find this root cause. |
| "Just restart it" | Restart masks, doesn't fix. Investigate. |
| Third fix attempt | You're thrashing. Start Phase 1 over. |

## Quick Reference

```
Issue reported
     │
     ▼
┌─────────────────┐
│ Phase 1:        │
│ INVESTIGATE     │
│ What? Where?    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Phase 2:        │
│ ANALYZE         │
│ Why?            │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Phase 3:        │
│ HYPOTHESIS      │
│ Test theory     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Phase 4:        │
│ IMPLEMENT       │
│ Fix + Test      │
└─────────────────┘
```
