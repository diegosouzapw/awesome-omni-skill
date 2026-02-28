---
name: adaptive-guardrail-calibrator
description: Calibrate guardrail thresholds from live hardware telemetry and emit environment presets. Use when thresholds are hand-tuned or drift with hardware changes.
---

# Adaptive Guardrail Calibrator

Use this skill to compute safe resource floors and caps from live telemetry.

## Workflow

1) Sample CPU, RAM, disk, and GPU metrics for a short window.
2) Compute conservative floors and caps.
3) Write environment presets for guardrails.

## Scripts

- Run: python scripts/calibrate_guardrails.py --duration 30 --interval 1 --apply

## References

- references/guardrail_defaults.json
