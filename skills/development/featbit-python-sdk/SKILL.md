---
name: featbit-python-sdk
description: Guides integration of the FeatBit Python Server-Side SDK for backend services. Use when users ask about FeatBit feature flags in Python, fbclient usage, or server frameworks like Flask, Django, or FastAPI. Not for client-side use.
appliesTo:
  - "**/*.py"
  - "**/app.py"
  - "**/main.py"
  - "**/server.py"
  - "**/views.py"
  - "**/api/**/*.py"
  - "**/backend/**/*.py"
---

# FeatBit Python Server-Side SDK

Expert guidance for integrating the FeatBit Python Server-Side SDK in backend applications. This skill is based on the official SDK README and keeps details in reference files for quick navigation.

## Activates When

- The user asks about FeatBit Python SDK setup, usage, or troubleshooting.
- The user mentions `fbclient`, `FBClient`, `variation`, `variation_detail`, or `flag_tracker`.
- The user needs server-side feature flag evaluation in Python (Flask, Django, FastAPI, etc.).

## Overview

Use this SDK to evaluate feature flags on the server. The SDK syncs flag data via websocket and stores it in memory by default. If you need a custom data source, use offline mode with bootstrap JSON.

## Core Knowledge Areas

### 1. Data Synchronization

- Websocket sync keeps local data updated (average sync < 100 ms).
- The connection can be interrupted but resumes automatically after outages.
- Offline mode disables remote sync; use bootstrap JSON if needed.

### 2. Installation and Prerequisites

- Install: `pip install fb-python-sdk` (Python 3.6–3.11).
- Required values: `env_secret`, `event_url`, `streaming_url`.
- Official FAQ: environment secret + SDK URLs.

### 3. Initialization and Client Lifecycle

- Preferred pattern: call `set_config(...)`, then `get()` for a singleton client.
- Direct instantiation: `FBClient(Config(...), start_wait=15)`.
- Async readiness: `client.update_status_provider.wait_for_OKState()`.

### 4. Evaluation and All Flags

- Evaluate: `variation()` for value, `variation_detail()` for value + reason.
- All flags: `get_all_latest_flag_variations(user)`.
- Defaults are returned when SDK is not initialized or keys are invalid.

### 5. User Model

- Use a dict with required `key` and `name`.
- Add custom properties (string, number, boolean).

### 6. Flag Change Tracking

- `flag_tracker.add_flag_value_maybe_changed_listener(...)`.
- `flag_tracker.add_flag_value_changed_listener(...)`.

### 7. Offline Mode and Bootstrap Data

- Enable: `Config(..., offline=True)`.
- Initialize with JSON: `client.initialize_from_external_json(json)`.
- Use FeatBit to export a bootstrap JSON snapshot.

### 8. Experiments (A/B/n)

- Track custom events: `client.track_metric(user, event_name, numeric_value)`.
- Ensure the flag is evaluated before tracking the metric.

## Quick Start (Concise)

```python
from fbclient import get, set_config
from fbclient.config import Config

set_config(Config(env_secret, event_url, streaming_url))
client = get()

if client.initialize:
    user = {"key": "bot-id", "name": "bot"}
    detail = client.variation_detail("flag-key", user, default=None)
    print(detail.variation, detail.reason)

client.stop()
```

For a complete walkthrough and full examples, see the reference guides below.

## Reference Guides

- [references/quick-start.md](references/quick-start.md) — installation, prerequisites, and full quick start
- [references/data-synchronization.md](references/data-synchronization.md) — websocket sync behavior and recovery
- [references/bootstrapping-and-lifecycle.md](references/bootstrapping-and-lifecycle.md) — singleton pattern, init waits, readiness
- [references/evaluation-and-tracking.md](references/evaluation-and-tracking.md) — evaluation, all-flags, change tracking, experiments
- [references/offline-mode.md](references/offline-mode.md) — offline setup and bootstrap JSON
- [references/support-and-links.md](references/support-and-links.md) — official docs, GitHub repo, support channels

## Best Practices

1. **Create one client per environment**: keep `FBClient` instances for the app lifetime.
2. **Check readiness only when needed**: evaluate with defaults if not ready, and log status for diagnostics.
3. **Prefer `variation_detail()` for debugging**: it returns evaluation reasons.
4. **Call `stop()` on shutdown**: allow pending events to flush.

## Documentation Reference

- Official Docs: https://docs.featbit.co/sdk/overview#python
- Complete Guide (full README and latest updates): https://github.com/featbit/featbit-python-sdk

## Related Topics

- FeatBit OpenFeature integration (if using OpenFeature with Python SDK)
- Server-side SDKs for other languages
