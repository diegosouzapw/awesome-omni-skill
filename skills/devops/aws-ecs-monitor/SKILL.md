---
name: aws-ecs-monitor
version: 1.0.1
description: AWS ECS production health monitoring with CloudWatch log analysis — monitors ECS service health, ALB targets, SSL certificates, and provides deep CloudWatch log analysis for error categorization, restart detection, and production alerts.
metadata:
  openclaw:
    requires:
      bins: ["aws", "curl", "python3"]
      anyBins: ["openssl"]
---

# AWS ECS Monitor

Production health monitoring and log analysis for AWS ECS services.

## What It Does

- **Health Checks**: HTTP probes against your domain, ECS service status (desired vs running), ALB target group health, SSL certificate expiry
- **Log Analysis**: Pulls CloudWatch logs, categorizes errors (panics, fatals, OOM, timeouts, 5xx), detects container restarts, filters health check noise
- **Auto-Diagnosis**: Reads health status and automatically investigates failing services via log analysis

## Prerequisites

- `aws` CLI configured with appropriate IAM permissions:
  - `ecs:ListServices`, `ecs:DescribeServices`
  - `elasticloadbalancing:DescribeTargetGroups`, `elasticloadbalancing:DescribeTargetHealth`
  - `logs:FilterLogEvents`, `logs:DescribeLogGroups`
- `curl` for HTTP health checks
- `python3` for JSON processing and log analysis
- `openssl` for SSL certificate checks (optional)

## Configuration

All configuration is via environment variables:

| Variable | Required | Default | Description |
|---|---|---|---|
| `ECS_CLUSTER` | **Yes** | — | ECS cluster name |
| `ECS_REGION` | No | `us-east-1` | AWS region |
| `ECS_DOMAIN` | No | — | Domain for HTTP/SSL checks (skip if unset) |
| `ECS_SERVICES` | No | auto-detect | Comma-separated service names to monitor |
| `ECS_HEALTH_STATE` | No | `./data/ecs-health.json` | Path to write health state JSON |
| `ECS_HEALTH_OUTDIR` | No | `./data/` | Output directory for logs and alerts |
| `ECS_LOG_PATTERN` | No | `/ecs/{service}` | CloudWatch log group pattern (`{service}` is replaced) |
| `ECS_HTTP_ENDPOINTS` | No | — | Comma-separated `name=url` pairs for HTTP probes |

## Directories Written

- **`ECS_HEALTH_STATE`** (default: `./data/ecs-health.json`) — Health state JSON file
- **`ECS_HEALTH_OUTDIR`** (default: `./data/`) — Output directory for logs, alerts, and analysis reports

## Scripts

### `scripts/ecs-health.sh` — Health Monitor

```bash
# Full check
ECS_CLUSTER=my-cluster ECS_DOMAIN=example.com ./scripts/ecs-health.sh

# JSON output only
ECS_CLUSTER=my-cluster ./scripts/ecs-health.sh --json

# Quiet mode (no alerts, just status file)
ECS_CLUSTER=my-cluster ./scripts/ecs-health.sh --quiet
```

Exit codes: `0` = healthy, `1` = unhealthy/degraded, `2` = script error

### `scripts/cloudwatch-logs.sh` — Log Analyzer

```bash
# Pull raw logs from a service
ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh pull my-api --minutes 30

# Show errors across all services
ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh errors all --minutes 120

# Deep analysis with error categorization
ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh diagnose --minutes 60

# Detect container restarts
ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh restarts my-api

# Auto-diagnose from health state file
ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh auto-diagnose

# Summary across all services
ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh summary --minutes 120
```

Options: `--minutes N` (default: 60), `--json`, `--limit N` (default: 200), `--verbose`

## Auto-Detection

When `ECS_SERVICES` is not set, both scripts auto-detect services from the cluster:

```bash
aws ecs list-services --cluster $ECS_CLUSTER
```

Log groups are resolved by pattern (default `/ecs/{service}`). Override with `ECS_LOG_PATTERN`:

```bash
# If your log groups are /ecs/prod/my-api, /ecs/prod/my-frontend, etc.
ECS_LOG_PATTERN="/ecs/prod/{service}" ECS_CLUSTER=my-cluster ./scripts/cloudwatch-logs.sh diagnose
```

## Integration

The health monitor can trigger the log analyzer for auto-diagnosis when issues are detected. Set `ECS_HEALTH_OUTDIR` to a shared directory and run both scripts together:

```bash
export ECS_CLUSTER=my-cluster
export ECS_DOMAIN=example.com
export ECS_HEALTH_OUTDIR=./data

# Run health check (auto-triggers log analysis on failure)
./scripts/ecs-health.sh

# Or run log analysis independently
./scripts/cloudwatch-logs.sh auto-diagnose --minutes 30
```

## Error Categories

The log analyzer classifies errors into:

- `panic` — Go panics
- `fatal` — Fatal errors
- `oom` — Out of memory
- `timeout` — Connection/request timeouts
- `connection_error` — Connection refused/reset
- `http_5xx` — HTTP 500-level responses
- `python_traceback` — Python tracebacks
- `exception` — Generic exceptions
- `auth_error` — Permission/authorization failures
- `structured_error` — JSON-structured error logs
- `error` — Generic ERROR-level messages

Health check noise (GET/HEAD `/health` from ALB) is automatically filtered from error counts and HTTP status distribution.
