---
name: spec-review
description: "How to verify implementation against OpenSpec artifacts"
---

# OpenSpec Verification

Guide for verifying that implementation matches spec artifacts.

## Input Files

| File | Required | Purpose |
|------|----------|---------|
| `{specs_path}/design.md` | Yes | Requirements and scenarios to verify |
| `{specs_path}/tasks.md` | Yes | Completion checklist |
| `{specs_path}/proposal.md` | Optional | Original intent reference |

## Three Verification Dimensions

### 1. Completeness

**Question**: Are all tasks done and all requirements covered?

| Check | Method | Issue Level |
|-------|--------|-------------|
| All checkboxes marked `[x]` | Parse tasks.md | CRITICAL if incomplete |
| All requirements have code | Search codebase for keywords | CRITICAL if missing |
| All new files exist | Verify file paths from tasks | CRITICAL if missing |

### 2. Correctness

**Question**: Does the code do what the spec says?

| Check | Method | Issue Level |
|-------|--------|-------------|
| GIVEN/WHEN/THEN satisfied | Trace scenario through code | WARNING if divergent |
| Tests cover scenarios | Match test names to scenarios | WARNING if uncovered |
| Edge cases handled | Check error paths in code | WARNING if missing |
| Validation commands pass | Run pytest / npm run validate | CRITICAL if failing |

### 3. Coherence

**Question**: Does the code match design decisions?

| Check | Method | Issue Level |
|-------|--------|-------------|
| Design decisions followed | Compare Decisions section to code | WARNING if violated |
| Patterns consistent | Check naming, structure, style | SUGGESTION |
| No undocumented changes | Diff scope vs design scope | WARNING if extra |
| No design deviations | Cross-reference architecture | WARNING if different |

## Verification Process

1. **Load artifacts** - Read design.md, tasks.md, proposal.md
2. **Check completeness** - Parse checkboxes, search for requirement implementations
3. **Check correctness** - Trace each scenario through code, verify test coverage
4. **Check coherence** - Compare decisions to implementation, check patterns
5. **Generate report** - Summarize findings with issue levels

## Report Format

```markdown
## Verification Report

### Summary
| Dimension    | Status              |
|--------------|---------------------|
| Completeness | X/Y tasks, N reqs   |
| Correctness  | M/N scenarios pass   |
| Coherence    | Followed / N issues  |

### Critical Issues
| # | Dimension | Issue | File | Recommendation |
|---|-----------|-------|------|----------------|
| 1 | Completeness | Task 2.3 incomplete | - | Complete or mark blocked |

### Warnings
| # | Dimension | Issue | File | Recommendation |
|---|-----------|-------|------|----------------|
| 1 | Correctness | Scenario X not tested | test_foo.py | Add test case |

### Suggestions
- [Pattern deviation details with file reference]

### Assessment
[CRITICAL: N issues | WARNINGS: N | Ready for archive: Yes/No]
```

## Graceful Degradation

| Available Artifacts | Checks Performed |
|--------------------|-----------------|
| tasks.md only | Completeness (checkboxes) only |
| tasks.md + design.md | Completeness + Correctness |
| All three | All three dimensions |

Always note which checks were skipped and why.

## Verification Heuristics

- **Completeness**: Focus on objective items (checkboxes, requirement lists)
- **Correctness**: Use keyword search + file path analysis; don't require certainty
- **Coherence**: Look for glaring inconsistencies, don't nitpick style
- **False positives**: Prefer SUGGESTION over WARNING, WARNING over CRITICAL when uncertain
- **Actionability**: Every issue must have a specific recommendation with file references
