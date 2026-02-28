---
name: dig
description: Investigation-only mode for understanding bugs, features, or system behavior without making changes. Use when the user says phrases like "let's investigate", "we need to figure out", or "we're debuggin", or asks to investigate or analyze before changing anything.
---

# Dig

## Overview

Run investigation-only mode to understand the situation before making any changes. Read files, gather context, and ask questions, but do not modify anything unless the user explicitly approves the specific change.

## Trigger Phrases

Use this skill when the user indicates investigation or debugging intent, including:

- "let's investigate"
- "we need to figure out"
- "we're debuggin"
- Any request to investigate, analyze, or understand before changing code or configs

## Operating Rules

- Do not edit files, run write operations, or make changes by default.
- Read files, inspect code, run read-only commands, and ask clarifying questions.
- If a change is needed, explain what you want to change and why, then ask for explicit permission.
- Treat permission as single-use: if additional changes are needed later, ask again.
- Stay in investigation mode until the user explicitly ends it.

## Investigation Workflow

1. Clarify the goal and constraints (what needs to be understood, scope, deadlines).
2. Gather context by reading relevant files, logs, configs, or outputs.
3. Ask focused questions to fill gaps (repro steps, environment, expected vs actual).
4. Summarize current understanding and list candidate causes or hypotheses.
5. Propose next investigative steps; request permission before any change.

## Exiting Investigation Mode

Exit only when the user explicitly says something like "ok, we're done with investigation". If unclear, ask for confirmation before changing anything.
