---
name: ops-detection-incident-routing
description: Detect agent runtime anomalies and route incidents through approval-safe guardrails. Use when you need deterministic checks for cron failures, context pressure, dangling sessions, token spikes, and a controlled incident workflow (detect -> route -> investigate -> remediate).
homepage: https://github.com/your-org/openclaw-public-skills
metadata: {"clawdbot":{"emoji":"🛟","requires":{"bins":["bash","jq","python3"]}}}
---

# ops-detection-incident-routing

Run deterministic operations checks and route incidents with guardrails.

This skill ships a small toolkit for:

1. detecting runtime anomalies from local state/log files
2. applying in-flight + cooldown guards
3. emitting structured incident actions for investigator/remediator flows

## Use This Skill

Use this skill when you need a production-safe ops loop for agent systems and do not want ad-hoc prompt-only monitoring.

## Files

- `scripts/ops-threshold-detector.sh`
  reads session/cron/snapshot state and appends detector JSONL events
- `scripts/incident-guard-check.sh`
  checks in-flight/cooldown guard status for a check id
- `scripts/incident-state-update.sh`
  updates guard state for start/complete/fail transitions
- `scripts/ops-incident-router.sh`
  converts detector alerts into structured actions
- `scripts/ops-detector-cycle.sh`
  detector + router cycle runner
- `scripts/setup.sh`
  dependency checks + local example scaffold
- `scripts/clean-generated.sh`
  removes generated `.jsonl` and lock artifacts before republishing from a used folder

## Setup

```bash
bash scripts/setup.sh
```

## Quick Start

Run one full dry-run cycle:

```bash
bash scripts/ops-detector-cycle.sh \
  --workspace "$(pwd)/examples/workspace" \
  --state-file "$(pwd)/examples/incident-state.json" \
  --detector-out "$(pwd)/examples/ops-detector.jsonl" \
  --router-out "$(pwd)/examples/router-actions.jsonl"
```

Run live mode (router also acquires in-flight locks):

```bash
bash scripts/ops-detector-cycle.sh \
  --workspace "$(pwd)/examples/workspace" \
  --state-file "$(pwd)/examples/incident-state.json" \
  --detector-out "$(pwd)/examples/ops-detector.jsonl" \
  --router-out "$(pwd)/examples/router-actions.jsonl" \
  --live
```

## Output Contract

Detector writes one JSON line per run:

```json
{
  "ts": "2026-02-24T02:30:00Z",
  "status": "ALERT",
  "checks": 5,
  "alerts": [{"sev":"Sev-2","trigger":"cron_failure","value":2,"threshold":0}],
  "gaps": []
}
```

Router emits one JSON action per alert decision:

```json
{"action":"spawn","check_id":"cron_failure","severity":"Sev-2","mode":"dry-run","task":"Investigate incident: cron_failure"}
```

## Operational Pattern

1. schedule `ops-threshold-detector.sh` (every 5-15 min)
2. feed the latest detector line to `ops-incident-router.sh`
3. spawn investigator/remediator only from router output
4. keep remediation behind explicit owner approval

For details, read `references/architecture.md`.
