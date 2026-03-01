---
name: analyzing-tdigest-metrics
description: Analyze percentile metrics (tdigest type) using OPAL for latency analysis and SLO tracking. Use when calculating p50, p95, p99 from pre-aggregated duration or latency metrics. Covers the critical double-combine pattern with align + m_tdigest() + tdigest_combine + aggregate. For simple metrics (counts, averages), see aggregating-gauge-metrics skill.
---

# Analyzing TDigest Metrics

TDigest metrics in Observe store pre-aggregated percentile data for efficient latency and duration analysis. This skill teaches the specialized pattern for querying tdigest metrics using OPAL.

## When to Use This Skill

- Calculating latency percentiles (p50, p95, p99) for services or endpoints
- Analyzing request duration distributions
- Setting or tracking SLOs (Service Level Objectives) based on percentiles
- Understanding performance characteristics beyond simple averages
- Working with any metric of type `tdigest`
- When you need accurate percentile calculations from pre-aggregated data

## Prerequisites

- Access to Observe tenant via MCP
- Understanding that tdigest metrics are pre-aggregated percentile structures
- Metric dataset with type: `tdigest`
- Familiarity with percentiles (p50 = median, p95 = 95th percentile, etc.)
- Use `discover_context()` to find and inspect tdigest metrics

## Key Concepts

### What Are TDigest Metrics?

**TDigest** (t-digest) is a probabilistic data structure for estimating percentiles efficiently:

**Pre-aggregated percentile data**: Not raw values, but compressed statistical summaries
- Stores distribution information in compact form
- Enables accurate percentile calculations
- Much more efficient than storing all raw values

**Why percentiles matter**:
- **Averages hide outliers**: A service with avg 100ms might have p99 at 10 seconds
- **SLOs use percentiles**: "p95 latency < 500ms" is a common SLO target
- **User experience**: p95/p99 show what real users experience, not just average case

**Common Examples**:
- `span_sn_service_node_duration_tdigest_5m` - Service-to-service latency percentiles
- `span_sn_service_edge_duration_tdigest_5m` - Edge latency percentiles
- `request_duration_tdigest_5m` - Request duration percentiles
- `database_query_duration_tdigest_5m` - Database query latency percentiles

### CRITICAL: The Double-Combine Pattern

TDigest metrics require a **special pattern** that's different from gauge metrics:

```opal
# WRONG - Missing second combine ❌
align options(bins: 1), combined:tdigest_combine(m_tdigest("metric"))
aggregate p95:tdigest_quantile(combined, 0.95)

# CORRECT - Double-combine pattern ✅
align options(bins: 1), combined:tdigest_combine(m_tdigest("metric"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```

**Why the double combine?**
1. **First `tdigest_combine`** (in `align`): Combines tdigest data points within time buckets
2. **Second `tdigest_combine`** (in `aggregate`): Re-combines tdigests across groups/dimensions
3. **Then `tdigest_quantile`**: Calculates the actual percentile value

**Pattern breakdown**:
```opal
align options(bins: 1),
      combined:tdigest_combine(m_tdigest("metric_name"))  ← First combine
aggregate p95:tdigest_quantile(
                tdigest_combine(combined),                ← Second combine (NESTED!)
                0.95),                                    ← Quantile value (0.0-1.0)
          group_by(service_name)
```

### Percentile Values

Percentiles are specified as decimal values from 0.0 to 1.0:

| Percentile | Value | Meaning |
|------------|-------|---------|
| p50 (median) | 0.50 | 50% of values are below this |
| p75 | 0.75 | 75% of values are below this |
| p90 | 0.90 | 90% of values are below this |
| p95 | 0.95 | 95% of values are below this |
| p99 | 0.99 | 99% of values are below this |
| p99.9 | 0.999 | 99.9% of values are below this |

**Common SLO targets**: p95 < 500ms, p99 < 1000ms

### Summary vs Time-Series (Same as Gauge Metrics)

| Output Type | Pattern | Result | Pipe? |
|-------------|---------|--------|-------|
| **Summary** | `options(bins: 1)` | One row per group | NO `\|` |
| **Time-Series** | `5m`, `1h` | Many rows per group | YES `\|` |

## Discovery Workflow

**Step 1: Search for tdigest metrics**
```
discover_context("duration tdigest", result_type="metric")
discover_context("latency percentile", result_type="metric")
```

**Step 2: Get detailed metric schema**
```
discover_context(metric_name="span_sn_service_node_duration_tdigest_5m")
```

**Step 3: Verify metric type**
Look for: `Type: tdigest` (critical!)

**Step 4: Note available dimensions**
Used for `group_by()`:
- `service_name`, `for_service_name`
- `environment`, `for_environment`
- etc. (shown in discovery output)

**Step 5: Write query**
Use double-combine pattern with correct dimensions

## Basic Patterns

### Pattern 1: Overall Percentiles (No Grouping)

Calculate percentiles across all data:

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p50:tdigest_quantile(tdigest_combine(combined), 0.50),
          p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          p99:tdigest_quantile(tdigest_combine(combined), 0.99)
```

**Output**: Single row with overall p50, p95, p99 across entire time range.

**Note**: Both combines present, no `group_by`.

### Pattern 2: Percentiles Per Service

Calculate percentiles broken down by dimension:

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p50:tdigest_quantile(tdigest_combine(combined), 0.50),
          p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          p99:tdigest_quantile(tdigest_combine(combined), 0.99),
          group_by(service_name)
```

**Output**: One row per service with percentiles.

### Pattern 3: Single Percentile (Common for SLOs)

Get just p95 for SLO tracking:

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          group_by(service_name)
| sort desc(p95)
| limit 10
```

**Output**: Top 10 services by p95 latency.

**Use case**: Identify slowest services for optimization.

### Pattern 4: Converting Units

TDigest values are often in nanoseconds - convert for readability:

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p50_ns:tdigest_quantile(tdigest_combine(combined), 0.50),
          p95_ns:tdigest_quantile(tdigest_combine(combined), 0.95),
          p99_ns:tdigest_quantile(tdigest_combine(combined), 0.99),
          group_by(service_name)
| make_col p50_ms:p50_ns / 1000000,
          p95_ms:p95_ns / 1000000,
          p99_ms:p99_ns / 1000000
```

**Output**: Percentiles in both nanoseconds and milliseconds.

**Note**: Check sample values in `discover_context()` to identify units.

### Pattern 5: Time-Series Percentiles

Track percentiles over time buckets:

```opal
align 5m, combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
| aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
            group_by(service_name)
```

**Output**: Multiple rows per service (one per 5-minute interval).

**Note**: Pipe `|` required for time-series pattern.

**Use case**: Dashboard charts showing latency trends over time.

## Common Use Cases

### SLO Tracking: p95 Latency Under Threshold

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95_ns:tdigest_quantile(tdigest_combine(combined), 0.95),
          group_by(service_name)
| make_col p95_ms:p95_ns / 1000000
| make_col slo_target:500,
          meets_slo:if(p95_ms < 500, "yes", "no")
| sort desc(p95_ms)
```

**Use case**: Check which services meet p95 < 500ms SLO target.

**Output**: Services with SLO compliance status.

### Latency Distribution Analysis

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p50:tdigest_quantile(tdigest_combine(combined), 0.50),
          p75:tdigest_quantile(tdigest_combine(combined), 0.75),
          p90:tdigest_quantile(tdigest_combine(combined), 0.90),
          p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          p99:tdigest_quantile(tdigest_combine(combined), 0.99),
          group_by(service_name)
| make_col p50_ms:p50 / 1000000,
          p95_ms:p95 / 1000000,
          p99_ms:p99 / 1000000
```

**Use case**: Understand full latency distribution to identify outliers.

**Insight**: Large gap between p95 and p99 indicates inconsistent performance.

### Comparing Services by Latency

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          group_by(service_name)
| make_col p95_ms:p95 / 1000000
| sort desc(p95_ms)
| limit 10
```

**Use case**: Find slowest services to prioritize optimization efforts.

### Time-Series for Incident Investigation

```opal
align 5m, combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
| aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
            group_by(service_name)
| filter service_name = "frontend"
| make_col p95_ms:p95 / 1000000
```

**Use case**: See when latency spiked during an incident.

**Output**: Timeline of p95 latency for specific service.

### Multi-Dimension Grouping

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          group_by(service_name, environment)
| make_col p95_ms:p95 / 1000000
| sort desc(p95_ms)
```

**Use case**: Compare latency across services AND environments.

## Complete Example

**Scenario**: You're tracking SLOs for your microservices. The target is p95 latency < 500ms and p99 latency < 1000ms for all production services.

**Step 1: Discover tdigest metrics**
```
discover_context("duration tdigest", result_type="metric")
```

Found: `span_sn_service_node_duration_tdigest_5m` (type: tdigest)

**Step 2: Get metric details**
```
discover_context(metric_name="span_sn_service_node_duration_tdigest_5m")
```

Available dimensions: `service_name`, `environment`, `for_service_name`

**Step 3: Query for SLO compliance**
```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95_ns:tdigest_quantile(tdigest_combine(combined), 0.95),
          p99_ns:tdigest_quantile(tdigest_combine(combined), 0.99),
          group_by(service_name, environment)
| make_col p95_ms:p95_ns / 1000000,
          p99_ms:p99_ns / 1000000
| make_col p95_slo:if(p95_ms < 500, "✓", "✗"),
          p99_slo:if(p99_ms < 1000, "✓", "✗")
| filter environment = "production"
| sort desc(p95_ms)
```

**Step 4: Interpret results**

| service_name | environment | p95_ms | p99_ms | p95_slo | p99_slo |
|--------------|-------------|--------|--------|---------|---------|
| frontend | production | 19373.5 | 5641328.2 | ✗ | ✗ |
| featureflagservice | production | 5838.8 | 7473.9 | ✗ | ✗ |
| cartservice | production | 4136.6 | 5898.3 | ✗ | ✗ |
| productcatalogservice | production | 257.0 | 313.1 | ✓ | ✓ |
| currencyservice | production | 54.1 | 125.1 | ✓ | ✓ |

**Insight**: Frontend, featureflagservice, and cartservice are violating SLOs - need optimization.

**Step 5: Investigate frontend latency over time**
```opal
align 1h, combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
| aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
            p99:tdigest_quantile(tdigest_combine(combined), 0.99),
            group_by(service_name)
| filter service_name = "frontend"
| make_col p95_ms:p95 / 1000000, p99_ms:p99 / 1000000
```

**Output**: Hourly p95/p99 trends to identify when latency degraded.

## Common Pitfalls

### Pitfall 1: Forgetting Second Combine

❌ **Wrong** (most common mistake):
```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("metric"))
aggregate p95:tdigest_quantile(combined, 0.95)
```

✅ **Correct**:
```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("metric"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```

**Why**: TDigest requires combining twice - once in align, once in aggregate.

**Error message**: "the field has to be aggregated or grouped"

### Pitfall 2: Using m() Instead of m_tdigest()

❌ **Wrong**:
```opal
align options(bins: 1), combined:tdigest_combine(m("duration_tdigest_5m"))
```

✅ **Correct**:
```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("duration_tdigest_5m"))
```

**Why**: Tdigest metrics require `m_tdigest()` function, not `m()`.

**Check**: Look for `Type: tdigest` in `discover_context()` output.

### Pitfall 3: Wrong Pipe Usage (Same as Gauge)

❌ **Wrong** (pipe with bins:1):
```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("metric"))
| aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```

✅ **Correct**:
```opal
# Summary - NO pipe
align options(bins: 1), combined:tdigest_combine(m_tdigest("metric"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)

# Time-series - YES pipe
align 5m, combined:tdigest_combine(m_tdigest("metric"))
| aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```

### Pitfall 4: Percentile Value Out of Range

❌ **Wrong**:
```opal
aggregate p95:tdigest_quantile(tdigest_combine(combined), 95)
```

✅ **Correct**:
```opal
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```

**Why**: Quantile values must be 0.0 to 1.0 (not 1 to 100).

### Pitfall 5: Not Converting Units

❌ **Wrong** (values in nanoseconds, hard to read):
```opal
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```
Result: `p95 = 14675991.25` (what unit is this?)

✅ **Correct** (convert to milliseconds):
```opal
aggregate p95_ns:tdigest_quantile(tdigest_combine(combined), 0.95)
| make_col p95_ms:p95_ns / 1000000
```
Result: `p95_ms = 14.68` (clearly milliseconds)

**Tip**: Check sample values in discovery to identify units (19-digit numbers = nanoseconds).

## Percentile Reference

Common percentiles and their meanings:

| Percentile | Decimal | Meaning | Common Use |
|------------|---------|---------|------------|
| p50 | 0.50 | Median (middle value) | Typical user experience |
| p75 | 0.75 | 75th percentile | Better than average case |
| p90 | 0.90 | 90th percentile | Catching most outliers |
| p95 | 0.95 | 95th percentile | Standard SLO target |
| p99 | 0.99 | 99th percentile | Tail latency / worst 1% |
| p99.9 | 0.999 | 99.9th percentile | Extreme outliers |

**SLO best practice**: Track p95 and p99, not just averages.

## Unit Conversion Reference

Common time unit conversions (assuming nanoseconds):

```opal
# Nanoseconds to milliseconds (most common)
make_col value_ms:value_ns / 1000000

# Nanoseconds to seconds
make_col value_sec:value_ns / 1000000000

# Nanoseconds to microseconds
make_col value_us:value_ns / 1000
```

**How to identify units**: Check sample values in `discover_context()`:
- 19 digits (1760201545280843522) = nanoseconds
- 13 digits (1758543367916) = milliseconds
- 10 digits (1758543367) = seconds

## Best Practices

1. **Always use double-combine pattern** - most critical rule for tdigest
2. **Verify metric type** - must be `tdigest` (not `gauge`)
3. **Check units** - convert nanoseconds to milliseconds for readability
4. **Use multiple percentiles** - p50, p95, p99 show full distribution
5. **Calculate SLO compliance** - add derived columns comparing to targets
6. **Sort and limit** - focus on worst offenders with `sort desc() | limit 10`
7. **Use time-series for investigation** - see when latency changed
8. **Group by relevant dimensions** - service, environment, endpoint, etc.

## Related Skills

- **aggregating-gauge-metrics** - For count/sum/avg metrics (NOT percentiles)
- **working-with-intervals** - For calculating percentiles from raw interval data (slower)
- **time-series-analysis** - For event/interval trending with timechart

## Summary

TDigest metrics enable efficient percentile calculations:

- **Core pattern**: `align` + `m_tdigest()` + **double** `tdigest_combine` + `tdigest_quantile`
- **Critical rule**: Use `tdigest_combine()` TWICE (in align AND in aggregate)
- **Metric function**: `m_tdigest()` (NOT `m()`)
- **Percentile values**: 0.0 to 1.0 (0.95 = p95)
- **Common percentiles**: p50 (median), p95 (SLO), p99 (tail latency)
- **Units**: Often nanoseconds - convert to milliseconds for readability

**Key distinction**: TDigest metrics use special double-combine pattern, while gauge metrics use simple `m()` + aggregate.

---
**Last Updated**: November 14, 2025
**Version**: 1.0
**Tested With**: Observe OPAL (ServiceExplorer/Service Inspector Metrics)
