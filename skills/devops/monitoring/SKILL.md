---
name: Monitoring
description: "Set up observability for applications and infrastructure with metrics, logs, traces, and alerts."
---

## Complexity Levels

| Level | Tools | Setup Time | Best For |
|-------|-------|------------|----------|
| **Minimal** | UptimeRobot, Healthchecks.io | 15 min | Side projects, MVPs |
| **Standard** | Uptime Kuma, Sentry, basic Grafana | 1-2 hours | Small teams, startups |
| **Professional** | Prometheus, Grafana, Loki, Alertmanager | 1-2 days | Production systems |
| **Enterprise** | Datadog, New Relic, or full OSS stack | Ongoing | Large-scale operations |

## The Three Pillars

| Pillar | What It Answers | Tools |
|--------|-----------------|-------|
| **Metrics** | "How is the system performing?" | Prometheus, Grafana, Datadog |
| **Logs** | "What happened?" | Loki, ELK, CloudWatch |
| **Traces** | "Why is this request slow?" | Jaeger, Tempo, Sentry |

## Quick Start by Use Case

**"I just want to know if it's down"**
→ UptimeRobot (free) or Uptime Kuma (self-hosted). See `simple.md`.

**"I need to debug production errors"**
→ Sentry with your framework SDK. 5-minute setup. See `apm.md`.

**"I want real observability"**
→ Prometheus + Grafana + Loki. See `prometheus.md`.

**"I need to centralize logs"**
→ Loki for simple, ELK for complex queries. See `logs.md`.

## What to Monitor

### Applications (RED Method)
- **R**ate — requests per second
- **E**rrors — error rate by endpoint
- **D**uration — latency (p50, p95, p99)

### Infrastructure (USE Method)
- **U**tilization — CPU, memory, disk usage
- **S**aturation — queue depth, load average
- **E**rrors — hardware/system errors

## Alerting Principles

| Do | Don't |
|----|-------|
| Alert on symptoms (user impact) | Alert on causes (CPU high) |
| Include runbook link | Require investigation to understand |
| Set appropriate severity | Make everything P1 |
| Require action | Alert on "interesting" metrics |

**Alert fatigue kills monitoring.** If alerts are ignored, you have no monitoring.

For alert configuration, severities, and on-call setup, see `alerting.md`.

## Cost Comparison

| Solution | Monthly Cost (small) | Monthly Cost (medium) |
|----------|---------------------|----------------------|
| UptimeRobot | Free | $7 |
| Uptime Kuma | $5 (VPS) | $5 (VPS) |
| Sentry | Free / $26 | $80 |
| Grafana Cloud | Free tier | $50+ |
| Datadog | $15/host | $23/host + features |
| Self-hosted stack | $10-20 (VPS) | $50-100 (VPS) |

## Common Mistakes

- Starting with Prometheus/Grafana when Uptime Kuma would suffice
- No alerting (dashboards nobody watches)
- Too many alerts (alert fatigue → ignored)
- Missing runbooks (alert fires, nobody knows what to do)
- Not monitoring from outside (only internal checks)
- Storing logs forever (cost explodes)
