---
name: codex-sessions-skill-scan
description: "Daily skill health scan: analyze ~/.codex/sessions plus per-repo session logs under ~/dev (default last 1 day) and summarize skill invocations + likely failures for personal skills in ~/dev/agent-skills (missing paths, tool failures, complex-task word triggers). Optional: include best-effort local OTel signals."
knowledge_graph_profile: references/task-profile.json
---

# Codex Sessions Skill Scan

## Overview
This skill runs a **daily scan** over Codex session logs (global `~/.codex/sessions` and per-repo sessions under `~/dev`) to catch repeated “paper cuts” when using skills (broken file paths, missing scripts, validation commands that don’t run) and produces a short report + suggested fixes. It also highlights user words that often indicate complex tasks and adds a step-by-step reminder.

## Scope and triggers
- “Scan yesterday’s Codex sessions for skill failures (personal skills only).”
- “Why does `$some-skill` keep failing? Look at recent session logs and tell me what’s wrong.”
- “Daily check: any broken skill references or validation commands in the last 24 hours?”

## Quick start (daily)
Run:
```bash
cd ~/dev/agent-skills
python3 utilities/codex-sessions-skill-scan/scripts/scan_codex_sessions.py --days 1 --include-otel
```

## Philosophy
- **Single-threaded:** scan → summarize → (optional) patch only the smallest diff.
- **Evidence-led:** cite the specific error snippets + point to the exact personal SKILL.md path when possible.
- **Safety-first:** redact secrets; do not copy full transcripts.

## Required inputs
- `--days <float>`: how far back to scan (default `1`).
- `--sessions-root <path>`: where to scan (default `~/.codex/sessions`).
- `--include-dev-project-sessions` (enabled by default): also scan per-project `.codex/sessions` under `~/dev`. Use `--no-include-dev-project-sessions` to disable.
- `--max-samples-per-skill <int>`: cap snippets per skill (default `3`).
- `--include-otel`: include best-effort OTel signals (Codex OTLP endpoint listening status + repo-local OTLP-derived trace artifacts under `.narrative/trace/`).
- `--include-otel-collector`: include summary from `~/.agents/otel-collector/data/processed/stats.json`.
- `--otel-collector-stats <path>`: override collector stats path (default `~/.agents/otel-collector/data/processed/stats.json`).
- `--codex-config-toml <path>`: path to read Codex `[otel]` endpoints from (default `~/.codex/config.toml`).

## Deliverables
- A “daily skill health report” in Markdown (skills invoked, skills with issues, sample error snippets).
- A list of *suggested fix patterns* (no changes applied).

## Constraints
- Personal skills only: do not patch `skills-system/` or `.system` skills.
- Do not auto-install dependencies or change system settings as part of scanning.
- Keep output small (snippets only); avoid dumping raw logs.
- Redact secrets/sensitive data by default; never paste tokens/keys from logs into chat or files.

## Reliability hardening (from recurring failures)
- **rg/fd preflight:** before any repo-wide search commands, run `command -v rg` and `command -v fd`. If missing, report the missing binary and stop (or use absolute paths such as `/opt/homebrew/bin/rg` and `/opt/homebrew/bin/fd` when available).
- **No direct network curl:** do not run external `curl` commands in this workflow. Use local scripts/MCP tools instead.
- **TTY for interactive/streaming commands:** if a command expects stdin or may run interactively (for example auth/login flows), run it with `tty=true`; otherwise avoid `write_stdin` follow-ups.
- **Claude auth check:** when `claude_projects` emits auth failures, verify with `claude auth status` and treat `loggedIn=false` as environment/auth state, not a skill regression.

## Procedure
### A) Scan (read-only)
1) Run the scan script (Quick start).
2) If exit code is `2`, issues were found (see report).

### B) Triage (human-in-the-loop)
For each flagged skill:
1) Identify if it’s a **personal skill** (lives under `~/dev/agent-skills/` and is *not* under `skills-system/`).
2) Confirm whether the failure is:
   - wrong file path in a SKILL.md reference
   - wrong interpreter for validation scripts (PyYAML required)
   - `~` not expanding (needs absolute path)
   - missing/moved file in the repo

### C) Patch (optional; requires explicit user request)
If the user asks to apply fixes:
1) Make the smallest possible edits to the referenced personal skill files.
2) Re-run skill gates (fail fast on first failure):
```bash
cd ~/dev/agent-skills
~/.venvs/pyyaml/bin/python utilities/skill-creator/scripts/quick_validate.py <skill-folder>
~/.venvs/pyyaml/bin/python utilities/skill-creator/scripts/skill_gate.py <skill-folder>
```

## Guardrails (non-negotiable)
- Personal skills only: do not patch `skills-system/` or `.system` skills.
- Redact: do not paste secrets/tokens from session logs into chat, files, or issues.
- Do not auto-fix: always propose patches first; only apply after confirmation.

## Anti-patterns
- Editing skills automatically just because the scan found an issue.
- “Fixing” system skills (`skills-system/`, `.system`) when the user asked for personal skills only.
- Copy/pasting full session logs into chat/issues (high risk for secrets/PII).

## Examples
- “Scan my Codex sessions from the last day and tell me if any skills are failing. Personal skills only.”
- “Why does `$product-spec` keep referencing missing files? Scan yesterday’s sessions and suggest the smallest fix.”
- “Daily scan: any broken validation commands or missing paths from the last 24 hours?”

## Validation
- Fail fast: stop at the first failed gate, fix it, then re-run.
- This skill’s scan script is stdlib-only; run it with `python3`.
- When changing skill files, validate with:
  - `~/.venvs/pyyaml/bin/python utilities/skill-creator/scripts/quick_validate.py utilities/codex-sessions-skill-scan`
  - `~/.venvs/pyyaml/bin/python utilities/skill-creator/scripts/skill_gate.py utilities/codex-sessions-skill-scan`

References used by skill-gate:
- `references/contract.yaml`
- `references/evals.yaml`

<!-- skill-score-boost-v1 -->
## Philosophy and tradeoffs
- Use this skill when consistent decision-making matters more than one-off execution because project context should drive the approach.
- Principle and mindset: prioritize tradeoffs and constraints over rigid checklists; understand why each step exists.
- Ask this to keep outcomes robust: Why is this the right default, and what could change this outcome?
- How do we adapt if constraints shift?
- What evidence is needed before choosing one path over another?

## Anti-patterns and caveats
- Avoid applying this playbook generically without checking repository-specific context.
- **NEVER** skip required validation gates when behavior changes.
- **DO NOT** use this skill as a rigid replacement for engineering judgment.
- **DON'T** ignore warnings or assume one pattern fits all repos.
- Common pitfall: treating anti-patterns as optional.
- Incorrect assumptions here can lead to fragile guidance.
- Warning: wrong sequencing can create avoidable regressions.

## Variation and adaptation
- Vary the workflow by team size, risk, and deployment target.
- Use different strategies for small, medium, and large changes.
- Adapt recommendations to the specific environment and avoid repetitive templates.
- Avoid generic or cookie-cutter responses; craft context-specific alternatives.
- Keep outputs diverse and not repetitive.
- Converge on a custom path only after evidence review.
- Different constraints should produce different, non-generic recommendations.

## Empowering execution style
- Be capable of exploring multiple options and enabling the team to make safe decisions.
- Unlock confidence by explaining options and tradeoffs clearly.
- Feel free to be creative while staying rigorous and precise.
- Push boundaries with practical alternatives when simple recipes fail.
- Enable outcomes-oriented problem solving.
