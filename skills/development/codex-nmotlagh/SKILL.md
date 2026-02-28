---
name: codex
description: Use the OpenAI Codex CLI (codex exec) as a coding agent for writing code, debugging, code review, and automated refactoring. Always runs with high reasoning effort for maximum depth.
---

# Codex CLI Skill

Use the OpenAI Codex CLI to delegate coding tasks that benefit from fast, focused code generation, debugging, and review.

## When to Use Codex

- **Code review** — reviewing diffs, files, or modules for bugs and edge cases
- **Writing new functions or modules** where requirements are clear
- **Debugging** — Codex has found nuanced bugs that are easy to miss
- **Targeted refactors** — renaming, restructuring single files or small groups
- **Boilerplate generation** — tests, config files, repetitive patterns

## When NOT to Use Codex

- Tasks requiring multi-file architectural reasoning across the whole repo
- Interactive exploration or codebase understanding (use Claude directly)
- Tasks that need internet access on an airgapped cluster
- When Codex is not installed (`which codex` returns nothing)

## Installation

```bash
npm i -g @openai/codex
```

Requires Node.js 20+. Authenticate on first run (ChatGPT Plus/Pro/API key).

## Required Flags

Every invocation MUST include these flags:

| Flag | Value | Why |
|------|-------|-----|
| `--full-auto` | (no value) | Non-interactive, no approval prompts |
| `-m` | `gpt-5.3-codex` | Best model for code reasoning |
| `-c` | `reasoning_effort="high"` | Maximum depth for catching subtle bugs |

## Core Usage

### Non-Interactive Execution

```bash
# Basic task
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" "your task here"

# Set working directory
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  -C ~/binom-abstain "your task here"

# Read prompt from stdin
echo "fix the failing test in tests/test_eval.py" | codex exec --full-auto \
  -m gpt-5.3-codex -c reasoning_effort="high" -

# Save output to file
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  -o result.md "explain what src/train/sft.py does"
```

## Code Review Patterns

### Quick Diff Review
Review only what changed — fast, focused.
```bash
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Review the uncommitted changes. Look for bugs, edge cases, and correctness issues."
```

### Deep File Review
Review an entire file for latent bugs, not just recent changes.
```bash
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Do a thorough review of src/pipeline/run_pipeline.py. Check for bugs, race conditions, and edge cases."
```

### Read-Only Review (No Edits)
Use `--sandbox read-only` to ensure Codex only reads and reports, never modifies files.
```bash
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  -s read-only \
  "Review src/train/sft.py. Report issues but do not modify any files."
```

### Multi-File Module Review
```bash
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Review all files in src/eval/ for correctness and consistency."
```

### Staged / PR Review
```bash
# Review staged changes (pre-commit)
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Review the staged git changes. Flag any issues before I commit."

# Review a PR diff against main
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Review the diff between main and HEAD. Summarize changes and flag problems."
```

## Debugging Patterns

```bash
# Point Codex at a failing test
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "the test tests/test_sft.py::test_lora_training is failing. diagnose and fix the bug."

# Focused debugging on a specific issue
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Review src/sampling/generate.py focusing on OOM risks and GPU memory usage."
```

## Writing & Refactoring Patterns

```bash
# Implement a new function
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Implement function X that does Y. File: src/module/file.py"

# Targeted refactor
codex exec --full-auto -m gpt-5.3-codex -c reasoning_effort="high" \
  "Refactor src/train/sft.py to extract the data loading into a separate function."
```

## All Flags Reference

| Flag | Short | Description |
|------|-------|-------------|
| `--model` | `-m` | Model to use (always `gpt-5.3-codex`) |
| `--full-auto` | | No approvals, workspace-write sandbox |
| `--config` | `-c` | Inline config (e.g., `reasoning_effort="high"`) |
| `--cd` | `-C` | Set working directory |
| `--output-last-message` | `-o` | Write final response to file |
| `--sandbox` | `-s` | `read-only`, `workspace-write`, `danger-full-access` |
| `--ask-for-approval` | `-a` | `untrusted`, `on-failure`, `on-request`, `never` |
| `--image` | `-i` | Attach screenshot/image to prompt |
| `--json` | | Emit newline-delimited JSON events |
| `--skip-git-repo-check` | | Run outside a git repo |
| `--search` | | Enable live web search |

## Resuming Sessions

```bash
codex exec resume --last        # Most recent session
codex exec resume <SESSION_ID>  # Specific session
```

## Guidelines

- Always verify Codex is installed first: `which codex`
- Always use `high` reasoning — research code demands maximum scrutiny
- Use `--sandbox read-only` when you only want a report, not fixes
- Use `-o review.md` to capture reviews for later reference
- Break large tasks into focused chunks (one module or file group at a time)
- After Codex makes changes, run `python -m pytest tests/ -v` to verify
- Review Codex output before committing — it may introduce subtle issues in unfamiliar codebases
- Codex cannot access the internet — it works only with local files
