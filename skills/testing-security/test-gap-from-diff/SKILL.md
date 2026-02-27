---
name: Test Gap from Diff
description: Given a code diff, identify exactly which tests are missing — map changed functions to required test types using the test requirement matrix
category: testing
version: 1.0.0
triggers:
  - code-change-without-test
  - pr-created
  - coverage-drop
globs: "**/*.go"
---

# Test Gap from Diff Skill

Analyze a code diff to identify exactly which tests need to be written, using the test requirement matrix (rule 123) to determine required test types.

## Trigger Conditions
- Source code changes without corresponding test changes
- PR is created for review
- Coverage drops below threshold
- User invokes with "test gaps" or "test-gap-from-diff"

## Input Contract
- **Required:** Git diff or list of changed files
- **Optional:** Current coverage report

## Output Contract
- Per-function list of missing tests with required test type
- Priority ranking (critical path functions first)
- Test skeleton code for top-priority gaps
- Coverage delta estimate if tests are added

## Tool Permissions
- **Read:** Source files, test files, coverage reports
- **Write:** None (analysis only; test-engineer writes the tests)
- **Search:** Grep for function signatures, test file patterns

## Execution Steps

1. **Parse diff**: Extract list of changed functions/methods with file paths
2. **Classify changes**: Categorize each change per rule 123 matrix (model, service, handler, etc.)
3. **Determine required tests**: Map each change type to required test types (unit, integration, property, concurrency, fuzz)
4. **Check existing coverage**: For each changed function, check if required test types already exist
5. **Identify gaps**: List functions where required tests are missing
6. **Prioritize**: Rank gaps by: financial operations first, then public API, then internal logic
7. **Report**: Produce gap report with test type, priority, and estimated effort

## Success Criteria
- All changed functions mapped to required test types
- All existing test coverage identified
- Clear gap report with actionable items

## References
- `.cursor/rules/123-test-requirement-matrix.mdc`
