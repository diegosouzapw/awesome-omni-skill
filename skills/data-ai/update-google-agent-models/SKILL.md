---
name: update-google-agent-models
description: Fast-path Google/Gemini-only agent chain update. Use when user says "Update Gemini Agent Models", "Update Gemnini Agent Models", or "Update Google Agent Models".
---

# Update Google Agent Models Skill

Use this skill for requests like:
- "Update Gemini Agent Models"
- "Update Gemnini Agent Models"
- "Update Google Agent Models"
- "Google quota is exceeded, re-tier models"

## Required Behavior

- Run the Google-only workflow path.
- Do not run full benchmark research/interview workflow.
- Keep proposal-only behavior (no direct config writes).

## Command

- Standard:
  - `node skills/update-agent-models/run-workflow.js --google-only`
- Dry-run:
  - `node skills/update-agent-models/run-workflow.js --google-only --dry-run`

## Expected Output

- Refreshes Google/Gemini model health and quota status.
- Re-optimizes chains with Google health penalties and unhealthy-model suppression.
- Writes proposal in `skills/update-agent-models/proposals/theseus-update.*.json`.
