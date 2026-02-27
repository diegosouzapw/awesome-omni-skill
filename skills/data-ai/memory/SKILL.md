---
name: memory
description: Save and retrieve experiment context using the local Obsidian vault. Use to preserve context across sessions and reduce context window usage.
disable-model-invocation: true
---

# Obsidian Memory Skill

Use a local Obsidian vault to maintain persistent context and memory across Claude Code sessions.

## Goals
- Reduce context window usage by storing detailed experiment logs and summaries in Obsidian.
- Maintain a searchable knowledge base of all experiments, results, and insights.
- Automate the creation of daily summaries and pipeline execution logs.

## When to use
- **Before ending a session**: Save a summary of current progress.
- **After an experiment finishes**: Use `experiment_summary` to record results.
- **When switching tasks**: Load relevant context from previous notes.
- **Daily**: Review or append to the daily summary.

## Tools
- `save_context(topic, content, tags, category)`: Save notes to Obsidian.
- `load_context(topic_or_query)`: Retrieve notes or search.
- `experiment_summary(exp_id)`: Generate a structured summary from an experiment ID.

## Folder Structure
- `experiments/`: One note per experiment.
- `pipeline-runs/`: Logs of automated runs.
- `daily/`: Daily activity summaries.
- `templates/`: Templates for new notes.

## Workflow
1. **Record**: As you work, use `save_context` to jot down observations.
2. **Summarize**: When an experiment is done, run `experiment_summary`.
3. **Recall**: At the start of a session, use `load_context` with a query like "yesterday" or "baseline" to get up to speed.
