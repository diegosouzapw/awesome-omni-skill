---
name: agentic-kpi-tracking
description: Track and measure agentic coding KPIs for ZTE progression. Use when measuring workflow effectiveness, tracking Size/Attempts/Streak/Presence metrics, or assessing readiness for autonomous operation.
allowed-tools: Read, Grep, Glob
---

# Agentic KPI Tracking Skill

Guide measurement and tracking of agentic coding KPIs to assess ZTE readiness.

## When to Use

- Measuring agentic workflow effectiveness
- Tracking progress toward ZTE
- Analyzing success patterns
- Identifying improvement areas

## Core KPIs

### Summary Metrics

| Metric | Calculation | Target |
| --- | --- | --- |
| **Current Streak** | Consecutive successes (Attempts <= 2) | Higher is better |
| **Longest Streak** | Best consecutive success run | Track improvement |
| **Average Presence** | Mean attempts across all runs | Target: 1 |
| **Total Plan Size** | Sum of all plan sizes | Track scaling |
| **Total Diff Size** | Sum of all changes (added + removed) | Track throughput |

### Per-Run Metrics

| Metric | Source | Meaning |
| --- | --- | --- |
| **Attempts** | Count of plan/patch runs | 1 = perfect, higher = retries |
| **Plan Size** | Lines in plan file | Task complexity |
| **Diff Size** | Lines added + removed | Change magnitude |
| **Files Changed** | Number of files modified | Change scope |

## Calculation Methods

### Attempts Count

Only count workflow restarts:

```python
attempts_incrementing = ["adw_plan_iso", "adw_patch_iso"]
attempts = count(workflow in all_adws if workflow in attempts_incrementing)
```

Build/test/review don't increment - only full replans.

### Streak Calculation

```python
current_streak = 0
for run in reversed(runs):
    if run.attempts <= 2:
        current_streak += 1
    else:
        break
```

### Diff Statistics

```bash
git diff origin/main --shortstat
# Output: X files changed, Y insertions(+), Z deletions(-)
```

## KPI File Format

Store in `app_docs/agentic_kpis.md` or equivalent:

```markdown
# Agentic KPIs

## Summary

| Metric | Value |
| --- | --- |
| Current Streak | 5 |
| Longest Streak | 12 |
| Average Presence | 1.3 |
| Total Plan Size | 450 lines |
| Total Diff Size | 2,340 lines |

## Detail

| Date | ADW ID | Issue | Class | Attempts | Plan Size | Diff +/- | Files |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2024-01-15 | abc123 | #45 | /bug | 1 | 35 | +45/-12 | 3 |
| 2024-01-14 | def456 | #44 | /feature | 2 | 85 | +120/-30 | 8 |
```

## Tracking Workflow

### Step 1: Gather Current Run Data

From state or git:

- ADW ID
- Issue number
- Issue classification
- Plan file path
- All workflows run (for attempts)

### Step 2: Calculate Metrics

```python
attempts = count_attempts(all_adws)
plan_size = wc_lines(plan_file)
diff_stats = parse_git_diff()
```

### Step 3: Update Detail Table

Add new row with current run data.

### Step 4: Recalculate Summary

Update all summary metrics based on full detail table.

### Step 5: Analyze Trends

- Is streak increasing?
- Is average presence decreasing?
- Are plan sizes growing (handling bigger tasks)?

## ZTE Readiness Indicators

Based on KPIs, assess ZTE readiness:

| Indicator | Threshold | Status |
| --- | --- | --- |
| Current Streak | >= 5 | Ready to try ZTE |
| Average Presence | <= 1.5 | Good efficiency |
| Recent Failures | 0 in last 10 | High confidence |
| Plan Size Trend | Increasing | Scaling up |

## Key Memory References

- @agentic-kpis.md - KPI definitions from Lesson 002
- @zte-progression.md - How KPIs relate to ZTE levels
- @zte-confidence-building.md - Using KPIs for confidence

## Output Format

Provide KPI update:

```markdown
## KPI Update

**Run:** {adw_id}
**Issue:** #{issue_number} ({issue_class})

### This Run
- Attempts: 1
- Plan Size: 45 lines
- Diff: +67/-23 (4 files)

### Updated Summary
- Current Streak: 6 (was 5)
- Longest Streak: 12 (unchanged)
- Average Presence: 1.28 (improved)

### Analysis
[Trend observations and recommendations]
```

## Anti-Patterns

- Gaming metrics (easy tasks only)
- Ignoring failures (not counting retries)
- Not tracking consistently
- Celebrating streaks over actual delivery

## Version History

- **v1.0.0** (2025-12-26): Initial release

---

## Last Updated

**Date:** 2025-12-26
**Model:** claude-opus-4-5-20251101
