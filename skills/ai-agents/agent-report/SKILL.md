---
name: agent-report
description: Extract and display the final message from a Claude agent JSONL file. Use when the user wants to see an agent report, view agent output, extract agent results, check what an agent produced, or read the final response from a subagent. Triggers on requests like "show me agent report", "what did agent X produce", "extract agent output", "view agent results", or "get the report from agent ad42ecb".
---

# Agent Report Extractor

Extract the final message from a Claude agent file and write it to a formatted markdown report.

## Usage

Run the script with an agent ID:

```bash
scripts/extract_agent_message.sh <agent_id> [output_file]
```

- `agent_id`: The short agent ID (e.g., `ad42ecb`)
- `output_file`: Optional. Defaults to `agent-<id>-report.md` in current directory

## Workflow

1. If the user provides an agent ID, run the script immediately
2. If no agent ID is provided, ask the user for the agent ID using AskUserQuestion
3. Run the script and confirm the output file location

## Output Format

The script produces a markdown file containing:
- Agent slug/name
- Agent ID
- Model used
- Timestamp
- Full text content of the final message
