---
name: aggregating-gauge-metrics
description: Aggregate pre-computed metrics (gauge, counter, delta types) using OPAL. Use when analyzing request counts, error rates, resource utilization, or any numeric metrics over time. Covers align + m() + aggregate pattern, summary vs time-series output, and common aggregation functions. For percentile metrics (tdigest), see analyzing-tdigest-metrics skill.
---

# Aggregating Gauge Metrics

Pre-computed metrics in Observe store aggregated measurements at regular intervals (typically every 5 minutes). This skill teaches how to query gauge, counter, and delta metric types using OPAL.

## When to Use This Skill

- Analyzing request counts, error rates, or throughput metrics
- Tracking resource utilization (CPU, memory, network)
- Computing totals, averages, or rates across time periods
- Creating dashboards with time-series charts
- Working with any gauge, counter, or delta metric type
- When you need summary statistics or trends over time

## Prerequisites

- Access to Observe tenant via MCP
- Understanding that metrics are pre-aggregated (not raw events)
- Metric dataset with type: gauge, counter, or delta
- Use `discover_context()` to find and inspect metrics

## Key Concepts

### What Are Gauge Metrics?

**Gauge metrics** are pre-aggregated numeric measurements collected at regular intervals:

**Pre-aggregated**: Already summarized at collection time (typically 5-minute intervals)
- More efficient than querying raw data
- Faster query performance
- Lower storage costs

**Common Metric Types**:
- **Gauge**: Point-in-time value (CPU utilization, memory usage, queue depth)
- **Counter**: Monotonically increasing value (total requests, bytes sent)
- **Delta**: Change between intervals (requests per interval, errors per interval)

**Examples**:
- `span_call_count_5m` - Number of requests per 5-minute interval
- `span_error_count_5m` - Number of errors per 5-minute interval
- `system_cpu_utilization_ratio` - CPU utilization percentage
- `k8s_pod_memory_available_bytes` - Available memory in bytes

### CRITICAL: The align Verb is REQUIRED

Unlike datasets (Events/Intervals), metrics **MUST** use the `align` verb:

```opal
# WRONG - Will not work ❌
m("span_call_count_5m")
| statsby total:sum(metric)

# CORRECT - Must use align ✅
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total_requests:sum(rate)
```

**Why align is required**: Metrics are stored as time-series data that must be aligned to a time grid before aggregation.

### Summary vs Time-Series Output

OPAL metrics queries can produce two different output types:

| Output Type | Pattern | Result | Use Case |
|-------------|---------|--------|----------|
| **Summary** | `options(bins: 1)` | One row per group | Totals, overall statistics |
| **Time-Series** | `5m`, `1h`, or default | Many rows per group | Trending, dashboards, charts |

**Summary pattern** - Single statistics across entire time range:
```opal
align options(bins: 1), rate:sum(m("metric"))
aggregate total:sum(rate), group_by(service_name)
```
Output: One row per service

**Time-series pattern** - Values over time buckets:
```opal
align 5m, rate:sum(m("metric"))
| aggregate total:sum(rate), group_by(service_name)
```
Output: Multiple rows per service (one per 5-minute bucket)

**CRITICAL Syntax Difference**:
- Summary (`bins: 1`): NO pipe `|` between align and aggregate
- Time-series (`5m`): YES pipe `|` between align and aggregate

## Discovery Workflow

**Step 1: Search for metrics**
```
discover_context("request count", result_type="metric")
discover_context("error", result_type="metric")
discover_context("cpu memory", result_type="metric")
```

**Step 2: Get detailed metric schema**
```
discover_context(metric_name="span_call_count_5m")
```

**Step 3: Verify metric type**
Look for: `Type: gauge` (or `counter`, `delta`)

**Step 4: Note available dimensions**
These are used for `group_by()`:
- `service_name`, `service_namespace`
- `environment`, `span_name`
- `k8s_namespace_name`, `k8s_pod_name`
- etc. (shown in discovery output)

**Step 5: Write query**
Use `align` + `m()` + `aggregate` pattern with correct dimensions

## Basic Patterns

### Pattern 1: Total Count Across Time Range

Get overall totals (summary output):

```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total_requests:sum(rate)
```

**Output**: Single row with total count across entire time range.

**No group_by**: Aggregates everything together.

### Pattern 2: Totals Per Group

Get totals broken down by dimension:

```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total_requests:sum(rate), group_by(service_name)
```

**Output**: One row per service with total requests.

**group_by**: Use any dimension from metric schema.

### Pattern 3: Average Values Per Group

Calculate averages across time range:

```opal
align options(bins: 1), cpu:avg(m("system_cpu_utilization_ratio"))
aggregate avg_cpu:avg(cpu), group_by(service_name)
```

**Output**: Average CPU utilization per service.

**avg() function**: Used twice - once in align, once in aggregate.

### Pattern 4: Multiple Aggregations

Compute several statistics together:

```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total:sum(rate),
          average:avg(rate),
          maximum:max(rate),
          group_by(service_name)
```

**Output**: Multiple columns per service (total, average, maximum).

### Pattern 5: Time-Series for Trending

Track values over time buckets:

```opal
align 5m, rate:sum(m("span_call_count_5m"))
| aggregate requests_per_5min:sum(rate), group_by(service_name)
```

**Output**: Multiple rows per service (one per 5-minute interval).

**Note**: Pipe `|` required after align for time-series pattern.

**Output columns**:
- `_c_bucket` - Time bucket identifier
- `valid_from`, `valid_to` - Bucket boundaries
- Metric values

## Common Use Cases

### Counting Total Requests by Service

```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total_requests:sum(rate), group_by(service_name)
| sort desc(total_requests)
| limit 10
```

**Use case**: Identify top services by request volume.

### Counting Errors with Fill for Zero Values

```opal
align options(bins: 1), errors:sum(m("span_error_count_5m"))
aggregate total_errors:sum(errors), group_by(service_name)
fill total_errors:0
```

**Use case**: Show all services, even those with zero errors.

**fill verb**: Replaces null values with 0.

### Tracking Request Rate Over Time

```opal
align 1h, rate:sum(m("span_call_count_5m"))
| aggregate requests_per_hour:sum(rate), group_by(service_name)
```

**Use case**: Hourly request trends for dashboards.

**Output**: Time-series data for charting.

### Multiple Metrics in One Query

```opal
align options(bins: 1),
      requests:sum(m("span_call_count_5m")),
      errors:sum(m("span_error_count_5m"))
aggregate total_requests:sum(requests),
          total_errors:sum(errors),
          group_by(service_name)
| make_col error_rate:float64(total_errors) / float64(total_requests)
```

**Use case**: Calculate error rate from two metrics.

**make_col**: Add derived column after aggregation.

### Resource Utilization Averages

```opal
align options(bins: 1), cpu:avg(m("system_cpu_utilization_ratio"))
aggregate avg_cpu:avg(cpu),
          max_cpu:max(cpu),
          group_by(k8s_pod_name)
| sort desc(avg_cpu)
| limit 20
```

**Use case**: Find pods with highest CPU usage.

## Complete Example

**Scenario**: You want to analyze request and error rates for your microservices over the last 24 hours.

**Step 1: Discover available metrics**
```
discover_context("request error", result_type="metric")
```

Found metrics:
- `span_call_count_5m` (type: gauge)
- `span_error_count_5m` (type: gauge)

**Step 2: Get metric details**
```
discover_context(metric_name="span_call_count_5m")
```

Available dimensions: `service_name`, `service_namespace`, `environment`, `span_name`

**Step 3: Query for summary statistics**
```opal
align options(bins: 1),
      requests:sum(m("span_call_count_5m")),
      errors:sum(m("span_error_count_5m"))
aggregate total_requests:sum(requests),
          total_errors:sum(errors),
          group_by(service_name)
fill total_errors:0
| make_col error_rate:float64(total_errors) / float64(total_requests) * 100.0
| sort desc(total_requests)
```

**Step 4: Interpret results**

| service_name | total_requests | total_errors | error_rate |
|--------------|----------------|--------------|------------|
| frontend-proxy | 15660 | 0 | 0.0 |
| frontend | 15263 | 35 | 0.23 |
| featureflagservice | 11693 | 0 | 0.0 |
| productcatalogservice | 8813 | 0 | 0.0 |

**Insight**: Frontend has a 0.23% error rate - investigate errors.

**Step 5: Get hourly trends**
```opal
align 1h,
      requests:sum(m("span_call_count_5m")),
      errors:sum(m("span_error_count_5m"))
| aggregate requests_per_hour:sum(requests),
            errors_per_hour:sum(errors),
            group_by(service_name)
| filter service_name = "frontend"
```

**Output**: Time-series showing frontend requests and errors per hour.

## Common Pitfalls

### Pitfall 1: Forgetting align Verb

❌ **Wrong**:
```opal
m("span_call_count_5m")
| statsby total:sum(metric)
```

✅ **Correct**:
```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total:sum(rate)
```

**Why**: Metrics MUST use `align` verb - it's required, not optional.

### Pitfall 2: Wrong Pipe Usage

❌ **Wrong** (pipe with bins:1):
```opal
align options(bins: 1), rate:sum(m("metric"))
| aggregate total:sum(rate)
```

❌ **Wrong** (no pipe with time duration):
```opal
align 5m, rate:sum(m("metric"))
aggregate total:sum(rate)
```

✅ **Correct**:
```opal
# Summary - NO pipe
align options(bins: 1), rate:sum(m("metric"))
aggregate total:sum(rate)

# Time-series - YES pipe
align 5m, rate:sum(m("metric"))
| aggregate total:sum(rate)
```

**Why**: Syntax differs between summary and time-series patterns.

### Pitfall 3: Grouping by Non-Existent Dimension

❌ **Wrong**:
```opal
align options(bins: 1), rate:sum(m("metric"))
aggregate total:sum(rate), group_by(service_name)
```
Error: "field 'service_name' does not exist"

✅ **Correct**:
```opal
# First: discover_context(metric_name="metric") to see available dimensions
# Then: use only dimensions that exist
align options(bins: 1), rate:sum(m("metric"))
aggregate total:sum(rate), group_by(correct_dimension_name)
```

**Why**: Not all metrics have the same dimensions - always check first.

### Pitfall 4: Using statsby Instead of aggregate

❌ **Wrong**:
```opal
align options(bins: 1), rate:sum(m("metric"))
statsby total:sum(rate)
```

✅ **Correct**:
```opal
align options(bins: 1), rate:sum(m("metric"))
aggregate total:sum(rate)
```

**Why**: After `align`, use `aggregate` (not `statsby` which is for datasets).

## Aggregation Functions Reference

Common functions used with gauge metrics:

```opal
# Summing values
align options(bins: 1), metric:sum(m("metric_name"))
aggregate total:sum(metric)

# Averaging values
align options(bins: 1), metric:avg(m("metric_name"))
aggregate average:avg(metric)

# Maximum value
align options(bins: 1), metric:max(m("metric_name"))
aggregate maximum:max(metric)

# Minimum value
align options(bins: 1), metric:min(m("metric_name"))
aggregate minimum:min(metric)

# Count of samples
align options(bins: 1), metric:count(m("metric_name"))
aggregate sample_count:count(metric)
```

**Pattern**: Function used in both `align` and `aggregate`.

## Time Bucket Options

Common time durations for time-series queries:

```opal
align 1m, ...    # 1-minute buckets
align 5m, ...    # 5-minute buckets (common)
align 15m, ...   # 15-minute buckets
align 1h, ...    # 1-hour buckets
align 1d, ...    # 1-day buckets
```

**Default**: `align` without duration uses automatic binning (300 bins).

## Best Practices

1. **Always use discover_context() first** to find metrics and check dimensions
2. **Verify metric type** - this skill is for gauge/counter/delta (NOT tdigest)
3. **Use summary pattern** (`bins: 1`) for single statistics, reports, totals
4. **Use time-series pattern** (`5m`, `1h`) for dashboards, trending, charts
5. **Remember pipe rule**: bins:1 = no pipe, time duration = yes pipe
6. **Use fill** to replace nulls with zeros for complete results
7. **Add sort + limit** for top-N queries to avoid overwhelming output
8. **Check available dimensions** before using group_by

## Related Skills

- **analyzing-tdigest-metrics** - For percentile metrics (latency, duration p95/p99)
- **time-series-analysis** - For event/interval trending with timechart (different from metrics)
- **aggregating-event-datasets** - For aggregating raw events with statsby (different from metrics)
- **working-with-intervals** - For calculating durations from raw interval data

## Summary

Gauge metrics are pre-aggregated measurements that **require** the `align` verb:

- **Core pattern**: `align` + `m()` + `aggregate`
- **Metric types**: gauge, counter, delta (NOT tdigest)
- **Two output modes**:
  - Summary: `options(bins: 1)` → one row per group, NO pipe
  - Time-series: `5m`, `1h` → many rows per group, YES pipe
- **Common functions**: sum, avg, max, min, count
- **Discovery**: Use `discover_context()` to find metrics and dimensions

**Key distinction**: Metrics are pre-aggregated (use `align`), while Events/Intervals are raw data (use `statsby`/`timechart`).

---
**Last Updated**: November 14, 2025
**Version**: 1.0
**Tested With**: Observe OPAL (ServiceExplorer/Service Metrics)
