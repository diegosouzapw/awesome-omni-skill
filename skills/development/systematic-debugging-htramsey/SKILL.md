---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior (including race conditions, deadlocks, concurrency issues) - four-phase framework (root cause investigation, pattern analysis, hypothesis testing, implementation) with specialized techniques for deep call stack tracing and concurrency debugging
---

# Systematic Debugging

**Persona:** Methodical diagnostician who never guesses - treats symptoms as clues, not targets.

## The Iron Law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you haven't completed Phase 1, you cannot propose fixes. Symptom fixes are failure.

## Should NOT Attempt

- Propose fixes before completing Phase 1
- Make multiple changes at once "to save time"
- Copy solutions from StackOverflow without understanding
- Add logging everywhere without hypothesis
- "Clean up" unrelated code while debugging
- Skip reproduction because "I know what happened"

## The Four Phases

### Phase 1: Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully** - They often contain the exact solution
2. **Reproduce Consistently** - If not reproducible, gather more data
3. **Check Recent Changes** - Git diff, recent commits, new dependencies

4. **Binary Search Isolation** (when bug location unknown):
   ```
   1. Identify range: known-good → known-bad
   2. Bisect: Add logging at midpoint
   3. Narrow: Bug in first or second half?
   4. Repeat until isolated
   ```
   Use `git bisect` for regression bugs.

5. **Gather Evidence in Multi-Component Systems**:
   ```
   For EACH component boundary:
     - Log what data enters
     - Log what data exits
     - Verify environment propagation
   ```

6. **Trace Data Flow (Deep Call Stack)**:
   - Observe symptom at error point
   - Find immediate cause (what function?)
   - Trace up the call chain
   - Keep tracing to original trigger
   - Fix at source, not symptom

   **Adding Instrumentation:**
   ```typescript
   console.error('DEBUG:', { directory, cwd: process.cwd(), stack: new Error().stack });
   ```

7. **For Concurrency Bugs**: See [resources/references/concurrency.md](resources/references/concurrency.md)
   - Race conditions, deadlocks, livelocks
   - Shared state identification
   - Detection tools by language

### Phase 2: Pattern Analysis

1. **Find Working Examples** - Similar working code in same codebase
2. **Compare Against References** - Read reference implementation COMPLETELY
3. **Identify Differences** - List every difference, however small
4. **Understand Dependencies** - Components, config, environment

### Phase 3: Hypothesis and Testing

1. **Form Single Hypothesis** - "X is root cause because Y"
2. **Test Minimally** - SMALLEST possible change, one variable
3. **Verify Before Continuing** - Worked? → Phase 4. Didn't? → NEW hypothesis

### Phase 4: Implementation

1. **Create Failing Test Case** - REQUIRED (use `test-driven-development` skill)
2. **Implement Single Fix** - ONE change, no "while I'm here" improvements
3. **Verify Fix** - Test passes? No regressions?
4. **If Fix Doesn't Work** - Return to Phase 1 if < 3 attempts
5. **If 3+ Fixes Failed** - STOP. Question architecture.

## Red Flags - STOP and Return to Phase 1

- "Quick fix for now, investigate later"
- "Just try changing X and see"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- Proposing solutions before tracing data flow

## Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |

## Escalation Triggers

| Situation | Action |
|-----------|--------|
| Root cause spans multiple systems | Involve system owners |
| 3+ fix attempts failed | Question architecture with user |
| Race condition in 3+ locations | `orchestrator` agent for planning |
| Cannot reproduce locally | Ask for exact reproduction steps |
| Security vulnerability discovered | `code-reviewer` agent |

**Format:**
```
BLOCKED: [description]
Root cause: [what you found]
Evidence: [key data points]
Attempted: [what you tried]
Recommendation: [path forward]
```

## When Blocked

If debugging stalls:
1. Document current hypotheses and what's been tested
2. Look for overlooked assumptions (threading, async, environment)
3. Try binary search on recent changes (git bisect)
4. Add more logging/tracing around the suspected area
5. Sleep on it - fresh eyes often see what tired ones miss
6. Ask for code review - another perspective helps

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple" | Simple issues have root causes too |
| "Emergency, no time" | Systematic is FASTER than thrashing |
| "Just try this first" | First fix sets the pattern |
| "One more fix attempt" | 3+ failures = architectural problem |

## Integration

**Required:** `test-driven-development` skill (Phase 4)
**Related:** `verification-before-completion` skill, `code-reviewer` agent
