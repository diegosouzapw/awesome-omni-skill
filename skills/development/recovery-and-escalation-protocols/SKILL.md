---
name: Recovery and Escalation Protocols
description: This skill should be used when encountering blockers, repeated failures (tests failing 3+ times), stuck states, error conditions, or degraded performance during any workflow phase. Provides systematic recovery procedures, escalation paths, rollback protocols, and debugging guidance to handle failures gracefully without compounding problems.
user-invocable: false
license: MIT
version: 0.9.0
metadata:
  author: Jason Hsieh
  category: recovery
  tags: [error-handling, recovery, debugging, escalation]
  documentation: https://github.com/jason-hchsieh/mycelium
---

# Recovery and Escalation Protocols

## Core Principle

**When things go wrong, follow explicit recovery paths rather than continuing blindly.**

This skill enforces systematic failure handling because unstructured debugging wastes time and compounds problems. Clear recovery protocols enable efficient problem resolution and prevent cascade failures.

## When to Use

Apply immediately when:
- 3+ fix attempts for same issue failed
- Tests pass but behavior is wrong
- Agent shows confusion or repeated questions
- User unavailable for blocking decision
- External dependency down or unavailable
- Task blocked by architectural issue
- Implementation growing beyond scope

Also use when:
- Error patterns emerge
- Time spent exceeds estimate by 2x
- Worktree state corrupted
- Git history problematic
- Need to abandon current approach

## Recovery Triggers and Actions

### Trigger 1: Three Failing Fix Attempts

**Symptoms:**
- Same test failing after 3+ fix attempts
- Each fix creates new problems
- Root cause unclear

**Action: ESCALATE to Architecture Review**

Why: After 3 failures, suspect design flaw not implementation error.

**Protocol:**
1. STOP attempting fixes
2. Document all attempts made
3. Identify suspected architectural issue
4. Present problem and request guidance:
   - "Attempted 3 fixes, all failed"
   - "Issue may be architectural, not implementation"
   - "Options: [A] redesign, [B] pivot approach, [C] simplify scope"
5. Wait for human decision
6. Do not proceed without direction

**Example:**
```markdown
## Escalation: Password Validation Failing

**Attempts:**
1. Added regex validation → rejected valid passwords
2. Changed regex → now too permissive
3. Switched to library → conflicts with other validators

**Suspected Issue:**
Validation architecture couples too many concerns.
May need separation of syntax vs. security validation.

**Requesting:** Architecture review of validation strategy
```

### Trigger 2: Tests Pass But Behavior Wrong

**Symptoms:**
- All tests green
- Manual verification shows incorrect behavior
- User reports it doesn't work as expected

**Action: QUESTION Test Validity**

Why: Tests may not cover actual requirement.

**Protocol:**
1. Review test assertions carefully
2. Compare tests to original requirements
3. Identify what's NOT being tested
4. Propose additional test cases
5. Add missing tests
6. Implement to pass new tests

**Example:**
```markdown
## Test Gap Identified

**Current Tests:** Verify email validation accepts/rejects format
**Missing Tests:** Don't verify case-insensitive matching
**Issue:** User@example.com and user@example.com treated as different

**Action:**
- Add test: "treats email as case-insensitive"
- Update implementation to lowercase before comparison
- Verify fix
```

### Trigger 3: Agent Confusion Detected

**Symptoms:**
- Asking same question twice
- Modifying wrong files
- Forgetting recent decisions
- Repeated context errors

**Action: SPAWN Fresh Agent with Compressed Context**

Why: Context corruption or overload causing errors.

**Protocol:**
1. Create context checkpoint
2. Compress session to progress.md (≤2000 tokens)
3. Document current task state
4. Spawn fresh agent
5. Fresh agent reads progress file
6. Continue with clean context

**Example:**
```markdown
## Context Reset Required

**Symptoms:**
- Asked about database schema twice
- Modified auth-service.ts instead of user-service.ts
- Context usage: 85%

**Action:**
Creating checkpoint and spawning fresh agent.
Progress file: .mycelium/progress.md
Resume at: Task 2.3 (password hashing)
```

### Trigger 4: User Unavailable for Blocking Decision

**Symptoms:**
- Task requires human decision
- User not responding
- Work cannot proceed

**Action: STASH and SWITCH to Unblocked Track**

Why: Maximize productivity while waiting.

**Protocol:**
1. Document the blocker clearly
2. Stash current work with descriptive message
3. Update task status to [!] (blocked)
4. Document blocker in plan file
5. Identify unblocked tasks
6. Switch to different task/track
7. Notify when switching back

**Example:**
```markdown
## Blocker: Database Migration Strategy

**Question:** Use online migration or downtime?
**Impact:** Affects task 3.2-3.4 (3 tasks blocked)
**Decision Required:** User approval for approach

**Action:**
- Stashed work: `git stash save "WIP: migration - awaiting decision"`
- Updated plan: Task 3.2 marked [!]
- Switching to: Task 4.1 (API documentation - unblocked)
- Will resume when decision received
```

### Trigger 5: External Dependency Down

**Symptoms:**
- API calls timing out
- Service unavailable
- Network errors
- Third-party service down

**Action: MOCK or DEFER**

Why: Can't control external services.

**Protocol:**
1. Identify affected tasks
2. Assess criticality
3. Choose path:
   - **Mock:** Create stub for testing
   - **Defer:** Mark blocked, continue other work
   - **Alternative:** Use backup service if available
4. Document dependency issue
5. Create test with mock
6. Mark as requiring real integration test later

**Example:**
```markdown
## External Dependency Issue: Payment API

**Service:** Stripe API
**Status:** Returning 503 errors
**Affected:** Tasks 5.1-5.3

**Action:**
- Created mock payment service for tests
- Tests pass with mock (12/12)
- Added TODO: integration test with real Stripe
- Marked in plan: "Requires real API test before merge"
- Continuing with mocked service
```

### Trigger 6: Test Suite Takes Too Long

**Symptoms:**
- Full test run > 5 minutes
- Blocking TDD cycle
- Slowing iteration

**Action: RUN Affected Tests Only**

Why: Optimize feedback loop during development.

**Protocol:**
1. Identify tests related to current work
2. Run only affected tests during development
3. Document that full suite pending
4. Run full suite at phase completion
5. Note in verification checklist

**Example:**
```markdown
## Test Strategy: Focused Runs

**Full Suite:** 342 tests, 8.5 minutes
**Affected:** Auth tests, 47 tests, 1.2 minutes

**During Development:**
- Run: `npm test -- auth.test.ts`
- Fast feedback: 1.2min vs 8.5min

**Before Completion:**
- Run full suite: `npm test`
- Verify no regressions
```

## Recovery Decision Tree

```
Problem Detected
    │
    ├─ First Attempt? → DEBUG
    │   • Read error message carefully
    │   • Check obvious issues
    │   • Fix and verify
    │   • Document fix
    │
    ├─ Second Attempt? → ANALYZE
    │   • Root cause investigation
    │   • Review related code
    │   • Check for patterns
    │   • Try systematic fix
    │
    ├─ Third Attempt? → PIVOT
    │   • Question approach
    │   • Consider alternatives
    │   • Try different strategy
    │   • Document why pivoting
    │
    └─ Fourth+ Attempt? → ESCALATE
        │
        ├─ Technical Complexity?
        │   → Request architecture review
        │   → Present problem + attempts
        │   → Suggest alternatives
        │
        ├─ Missing Information?
        │   → Ask user for clarification
        │   → Document what's unclear
        │   → Block until answered
        │
        ├─ Wrong Requirement?
        │   → Revisit Phase 1 (Clarify)
        │   → Verify understanding
        │   → Replan if needed
        │
        └─ Beyond Capability?
            → Document limitation
            → Ask for help
            → Suggest alternatives
```

## Systematic Debugging Process

When debugging (attempts 1-3):

### Phase 1: Reproduce
1. Create minimal reproduction case
2. Document exact steps to trigger
3. Verify reproduction consistent
4. Capture error messages completely

### Phase 2: Isolate
1. Remove unrelated code
2. Test in isolation
3. Identify minimal failing case
4. Verify issue persists

### Phase 3: Investigate
1. Use git bisect to find breaking commit
   ```bash
   git bisect start
   git bisect bad
   git bisect good <known-good-sha>
   # Test at each step
   ```
2. Review changes in breaking commit
3. Identify exact change causing issue
4. Understand why it breaks

### Phase 4: Hypothesize
1. Form hypothesis about root cause
2. Predict what fix should achieve
3. Identify test that would verify
4. Document hypothesis

### Phase 5: Fix and Verify
1. Implement targeted fix
2. Verify fix resolves issue
3. Verify no regressions introduced
4. Document fix and reasoning

## Blocker Types and Escalation

### Technical Blockers
**Examples:**
- Architectural limitation
- Performance issue beyond quick fix
- Security concern
- Complex algorithm needed

**Escalation Path:**
1. Document technical challenge
2. Present attempted solutions
3. Request architecture review or expert input
4. Suggest alternatives if any

### Clarification Blockers
**Examples:**
- Ambiguous requirements
- Conflicting specifications
- Unknown edge case handling
- Unclear success criteria

**Escalation Path:**
1. Document specific ambiguity
2. Present interpretation options
3. Ask specific questions
4. Wait for user clarification

### External Dependency Blockers
**Examples:**
- Third-party API down
- Library bug
- Infrastructure issue
- External team dependency

**Escalation Path:**
1. Document dependency and issue
2. Check for workarounds/alternatives
3. Mock if possible, defer if not
4. Continue unblocked work

### Resource Blockers
**Examples:**
- Missing credentials
- No access to system
- Missing documentation
- Tool not available

**Escalation Path:**
1. Document needed resource
2. Request access/provision
3. Switch to unblocked work
4. Resume when available

### Approval Blockers
**Examples:**
- Architecture decision needed
- Security policy unclear
- Business rule ambiguous
- Design approval needed

**Escalation Path:**
1. Present decision with options
2. Show pros/cons of each
3. Recommend approach (with reasoning)
4. Wait for approval

## Rollback Procedures

### Task-Level Rollback

When task fails irreparably:

```bash
# In worktree
git reset --hard origin/main
git clean -fd
# Start task fresh
```

### Phase-Level Rollback

When phase approach is wrong:

```bash
# Return to last phase checkpoint
git reset --hard <checkpoint-sha>
git clean -fd
# Replan phase
```

### Track-Level Rollback

When track is fundamentally flawed:

```bash
# Abandon worktree
cd <main-repo>
git worktree remove .worktrees/<track-id>
git branch -D <track-id>
# Document learnings
# Start new track with different approach
```

### Selective Rollback

When only specific changes need reverting:

```bash
# Revert specific commits
git revert <commit-sha>
# Or reset specific files
git checkout <good-sha> -- path/to/file.ts
```

## Track Abandonment Criteria

Abandon track when:
- 5+ tasks blocked by same fundamental issue
- Architecture review reveals incompatible approach
- Requirements changed significantly mid-track
- External blocker has no ETA
- Cost/effort exceeds estimate by 3x+

### Abandonment Protocol

1. **Document** why abandoning
   ```markdown
   ## Track Abandonment: user-auth_20260203

   **Reason:** JWT architecture incompatible with
   requirement for instant token revocation (discovered
   in task 5.2).

   **Completed Work:** Tasks 1.1-4.3 (15 tasks)
   **Abandoned:** Tasks 5.1-6.4 (8 tasks)

   **Learnings:** JWT stateless tokens can't be
   revoked without central store, defeating stateless
   benefit. Need session-based auth instead.
   ```

2. **Capture learnings** to `.mycelium/solutions/`
   - What worked
   - What didn't
   - Why abandoned
   - What to do differently

3. **Revert** to last known good state
   ```bash
   git checkout main
   git worktree remove .worktrees/user-auth_20260203
   git branch -D user-auth_20260203
   ```

4. **Archive plan** with [-] markers on remaining tasks
   ```markdown
   ### Task 5.1: Token revocation
   **Status:** [-] Abandoned - JWT limitation discovered
   ```

5. **Create new track** if work should continue
   - New plan with lessons applied
   - Different approach
   - Corrected architecture

## Error Pattern Recognition

Watch for recurring patterns:

### Pattern: Same Error in Multiple Tasks
**Indicates:** Systematic issue in approach
**Action:** Stop, review pattern, fix root cause once

### Pattern: Increasing Complexity
**Indicates:** Scope creep or wrong abstraction
**Action:** Revisit requirements, simplify

### Pattern: Test Fragility
**Indicates:** Over-mocking or testing implementation
**Action:** Refactor tests to test behavior

### Pattern: Merge Conflicts
**Indicates:** Poor task decomposition or timing
**Action:** Review dependency graph, resequence

## Recovery Metrics

Track recovery actions in session state:

```json
{
  "recovery_actions": [
    {
      "trigger": "3_failed_attempts",
      "action": "escalate",
      "task_id": "2.3",
      "timestamp": "2026-02-03T14:45:00Z",
      "resolution": "User provided alternative approach"
    },
    {
      "trigger": "external_dependency_down",
      "action": "mock",
      "task_id": "5.2",
      "timestamp": "2026-02-03T15:30:00Z",
      "resolution": "Created mock, deferred integration test"
    }
  ],
  "escalations": 2,
  "pivots": 1,
  "rollbacks": 0
}
```

## Integration with Workflow

**Phase 4: Implementation**
- Watch for trigger conditions
- Apply recovery protocols
- Document issues
- Escalate when needed

**Phase 4.5: Verification**
- If verification fails repeatedly → Recovery skill
- Systematic debugging process
- Escalate if architecture issue

**Phase 6: Learning**
- Capture recovery actions as solutions
- Document what went wrong
- Prevent recurrence

## Human-AI Boundaries

**AI Must Escalate:**
- Architecture decisions
- Security concerns
- Breaking changes
- Data schema changes
- Irreversible operations

**AI Can Handle:**
- Implementation bugs (first 3 attempts)
- Test fixes
- Refactoring
- Minor optimizations
- Code formatting

## Prevention vs. Recovery

Best recovery is prevention:

### Prevent Through
- Clear requirements (Phase 1)
- Detailed planning (Phase 3)
- TDD discipline (Phase 4)
- Evidence-based verification (Phase 4.5)
- Pattern learning (Phase 6)

### Recover When
- Prevention insufficient
- Unexpected conditions
- External factors
- Learning new patterns

## Summary

**Key principles:**
- After 3 failures, escalate (don't keep trying)
- Use systematic debugging (5 phases)
- Document all recovery actions
- Stash and switch when blocked
- Mock or defer external dependencies
- Abandon track when fundamentally flawed
- Learn from every failure
- Prevention better than recovery

**Recovery Actions:**
- `retry` - Try again with fix
- `escalate` - Request human help
- `pivot` - Change approach
- `rollback` - Revert changes
- `abandon` - Give up on track

Recovery protocols exist because problems are inevitable. Handle them systematically to minimize waste and maximize learning.

## References

- [`.mycelium/` directory structure][mycelium-dir]
- [Session state docs][session-state-docs]
- [Session state schema][session-state-schema]
- [Metrics schema][metrics-schema]
- [Enum definitions][enums]

[mycelium-dir]: ../../docs/mycelium-directory.md
[session-state-docs]: ../../docs/session-state.md
[session-state-schema]: ../../schemas/session-state.schema.json
[metrics-schema]: ../../schemas/metrics.schema.json
[enums]: ../../schemas/enums.json
