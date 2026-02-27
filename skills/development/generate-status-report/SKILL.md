---
name: generate-status-report
description: Comprehensive system status report with services, infrastructure, performance metrics, and recommendations
---

# Generate Status Report

Generate comprehensive system status report with complete metrics and analysis.

## What It Generates

- Complete system overview
- All services status
- Infrastructure health
- Performance metrics (1hr, 24hr, 7d)
- Recent activity summary
- Top agents and patterns
- Issues and recommendations
- Trend analysis
- Export formats: JSON, Markdown, HTML

## When to Use

- **Daily reports**: Automated daily status reports
- **Weekly reviews**: Comprehensive system review
- **Post-incident**: Document system state
- **Audits**: Compliance and audit documentation
- **Stakeholder updates**: Share system health status

## How to Use

```bash
# Generate JSON report
python3 ${CLAUDE_PLUGIN_ROOT}/skills/system-status/generate-status-report/execute.py

# Generate Markdown report
python3 ${CLAUDE_PLUGIN_ROOT}/skills/system-status/generate-status-report/execute.py \
  --format markdown \
  --output /tmp/system-status-report.md

# Include detailed trends
python3 ${CLAUDE_PLUGIN_ROOT}/skills/system-status/generate-status-report/execute.py \
  --include-trends \
  --timeframe 7d
```

### Arguments

#### `--format`
Output format: `json`, `markdown`, or `text` [default: `json`]
- **json**: Structured JSON output for programmatic processing
- **markdown**: Human-readable Markdown with tables and formatting
- **text**: Plain text format (uses markdown as base)

#### `--output`
Output file path (stdout if not specified)
- If provided, report is written to file
- If omitted, report is printed to stdout

#### `--include-trends`
Enable historical trend analysis comparing current period with previous period

**What it provides**:
- Routing decisions change percentage
- Average routing time change percentage
- Average confidence change percentage
- Comparison with equivalent previous timeframe

**Example output**:
```markdown
## Trends Analysis

**Comparison with Previous Period**:
- **Routing Decisions**: +15.3% change
- **Average Routing Time**: -8.2% change (improvement)
- **Average Confidence**: +2.1% change

*Positive values indicate increase, negative values indicate decrease*
```

**Performance note**: Adds ~100-200ms to report generation time due to additional database query.

#### `--timeframe`
Data collection period: `1h`, `24h`, or `7d` [default: `24h`]
- `1h`: Last 1 hour (recent activity snapshot)
- `24h`: Last 24 hours (daily overview)
- `7d`: Last 7 days (weekly trends)

**Note**: When using `--include-trends`, the comparison is with the equivalent previous period (e.g., `24h` compares last 24h vs previous 24h)

## Output Sections

1. **Executive Summary** - Overall system health and key metrics
2. **Service Health Matrix** - All services with status indicators
3. **Infrastructure Status** - Kafka, PostgreSQL, Qdrant health
4. **Performance Metrics** - Routing times, query latencies, throughput
5. **Recent Activity** - Agent executions, routing decisions, actions
6. **Top Agents** - Most frequently routed agents
7. **Pattern Discovery** - Collection stats and vector counts
8. **Issues & Recommendations** - Problems found and suggestions
9. **Trends** (optional) - Historical comparison and trends

## Example Markdown Output

```markdown
# System Status Report

**Generated**: 2025-11-12T14:30:00Z
**Timeframe**: 24 hours

## Executive Summary

- **System Health**: Healthy
- **Services Running**: 12/12
- **Agent Executions**: 452
- **Routing Decisions**: 890
- **Average Confidence**: 89%

## Service Health

| Service | Status | Uptime |
|---------|--------|--------|
| archon-intelligence | ✓ Healthy | 5d 3h |
| archon-qdrant | ✓ Healthy | 5d 3h |
...

## Issues

No critical issues found.

## Recommendations

1. Consider increasing Qdrant collection size
2. Monitor connection pool usage
```

## See Also

- **check-system-health** - Quick health check
- **diagnose-issues** - Detailed issue diagnosis
