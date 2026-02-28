---
name: aggregating-event-datasets
description: Aggregate and summarize event datasets (logs) using OPAL statsby. Use when you need to count, sum, or calculate statistics across log events. Covers make_col for derived columns, statsby for aggregation, group_by for grouping, aggregation functions (count, sum, avg, percentile), and topk for top N results. Returns single summary row per group across entire time range. For time-series trends, see time-series-analysis skill.
---

# Aggregating Event Datasets

Event datasets (logs) can be aggregated to create summaries and statistics. This skill teaches you how to use `statsby` to aggregate log data into meaningful insights using OPAL.

## When to Use This Skill

- Counting occurrences (error count by namespace, log volume by pod)
- Calculating statistics (average, sum, percentiles) across events
- Grouping events by dimensions (namespace, pod, container, service)
- Finding top N results by a metric (top 10 error sources, busiest pods)
- Creating summary reports across entire time range

**Note**: This skill covers `statsby` which returns **one summary row per group** across the entire time range. For time-series trends (multiple rows per group over time), see the `time-series-analysis` skill.

## Prerequisites

- Access to Observe tenant via MCP
- Understanding of event datasets (see filtering-event-datasets skill)
- Dataset with `log` interface (or any Event dataset)

## Key Concepts

### statsby - Statistical Aggregation

`statsby` is the primary aggregation verb for event datasets. It:
- Groups events by specified dimensions
- Applies aggregation functions (count, sum, avg, etc.)
- Returns **one row per group** across the entire query time range

**Syntax**:
```opal
statsby aggregation_function(), group_by(dimension1, dimension2, ...)
```

### Common Aggregation Functions

- `count()` - Count number of events
- `sum(field)` - Sum values of a field
- `avg(field)` - Average value of a field
- `min(field)` - Minimum value
- `max(field)` - Maximum value
- `percentile(field, p)` - Percentile (e.g., p=0.95 for 95th percentile)
- `any_not_null(field)` - Any non-null value from the group

### topk vs sort/limit

- **`topk N, max(metric)`** - Get top N results by a specific metric (semantically correct for "top performers")
- **`sort desc(metric) | limit N`** - Alternative but less clear intent
- **Use topk** for aggregated results - it's more explicit about intent

## Discovery Workflow

Start with dataset discovery (same as filtering-event-datasets):

**Step 1: Find dataset**
```
discover_context("kubernetes logs")
```

**Step 2: Get schema**
```
discover_context(dataset_id="YOUR_DATASET_ID")
```

Note fields you'll use for:
- **Filtering** (before aggregation)
- **Grouping** (dimensions to aggregate by)
- **Calculating** (fields to sum, average, etc.)

## Basic Patterns

### Pattern 1: Simple Count

**Use case**: Count total events

```opal
statsby count()
```

**Explanation**: Counts all events in the time range. Returns single row with total count.

**Output**:
```
count
5831
```

### Pattern 2: Count by Dimension

**Use case**: Count events grouped by a field (e.g., namespace)

```opal
make_col namespace:string(resource_attributes."k8s.namespace.name")
| statsby count(), group_by(namespace)
| topk 10, max(count)
```

**Explanation**:
1. `make_col` creates a derived column `namespace` from nested field
2. `statsby` counts events, grouped by namespace
3. `topk` returns top 10 namespaces by count

**Output**:
```
namespace,count,_c_rank
default,5805,1
kube-system,648,2
observe,64,3
```

### Pattern 3: Count with Filtering

**Use case**: Count errors per namespace

```opal
filter contains(body, "error")
| make_col namespace:string(resource_attributes."k8s.namespace.name")
| statsby error_count:count(), group_by(namespace)
| topk 10, max(error_count)
```

**Explanation**: Filters for errors first, then counts by namespace. Notice we name the count `error_count` for clarity.

### Pattern 4: Multiple Dimensions

**Use case**: Count by namespace AND pod

```opal
make_col
    namespace:string(resource_attributes."k8s.namespace.name"),
    pod:pod
| statsby count(), group_by(namespace, pod)
| topk 20, max(count)
```

**Explanation**: Groups by multiple dimensions. Each unique (namespace, pod) combination gets one row.

### Pattern 5: Multiple Aggregations

**Use case**: Calculate multiple statistics in one query

```opal
filter stream = "stderr"
| make_col namespace:string(resource_attributes."k8s.namespace.name")
| statsby
    stderr_count:count(),
    group_by(namespace)
| topk 10, max(stderr_count)
```

**Explanation**: You can calculate multiple aggregations in a single `statsby` call.

## Complete Example

End-to-end workflow for analyzing errors across your infrastructure.

**Scenario**: Find which services, namespaces, and pods are producing the most errors in the last 24 hours.

**Step 1: Discovery**

```
discover_context("kubernetes logs")
```

Found: Dataset "Kubernetes Explorer/Kubernetes Logs" (ID: 42161740)

**Step 2: Build query**

```opal
filter contains(body, "error") or contains(body, "ERROR")
| make_col
    namespace:string(resource_attributes."k8s.namespace.name"),
    pod:pod,
    container:container
| statsby error_count:count(), group_by(namespace, pod, container)
| topk 20, max(error_count)
```

**Step 3: Execute**

```
execute_opal_query(
    query="[query above]",
    primary_dataset_id="42161740",
    time_range="24h"
)
```

**Step 4: Interpret results**

```csv
namespace,pod,container,error_count,_c_rank
kube-system,calico-node-74d4r,calico-node,33,1
kube-system,calico-node-hhvbf,calico-node,31,2
kube-system,calico-node-ghk2s,calico-node,31,3
kube-system,calico-kube-controllers-759cd8b574-fzr49,calico-kube-controllers,31,4
```

**Analysis**:
- Most errors are in `kube-system` namespace
- `calico-node` pods are the primary error source
- All errors are from the same container (`calico-node`)
- Total of 126 errors across top 4 sources in 24h

**Next steps**: Investigate the specific calico-node errors to understand the root cause.

## Advanced Patterns

### Pattern 6: Conditional Aggregation

**Use case**: Count errors vs total, calculate error rate

```opal
make_col
    namespace:string(resource_attributes."k8s.namespace.name"),
    is_error:if(contains(body, "error"), 1, 0)
| statsby
    total:count(),
    error_count:sum(is_error),
    group_by(namespace)
| make_col error_rate:float64(error_count)/float64(total)
| topk 10, max(error_rate)
```

**Explanation**:
1. Create boolean flag `is_error` (1 or 0)
2. Count total events and sum error flags
3. Calculate error rate as derived column
4. Show top 10 by error rate

**Note**: OPAL doesn't have `count_if()`, so use `if()` + `sum()` pattern.

### Pattern 7: Type Conversions

**Use case**: Safely handle type conversions for nested fields

```opal
make_col
    namespace:string(resource_attributes."k8s.namespace.name"),
    pod:string(pod),
    container:string(container)
| statsby count(), group_by(namespace, pod, container)
| topk 20, max(count)
```

**Explanation**: Wrap fields in `string()`, `int64()`, `float64()` for type safety, especially with nested fields.

## Common Pitfalls

### Pitfall 1: Forgetting make_col Before statsby

❌ **Wrong**:
```opal
statsby count(), group_by(resource_attributes."k8s.namespace.name")
# Error: Can't group by nested field directly
```

✅ **Correct**:
```opal
make_col namespace:string(resource_attributes."k8s.namespace.name")
| statsby count(), group_by(namespace)
```

**Why**: `statsby` group_by needs simple column names. Use `make_col` to extract nested fields first.

### Pitfall 2: Using align Instead of statsby

❌ **Wrong**:
```opal
align options(bins: 1), count:count()
aggregate total:sum(count)
# align is for METRICS only!
```

✅ **Correct**:
```opal
statsby count()
# statsby is for EVENTS
```

**Why**: `align` is only for metric datasets. Events use `statsby` for aggregation.

### Pitfall 3: Using limit Instead of topk After Aggregation

❌ **Wrong** (less clear):
```opal
statsby error_count:count(), group_by(namespace)
| sort desc(error_count)
| limit 10
```

✅ **Correct**:
```opal
statsby error_count:count(), group_by(namespace)
| topk 10, max(error_count)
```

**Why**: `topk` explicitly states "top N by this metric" - clearer intent than arbitrary limit.

### Pitfall 4: Confusing statsby with timechart

❌ **Wrong** (if you want summary):
```opal
timechart 1h, count(), group_by(namespace)
# Returns multiple rows per namespace (time-series)
```

✅ **Correct** (for summary):
```opal
statsby count(), group_by(namespace)
# Returns one row per namespace (total)
```

**Why**:
- `statsby` = Single summary across time range
- `timechart` = Time-series with multiple rows per group

## Tips and Best Practices

- **Name your aggregations**: Use descriptive names like `error_count:count()` instead of just `count()`
- **Filter before aggregating**: Apply filters before `statsby` for better performance
- **Use topk for top N**: More explicit than sort/limit
- **Type conversion**: Wrap nested fields in `string()` for safety
- **Test with limit first**: When developing, filter to small dataset before aggregating
- **Small time ranges**: Start with 1h or 24h, expand once query is working

## Aggregation Function Reference

**Counting**:
- `count()` - Count all events in group

**Numeric**:
- `sum(field)` - Sum values
- `avg(field)` - Average
- `min(field)` - Minimum
- `max(field)` - Maximum
- `percentile(field, p)` - Percentile (0.0 to 1.0)

**String/Any**:
- `any_not_null(field)` - Any non-null value from group

## Additional Resources

For more details, see:
- [RESEARCH.md](../../RESEARCH.md) - Tested patterns and findings
- [OPAL Documentation](https://docs.observeinc.com/en/latest/content/query-language-reference/) - Official OPAL docs

## Related Skills

- [filtering-event-datasets] - For filtering events before aggregation
- [time-series-analysis] - For time-series trends with timechart
- [working-with-nested-fields] - Deep dive on nested field access

---

**Last Updated**: November 14, 2025
**Version**: 1.0
**Tested With**: Observe OPAL v2.x
