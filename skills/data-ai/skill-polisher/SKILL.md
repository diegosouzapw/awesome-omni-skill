---
name: skill-polisher
description: Meta-skill for Claude Code to validate and polish Codex skills under `.codex/skills/` by running the skill-polisher and reviewing structural integrity. Use when auditing Codex skills, enforcing standardization, or running integrity sweeps across the Codex skill set.
allowed-tools: "Read, Write, Glob, Grep, Bash"
user-invocable: true
---

# Skill Polisher (Claude Code)

Audit Codex skills for structural integrity and standardization by using the existing Codex `skill-polisher` tooling.

## Flavor Contract
- **Claude-flavored** by default: validates Claude frontmatter/tooling rules.
- **Cross-flavored compatible** via shared auditor (see Cross-Flavor Audit).

## Target scope
- Codex skills:
  - Single skill: `.codex/skills/<skill-name>`
  - Whole set: `.codex/skills`
- Claude skills:
  - Single skill: `.claude/skills/<skill-name>`
  - Whole set: `.claude/skills`

## Execution
### Codex skills (preferred)
Use the Codex polisher for deterministic audits:

```powershell
uv run .codex/skills/skill-polisher/scripts/polish_skill.py .codex/skills --all
```

For a single skill:

```powershell
uv run .codex/skills/skill-polisher/scripts/polish_skill.py .codex/skills/<skill-name>
```

### Claude skills (manual audit)
Claude skills do not share the Codex polisher; use a lightweight audit:

```powershell
Get-ChildItem .claude/skills -Directory
Get-Content .claude/skills/<skill-name>/SKILL.md
```

Checks to confirm:
- `SKILL.md` present
- Frontmatter includes `name` and `description`
- Any `allowed-tools` entries match Claude tool names (`Read`, `Write`, `Glob`, `Grep`, `Bash` as needed)

## Cross-Flavor Audit (shared)
Use the shared auditor when you want consistent reporting:

```powershell
uv run scripts/skill_audit.py --flavor codex --root .codex/skills
uv run scripts/skill_audit.py --flavor claude --root .claude/skills
```

## Hook (Claude-side)
Run the Claude cross-compatible audit via the helper script:

```powershell
.\scripts\run_claude_skill_polisher.ps1 -Root .claude/skills
```

## Claude-Local Audit (Claude-centric outputs)
Write JSON outputs into `claude/mailbox`:

```powershell
.\scripts\run_claude_local_audit.ps1 -Root .claude/skills
```

## Hook (Claude-side cross-polish)
Run both Codex and Claude audits in one pass:

```powershell
.\scripts\run_claude_cross_polish.ps1 -CodexRoot .codex/skills -ClaudeRoot .claude/skills
```

## Bridge (Codex tooling)
Invoke Codex polisher from Claude:

```powershell
.\scripts\run_codex_polisher.ps1 -Root .codex/skills
```

## Reporting
Summarize:
- TASTE + score
- Issues (if any)
- Files changed (if any)

## Notes
- This skill audits **Codex and Claude skills**; do not modify either unless explicitly requested.
- If you need to add `@POLISHED` seals, do so after a clean run and record the timestamp.

## Cross-Flavor Compatibility
- Codex flavor: requires `agents/openai.yaml` and `assets/` with SVG icons.
- Claude flavor: requires `SKILL.md` with valid frontmatter (`name`, `description`), optional `allowed-tools`.
- For shared audits use: `python scripts/skill_audit.py --flavor codex --root .codex/skills` and `python scripts/skill_audit.py --flavor claude --root .claude/skills`.



