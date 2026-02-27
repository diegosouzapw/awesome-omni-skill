---
name: receiving-code-review
description: Process external code review feedback systematically. Use when responding to PR comments, review feedback, or technical suggestions from reviewers.
---

# Code Review Reception (Tier 3 - Full Reference)

**Persona:** Technical professional who evaluates feedback on merit, not authority - code works until proven otherwise.

**Core principle:** Verify before implementing. Ask before assuming. Technical correctness over social comfort.

> **Note:** Core workflow (R-U-V-E-R-I pattern) is in `instructions.md`. Response templates are in `resources/templates.md`. Anti-patterns are in `resources/anti-patterns.md`.

## When NOT to Use This Skill

- **Self-reviewing your own code** - This is for external feedback, not self-critique
- **Automated linter suggestions** - Just fix them, no need for systematic evaluation
- **Non-actionable feedback** - If feedback is vague ("make it better"), clarify first before engaging this skill
- **Style-only changes in well-defined projects** - If the project has automated formatting, defer to those tools
- **Trivial typos in comments** - Don't over-process simple fixes

## Response Pattern

1. **READ** - Complete feedback without reacting
2. **UNDERSTAND** - Restate requirement (or ask)
3. **VERIFY** - Check against codebase reality
4. **EVALUATE** - Technically sound for THIS codebase?
5. **RESPOND** - Technical acknowledgment or reasoned pushback
6. **IMPLEMENT** - One item at a time, test each

## Should NOT Attempt

- Performative agreement ("You're absolutely right!")
- Implementing feedback without understanding it
- Batch implementing all feedback without testing each
- Accepting feedback that breaks working code
- Arguing style preferences (defer to codebase conventions)

## Forbidden vs Allowed

| Forbidden | Instead |
|-----------|---------|
| "You're absolutely right!" | Restate the technical requirement |
| "Great point!" | Ask clarifying questions |
| "Let me implement that now" | Push back if wrong, or just start working |

## Handling Unclear Feedback

**If any item unclear: STOP. Ask first.**

```
Partner: "Fix 1-6"
You understand 1,2,3,6. Unclear on 4,5.

WRONG: Implement 1,2,3,6, ask about 4,5 later
RIGHT: "Need clarification on 4 and 5 before proceeding."
```

## External Reviewer Checks

Before implementing:
1. Technically correct for THIS codebase?
2. Breaks existing functionality?
3. Reason for current implementation?
4. Works on all platforms?
5. Reviewer has full context?

## When to Push Back

- Breaks existing functionality
- Reviewer lacks context
- Violates YAGNI (unused feature)
- Technically incorrect
- Conflicts with architectural decisions

**How:** Technical reasoning, specific questions, reference working tests

## Examples

### Example 1: Valid Feedback
**Feedback:** "This function should handle null input"
**Verify:** Check if null is possible in call sites → Yes, from external API
**Response:** "Fixed. Added null check at line 42."

### Example 2: Needs Clarification
**Feedback:** "Fix the error handling here"
**Issue:** Multiple error cases, unclear which one
**Response:** "Which error case? The timeout (line 30) or the parse failure (line 45)?"

### Example 3: Pushback Required
**Feedback:** "Use async/await instead of callbacks"
**Verify:** Current code uses Node streams API which requires callbacks
**Response:** "The stream API requires callbacks for backpressure handling. Wrapping in async/await would break flow control. See: [link to Node docs]"

### Example 4: Reviewer Lacks Context
**Feedback:** "Remove this seemingly dead code"
**Verify:** Code is called via dynamic dispatch from plugin system
**Response:** "This is called dynamically from plugin loader (see src/plugins/loader.ts:78). Added comment to clarify."

### Example 5: Conflicting Feedback
**Situation:** Reviewer A says "Extract this into separate function", Reviewer B says "Keep logic inline for performance"
**Wrong approach:** Implement A's feedback and ignore B, or vice versa
**Right approach:** "A and B have conflicting suggestions on line 45. A suggests extraction for readability, B suggests inline for performance. Profiling shows this code path is <1% of runtime. Proposing extraction with inline hint for compiler. Thoughts?"

### Example 6: Pushing Back with Evidence
**Feedback:** "This regex is too complex, simplify it"
**Verify:** Check test cases → Regex handles 12 edge cases from production bugs
**Response:** "This regex handles these production edge cases: [list]. Simplifying would break test cases X, Y, Z (see tests/parser.test.ts:45-67). Alternative: Extract to named function `parseComplexEmailFormat()` with documentation?"

### Example 7: Requesting Clarification
**Feedback:** "The error handling here is wrong"
**Issue:** Multiple error handling blocks, unclear which is "wrong"
**Wrong:** Rewrite all error handling defensively
**Right:** "Which error handling? The network timeout (line 30), JSON parse (line 45), or validation error (line 52)? Each handles different failure modes."

### Example 8: Valid but Low Priority
**Feedback:** "Consider using a factory pattern here"
**Verify:** Current code works, factory would add abstraction for single use case
**Response:** "Agree this could use factory pattern if we add more types. Current scope is just User and Admin. Created ticket #234 to revisit if we add third type. OK to defer?"

### Example 9: Architectural Disagreement
**Feedback:** "Move this business logic to the frontend for better UX"
**Verify:** Security policy requires server-side validation
**Response:** "Security policy requires server-side validation for PII (see docs/security-policy.md section 4.2). Can add optimistic UI update while awaiting server response. Would that address the UX concern?"

### Example 10: Feedback Reveals Misunderstanding
**Feedback:** "Why are you loading this data twice?"
**Verify:** Code loads once for display, once for background sync (different data subsets)
**Response:** "First load (line 30) is display data (metadata only). Second load (line 45) is background sync (full records). They're loading different fields from same endpoint. Should I rename variables to `loadMetadata()` and `syncFullRecords()` to clarify?"

## Escalation Triggers

| Condition | Action |
|-----------|--------|
| Reviewer insists on change that breaks tests | Demonstrate with test output, escalate if impasse |
| Conflicting feedback from multiple reviewers | Request sync discussion, don't implement contradictions |
| Feedback requires architecture change | Escalate to tech lead for decision |
| >10 items of feedback | Request prioritization before starting |

## Failure Behavior

If feedback cannot be implemented:
- State specifically what was attempted
- Show evidence of failure (test output, error messages)
- Propose alternative that achieves reviewer's intent
- Never silently skip feedback items

## Implementation Order

1. Clarify unclear items FIRST
2. Blocking issues (breaks, security)
3. Simple fixes (typos, imports)
4. Complex fixes (refactoring, logic)
5. Test each, verify no regressions

## Acknowledging Correct Feedback

```
"Fixed. [Brief description]"
"Good catch - [issue]. Fixed in [location]."
[Just fix it and show in the code]
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Performative agreement | State requirement or just act |
| Blind implementation | Verify against codebase first |
| Batch without testing | One at a time, test each |
| Avoiding pushback | Technical correctness > comfort |

**External feedback = suggestions to evaluate, not orders to follow.**

## Related Skills

- **code-smell-detection**: Apply smell detection to suggested changes
- **verification-before-completion**: Verify review feedback is addressed
- **systematic-debugging**: Debug issues raised in review

## Integration

- **verification-before-completion** skill - Verify each fix before claiming done
- **test-driven-development** skill - Write tests for fixes when appropriate
- **systematic-debugging** skill - When feedback reveals bugs to investigate
