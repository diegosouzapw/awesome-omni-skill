---
name: chatgpt-archive-topic-background-report
description: Build a topic-focused research collection from ChatGPT archive viewer conversations (latest archive or all archives), run a background Responses API consolidation job with web search, and save markdown plus raw response artifacts. Use when the user asks to find archive threads by topic, reconcile repetition/contradictions, and generate a saved report with minimal polling noise.
---

# ChatGPT Archive Topic Background Report

Use this skill when a user wants end-to-end topic consolidation from ChatGPT archive data:
- Search relevant threads by topic in latest archive or all archives.
- Assemble a deduplicated JSON research collection.
- Run a background Responses API call with reasoning + verbosity + web search.
- Save markdown output and response payload to disk.

## Preconditions

- Ensure `OPENAI_API_KEY` is set (environment or `.env`).
- Ensure outbound access to `api.openai.com` is available.
- Confirm `chatgpt_viewer_sites/` exists with archive folders.

## Core Workflow

1. Run the pipeline script with topic and archive scope:

```bash
python -u .agents/skills/chatgpt-archive-topic-background-report/scripts/run_topic_report.py \
  --topic "OpenClaw / Moltbot / Clawdbot" \
  --archive-scope latest
```

2. For all archives instead of latest:

```bash
python -u .agents/skills/chatgpt-archive-topic-background-report/scripts/run_topic_report.py \
  --topic "OpenClaw / Moltbot / Clawdbot" \
  --archive-scope all
```

3. Add custom prompt constraints (topic-specific or ad hoc):

```bash
python -u .agents/skills/chatgpt-archive-topic-background-report/scripts/run_topic_report.py \
  --topic "OpenClaw / Moltbot / Clawdbot" \
  --archive-scope all \
  --ad-hoc-requirements "Prioritize security incidents and timeline precision."
```

## Quiet Monitoring Rule

- Keep polling output minimal by default.
- The script prints status changes and sparse heartbeats only.
- Use `--verbose-poll` only if detailed polling lines are explicitly needed.

## Key Parameters

- `--archive-scope latest|all`: search latest archive or all archive folders.
- `--keyword-regex`: override keyword matching regex.
- `--keyword`: add extra keyword terms (repeatable).
- `--model`: default `gpt-5.2`.
- `--reasoning-effort`: `none|low|medium|high|xhigh`.
- `--verbosity`: `low|medium|high`.
- `--search-context-size`: `low|medium|high` for web search tool.
- `--ad-hoc-requirements` / `--ad-hoc-requirements-file`: append run-specific requirements.

## Reporting Standard

- The default developer prompt template enforces an academic/journal-style report structure.
- Reports must state they are based on the provided research collection, augmented by targeted web search only when contradictions or ambiguities remain.
- Reports should avoid vague, repetitive references to "the dataset" and instead use precise source-language.

## Outputs

Each run writes an output directory under `scratchpad/topic_background_reports/<timestamp>_<topic>/` with:
- `assembled_dataset.json` (JSON artifact containing the provided research collection)
- `developer_prompt_effective.txt`
- `user_task_effective.txt`
- `report.md`
- `response_payload.json`
- `run_manifest.json`

## Bundled Resources

- `scripts/run_topic_report.py`: complete pipeline (assemble -> background API -> poll -> save outputs).
- `references/developer_prompt_template.txt`: base developer prompt template with `{{TOPIC}}` placeholder.
