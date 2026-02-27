---
name: agent-evaluator
description: Deterministic custom subagent selection helper. Use when you need a reproducible, auditable decision on which custom subagents to activate for a user query (runs scripts/agent_evaluator.py).
---

# Agent Evaluator

Evaluate a user query against the workspace's available subagents and return a JSON decision payload (activated/required/suggested agents and scoring).

## Mechanism

Run the evaluator script (located in scripts folder relative this skill file) with the user query as an argument.:

```shell
python scripts/agent_evaluator.py "YOUR_QUERY_HERE"
```

Optional: include a contextual file path as the second argument:

```shell
python scripts/agent_evaluator.py "YOUR_QUERY_HERE" "path/to/file.ext"
```

## Output

- Writes a JSON object to stdout.
- Key fields include:
  - `activated_agents`
  - `required_agents`
  - `suggested_agents`
  - `evaluations` (per-agent score + reasoning)

## Examples

Evaluate a query:

```shell
python scripts/agent_evaluator.py "Please help refine our custom instruction file"
```

Evaluate a query with file context:

```shell
python scripts/agent_evaluator.py "Update this instruction" "instructions/agent-forced-eval.instructions.md"
```
