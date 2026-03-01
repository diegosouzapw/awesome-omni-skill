---
name: openai-usage
description: Report current OpenAI usage/rate-limit health from Codex ChatGPT limits.
---

# OpenAI Usage Skill

Use this skill when the user asks things like:
- "what is my OpenAI usage at?"
- "show my OpenAI limits"
- "how much OpenAI budget do I have left?"

## Command

- Human-readable output:
  - `node skills/openai-usage/check-openai-usage.js`
- JSON output:
  - `node skills/openai-usage/check-openai-usage.js --json`

## Data Source

- Codex app-server `account/rateLimits/read` (ChatGPT mode)
- No manual fallback. If Codex rate limits are unavailable, return an actionable error.

## Behavior

- Reports 5-hour and weekly remaining percentages.
- Computes one health stage (`healthy`, `medium`, `low`, `critical`) using separate 5-hour and weekly thresholds.
- Stage rules:
  - `critical` if 5-hour <= 10% OR weekly <= 10%
  - `low` if 5-hour <= 25% OR weekly <= 20%
  - `medium` if 5-hour <= 50% OR weekly <= 30%
  - otherwise `healthy`
