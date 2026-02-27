---
name: self-improvement
description: "Mulch Self Improver — Let your agents grow 🌱. Captures learnings with Mulch so expertise compounds across sessions. Use when: command/tool fails, user corrects you, missing feature, API fails, knowledge was wrong, or better approach found. Run mulch prime at session start; mulch record before finishing. Benefits: better and more consistent coding, improved experience, less hallucination."
metadata:
---

# Mulch Self Improver — Let your agents grow 🌱

Structured expertise that accumulates over time, lives in git, and works with any agent. Agents start each session from zero; the pattern discovered yesterday is forgotten today. This skill uses [Mulch](https://github.com/jayminwest/mulch): agents call `mulch record` to write learnings and `mulch query` to read them. Expertise compounds across sessions, domains, and teammates. **Mulch is a passive layer** — it does not contain an LLM. Agents use Mulch; Mulch does not use agents.

**Benefits:** Better and more consistent coding · Improved experience · Less hallucination (grounding in project expertise)

**When to use:** Command/tool fails, user corrects you, user wants a missing feature, your knowledge was wrong, or you found a better approach — record with Mulch and promote proven patterns to project memory. **Auto-detection:** The hook now detects errors and corrections automatically and prompts to record.

**Mechanics:** One learning store — `.mulch/` (append-only JSONL, git-tracked, queryable). Session start: `mulch prime`. Recording: `mulch record <domain> --type <type> ...`. No `.learnings/` markdown files.

**Qualification (features, benefits, pain points):** See [QUALIFICATION.md](QUALIFICATION.md). **Benchmark (token efficiency, troubleshooting skill improvement):** See [BENCHMARK.md](BENCHMARK.md) — e.g. ~54% fewer chars to get same resolutions; find rate same or better; less context → fewer tokens, less noise, lower risk of wrong fix.

## New Features (v1.1)

### Auto-Detection
The hook now automatically detects learning moments:
- **Errors/failures** — When commands fail or return errors
- **Corrections** — When you say "no", "actually", "wrong", etc.
- **Retries** — When you ask to try again

The agent will prompt: "Want me to record this for next time?"

### Pre-loaded Domains
24 preset domains included in `config/domains.json`:
```
api, database, testing, frontend, backend, infra, docs, config,
security, performance, deployment, auth, errors, debugging,
workflow, customer, system, marketing, sales, content,
competitors, crypto, automation, openclaw
```

### Notifications
When a learning is recorded, you're notified via Telegram.

---

## Quick Reference

| Situation | Action |
|-----------|--------|
| Command/operation or API fails | `mulch record <domain> --type failure --description "..." --resolution "..."` |
| User corrects you / knowledge was wrong | `mulch record <domain> --type convention "..."` or `--type pattern --name "..." --description "..."` |
| Found better approach, best practice | `mulch record <domain> --type convention "..."` or `--type guide --name "..." --description "..."` |
| Architectural or tech decision | `mulch record <domain> --type decision --title "..." --rationale "..."` |
| Feature request (tracking) | `mulch record <domain> --type decision --title "..." --rationale "..."` |
| Key file/endpoint to remember | `mulch record <domain> --type reference --name "..." --description "..."` |
| Similar to existing record | Use `--relates-to <domain>:<id>` or `--supersedes`; run `mulch search "..."` first |
| Broadly applicable pattern | Promote to `CLAUDE.md`, `AGENTS.md`, SOUL.md, TOOLS.md; use `mulch onboard` for snippets |
| Session start (project has .mulch/) | Run `mulch prime` to load expertise into context |

## Mulch Setup

**Install (optional; npx works without install):**
```bash
npm install -g mulch-cli
# or: npx mulch-cli <command>
```

**Initialize in project:**
```bash
mulch init
# Quick: add all preset domains at once
cat config/domains.json | jq -r '.domains[].name' | xargs -I {} mulch add {}
# Or add individually:
mulch add api
mulch add database
mulch add testing
# add domains that match your areas: frontend, backend, infra, docs, config
```

**Provider hooks (remind agent to record):**
```bash
mulch setup cursor   # or: claude, codex, gemini, windsurf, aider
```

**Onboarding snippet for AGENTS.md/CLAUDE.md:**
```bash
mulch onboard
```

## Record Types (Mulch)

| Type | Required | Use Case |
|------|----------|----------|
| `failure` | description, resolution | What went wrong and how to avoid it |
| `convention` | content | "Use pnpm not npm"; "Always WAL mode for SQLite" |
| `pattern` | name, description | Named patterns, optional `--file` |
| `decision` | title, rationale | Architecture, tech choices, feature tracking |
| `reference` | name, description | Key files, endpoints, resources |
| `guide` | name, description | Step-by-step procedures |

Optional on any record: `--classification` (foundational | tactical | observational), `--tags`, `--relates-to`, `--supersedes`, `--evidence-commit`, `--evidence-file`, `--outcome-status` (success | failure).

## Workflow

1. **Session start:** If `.mulch/` exists, run `mulch prime` (or `mulch prime <domain>` for focus).
2. **During work:** When something fails or you learn something, run `mulch record <domain> --type <type> ...`.
3. **Before finishing:** Review; record any remaining insights with `mulch record`.
4. **Promote:** When a pattern is proven and broadly applicable, add to CLAUDE.md / AGENTS.md / SOUL.md / TOOLS.md; use `mulch onboard` to generate snippets.

## Finding Domain

- Use existing domains from `mulch status` or `mulch query --all`.
- Run `mulch learn` to get domain suggestions from changed files.
- Common domains: `api`, `database`, `testing`, `frontend`, `backend`, `infra`, `docs`, `config`.

## Recurring Patterns and Linking

- **Search first:** `mulch search "keyword"` or `mulch query <domain>`.
- **Link records:** `mulch record ... --relates-to <domain>:<id>` or `--supersedes <domain>:<id>`.
- Recurring issues → promote to CLAUDE.md/AGENTS.md or add to TOOLS.md/SOUL.md so all agents see them.

## Simplify & Harden Feed

For candidates from the simplify-and-harden skill:

1. Use `pattern_key` as a stable tag: `mulch record <domain> --type pattern --name "<pattern_key>" --description "..." --tags "simplify-and-harden"`.
2. Search first: `mulch search "<pattern_key>"`; if found, use `--relates-to` or add to existing via `mulch edit` if needed.
3. When recurrence is high, promote to CLAUDE.md/AGENTS.md/SOUL.md/TOOLS.md as short prevention rules.

## Periodic Review

- **When:** Before major tasks, after features, weekly.
- **Commands:** `mulch status`, `mulch ready --since 7d`, `mulch query --all`.
- **Actions:** Promote high-value records to project memory; run `mulch prune` for stale tactical/observational entries if desired; `mulch doctor --fix` for health.

## Promotion Targets

| Learning Type | Promote To |
|----------------|------------|
| Behavioral patterns | `SOUL.md` (OpenClaw workspace) |
| Workflow improvements | `AGENTS.md` |
| Tool gotchas | `TOOLS.md` (OpenClaw workspace) |
| Project facts, conventions | `CLAUDE.md` |
| Copilot context | `.github/copilot-instructions.md` |

Use `mulch onboard` to generate AGENTS.md/CLAUDE.md snippets.

## Detection Triggers

**Record when you notice:**

- User corrects you ("No, that's not right...", "Actually...") → convention or pattern
- Command/API/tool fails → failure (description + resolution)
- User wants missing capability → decision (title + rationale)
- Your knowledge was wrong or outdated → convention
- You found a better approach → convention or guide

## OpenClaw Setup

OpenClaw injects workspace files; use Mulch for learnings.

### Installation

```bash
clawdhub install self-improving-agent
# or: git clone ... ~/.openclaw/skills/self-improving-agent
```

### Workspace and Mulch

- **Session start:** Run `mulch prime` when the project (or workspace) has `.mulch/`. Optionally add `mulch prime` output to workspace context if your setup supports it.
- **Recording:** Use `mulch record` from the project or workspace directory that contains `.mulch/`.
- **Promotion:** SOUL.md, AGENTS.md, TOOLS.md live in `~/.openclaw/workspace/`; add promoted rules there.

### Enable Hook (reminder at bootstrap)

```bash
cp -r hooks/openclaw ~/.openclaw/hooks/self-improvement
openclaw hooks enable self-improvement
```

See `references/openclaw-integration.md`.

## Generic Setup (Other Agents)

1. In project: `mulch init` and `mulch add <domain>` as needed.
2. Use `mulch setup <provider>` (cursor, claude, codex, etc.) for hooks.
3. Add to CLAUDE.md/AGENTS.md: "Run mulch prime at session start. Record learnings with mulch record <domain> --type failure|convention|decision|pattern|guide|reference."
4. Run `mulch onboard` and paste the snippet into your agent docs.

## Multi-Agent Safety

Mulch is safe for concurrent use: advisory file locking, atomic writes, and `merge=union` in `.gitattributes` for JSONL. Multiple agents can run `mulch prime` and `mulch record` in parallel; locks serialize writes per domain.

## Skill Extraction

When a Mulch record is valuable as a reusable skill:

1. Get content from `mulch query <domain>` or `mulch search "..."`.
2. Create `skills/<skill-name>/SKILL.md` (template in `assets/SKILL-TEMPLATE.md`).
3. Optionally note in the record (e.g. via `mulch edit`) that it was promoted to a skill.

## Best Practices

1. **Record immediately** — context is freshest after the issue.
2. **Pick the right type** — failure (description+resolution), convention (short rule), decision (title+rationale), etc.
3. **Use domains consistently** — e.g. same `api` domain for all API-related learnings.
4. **Link related records** — `--relates-to`, `--supersedes`.
5. **Run mulch prime at session start** — so the agent is grounded in existing expertise.
6. **Promote when proven** — move broadly applicable rules to CLAUDE.md, AGENTS.md, SOUL.md, TOOLS.md.

## No .learnings/

This skill does not use `.learnings/` or markdown log files. All learnings live in `.mulch/` and are recorded via the Mulch CLI. If you see references to `.learnings/` in older docs, treat them as superseded by Mulch.
