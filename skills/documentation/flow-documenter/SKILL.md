---
name: flow-documenter
description: Document findings and maintain task notes using Flow framework. Use when user says "document", "document this", "document finding", "add notes", "add this to notes", "write this down", "summarize", "summarize this", "generate changelog", "create changelog", or wants to capture discoveries. Helps update task Notes sections, create summaries with /flow-summarize, and keep documentation synchronized with work. Focuses on concise, actionable documentation.
---

# Flow Documenter

Help users document discoveries, maintain task notes, and generate summaries using Flow framework projects. This Skill ensures documentation stays synchronized with actual work and follows Flow patterns.

## When to Use This Skill

Activate when the user wants to document:
- "Document this finding"
- "Add notes to the task"
- "Write this down"
- "Summarize what we did"
- "Generate a changelog"
- "Update the task notes"
- "Record this discovery"

## Documentation Philosophy

**Keep it Actionable**: Documentation should help future work, not just record history.

**Where to Document**:
- **Task ## Notes**: Discoveries, decisions, patterns found during work
- **Implementation Notes**: What was built, why, challenges solved
- **DASHBOARD.md**: Phase/task completion summaries
- **PLAN.md**: Architecture decisions, DO/DON'T guidelines (via flow-architect)

**Slash Commands**:
- `/flow-summarize`: Generate project summary from all phases/tasks/iterations

## Task Notes Section

### What Goes in ## Notes

**Discoveries**:
- Patterns found during implementation
- Unexpected behaviors discovered
- Design insights that emerged
- Technical constraints learned

**Decisions**:
- Choices made and why
- Trade-offs accepted
- Approaches tried and rejected

**References**:
- Related tasks or iterations
- External documentation consulted
- Framework patterns followed

### Task Notes Template

```markdown
## Notes

**Key Discoveries**:
- [Discovery 1]: [What was found and why it matters]
- [Discovery 2]: [What was found and why it matters]

**Design Decisions**:
- [Decision 1]: Chose [approach] because [rationale]
- [Decision 2]: Deferred [feature] to V2 due to [constraint]

**Challenges & Solutions**:
- **Challenge**: [Problem encountered]
  - **Solution**: [How it was solved]
  - **Impact**: [Effect on design/implementation]

**References**:
- Related to [Task X, Iteration Y]
- Pattern from DEVELOPMENT_FRAMEWORK.md lines [X-Y]
- External: [URL or doc reference]
```

### Example Task Notes

```markdown
## Notes

**Key Discoveries**:
- **Stripe SDK Limitation**: v12 doesn't expose retry configuration hooks
  - Led to custom RetryPolicy implementation
  - See PLAN.md Technology Choices section for detailed rationale
- **Error Classification Pattern**: Transient vs permanent errors need different handling
  - Implemented ErrorMapper with strategy pattern
  - Allows easy extension for new error types

**Design Decisions**:
- Chose exponential backoff over linear for retry logic
  - Rationale: Better for API rate limiting (Stripe recommendation)
  - Added jitter to prevent thundering herd problem
- Deferred refund processing to V2
  - Rationale: Not needed for initial launch, adds complexity
  - Can be added later without refactoring payment flow

**Challenges & Solutions**:
- **Challenge**: Testing retry timing is difficult (delays slow down test suite)
  - **Solution**: Made clock injectable for testing (dependency injection)
  - **Impact**: Tests run fast, retry logic still thoroughly tested
- **Challenge**: Stripe test API has rate limit (100 req/hour)
  - **Solution**: Used mocks for most tests, real API for critical paths
  - **Impact**: Test suite completes in < 10 seconds

**References**:
- Related to Task 3, Iteration 2 (Error Handling)
- Pattern from DEVELOPMENT_FRAMEWORK.md lines 1798-1836 (Implementation Pattern)
- Stripe API Docs: https://stripe.com/docs/api/errors
```

## Bug Documentation

### When to Document Bugs

**In Task Notes** (if discovered during implementation):
- Bugs found in existing code (not your changes)
- Workarounds applied
- Issues deferred to future tasks

**Use Bug Documentation Template**:

```markdown
**Bugs Discovered**:

### Bug 1: [Short Description]

**Location**: `path/to/file.ts:lines X-Y`

**Problem**:
```[language]
// Current buggy code
```

**Impact**: [What breaks or degrades]

**Action Taken**: [Fixed immediately | Documented for future | Workaround applied]

**Fix** (if applied):
```[language]
// Corrected code
```

**Recommendation**: [Priority level and next steps]
```

### Example Bug Documentation

```markdown
**Bugs Discovered**:

### Bug 1: Race Condition in PaymentService

**Location**: `src/services/PaymentService.ts:145-152`

**Problem**:
```typescript
async processPayment(amount: number) {
  const status = await this.checkStatus();
  // Race condition: status can change between check and update
  await this.updatePayment(status);
}
```

**Impact**: Could cause duplicate charges in concurrent requests

**Action Taken**: Documented here, created separate task for fix (not in scope of current iteration)

**Recommendation**: High priority fix for next sprint - use database transactions
```

## Changelog Generation

### Using /flow-summarize

The `/flow-summarize` command generates summaries from PLAN.md structure:

```bash
/flow-summarize
```

**Outputs**:
- All phases with completion status
- All tasks with iteration counts
- High-level summary of what was accomplished

### Manual Changelog Format

For release notes or detailed changelogs:

```markdown
## Changelog - [Version/Date]

### Added
- [Feature 1]: [Brief description]
- [Feature 2]: [Brief description]

### Changed
- [Change 1]: [What changed and why]
- [Change 2]: [What changed and why]

### Fixed
- [Bug 1]: [What was fixed]
- [Bug 2]: [What was fixed]

### Deprecated
- [Old feature]: [Replacement or removal plan]
```

### Example Changelog

```markdown
## Changelog - Payment Integration V1

### Added
- Payment processing via Stripe API
  - Credit card charges (Visa, Mastercard, Amex)
  - Retry logic with exponential backoff
  - Error classification (transient vs permanent)
- Transaction logging for audit trail
- Error handling with user-friendly messages

### Changed
- Refactored PaymentService to use dependency injection
  - Improves testability
  - Allows easy swapping of payment providers
- Updated error responses to include retry guidance

### Fixed
- Race condition in payment status checks
  - Now uses database transactions
  - Prevents duplicate charge scenarios

### Deferred to V2
- Refund processing
- Subscription billing
- Multi-currency support
```

## DASHBOARD Updates

### When to Update DASHBOARD

**Task completion**:
- Mark task ✅ COMPLETE
- Update completion percentages
- Update "📍 Current Work" section

**Phase completion**:
- Mark phase ✅ COMPLETE
- Add phase summary
- Update overall project status

### DASHBOARD Update Pattern

```markdown
## 📍 Current Work

- **Phase**: [Current Phase Name](phase-X/)
- **Task**: [Current Task Name](phase-X/task-Y.md)
- **Status**: [Brief status summary]
- **Next**: [What to do next]

---

## 📊 Progress Overview

### Phase X: [Phase Name] ✅ COMPLETE

**Summary**: [1-2 sentence summary of what was accomplished]

**Key Deliverables**:
- [Deliverable 1]
- [Deliverable 2]

**Tasks**:
- ✅ **Task 1**: [Name] (X/Y iterations)
- ✅ **Task 2**: [Name] (X/Y iterations)
```

## Pre-Implementation Task Documentation

### Pre-Task Notes Pattern

When documenting completed pre-implementation tasks:

```markdown
#### Pre-Implementation Tasks

##### ✅ Pre-Task 1: [Name]

**Completed**: 2025-01-15

**Why Blocking**: [Explanation of why this had to be done first]

**Changes Made**:
- [Change 1]: [Description]
- [Change 2]: [Description]
- [Change 3]: [Description]

**Files Modified**:
- `path/to/file1.ts` (+X lines, -Y lines)
- `path/to/file2.ts` (+X lines, -Y lines)

**Impact on Main Iteration**:
[How this pre-task enables or simplifies the main work]
```

## Discovery Documentation

### When You Discover Something Important

**Document immediately** in task notes if it's:
- A pattern that should be followed consistently
- A constraint that affects future work
- An insight that changes understanding
- A decision that future developers need to know

### Discovery Template

```markdown
**Discovery: [Short Title]**

**Context**: [When/how this was discovered]

**What We Learned**:
[Detailed explanation of the discovery]

**Implications**:
- [Impact 1]: [How this affects current or future work]
- [Impact 2]: [How this affects current or future work]

**Action Taken**:
- [Action 1]: [What was done in response]
- [Action 2]: [What was done in response]
```

### Example Discovery

```markdown
**Discovery: Stripe Webhook Signatures Expire After 5 Minutes**

**Context**: While implementing webhook endpoint, discovered signature validation fails for delayed webhooks

**What We Learned**:
Stripe webhook signatures include a timestamp and expire after 5 minutes to prevent replay attacks. If webhook processing is delayed (queue backlog, system downtime), validation will fail even for legitimate webhooks.

**Implications**:
- **Current Work**: Need to capture raw webhook payload before validation for debugging
- **Future Work**: V2 webhook processing must handle signature expiration gracefully
- **Monitoring**: Add alerts for webhook validation failures

**Action Taken**:
- Documented in PLAN.md Scope section (V1 assumes < 5min processing)
- Added pre-validation logging of raw payload
- Created V2 task for robust webhook handling
```

## Interaction with Other Flow Skills

**Planning Stage** (flow-planner Skill):
- Planner creates structure
- Documenter captures decisions made

**Architecture Stage** (flow-architect Skill):
- Architect updates PLAN.md
- Documenter adds task-specific notes

**Implementation Stage** (flow-implementer Skill):
- Implementer executes work
- Documenter records discoveries ← YOU ARE HERE

**Review Stage** (flow-reviewer Skill):
- Reviewer validates consistency
- Documenter updates based on findings

## References

- **Task Structure**: DEVELOPMENT_FRAMEWORK.md lines 238-566
- **Implementation Pattern**: DEVELOPMENT_FRAMEWORK.md lines 1798-1836
- **Status Markers**: DEVELOPMENT_FRAMEWORK.md lines 1872-1968
- **Slash Command**: `/flow-summarize` (generate summaries)

## Key Reminders

**Before documenting**:
- [ ] Identify correct location (Task Notes, DASHBOARD, PLAN.md)
- [ ] Keep it concise and actionable
- [ ] Focus on "why" not just "what"

**During documentation**:
- [ ] Use templates for consistency
- [ ] Link to related tasks/iterations
- [ ] Include impact/implications

**After documenting**:
- [ ] Verify documentation is findable (proper section, clear title)
- [ ] Check if DASHBOARD needs updating
- [ ] Consider if discovery should go in PLAN.md (via flow-architect)
