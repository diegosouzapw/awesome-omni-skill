---
name: asking-gemini
description: Architecture advice, design trade-offs, brainstorming, comparing approaches via Gemini. Use when user asks about architecture decisions, system design, design patterns, trade-offs analysis, brainstorming ideas, comparing options, or creative problem-solving. Supports SCAMPER, design thinking, divergent/convergent thinking methodologies. Do not use for web searches or shell commands.
allowed-tools: Task
---

# Gemini Consultation

Spawn the **gemini-consultant** agent for architecture and design questions.

## Foreground (blocking)

```
Task(subagent_type="gemini-consultant", prompt="[mode]: <question>")
```

## Background (for context efficiency)

```
Task(subagent_type="gemini-consultant", prompt="[mode]: <question>", run_in_background=true)
```

Use `TaskOutput(task_id="<id>")` to retrieve results.

**Modes:** prompt (default), brainstorm, review, compare

**Brainstorming Methodologies:**

- `divergent` - Generate many diverse ideas
- `convergent` - Refine and evaluate existing ideas
- `scamper` - Substitute, Combine, Adapt, Modify, Put to other uses, Eliminate, Reverse
- `design-thinking` - Human-centered, empathy-driven approach
- `lateral` - Unexpected connections, challenge assumptions
- `auto` - AI selects best methodology

The agent uses Gemini MCP tools (`mcp__gemini__gemini`, `mcp__gemini__brainstorm`) directly.
