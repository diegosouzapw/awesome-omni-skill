---
name: skill-claude-code
description: Use when the user asks to run Claude Code CLI (claude) for code analysis, refactoring, debugging, or automated edits, or references Claude Code workflows, sessions, permissions, or CLI flags.
---

# Claude Code

## Overview

Run Claude Code from the terminal to analyze, plan, refactor, or edit codebases via the `claude` CLI. Default to non-interactive print mode in this environment and summarize results back to the user.

## Quick Start

1. Use non-interactive (print) mode: `claude -p "prompt"` for one-shot tasks. Interactive sessions are disabled here due to flaky TTY prompts.
2. Select model (ask the user). Default suggestion: `opus`.
3. Choose permissions and scope.
4. Run the command, capture output, and summarize key results and next steps.

## Workflow

### 1) Ask for model and session intent
- Ask the user which model to use.
- If the user doesn't specify, propose `--model opus` and confirm.
- `--model` accepts `sonnet` or `opus` aliases, or a full model name.

### 2) Pick the right command form

Non-interactive / print mode (default):
- `claude -p "prompt"` to run a single request and exit.
- `cat file | claude -p "prompt"` to pipe content in.
- Use `--output-format json` or `--output-format stream-json` for machine-readable output when scripting.
- Use `--max-turns N` to limit agentic turns in print mode.
- Use `--input-format` for `text` or `stream-json` inputs.
- Use `--no-session-persistence` to avoid saving sessions in print mode.
- Use `--include-partial-messages` with `--output-format stream-json` to emit partial streaming events.

Session management (print mode only):
- Continue the most recent session in print mode: `claude -c -p "prompt"`.
- Resume a specific session in print mode: `claude -r "<session-id>" -p "prompt"`.
- `--fork-session` creates a new session ID when resuming.

### 3) Permissions and safety

- Prefer default permission prompts. Only use `--dangerously-skip-permissions` if the user explicitly approves and the task is low-risk or sandboxed.
- Use `--permission-mode` to start in a specific mode and `--allow-dangerously-skip-permissions` only to permit the bypass option.
- For larger changes, explore first, then plan, then implement.
- Use `--allowedTools`/`--disallowedTools` or `--tools` to control tool access when needed.

### 4) Run, summarize, and follow up

- Execute the command, capture stdout (and stderr if needed).
- Summarize results, list changes or findings, and ask the user for next steps.

## Notes

Assume the user already has Claude Code installed and configured. If `claude` is not found, report the error and ask the user to fix their environment before retrying.
When prompting, prefer rich context: reference files with `@` and pipe logs.

## Troubleshooting

If the prompt passed to `--print` / `-p` is misinterpreted (e.g., flags inside the prompt), use `--` before the prompt or pipe it via stdin:
- `claude -p -- "prompt with -dashes or --flags"`
- `echo "prompt" | claude -p`
