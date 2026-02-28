---
name: codex-agent
description: Invoke OpenAI Codex CLI for coding and complex tasks. ALWAYS use the codex_agent_direct TOOL — NEVER exec codex directly. Direct codex exec opens an interactive TUI that hangs and gets killed. The tool handles non-interactive execution correctly.
metadata: {"openclaw":{"requires":{"bins":["codex"]},"emoji":"🤖"}}
---

# Codex Agent — Direct Tool

**CRITICAL: NEVER run `exec codex ...` directly.** Running codex without the tool opens an interactive TUI that hangs and gets killed with signal 9. Always use the `codex_agent_direct` tool below.

**Use your own model first.** Only delegate to Codex when the user explicitly asks, or the task is large/multi-file.

## Primary Method: `codex_agent_direct` Tool

The **`codex_agent_direct`** tool invokes Codex CLI non-interactively (`codex exec`) with structured JSON output. This is the preferred method for all Codex interactions.

```
codex_agent_direct(
  prompt: "Your detailed task description",
  workspace: "/path/to/project",    // optional, passed as --cd
  model: "gpt-4.1-mini",            // optional, default "gpt-4.1-mini"
  mode: "agent" | "plan" | "ask"    // optional, default "agent"
)
```

### Modes

- **agent** (default): Execute code changes and commands. Codex runs with `auto` approval — it reads, edits, and runs commands in the workspace. Use for "implement", "refactor", "fix" tasks.
- **plan**: Plan before executing. Passes `--plan` to `codex exec`. Codex proposes a plan before making changes. Use when the user says "plan first", "design approach", or for large/ambiguous tasks.
- **ask**: Read-only / suggest-only. Codex is set to `suggest` approval mode — it explains and proposes but does not auto-run. Use for "explain", "where is", "why does" — no edits.

### When to Use

- User explicitly asks for "Codex", "openai codex", or "use Codex"
- **Coding tasks**: write, edit, refactor, review code
- **Complex multi-file operations**: Codex understands full codebase context
- **Analysis**: deep code analysis, dependency checking, architecture review
- Task is large, multi-file, or benefits from GPT-5 Codex reasoning

### Model Selection

| Model | Use when |
|---|---|
| `gpt-4.1-mini` | Default — fast, capable, cost-effective |
| `gpt-4.1` | Complex reasoning, large codebases |
| `gpt-5.3-codex` | Maximum capability (requires ChatGPT Pro/Plus auth) |

### Output

The tool returns structured JSON:
- `output`: Codex's text response (may include JSON transcript from `--json` flag)
- `exit_code`: 0 for success
- `duration_ms`: execution time
- `files_created`: list of new file paths detected after the run
- `truncated`: true if output was capped at 100KB

## There Is No Fallback — Handle Errors Directly

**DO NOT exec codex under any circumstances.** Running `codex` or `exec codex ...` opens an interactive TUI that hangs forever and gets killed with signal 9. There is no safe direct exec fallback.

### When the tool returns `ok: false` or an error:

1. **Read the `error` or `output` field** — it contains the failure reason (e.g. "not a trusted directory", "model not found")
2. **Report it to the user directly** — e.g. "Codex failed: [reason]. You may need to run `codex` interactively first to trust this directory."
3. **Do NOT** retry with exec, sessions_spawn, process polling, or any other approach
4. **Do NOT** start a background process and poll it

### When the tool returns empty output:

Report to the user: "Codex returned no output. The workspace may not be a trusted git repo, or Codex may need re-authentication."

### When output is raw JSONL:

The tool parses JSONL and returns clean text. If you still see raw `{"type":...}` JSON in the output, just summarize what you can see and tell the user the response format was unexpected.

### Common errors and what to tell the user:

| Error | Tell user |
|---|---|
| `Not inside a trusted directory` | Run `codex` interactively in the workspace first to trust it, or pass `--skip-git-repo-check` |
| `model not supported` | Your ChatGPT account uses `gpt-5.3-codex` — omit `--model` to use config default |
| `stream disconnected` | Codex authentication may have expired — run `codex` interactively to re-login |
| `signal 9` / process killed | The codex TUI was opened instead of non-interactive exec — this means exec was called directly, which is forbidden |

## Prerequisites

- Codex CLI installed: `npm i -g @openai/codex`
- Authenticated: run `codex` once interactively to sign in with your ChatGPT account or API key
- Verify: `python3 plugin/scripts/codex_agent_direct.py --check`

## Project Config

The project includes `.codex/config.toml` with sensible defaults:
- Model: `gpt-4.1-mini`
- Approval: `on-request` (Codex asks before running commands)
- Web search: `cached`

Override per-run with `--model` or `--approval` flags.

## Do NOT

- Do NOT run `codex` interactively from exec — it opens a TUI and hangs
- Do NOT use `codex` without the `exec` subcommand for automation
- Do NOT expose API keys in prompts or command arguments
