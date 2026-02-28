---
name: admin-dashboard-qa
description: Use this skill when implementing, modifying, or fixing the admin dashboard (admin-dashboard-v2). Triggers for tasks involving dashboard UI, components, pages, features, hooks, or API integration. Orchestrates a rigorous QA workflow with PM review, use case writing, testing, and bug fixing cycles.
---

# Admin Dashboard QA Workflow

This skill implements a rigorous multi-agent testing workflow for admin dashboard frontends. It automatically adapts to whichever dashboard version you're working on.

## Auto-Detection

When this skill activates, first determine which dashboard you're working on:

1. **Check the task context** - Does it mention a specific version?
2. **Check file paths** - Which package directory is involved?
3. **Default to active version** - Currently `admin-dashboard-v2`

**Dashboard Packages:**
- `packages/admin-dashboard-v2` - Current production version
- Any future `packages/admin-dashboard-*` directories

## Adaptive Configuration

Before starting, detect the dashboard's test setup:

```bash
# Find the correct package
DASHBOARD_DIR="packages/admin-dashboard-v2"  # or detected version

# Check for test config
ls $DASHBOARD_DIR/vitest.config.ts   # Vitest
ls $DASHBOARD_DIR/jest.config.js     # Jest
ls $DASHBOARD_DIR/package.json       # Check test scripts
```

Then adapt commands accordingly:
- **Vitest:** `pnpm test`, `pnpm test:coverage`
- **Jest:** `pnpm test`, `pnpm test -- --coverage`
- **Other:** Check `package.json` scripts

## Workflow Overview

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  PROJECT MANAGER │────▶│  USE CASE WRITER │────▶│     TESTER      │
│  (Reviews scope) │     │ (Defines tests)  │     │ (Executes tests)│
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                          │
                        ┌──────────────────┐              │
                        │    DEVELOPER     │◀─────────────┘
                        │   (Fixes bugs)   │
                        └──────────────────┘
                                │
                                ▼
                        ┌──────────────────┐
                        │  FULL REGRESSION │
                        │      TEST        │
                        └──────────────────┘
```

## Phase 1: Project Manager Review

Before any implementation begins:
1. Review the feature request/task description
2. Define acceptance criteria
3. Identify affected components
4. Estimate risk level (low/medium/high)
5. Create implementation checklist

Output a PM Summary like:
```markdown
## PM Summary: [Feature Name]
**Risk Level:** [low/medium/high]
**Affected Components:** [list]
**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2
**Implementation Checklist:**
- [ ] Step 1
- [ ] Step 2
```

## Phase 2: Use Case Writer

Before testing, define comprehensive use cases by consulting `TESTING-MANUAL.md`:
1. Identify all user flows affected
2. Write test cases for happy paths
3. Write test cases for edge cases
4. Write test cases for error states
5. Define expected outcomes

Format each use case as:
```markdown
### UC-[ID]: [Use Case Name]
**Preconditions:** [Setup required]
**Steps:**
1. Step 1
2. Step 2
**Expected Result:** [What should happen]
**Test Type:** [unit/integration/e2e/manual]
```

## Phase 3: Developer Implementation

Implement the feature following:
1. Write unit tests FIRST (TDD when possible)
2. Implement the feature
3. Run `pnpm test` to verify unit tests pass
4. Self-review code for common issues
5. Hand off to Tester

## Phase 4: Tester Execution

Execute all defined use cases:
1. Run automated tests: `cd packages/admin-dashboard-v2 && pnpm test`
2. Check coverage: `pnpm test:coverage`
3. Execute manual test cases if needed
4. Document any failures with:
   - Steps to reproduce
   - Expected vs actual behavior
   - Screenshot/error message if applicable
5. Report bugs to Developer

**Test Report Format:**
```markdown
## Test Report: [Feature Name]
**Date:** [YYYY-MM-DD]
**Tester:** Claude AI

### Automated Tests
- Total: X
- Passed: X
- Failed: X
- Coverage: X%

### Use Case Results
| UC ID | Description | Status | Notes |
|-------|-------------|--------|-------|
| UC-01 | ... | PASS/FAIL | ... |

### Bugs Found
1. BUG-001: [Description]
   - Severity: [critical/high/medium/low]
   - Steps to reproduce: ...
```

## Phase 5: Bug Fix Cycle

For each bug found:
1. Developer analyzes root cause
2. Developer implements fix
3. Developer writes regression test
4. Tester re-verifies the specific use case
5. Repeat until all bugs fixed

## Phase 6: Full Regression Test

After all bugs are fixed:
1. Run complete test suite: `pnpm test`
2. Verify coverage thresholds met (95% lines, 90% branches)
3. Re-execute ALL use cases from Testing Manual for affected features
4. Generate final test report

**IMPORTANT:** Only mark feature as complete when:
- [ ] All unit tests pass
- [ ] Coverage thresholds met
- [ ] All defined use cases pass
- [ ] No open bugs remain

## Commands

Replace `$DASHBOARD` with the detected dashboard directory (e.g., `admin-dashboard-v2`):

```bash
# Run all tests
cd planted-availability-db/packages/$DASHBOARD && pnpm test

# Run tests with coverage
cd planted-availability-db/packages/$DASHBOARD && pnpm test:coverage

# Run specific test file
cd planted-availability-db/packages/$DASHBOARD && pnpm test [path/to/test.tsx]

# Run tests in watch mode
cd planted-availability-db/packages/$DASHBOARD && pnpm test:watch

# Build check
cd planted-availability-db/packages/$DASHBOARD && pnpm build

# Lint check
cd planted-availability-db/packages/$DASHBOARD && pnpm lint
```

## Feature Discovery

When working on a NEW feature not in the Testing Manual:

1. **Explore the feature area:**
   ```bash
   # Find related components
   ls planted-availability-db/packages/$DASHBOARD/src/features/
   ls planted-availability-db/packages/$DASHBOARD/src/pages/
   ```

2. **Find existing tests as patterns:**
   ```bash
   # List test files for similar features
   find planted-availability-db/packages/$DASHBOARD -name "*.test.tsx" -o -name "*.test.ts"
   ```

3. **Define NEW use cases** following the format in TESTING-MANUAL.md

4. **Add new use cases to TESTING-MANUAL.md** for future regression testing

## Reference Documents

- `TESTING-MANUAL.md` - Test case library (expand as needed)
- `TEST-REPORT-TEMPLATE.md` - Template for test reports
- Dashboard's `vitest.config.ts` or `jest.config.js` - Test configuration
- Dashboard's `package.json` - Available scripts
