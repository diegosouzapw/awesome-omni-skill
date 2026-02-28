---
name: Coverage Ratcheting
description: Enforce that test coverage never decreases — compare current coverage against baseline, flag regressions, and track per-package coverage trends
category: testing
version: 1.0.0
triggers:
  - test-run
  - coverage-report
  - pr-created
globs: "**/*_test.go,**/coverage*"
---

# Coverage Ratcheting Skill

Enforce a coverage ratchet: coverage for each package must never decrease from its high-water mark.

## Trigger Conditions
- Test suite run completes with coverage data
- PR is submitted for review
- Coverage thresholds are checked in CI
- User invokes with "coverage check" or "coverage-ratcheting"

## Input Contract
- **Required:** Current coverage data (from `go test -cover`)
- **Optional:** Previous coverage baseline for comparison

## Output Contract
- Per-package coverage percentages
- Coverage delta from baseline (improved / unchanged / regressed)
- Packages below minimum threshold per rule 123
- Ratchet violations (packages that decreased from high-water mark)
- Recommended test additions to close gaps

## Tool Permissions
- **Read:** Test files, coverage reports, source files
- **Write:** Coverage baseline file (if updating)
- **Shell:** Run `go test -coverprofile`
- **Search:** Grep for untested functions

## Execution Steps

1. **Run tests with coverage**: Execute `go test -coverprofile=coverage.out ./...`
2. **Parse coverage**: Extract per-package and per-function coverage
3. **Compare to baseline**: Check each package against its previous high-water mark
4. **Check thresholds**: Verify each package meets minimum thresholds per rule 123
5. **Identify regressions**: Flag any package where coverage decreased
6. **Recommend**: For packages below threshold, identify the uncovered functions
7. **Update baseline**: If coverage improved, update the high-water mark

## Minimum Thresholds (from rule 123)
- `internal/service/` — 90%
- `internal/repository/` — 85%
- `internal/handler/` — 80%
- `internal/middleware/` — 80%
- `internal/models/` — 75%

## References
- `.cursor/rules/123-test-requirement-matrix.mdc`
