---
name: Algorithmic Command Enforcement
description: Use boolean decision trees instead of imperatives for 100% compliance under pressure
when_to_use: when writing commands or agents that enforce discipline (TDD, code review, git workflows) where compliance is required even under time pressure, sunk cost, exhaustion, or authority pressure
version: 1.0.0
---

# Algorithmic Command Enforcement

## Overview

Agents follow **algorithmic decision trees** (100% compliance) better than **imperative instructions** (0-33% compliance), even with MUST/DELETE language. LLMs treat algorithms as deterministic systems requiring execution, but treat imperatives as suggestions open to interpretation.

**Core principle:** Stop writing imperatives. Start writing algorithms.

## When to Use

**Use algorithmic format when:**
- Discipline-enforcing workflows (TDD, code review, verification)
- High compliance required (no acceptable bypass cases)
- Agents are under pressure (time, authority, sunk cost, exhaustion)
- Multiple escape hatches exist (simplicity, pragmatism, efficiency)
- Cost of non-compliance is high (technical debt, bugs, process violations)
- Decision is binary (yes/no question, not judgment call)

**Use imperative format when:**
- Suggestions/guidance (flexibility desired)
- Context determines best action (judgment required)
- Compliance nice-to-have but not critical
- Decision is subjective (quality, style, approach)

**Hybrid approach:**
- Algorithm for WHEN to use workflow (binary decision)
- Imperative for HOW to execute workflow (implementation details)

## Core Pattern

### ❌ Imperative Version (0-33% compliance)

```markdown
You MUST use /execute for any implementation plan.

DO NOT bypass this workflow for:
- "Simple" tasks
- Time pressure
- Tasks you've already started

If you wrote code without tests, DELETE it and start over.
```

**Agent rationalizations:**
- "Any could mean any complex plan. Mine are simple."
- "These are just markdown edits, don't need formal process"
- "I'll test after - achieves same goal"
- "Deleting 2 hours work is wasteful"

**Result:** Agents acknowledge rules then bypass them anyway.

### ✅ Algorithmic Version (100% compliance)

```markdown
## Decision Algorithm: When to Use /execute

## 1. Check for plan file

Does a file matching `docs/plans/*.md` exist?

- PASS: CONTINUE
- FAIL: GOTO 5

## 2. Check for exploration only

Is the task exploration/research only (no commits)?

- PASS: GOTO 5
- FAIL: CONTINUE

## 3. Execute /execute

Execute `/execute [plan-file-path]`

STOP reading this algorithm

## 4. [UNREACHABLE - if you reach here, you violated Step 3]

## 5. Proceed without /execute

Proceed without /execute (valid cases only)

## Recovery Algorithm: Already Started Without /execute?

## 1. Check for code

Have you written ANY code?

- PASS: CONTINUE
- FAIL: GOTO 4

## 2. Check for tests

Does that code have tests?

- PASS: GOTO 4
- FAIL: CONTINUE

## 3. Delete untested code

Delete the untested code

Execute: rm [files] OR git reset --hard

Then create/use plan file with /execute

- PASS: STOP
- FAIL: STOP

## 4. Continue current work

Tests exist OR no code written yet

## INVALID conditions (NOT in algorithm, do NOT use):
- "Is task simple?" → NOT A VALID CONDITION
- "Is there time pressure?" → NOT A VALID CONDITION
- "Should I be pragmatic?" → NOT A VALID CONDITION
- "Is there sunk cost?" → NOT A VALID CONDITION
- "Am I exhausted?" → NOT A VALID CONDITION

## Self-Test

Q1: Does file `docs/plans/my-task.md` exist?
    If YES: What does Step 3 say to do?
    Answer: Execute /execute and STOP

Q2: I wrote code 2 hours ago without tests. Recovery algorithm Step 3 says?
    Answer: Delete the untested code

Q3: "These are simple markdown tasks" - is this a valid algorithm condition?
    Answer: NO. Listed under INVALID conditions
```

**Agent recognition:**
- "Step 2: Does code have tests? → NO"
- "Step 3: Delete the untested code"
- "Non-factors correctly ignored: ❌ 2 hours sunk cost, ❌ Exhaustion"
- "The algorithm prevented me from rationalizing based on 'simple tasks'"

## Five Mechanisms That Work

### 1. Boolean Conditions (No Interpretation)

**Imperative:** "Use /execute for any implementation plan"
**Agent:** "Any could mean any complex plan"

**Algorithmic:** "Does file `docs/plans/*.md` exist? → YES/NO"
**Agent:** Binary evaluation. No room for interpretation.

### 2. Explicit Invalid Conditions List

**Imperative:** "Regardless of time pressure or sunk cost..."
**Agent:** Still debates what these mean

**Algorithmic:**
```markdown
INVALID conditions (NOT in algorithm):
- "Is task simple?" → NOT A VALID CONDITION
- "Is there sunk cost?" → NOT A VALID CONDITION
```
**Agent:** Sees rationalization listed as explicitly invalid. Creates meta-awareness.

### 3. Deterministic Execution Path with STOP

**Imperative:** Multiple "MUST" statements → agent prioritizes/balances them

**Algorithmic:**
```markdown
Step 3: Execute /execute [plan]
        STOP reading this algorithm
        Do not proceed to Step 4
```
**Result:** Single path from conditions. No choices. STOP prevents further processing.

### 4. Self-Test Forcing Comprehension

Include quiz with correct answers:
```markdown
Q1: Does file `docs/plans/my-task.md` exist?
    If YES: What does Step 3 say to do?
    Answer: Execute /execute and STOP
```

Agents must demonstrate understanding before proceeding. Catches comprehension failures early.

### 5. Unreachable Steps Proving Determinism

```markdown
Step 4: [UNREACHABLE - if you reach here, you violated Step 3]
Step 5: [UNREACHABLE - if you reach here, you violated Step 3]
```

Demonstrates algorithm is deterministic. Reaching unreachable steps = violation.

## Quick Reference: Algorithm Template

```markdown
## Decision Algorithm: [When to Use X]

## 1. Check [Boolean condition]

[Boolean condition]?

- PASS: CONTINUE
- FAIL: GOTO N (skip workflow)

## 2. Check [Boolean exception]

[Boolean exception]?

- PASS: GOTO N (skip workflow)
- FAIL: CONTINUE

## 3. Execute [action]

Execute [action]

STOP reading this algorithm

## N. [Alternative path or skip]

[Alternative path or skip]

## Recovery Algorithm: [Already Started Wrong?]

## 1. Check [Have you done X]

Have you done X?

- PASS: CONTINUE
- FAIL: GOTO N

## 2. Delete/undo the work

Delete/undo the work

- PASS: STOP
- FAIL: STOP

## N. Continue

Continue

## INVALID conditions (NOT in algorithm):
- "[Rationalization]" → NOT A VALID CONDITION
- "[Excuse]" → NOT A VALID CONDITION

## Self-Test

Q1: [Scenario] → What does Step X say?
    Answer: [Expected action]
```

## Common Mistakes

| Mistake | Why It Fails | Fix |
|---------|--------------|-----|
| Using "MUST" language | Agents treat as strong suggestion | Use boolean Step conditions |
| Rationalization defense tables | Agents acknowledge then use anyway | List as INVALID conditions |
| Missing STOP command | Agents continue reading and find loopholes | Explicit STOP after action |
| No self-test section | Comprehension failures go undetected | Include quiz with answers |
| Subjective conditions | "Complex", "simple", "important" are debatable | Only boolean yes/no conditions |

## Real-World Impact

**Evidence from pressure testing:**
- Imperative format: 33% compliance (1/3 scenarios passed)
- Same content, algorithmic format: 100% compliance (3/3 scenarios passed)
- **0% → 100% improvement** from format change alone

**Pressure scenarios that failed with imperatives, passed with algorithms:**
1. Simple tasks + 30-minute deadline → Algorithm prevented "too simple for process" rationalization
2. 2 hours untested code + exhaustion + sunk cost → Algorithm mandated deletion despite investment
3. Authority pressure + economic stakes → Algorithm enforced despite manager directive

**Agent quotes:**
> "The algorithm successfully prevented me from rationalizing based on 'simple markdown edits'"

> "Non-factors correctly ignored: ❌ 2 hours sunk cost, ❌ Exhaustion, ❌ Time pressure"

> "The algorithmic documentation eliminated ambiguity - every condition is boolean (YES/NO)"

## High-Priority Applications

Convert these workflows to algorithmic format:

1. **TDD enforcement** - "Does code have tests? NO → Delete"
2. **Code review trigger** - "Changes committed? YES + not reviewed? YES → Run review"
3. **Git workflow** - Based on test status, review status
4. **Verification before completion** - Binary checks before claiming "done"

## Testing Evidence

See test artifacts for full RED-GREEN-REFACTOR campaign:
- `docs/tests/execute-command-test-scenarios.md` - Pressure scenarios
- `docs/tests/execute-command-test-results.md` - Baseline (RED) and imperative (GREEN) results
- `docs/tests/execute-command-algorithmic.md` - Algorithmic version (REFACTOR) results
- `docs/learning/2025-10-16-algorithmic-command-enforcement.md` - Complete retrospective

**Methodology:** Following `${CLAUDE_PLUGIN_ROOT}skills/testing-skills-with-subagents/SKILL.md` - pressure scenarios with time, sunk cost, authority, and exhaustion combined.

## Agent vs Command Documentation

**Key distinction:**

- **Agents** (specialized subagents): Use persuasion principles (Authority, Commitment, Scarcity, Social Proof)
- **Commands** (read by main Claude): Use algorithmic decision trees

**Why different:**
- Agents operate in closed system (dedicated to one task)
- Commands operate in open system (competing priorities)
- Agents need motivation (persuasion)
- Commands need determinism (algorithms)

Don't copy agent template principles to commands. Use appropriate format for context.
