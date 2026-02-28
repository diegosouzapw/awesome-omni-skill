---
description: Systematic debugging workflow with parallel agent exploration, root cause analysis, and fix verification. Adapted from feature-dev methodology for bug investigation.
name: debugging-workflow
allowed-tools: ["Read", "Grep", "Glob", "Bash", "Task", "AskUserQuestion", "TodoWrite"]
---
# Debugging Workflow Rules

## Core Philosophy

**Understand before you fix.**

Debugging is not about quick patches - it's about:
1. Deep understanding of the bug's manifestation
2. Systematic tracing to root cause
3. Strategic fix design considering side effects
4. Thorough verification of the fix

---

## 6-Phase Debugging Workflow

| Phase | Name | Purpose | Agents |
|-------|------|---------|--------|
| 1 | Discovery | Understand bug symptoms | - |
| 2 | Exploration | Trace execution paths | 2-3 debug-explorer |
| 3 | Root Cause | Identify true cause | Analysis synthesis |
| 4 | Strategy | Design fix approach | 2-3 debug-strategist |
| 5 | Implementation | Apply the fix | User-approved |
| 6 | Verification | Confirm fix works | 2-3 debug-verifier |

---

## Phase Details

### Phase 1: Bug Discovery

Gather complete information about the bug:

```yaml
BugReport:
  symptoms: What is happening?
  expected: What should happen?
  reproduction: Steps to reproduce
  frequency: Always / Sometimes / Rare
  context: Environment, version, user actions
  error_output: Error messages, stack traces, logs
```

**Key Questions:**
- When did this start happening?
- What changed recently?
- Is it environment-specific?
- Can it be reliably reproduced?

---

### Phase 2: Codebase Exploration

Launch 2-3 **debug-explorer** agents in parallel with different focuses:

| Focus | Investigation Target |
|-------|---------------------|
| Execution Path | Trace the failing code path step by step |
| Data Flow | Track data transformations and mutations |
| Dependencies | Check related components and integrations |

**Serena MCP Integration:**
```
# Symbol-level tracing
find_symbol: Locate relevant functions/classes
find_referencing_symbols: Find all callers
search_for_pattern: Search for error patterns
```

---

### Phase 3: Root Cause Analysis

Synthesize exploration findings:

```yaml
RootCauseAnalysis:
  immediate_cause: What directly triggers the error?
  underlying_cause: Why does this condition exist?
  contributing_factors: What else enables this bug?
  timeline: When was this introduced?
```

**Common Root Cause Categories:**

| Category | Examples |
|----------|----------|
| Logic Error | Wrong condition, off-by-one, null check missing |
| State Management | Race condition, stale data, unexpected mutation |
| Integration | API contract mismatch, version incompatibility |
| Resource | Memory leak, connection exhaustion, timeout |
| Configuration | Wrong settings, missing env vars, path issues |

---

### Phase 4: Fix Strategy Design

Launch 2-3 **debug-strategist** agents with different approaches:

| Approach | Strategy |
|----------|----------|
| Minimal | Smallest change to fix the symptom |
| Comprehensive | Address root cause and prevent recurrence |
| Defensive | Add guards, validation, error handling |

**Strategy Evaluation Criteria:**
- Risk of regression
- Impact on related functionality
- Code quality improvement
- Testing feasibility

---

### Phase 5: Implementation

**Requires explicit user approval before proceeding.**

Implementation checklist:
- [ ] Apply the chosen fix strategy
- [ ] Update related code if needed
- [ ] Add defensive checks where appropriate
- [ ] Document the fix rationale

---

### Phase 6: Verification

Launch 2-3 **debug-verifier** agents with different focuses:

| Focus | Verification Target |
|-------|---------------------|
| Direct | Does the original bug no longer occur? |
| Regression | Are there any new issues introduced? |
| Edge Cases | Does it handle boundary conditions? |

**Verification Methods:**
- Manual reproduction attempt
- Related test execution
- Code review for side effects
- Static analysis check

---

## Confidence Scoring

All agents use confidence scoring (0-100):

| Score | Meaning | Action |
|-------|---------|--------|
| 0-25 | Speculation | Do not report |
| 26-50 | Possible | Mention if relevant |
| 51-79 | Likely | Report with caveats |
| 80-100 | Confident | Report as finding |

**Only report findings with confidence ≥ 80%**

---

## Integration with Serena MCP

| Task | Serena Tool |
|------|-------------|
| Find function definition | `find_symbol` |
| Trace all callers | `find_referencing_symbols` |
| Search error patterns | `search_for_pattern` |
| Get file overview | `get_symbols_overview` |
| Read specific symbol | `find_symbol` with `include_body=true` |

---

## Output Templates

### Exploration Report

```markdown
## Exploration: [Focus Area]

### Entry Points Found
- `file.ts:123` - `functionName` - [description]

### Execution Path
1. [Step 1 description]
2. [Step 2 description]

### Key Components
| Component | File | Responsibility |
|-----------|------|----------------|

### Architecture Insights
- [Insight 1]
- [Insight 2]

### Critical Files to Review
1. `path/to/file.ts` - [reason]
```

### Strategy Report

```markdown
## Fix Strategy: [Approach Name]

### Proposed Change
[Description of the fix]

### Files to Modify
| File | Change Type | Description |
|------|-------------|-------------|

### Risk Assessment
- Regression risk: Low/Medium/High
- Impact scope: [affected areas]

### Implementation Steps
1. [Step 1]
2. [Step 2]
```

### Verification Report

```markdown
## Verification: [Focus Area]

### Test Results
| Test | Status | Notes |
|------|--------|-------|

### Issues Found
- Severity: Critical/Important
- Confidence: [score]%
- Location: `file:line`
- Description: [issue]
- Suggested Fix: [fix]

### Final Assessment
[Pass/Fail with reasoning]
```

> **Detailed templates**: `Read("references/debug-patterns.md")`
