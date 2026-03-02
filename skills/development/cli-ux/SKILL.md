---
name: cli-ux
description: "Agent-first CLI design: JSON envelope output, Rich human UX, progress indicators, and dual-mode routing."
version: 1.0.0
category: dev
tags: [cli, ux, json, rich, terminal, agent-first]
metadata:
  ai-engineering:
    scope: read-write
    token_estimate: 900
---

# CLI UX Design

## Purpose

Agent-first CLI design skill for commands that serve both machine consumers (JSON envelopes via `--json`) and human operators (Rich terminal output). Covers the dual-output routing pattern, structured JSON envelopes with HATEOAS next actions, Rich progress indicators, data model conventions, and terminal visual language used across the `ai-eng` CLI.

## Trigger

- Command: agent invokes cli-ux skill or user requests CLI command design/review.
- Context: new CLI command, CLI output review, adding `--json` support, improving terminal UX, adding progress indicators.

## When NOT to Use

- **API endpoint design** — use `dev:api-design` for REST/GraphQL contract-first design.
- **CI/CD pipeline configuration** — use `dev:cicd-generate` for pipeline setup.
- **General code review** — use `dev:code-review` for implementation review.
- **Test strategy for CLI commands** — use `dev:test-strategy` for test design.

## Procedure

1. **Understand output context** — determine if the command needs JSON mode, human mode, or both.
   - All user-facing commands MUST support both modes (dual-output).
   - Internal/plumbing commands may be JSON-only.
   - Read `src/ai_engineering/cli_output.py` for the routing pattern.

2. **Design data model** — every result gets both `to_dict()` and `to_markdown()`.
   - `to_dict()`: returns a JSON-serializable dict for the envelope.
   - `to_markdown()`: returns a human-readable markdown string.
   - Serialization rules: `Path` → `.as_posix()`, dates → `.isoformat()`, enums → `.value`.
   - Use Pydantic models or `@dataclass` — never raw dicts as primary data structures.

3. **Implement dual-output routing** — branch on `is_json_mode()`.
   - JSON path: `emit_success(command_name, data_dict, [NextAction(...)])` → stdout.
   - JSON errors: `emit_error(command_name, message, error_code, fix, [NextAction(...)])` → stdout.
   - Human path: `result_header()`, `kv()`, `status_line()`, `suggest_next()` → stderr.
   - Rule: JSON goes to stdout, human messaging goes to stderr (CLIG guideline).
   - Use `output()` router from `cli_output.py` for clean branching.

4. **Design JSON envelope** — follow the `SuccessEnvelope`/`ErrorEnvelope` contract.
   - Success: `{ ok: true, command: str, result: dict, next_actions: [...] }`.
   - Error: `{ ok: false, command: str, error: { message, code }, fix: str, next_actions: [...] }`.
   - HATEOAS `NextAction`: `{ command: str, description: str, params?: dict }` — suggest follow-up commands.
   - Use `truncate_list(items, max_items=20)` for large collections to protect agent context windows.

5. **Add progress indicators** — wrap long operations in `spinner()` or `step_progress()`.
   - `spinner(description)`: single-step context manager for indeterminate waits.
   - `step_progress(total, description)`: multi-step tracker with `tracker.step(msg)`.
   - Auto-suppressed in JSON mode (`is_json_mode()`) and non-TTY (CI, piped output).
   - Transient by default — spinners disappear when done, leaving clean output.
   - **Gate hooks (pre-commit, commit-msg, pre-push): NEVER add progress indicators.**

6. **Apply visual language** — consistent Rich markup and color semantics.
   - Success: `[success]` (green). Error: `[error]` (red). Warning: `[warning]` (yellow). Info: `[info]` (blue).
   - Brand accent: `[brand]` (teal `#00D4AA`). Muted: `[muted]` (dim). Paths: `[path]` (teal underline).
   - Key-value pairs: `kv(key, value)`. File counts: `file_count(label, count)`.
   - Section dividers: `header(title)`. Result summary: `result_header(label, status, detail)`.
   - Next steps: `suggest_next([(command, description), ...])`.
   - Respect `NO_COLOR`, `TERM=dumb`, and non-TTY detection (handled by `get_console()`).

7. **Validate output contract** — verify both output modes work correctly.
   - JSON: output is valid JSON, parseable by `json.loads()`.
   - JSON: envelope matches `SuccessEnvelope`/`ErrorEnvelope` schema.
   - Human: output is readable, no raw dicts or repr strings.
   - Test: `--json` flag produces stdout-only JSON; human mode produces stderr-only Rich output.

## Output Contract

- CLI command with dual-output support (JSON + human).
- Data model with `to_dict()` and `to_markdown()` methods.
- JSON envelope conforming to `SuccessEnvelope`/`ErrorEnvelope`.
- Progress indicators that auto-suppress in JSON/non-TTY contexts.

## Governance Notes

- CLI output modules are framework-managed — changes require governance review.
- JSON envelope schema (`SuccessEnvelope`/`ErrorEnvelope`) is a contract — do not modify fields without versioning.
- Human output primitives (`cli_ui.py`) are shared — additions are welcome, removals require deprecation.
- All new commands must pass `--json` output validation in tests.

### Iteration Limits

- Max 3 attempts to resolve the same CLI output issue. After 3 failures, escalate to user with evidence.

### Post-Action Validation

- After implementing dual-output, verify JSON is parseable with `json.loads()`.
- Run the command with and without `--json` to confirm both paths work.
- If validation fails, fix issues and re-validate (max 3 attempts).

## References

- `standards/framework/stacks/python.md` — Python stack patterns.

### Implementation Files (repo root-relative)

Read these on-demand when implementing or reviewing CLI commands:

1. `src/ai_engineering/cli_envelope.py` — JSON envelope (`SuccessEnvelope`, `ErrorEnvelope`, `NextAction`).
2. `src/ai_engineering/cli_ui.py` — Rich human output primitives (`kv`, `status_line`, `result_header`, `suggest_next`).
3. `src/ai_engineering/cli_output.py` — dual-mode router (`is_json_mode`, `output`).
4. `src/ai_engineering/cli_progress.py` — spinner and step_progress context managers.
5. `src/ai_engineering/cli_commands/core.py` — reference implementation (install, doctor commands).
