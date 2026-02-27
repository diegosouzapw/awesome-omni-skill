---
name: collaborating-with-gemini-cli
description: "Delegates code review, debugging, and alternative implementation comparisons to Google Gemini CLI (`gemini`) via a JSON bridge script (default model: `gemini-3-pro-preview`). Supports headless one-shot and multi-turn sessions via `SESSION_ID`, with conservative defaults for Gemini effective-context constraints (file-scoped, `--no-full-access` by default) while allowing user override."
---

# Collaborating with Gemini CLI

Use this skill when you want a second model (Gemini) to sanity-check a solution, spot edge cases, propose tests, or suggest an alternative implementation approach.

This skill provides a small JSON bridge script that runs `gemini` (Gemini CLI) in non-interactive **headless** mode and returns structured output.

Compared to `collaborating-with-claude-code`, this skill defaults to **read-only** and is optimized for **file-scoped, one-shot** requests to avoid practical effective-context degradation.

## Requirements

- Gemini CLI installed (`gemini --version`).
  - Install via npm: `npm i -g @google/gemini-cli`
- Gemini CLI authenticated (Google account login or API key auth, depending on your local setup).
- Python 3 (to run the bridge script).

## Quick start

```bash
python scripts/gemini_cli_bridge.py --cd "/path/to/repo" --PROMPT "Review src/auth/login.py for bypasses; propose fixes as a unified diff."
```

Recommended (explicit file scope, best for effective context):

```bash
python scripts/gemini_cli_bridge.py --cd "/path/to/repo" --file "src/auth/login.py" --PROMPT "Review this file for bypasses; propose fixes as a unified diff."
```

## Multi-turn sessions

Always capture the returned `SESSION_ID` and pass it back on follow-ups:

```bash
# Start a new session (one-shot by default)
python scripts/gemini_cli_bridge.py --cd "/repo" --file "src/auth/login.py" --PROMPT "Summarize issues and propose a patch."

# Continue the same session
python scripts/gemini_cli_bridge.py --cd "/repo" --SESSION_ID "uuid-from-response" --PROMPT "Now propose 5 targeted tests for the fix."
```

## Access modes

This skill defaults to `--no-full-access` (read-only).

- **Read-only (default)**: `--no-full-access` → maps to `gemini --approval-mode default`
- **Full access (edits allowed)**: `--full-access` → maps to `gemini --approval-mode auto_edit`
- **YOLO (auto-approve everything)**: `--yolo` → maps to `gemini --approval-mode yolo`

Examples:

```bash
# Allow edits (auto-approve edit tools)
python scripts/gemini_cli_bridge.py --full-access --cd "/repo" --file "src/foo.py" --PROMPT "Refactor for clarity; keep behavior; apply edits."
```

```bash
# YOLO mode (dangerous): allow any tool calls without confirmation
python scripts/gemini_cli_bridge.py --yolo --cd "/repo" --PROMPT "Run tests, fix failures, and apply edits."
```

## Effective-context guardrails (default ON)

By default the bridge enables conservative guardrails designed for **practical effective context**:

- Adds a preamble instructing Gemini to **stop and ask** before reading additional files.
- Strongly encourages **file-scoped** runs (use `--file` and keep each call small).
- Uses `--max-files` as a **preference** for “how many files per turn” and as a **cap for auto-extracted files** (only when you did not pass explicit `--file`).

User override options:

- `--file PATH` (repeatable): explicitly decide the focus file set (recommended; not blocked by `--max-files`).
- `--max-files N`: raise the preferred cap / auto-extraction cap.
- `--no-guardrails`: disable guardrails (Gemini may read many files; treat like a normal agent).

### Session rotation guidance (effective context)

The bridge exposes Gemini CLI token stats when available:

- `meta.prompt_tokens`: prompt tokens reported by Gemini CLI
- `meta.over_effective_context_limit`: `true` when `prompt_tokens > --effective-context-tokens`

When `meta.over_effective_context_limit` is `true`, prefer starting a **new session** for the next turn (omit `--SESSION_ID`) and/or reduce the focus scope.

## Parameters (bridge script)

- `--PROMPT` (required): Instruction to send to Gemini.
- `--cd` (required): Working directory to run Gemini CLI in (typically repo root).
- `--SESSION_ID` (optional): Resume an existing Gemini CLI session (uuid).
- `--model` (optional): Defaults to `gemini-3-pro-preview`.
- `--full-access` / `--no-full-access` (optional): Defaults to `--no-full-access`.
- `--yolo` (optional): YOLO approval mode (implies full access).
- `--sandbox` (optional): Run Gemini CLI in sandbox mode (requires Docker). Default: off.
- `--return-all-messages` (optional): Return the full streamed event list (debugging).
- `--file PATH` (repeatable): Focus files to prepend as `@PATH` (recommended).
- `--max-files` / `--no-guardrails` / `--effective-context-tokens`: Guardrail controls (`--max-files` is a preference + auto-extraction cap; it does not block explicit `--file`).
- `--timeout-s` (optional): Defaults to 1800 seconds.

## Output format

The bridge prints JSON:

```json
{
  "success": true,
  "SESSION_ID": "uuid",
  "agent_messages": "…Gemini output…",
  "all_messages": [],
  "meta": {}
}
```

`meta` includes the normalized focus file list and (when available) token stats extracted from Gemini CLI output.
