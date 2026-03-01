---
name: monitor
description: Observe application health and gather feedback after deployment. Use to validate success criteria, collect metrics, and feed issues back to the backlog. Closes the feedback loop.
---

# Monitor

Observe deployed application health and gather feedback to inform future development.

## Purpose

Monitoring completes the feedback loop:
- Validate success criteria from intent
- Collect performance and health metrics
- Gather user feedback
- Identify issues for the backlog
- Ensure deployed features work as expected

## Input

Expect from orchestrator:
- Deployed application URL/endpoints
- Success criteria from intent
- Metrics to track
- Monitoring duration/frequency

## Process

### 1. Health monitoring

Check application health continuously:

```yaml
health_checks:
  - name: "API availability"
    endpoint: "/health"
    method: "GET"
    expected_status: 200
    interval: "60s"
    timeout: "5s"

  - name: "Database connectivity"
    endpoint: "/health/db"
    method: "GET"
    expected_status: 200
    interval: "60s"

  - name: "External dependencies"
    endpoint: "/health/deps"
    method: "GET"
    expected_status: 200
    interval: "300s"
```

### 2. Performance metrics

Collect performance data:

```yaml
metrics:
  - name: "response_time_p50"
    source: "apm"
    threshold: 200  # ms
    alert_above: 500

  - name: "response_time_p99"
    source: "apm"
    threshold: 1000  # ms
    alert_above: 2000

  - name: "error_rate"
    source: "logs"
    threshold: 0.01  # 1%
    alert_above: 0.05

  - name: "throughput"
    source: "apm"
    baseline: 100  # req/s
    alert_below: 50

  - name: "cpu_usage"
    source: "infrastructure"
    threshold: 70  # %
    alert_above: 90

  - name: "memory_usage"
    source: "infrastructure"
    threshold: 80  # %
    alert_above: 95
```

### 3. Success criteria validation

Verify intent success criteria are met:

```yaml
success_validation:
  - criterion: "Users can register successfully"
    metric: "registration_success_rate"
    target: ">= 99%"
    actual: "99.5%"
    status: "pass"

  - criterion: "Page loads under 2 seconds"
    metric: "page_load_time_p95"
    target: "< 2000ms"
    actual: "1850ms"
    status: "pass"

  - criterion: "Zero critical errors in first 24h"
    metric: "critical_error_count"
    target: "= 0"
    actual: "0"
    status: "pass"
```

### 4. Error tracking

Monitor for errors and exceptions:

```yaml
error_tracking:
  sources:
    - "application_logs"
    - "error_reporting_service"  # Sentry, Bugsnag, etc.
    - "browser_console"

  categories:
    - severity: "critical"
      pattern: "uncaught exception|fatal error|500"
      action: "immediate_alert"

    - severity: "error"
      pattern: "error|failed|exception"
      action: "aggregate_and_review"

    - severity: "warning"
      pattern: "warning|deprecated|slow"
      action: "log_for_review"
```

### 5. User feedback collection

Gather feedback from various sources:

```yaml
feedback_sources:
  - source: "support_tickets"
    integration: "zendesk"
    filter: "tag:new-feature"

  - source: "user_surveys"
    integration: "typeform"
    trigger: "post_feature_use"

  - source: "analytics_events"
    integration: "mixpanel"
    events: ["feature_used", "feature_abandoned", "error_encountered"]

  - source: "social_mentions"
    integration: "twitter_api"
    keywords: ["@ourapp", "ourapp bug"]
```

### 6. Analyse and categorise

Process collected data:

```yaml
analysis:
  - type: "error_pattern"
    description: "Timeout errors spike during peak hours"
    frequency: "12 occurrences/hour"
    impact: "medium"
    recommendation: "Investigate database connection pooling"

  - type: "user_friction"
    description: "Users abandoning registration at email step"
    metric: "30% drop-off"
    impact: "high"
    recommendation: "Review email validation UX"

  - type: "performance_regression"
    description: "API response time increased 40% after deploy"
    metric: "p99: 800ms → 1120ms"
    impact: "medium"
    recommendation: "Profile recent changes"
```

### 7. Generate backlog items

Create stories from feedback:

```yaml
backlog_additions:
  - id: "US-015"
    title: "Optimise database queries for peak load"
    source: "monitor:error_pattern"
    description: |
      As a user,
      I want fast response times during peak hours,
      So that I can complete tasks without delays.
    priority: 2
    evidence: "Timeout errors during 9-11am, 2-4pm"

  - id: "US-016"
    title: "Improve email validation feedback"
    source: "monitor:user_friction"
    description: |
      As a new user,
      I want clear feedback when my email is invalid,
      So that I can correct it and complete registration.
    priority: 1
    evidence: "30% drop-off at email step, support tickets about 'email not accepted'"
```

## Output

```yaml
monitor:
  period:
    start: "2024-01-15T10:00:00Z"
    end: "2024-01-16T10:00:00Z"
    duration: "24h"

  status: "healthy | degraded | unhealthy"

  health:
    uptime: "99.95%"
    incidents: 0
    health_checks:
      passed: 1440
      failed: 0

  performance:
    response_time_p50: "145ms"
    response_time_p99: "890ms"
    error_rate: "0.02%"
    throughput: "125 req/s"

  success_criteria:
    met: 3
    total: 3
    details:
      - criterion: "Users can register"
        status: "pass"
      - criterion: "Page loads < 2s"
        status: "pass"
      - criterion: "Zero critical errors"
        status: "pass"

  errors:
    critical: 0
    error: 12
    warning: 45
    top_errors:
      - message: "Connection timeout"
        count: 8
        first_seen: "2024-01-15T14:23:00Z"

  feedback:
    support_tickets: 3
    user_surveys: 15
    nps_score: 42
    sentiment: "positive"

  backlog_additions:
    - id: "US-015"
      title: "Optimise database queries"
      priority: 2
    - id: "US-016"
      title: "Improve email validation"
      priority: 1

  recommendations:
    - "Monitor database connection pool during peak hours"
    - "Consider adding email format hints to registration form"
    - "Review timeout thresholds for external API calls"
```

Update manifest:
```yaml
releases:
  - version: "1.2.3"
    monitoring:
      status: "healthy"
      period: "2024-01-15 to 2024-01-16"
      success_criteria_met: true

backlog:
  # New items added from monitoring
  - id: "US-015"
    source: "monitor"
    # ...
```

## Monitoring duration

```yaml
monitoring_schedule:
  post_deploy:
    duration: "24h"
    intensity: "high"
    checks_interval: "1m"

  steady_state:
    duration: "ongoing"
    intensity: "normal"
    checks_interval: "5m"

  alerts:
    critical: "immediate"
    error: "within 15m"
    warning: "daily digest"
```

## Alert escalation

```yaml
escalation:
  level_1:
    condition: "health_check_failed"
    wait: "5m"
    action: "slack_notification"

  level_2:
    condition: "health_check_failed for 15m"
    action: "pagerduty_alert"

  level_3:
    condition: "health_check_failed for 30m"
    action: "phone_call_oncall"
```

## Tips

- Set baselines before comparing metrics
- Alert on symptoms, not just causes
- Avoid alert fatigue with appropriate thresholds
- Correlate metrics across services
- Keep monitoring configs in version control
- Review and tune thresholds regularly
- Document what "normal" looks like
