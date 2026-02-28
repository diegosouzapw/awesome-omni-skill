---
name: administration
description: How to monitor usage, track costs, configure analytics, and measure ROI for Claude Code. Use when user asks about monitoring, telemetry, metrics, costs, analytics, or OpenTelemetry.
---

# Claude Code Administration

## Monitoring Overview

Claude Code supports **OpenTelemetry (OTel) for metrics and events**. The system exports time series data via standard metrics protocol and events through logs/events protocol.

### Quick Setup

Enable telemetry:
```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
```

Configure exporters (optional, pick what you need):
```bash
# Metrics
export OTEL_METRICS_EXPORTER=otlp  # Options: otlp, prometheus, console

# Logs
export OTEL_LOGS_EXPORTER=otlp     # Options: otlp, console
```

### Export Intervals

Default intervals:
- **Metrics**: 60 seconds
- **Logs**: 5 seconds

Customize intervals:
```bash
export OTEL_METRIC_EXPORT_INTERVAL=30000  # milliseconds
export OTEL_LOGS_EXPORT_INTERVAL=10000    # milliseconds
```

## Available Metrics

Claude Code tracks eight core metrics:

### 1. Session Counter
CLI sessions started

**Use for:** Tracking adoption and active users

### 2. Lines of Code
Code additions/removals tracked by type

**Use for:** Measuring productivity and code generation volume

### 3. Pull Requests
Creation count

**Use for:** Tracking automated PR generation

### 4. Commits
Git commits via Claude Code

**Use for:** Measuring development activity

### 5. Cost Usage
Session costs in USD (model-segmented)

**Use for:** Budget tracking and cost allocation

**Important:** Cost metrics are approximations. For official billing data, refer to your API provider (Claude Console, AWS Bedrock, or Google Cloud Vertex).

### 6. Token Usage
Tokens consumed (input/output/cache types)

**Use for:** Understanding API usage patterns and optimizing costs

### 7. Code Edit Tool Decisions
Accept/reject counts per tool

**Use for:** Understanding user trust and automation acceptance

### 8. Active Time
Actual usage duration in seconds

**Use for:** Measuring engagement and productivity time

## Metric Segmentation

Segment metrics by:
- `user.account_uuid` - Individual user tracking
- `organization.id` - Team/organization grouping
- `session.id` - Session-specific analysis
- `model` - Model usage breakdown
- `app.version` - Version tracking

## Events & Logging

Five event types are exported:

### 1. User Prompt Events
Prompt submissions (content redacted by default)

**Enable prompt logging:**
```bash
export OTEL_LOG_USER_PROMPTS=1
```

**Use for:** Understanding user interaction patterns

### 2. Tool Result Events
Tool execution completion with success status and duration

**Use for:** Monitoring tool performance and reliability

### 3. API Request Events
Claude API calls with cost and token data

**Use for:** Detailed cost analysis and API usage tracking

### 4. API Error Events
Failed requests with HTTP status codes

**Use for:** Troubleshooting and reliability monitoring

### 5. Tool Decision Events
User accept/reject actions with decision source

**Use for:** Understanding automation trust and user preferences

## Cost Monitoring

### Cost Tracking Setup

Monitor costs by model and user:
```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=prometheus
```

### Cost Analysis

View costs segmented by:
- Model (Sonnet vs Haiku)
- User/account
- Session
- Time period

### Budget Alerts

Implement budget monitoring:
1. Export cost metrics to your monitoring system
2. Set up alerts for cost thresholds
3. Review high-cost sessions
4. Optimize model selection and usage patterns

## Analytics & ROI

### ROI Measurement Guide

Reference the [Claude Code ROI Measurement Guide](https://github.com/anthropics/claude-code-monitoring-guide) for:
- Docker configurations
- Productivity report templates
- ROI calculation methods
- Team analytics dashboards

### Key Metrics for ROI

**Productivity Metrics:**
- Lines of code generated per hour
- Time saved vs manual coding
- PRs created automatically
- Issues resolved automatically

**Quality Metrics:**
- Code review findings
- Test coverage improvements
- Bug reduction rate
- Technical debt reduction

**Adoption Metrics:**
- Active users
- Session frequency
- Feature usage patterns
- User satisfaction scores

## Monitoring Backend Setup

### Prometheus Setup

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'claude-code'
    static_configs:
      - targets: ['localhost:9464']
```

Start with Prometheus exporter:
```bash
export OTEL_METRICS_EXPORTER=prometheus
claude
```

### Grafana Dashboard

Create dashboards to visualize:
- Cost over time
- Token usage trends
- Session counts
- User activity
- Tool acceptance rates

### Custom Analytics

Export to your own backend:
```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=https://your-backend.com
export OTEL_EXPORTER_OTLP_HEADERS="api-key=your-key"
```

## Best Practices

### 1. Enable Monitoring Early
Set up telemetry from day one to establish baselines

### 2. Segment by Team/Project
Use organization and user IDs for proper attribution

### 3. Monitor Costs Regularly
Review cost metrics weekly to identify trends

### 4. Track Adoption
Monitor active users and session frequency

### 5. Measure Quality Impact
Track bug rates and code review findings

### 6. Set Alert Thresholds
Configure alerts for:
- Unusual cost spikes
- Error rate increases
- Low adoption indicators

### 7. Review Metrics with Teams
Share analytics to demonstrate value and identify improvements

### 8. Optimize Based on Data
Use metrics to:
- Identify high-value use cases
- Optimize model selection
- Improve automation acceptance
- Reduce costs

## Privacy Considerations

**User Prompts:**
- Disabled by default
- Enable only with user consent: `OTEL_LOG_USER_PROMPTS=1`
- Consider data retention policies

**Sensitive Data:**
- Avoid logging sensitive information
- Implement data filtering
- Review compliance requirements

**Access Control:**
- Restrict metrics access appropriately
- Use secure connections for exporters
- Encrypt data in transit and at rest

## Troubleshooting Monitoring

### Metrics Not Appearing

1. Verify telemetry is enabled: `CLAUDE_CODE_ENABLE_TELEMETRY=1`
2. Check exporter configuration
3. Verify backend connectivity
4. Review export intervals
5. Check for error logs

### High Costs

1. Review token usage by model
2. Identify high-usage sessions
3. Check for inefficient prompts
4. Consider using Haiku for simple tasks
5. Implement cost controls

### Low Adoption

1. Review active user metrics
2. Identify barriers to usage
3. Provide training and documentation
4. Gather user feedback
5. Highlight success stories

## Example Monitoring Stack

```bash
# docker-compose.yml for full monitoring stack
version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

Configure Claude Code:
```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=prometheus
export OTEL_EXPORTER_PROMETHEUS_PORT=9464
```
