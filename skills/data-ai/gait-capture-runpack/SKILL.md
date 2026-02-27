---
name: gait-capture-runpack
description: Capture and verify deterministic Gait runpacks from normalized run input. Use when asked to record a run, produce run_id or runpack artifacts, generate ticket-ready proof, or validate artifact integrity before handoff.
disable-model-invocation: true
---
# Capture Runpack

Execute this workflow to record an artifact safely and deterministically.

## Gait Context

Gait is an offline-first runtime for AI agents that enforces tool-boundary policy, emits signed and verifiable evidence artifacts, and supports deterministic regressions.

Use this skill when:
- incident triage needs captured run artifacts for reproduction
- CI gate failures need verifiable artifact identity and integrity
- receipt/evidence generation depends on runpack metadata and digests

Do not use this skill when:
- Gait CLI is unavailable in the environment
- no Gait run input or artifact source is available

## Workflow

1. Validate required input path: `<run_record.json>`.
2. Run record with JSON output (required):
   - `gait run record --input <run_record.json> --json`
3. Parse output fields:
   - `ok`, `run_id`, `bundle`, `manifest_digest`, `ticket_footer`
4. Verify artifact integrity (required):
   - `gait verify <run_id_or_bundle_path> --json`
5. Return a concise handoff block that includes:
   - `run_id`
   - `bundle`
   - `manifest_digest`
   - `ticket_footer`
   - verify status

## Safety Rules

- Keep default capture mode as `reference`.
- Do not switch to raw capture unless explicitly requested.
- For replay workflows, prefer `gait run replay` (stub mode default); require explicit unsafe flags for real tool replay.
- Do not invent `run_id`, digests, or verify results.
- Treat non-zero exit from `gait run record` or `gait verify` as blocking errors.
- Keep this skill wrapper-only: do not parse policy YAML or implement custom evaluator logic.

## Determinism Rules

- Always use `--json` and parse structured fields.
- Do not rely on text-only output for workflow decisions.
- Keep output grounded in recorded artifact values only.
