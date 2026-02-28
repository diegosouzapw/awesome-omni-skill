---
name: analyzing-apm-data
description: Monitor application performance using the RED methodology (Rate, Errors, Duration) with Observe. Use when analyzing service health, investigating errors, tracking latency, or building APM dashboards. Covers when to use metrics vs spans, combining RED signals, and troubleshooting workflows. Cross-references working-with-intervals, aggregating-gauge-metrics, and analyzing-tdigest-metrics skills.
---

# Analyzing APM Data

Application Performance Monitoring (APM) tracks the health and performance of your services using telemetry data. This skill teaches the RED methodology and how to choose between pre-aggregated metrics and raw span data for different APM use cases.

## When to Use This Skill

- Monitoring microservices health and performance
- Building service dashboards with Rate, Errors, Duration (RED)
- Investigating production incidents and errors
- Tracking latency and throughput across services
- Setting up SLO (Service Level Objective) monitoring
- Understanding when to use metrics vs spans for APM
- Root cause analysis of service degradation
- Analyzing service dependencies and call patterns
- Identifying database bottlenecks and slow queries
- Understanding which services depend on a failing component

## Prerequisites

- Access to Observe tenant with OpenTelemetry span data
- Understanding of distributed tracing concepts
- Familiarity with metrics queries (see aggregating-gauge-metrics, analyzing-tdigest-metrics)
- Familiarity with interval queries (see working-with-intervals)

## Key Concepts

### The RED Methodology

RED is a standard approach for monitoring microservices health:

**R - Rate**: Request volume (requests per second/minute/hour)
- How much traffic is each service handling?
- Are there traffic spikes or drops?

**E - Errors**: Error count and error rate (percentage)
- How many requests are failing?
- Which services have the highest error rates?

**D - Duration**: Latency percentiles (p50, p95, p99)
- How fast are services responding?
- Are there latency spikes?

### Metrics vs Spans Decision Tree

Observe provides two ways to analyze APM data:

**Metrics** (Pre-aggregated, Fast)
- Request counts: `span_call_count_5m`
- Error counts: `span_error_count_5m`
- Latency percentiles: `span_*_duration_tdigest_5m`
- Pre-aggregated at 5-minute intervals

**Spans** (Raw Data, Detailed)
- Dataset: OpenTelemetry/Span (interface: `otel_span`)
- Complete trace information
- All span attributes available

**Decision Matrix:**

| Need | Use | Why |
|------|-----|-----|
| Dashboard, 24h+ range | Metrics | Fast, efficient, pre-aggregated |
| "How many errors?" | Metrics | Quick counts, good for alerts |
| "What went wrong?" | Spans | Error messages, stack traces |
| Latency overview | Metrics (tdigest) | Efficient percentiles |
| Filter by endpoint | Spans | Full attribute filtering |
| Real-time monitoring | Metrics | Consistent performance |
| Root cause analysis | Spans | Complete context, trace IDs |
| Service dependencies | Spans | Trace relationships, span types |
| Database bottlenecks | Spans | DB attributes, query details |

**Recommended Workflow:**
1. Start with metrics (fast overview)
2. Identify anomalies (high errors, latency spikes)
3. Drill down with spans (detailed investigation)

## Discovery Workflow

**Step 1: Find APM metrics**
```
discover_context("request error latency", result_type="metric")
```

Common metrics:
- `span_call_count_5m` - Request counts (gauge)
- `span_error_count_5m` - Error counts (gauge)
- `span_sn_service_node_duration_tdigest_5m` - Latency (tdigest)

**Step 2: Find span dataset**
```
discover_context("opentelemetry span trace")
```

Look for: OpenTelemetry/Span (interface: `otel_span`)

**Step 3: Get detailed schemas**
```
discover_context(metric_name="span_call_count_5m")
discover_context(dataset_id="<span_dataset_id>")
```

## RED Methodology Patterns

### Pattern 1: Rate - Request Volume (Metrics)

Get total requests per service over 24 hours:

```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total_requests:sum(rate), group_by(service_name)
| sort desc(total_requests)
| limit 10
```

**Output**: One row per service with total request count.

**Use case**: Identify highest-traffic services, detect traffic drops.

### Pattern 2: Errors - Error Count (Metrics)

Get total errors per service:

```opal
align options(bins: 1), errors:sum(m("span_error_count_5m"))
aggregate total_errors:sum(errors), group_by(service_name)
fill total_errors:0
| sort desc(total_errors)
| limit 10
```

**Output**: Services ranked by error count (including zeros with `fill`).

**Use case**: Quick error overview, dashboard widgets.

### Pattern 3: Duration - Latency Percentiles (Metrics)

Get latency percentiles per service:

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p50:tdigest_quantile(tdigest_combine(combined), 0.50),
          p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          p99:tdigest_quantile(tdigest_combine(combined), 0.99),
          group_by(service_name)
| make_col p50_ms:p50/1000000, p95_ms:p95/1000000, p99_ms:p99/1000000
| sort desc(p95_ms)
| limit 10
```

**Output**: Latency percentiles in milliseconds per service.

**Use case**: SLO tracking, latency monitoring, performance comparison.

**Note**: Uses tdigest double-combine pattern (see analyzing-tdigest-metrics skill).

### Pattern 4: Combined RED Dashboard (Metrics)

Calculate Rate, Errors, and Error Rate together:

```opal
align options(bins: 1),
      requests:sum(m("span_call_count_5m")),
      errors:sum(m("span_error_count_5m"))
aggregate total_requests:sum(requests),
          total_errors:sum(errors),
          group_by(service_name)
fill total_errors:0
| make_col error_rate:float64(total_errors) / float64(total_requests) * 100.0
| sort desc(error_rate)
| limit 10
```

**Output**: Services with request count, error count, and error rate percentage.

**Use case**: Complete service health dashboard.

### Pattern 5: Trending - Requests Over Time (Metrics)

Track request volume over time for charting:

```opal
align 1h, rate:sum(m("span_call_count_5m"))
| aggregate requests_per_hour:sum(rate), group_by(service_name)
| filter service_name = "frontend"
```

**Output**: Hourly time-series data for one service.

**Use case**: Dashboard charts, trend analysis, capacity planning.

**Pipe rule**: Time-series (`1h`) requires pipe `|` after `align`.

## Detailed Investigation Patterns (Spans)

### Pattern 6: Rate from Spans (Detailed)

Count requests from raw spans (1-hour window recommended):

```opal
make_col svc:service_name
| statsby request_count:count(), group_by(svc)
| sort desc(request_count)
| limit 10
```

**Use case**: When you need exact counts for short time ranges or want to filter by span attributes first.

**Note**: Use shorter time ranges (1h) for performance.

### Pattern 7: Error Analysis with Messages (Spans)

Get error details including messages:

```opal
filter error = true
| make_col svc:service_name,
          error_msg:string(error_message),
          span:span_name,
          status:string(status_code)
| statsby error_count:count(), group_by(svc, error_msg, span)
| sort desc(error_count)
| limit 10
```

**Output**: Error counts WITH full error messages and span names.

**Use case**: Root cause analysis - understand WHY errors happened.

**Key advantage**: Spans show WHAT went wrong, metrics only show HOW MANY.

### Pattern 8: Latency from Spans (Filtered)

Calculate latency percentiles for specific span types:

```opal
filter span_type = "Service entry point"
| make_col svc:service_name, dur_ms:duration / 1ms
| statsby p50:percentile(dur_ms, 0.50),
          p95:percentile(dur_ms, 0.95),
          p99:percentile(dur_ms, 0.99),
          group_by(svc)
| sort desc(p95)
| limit 10
```

**Output**: Latency percentiles for user-facing requests only.

**Use case**: Focus on end-user experience, exclude internal calls.

**Duration conversion**: `duration / 1ms` converts nanoseconds to milliseconds.

### Pattern 9: Error Rate from Spans

Calculate error rate using raw span data:

```opal
make_col svc:service_name, is_error:if(error = true, 1, 0)
| statsby total:count(), error_count:sum(is_error), group_by(svc)
| make_col error_rate:float64(error_count) / float64(total) * 100.0
| sort desc(error_rate)
| limit 10
```

**Output**: Services with total requests, errors, and error rate.

**Use case**: When you need to filter spans first (e.g., only specific endpoints).

## Complete APM Workflow Example

**Scenario**: You notice performance issues and need to investigate.

**Step 1: Check overall service health (Metrics - Fast)**

```opal
align options(bins: 1),
      requests:sum(m("span_call_count_5m")),
      errors:sum(m("span_error_count_5m"))
aggregate total_requests:sum(requests),
          total_errors:sum(errors),
          group_by(service_name)
fill total_errors:0
| make_col error_rate:float64(total_errors) / float64(total_requests) * 100.0
| sort desc(error_rate)
```

**Finding**: `cartservice` has 0.48% error rate - investigate further.

**Step 2: Get error details (Spans - Detailed)**

```opal
filter error = true
| filter service_name = "cartservice"
| make_col error_msg:string(error_message),
          span:span_name,
          endpoint:string(attributes.\"http.target\")
| statsby error_count:count(), group_by(error_msg, span, endpoint)
| sort desc(error_count)
```

**Finding**: "Can't access cart storage. System.ApplicationException: Wasn't able to connect to redis" - Redis connection issue!

**Step 3: Check latency impact (Metrics)**

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          group_by(service_name)
| filter service_name = "cartservice"
| make_col p95_ms:p95/1000000
```

**Finding**: p95 latency is elevated - errors are impacting response time.

**Step 4: View hourly trend (Metrics)**

```opal
align 1h, errors:sum(m("span_error_count_5m"))
| aggregate errors_per_hour:sum(errors), group_by(service_name)
| filter service_name = "cartservice"
```

**Finding**: Errors started 3 hours ago - correlate with Redis deployment?

**Result**: Complete picture of issue (what, when, impact) using metrics + spans.

## Dependency Tracking and Service Relationships

Understanding service dependencies is critical for root cause analysis. Spans contain relationship information through trace IDs and span types.

### Key Span Fields for Dependency Tracking

**Trace relationships**:
- `trace_id` - Links all spans in a single request
- `span_id` - Unique identifier for this span
- `parent_span_id` - Links to calling span

**Span direction**:
- `kind` - "CLIENT" (outgoing call) or "SERVER" (incoming request)
- `span_type` - "Service entry point", "Remote call", "Database call"

**Target identification**:
- `span_name` - Operation name (e.g., "grpc.oteldemo.CartService/GetCart")
- `attributes.db.*` - Database connection details

### Pattern 10: What Services Does X Depend On? (Downstream)

Find all services that a given service calls:

```opal
filter service_name = "frontend" and kind = "CLIENT"
| make_col target:span_name, dur_ms:duration / 1ms
| statsby calls:count(),
          p95_ms:percentile(dur_ms, 0.95),
          group_by(target)
| sort desc(calls)
```

**Output**: All downstream dependencies with call volume and latency.

**Use case**: "Frontend is slow - which downstream services are contributing?"

### Pattern 11: What Services Depend On X? (Upstream)

Find all services that call a given service (requires trace traversal):

```opal
filter span_type = "Service entry point"
| make_col svc:service_name
| statsby total:count(), group_by(svc)
| sort desc(total)
```

**Output**: Services ordered by request volume (entry points).

**Note**: Full upstream analysis requires joining spans by trace_id (advanced).

### Pattern 12: Database Dependencies

Identify which services are making database calls:

```opal
filter string(attributes."db.type") = "sql"
| make_col caller:service_name,
          db_name:string(attributes."db.name"),
          dur_ms:duration / 1ms
| statsby call_count:count(),
          p95_latency:percentile(dur_ms, 0.95),
          group_by(caller, db_name)
| sort desc(call_count)
```

**Output**: Service-to-database call patterns with latency.

**Use case**: "Are there any slow database queries impacting my services?"

### Pattern 13: Service Call Patterns (All Outbound)

Get a complete picture of all outbound service calls:

```opal
filter kind = "CLIENT"
| make_col caller:service_name, target:span_name
| statsby call_count:count(), group_by(caller, target)
| sort desc(call_count)
| limit 20
```

**Output**: Complete service-to-service call matrix.

**Use case**: Understanding service dependencies architecture-wide.

### Dependency Troubleshooting Workflow

**Scenario**: Service X has elevated latency - is it a downstream dependency?

**Step 1: Identify service's dependencies**
```opal
filter service_name = "X" and kind = "CLIENT"
| make_col target:span_name, dur_ms:duration / 1ms
| statsby calls:count(),
          p95_ms:percentile(dur_ms, 0.95),
          group_by(target)
| sort desc(p95_ms)
```

**Step 2: Check if database calls are slow**
```opal
filter service_name = "X"
| filter string(attributes."db.type") = "sql"
| make_col db:string(attributes."db.name"),
          query:string(attributes."db.statement"),
          dur_ms:duration / 1ms
| statsby avg_ms:avg(dur_ms),
          p95_ms:percentile(dur_ms, 0.95),
          count:count(),
          group_by(db, query)
| sort desc(p95_ms)
| limit 10
```

**Step 3: Compare to baseline**
Run same queries for different time range to see if latency changed.

**Result**: Identify which downstream dependency is causing the issue.

## Common Use Cases

### Use Case 1: Service Health Dashboard

**Goal**: Real-time overview of all services.

**Solution**: Use metrics with summary pattern:

```opal
align options(bins: 1),
      requests:sum(m("span_call_count_5m")),
      errors:sum(m("span_error_count_5m"))
aggregate total_requests:sum(requests),
          total_errors:sum(errors),
          group_by(service_name)
fill total_errors:0
| make_col error_rate:float64(total_errors) / float64(total_requests) * 100.0
```

Add latency query separately (tdigest metrics).

**Why metrics**: Fast, efficient, updates every 5 minutes.

### Use Case 2: SLO Tracking

**Goal**: Track p95 latency against 100ms SLO.

**Solution**: Use tdigest metrics:

```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_sn_service_node_duration_tdigest_5m"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95),
          group_by(service_name)
| make_col p95_ms:p95/1000000, slo_breach:if(p95/1000000 > 100, true, false)
| filter slo_breach = true
```

**Output**: Services exceeding 100ms p95 SLO.

### Use Case 3: Error Investigation

**Goal**: Understand why a specific service is failing.

**Solution**: Start with metrics (count), drill down with spans (details):

```opal
filter error = true
| filter service_name = "target-service"
| make_col error_msg:string(error_message),
          trace:trace_id,
          timestamp:start_time
| sort desc(timestamp)
| limit 50
```

**Output**: Recent errors with messages and trace IDs for further investigation.

### Use Case 4: Capacity Planning

**Goal**: Understand traffic patterns over 30 days.

**Solution**: Use metrics with daily buckets:

```opal
align 1d, rate:sum(m("span_call_count_5m"))
| aggregate requests_per_day:sum(rate), group_by(service_name)
```

**Time range**: Set to `30d` in query parameters.

**Output**: Daily request volume for capacity analysis.

## Best Practices

1. **Start with metrics** for dashboards, alerts, and overview queries
2. **Use spans for investigation** when you need details or filtering
3. **Keep time ranges short** when querying spans (1h recommended, max 24h)
4. **Use time-series patterns** (`align 1h`) for trending charts
5. **Use summary patterns** (`options(bins: 1)`) for single statistics
6. **Combine metrics** (requests + errors) in single query for efficiency
7. **Filter spans early** to improve performance (`filter` before `make_col`)
8. **Use fill** to show services with zero errors in dashboards

## Performance Considerations

**Metrics queries:**
- Consistent performance across time ranges (1h same speed as 30d)
- Ideal for: Dashboards, real-time monitoring, long time ranges
- Limitation: Fixed dimensions, no custom filtering

**Span queries:**
- Performance degrades with longer time ranges
- Ideal for: Short time windows (1h), detailed investigation, filtered analysis
- Limitation: Slower for high-volume services over long periods

**Recommended approach:**
- Dashboards: 100% metrics
- Alerts: Metrics
- Investigation: Metrics (overview) → Spans (details)
- Trace analysis: Spans only

## Common Pitfalls

### Pitfall 1: Using Spans for Long Time Ranges

❌ **Wrong**:
```opal
make_col svc:service_name
| statsby request_count:count(), group_by(svc)
```
With 30-day time range - very slow!

✅ **Correct**:
```opal
align options(bins: 1), rate:sum(m("span_call_count_5m"))
aggregate total_requests:sum(rate), group_by(service_name)
```
Metrics handle 30 days efficiently.

### Pitfall 2: Forgetting Time Units

❌ **Wrong**:
```opal
make_col dur_ms:float64(duration)/1000000
```

✅ **Correct**:
```opal
make_col dur_ms:duration / 1ms
```

Use OPAL duration units (see working-with-intervals skill).

### Pitfall 3: Not Using fill for Zeros

❌ **Wrong**:
```opal
aggregate total_errors:sum(errors), group_by(service_name)
```
Services with zero errors won't appear.

✅ **Correct**:
```opal
aggregate total_errors:sum(errors), group_by(service_name)
fill total_errors:0
```
All services shown (important for dashboards).

### Pitfall 4: Wrong Metric for Latency

❌ **Wrong**:
```opal
align options(bins: 1), dur:avg(m("span_duration_5m"))
```
Don't use avg() on pre-aggregated duration.

✅ **Correct**:
```opal
align options(bins: 1), combined:tdigest_combine(m_tdigest("span_*_duration_tdigest_5m"))
aggregate p95:tdigest_quantile(tdigest_combine(combined), 0.95)
```
Use tdigest metrics for percentiles.

## Related Skills

- **working-with-intervals** - Understanding span duration and temporal queries
- **aggregating-gauge-metrics** - Request and error count metrics (gauge type)
- **analyzing-tdigest-metrics** - Latency percentile metrics (tdigest type)
- **filtering-event-datasets** - Filtering techniques applicable to spans
- **time-series-analysis** - Time-bucketed analysis with timechart (alternative to align)

## Summary

APM in Observe uses the RED methodology:
- **Rate**: Request counts from `span_call_count_5m` metric
- **Errors**: Error counts from `span_error_count_5m` metric
- **Duration**: Latency percentiles from `span_*_duration_tdigest_5m` metric

**Key decision**: Metrics for speed and overview, Spans for detail and investigation.

**Workflow**: Metrics (identify issues) → Spans (root cause analysis) → Traces (follow request flow).

**Performance**: Metrics scale to long time ranges, spans are best for short windows.

---
**Last Updated**: November 14, 2025
**Version**: 1.0
**Tested With**: Observe OPAL (OpenTelemetry/Span + ServiceExplorer Metrics)
