---
name: skill-refactor
description: Reviews and refactors an existing skill against quality standards. Use when asked to review, audit, improve, or refactor a skill. Triggers on phrases like "refactor this skill", "review my skill", "improve skill quality", "audit skill-name", "clean up my skill". Do NOT use to create a new skill from scratch.
---

# Skill Refactor

Standalone refactor review for existing skills. Runs critique and red-team agents in parallel, compiles a review summary, and gates all changes behind explicit user approval.

## Input

Accept one of:

- Project-bound path: `.claude/skills/<skill-name>/`
- Template path: `skills/<skill-name>/` (or any other project-relative path)

If the user does not provide a path, ask which skill to refactor before proceeding.

Confirm the directory exists and contains `SKILL.md` before launching sub-agents.

## Workflow

See `references/refactor-protocol.md` for full sub-agent prompt templates, output formats, and AskUserQuestion schemas.

### 1. Locate skill

Resolve the skill path. Read all files in the directory. If `SKILL.md` is missing, stop and report.

### 2. Launch parallel sub-agents

Launch both simultaneously:

- **Critique agent** — evaluates against internal quality standards (conciseness, degrees of freedom, progressive disclosure, structure, forbidden files). Read `references/refactor-protocol.md` for the prompt template.
- **Red-team agent** — evaluates against `references/anthropic-best-practices.md` (naming, frontmatter, trigger quality, instruction quality, error handling, file conventions). Read `references/refactor-protocol.md` for the prompt template.

Write outputs to:

- `temp/<skill-name>/critique/feedback.md`
- `temp/<skill-name>/red-team/feedback.md`

### 3. Compile review summary

Merge both feedback files into `temp/<skill-name>/refactor/review-summary.md`. Present the path to the user and ask them to review it before proceeding.

### 4. Approval gate

Use AskUserQuestion. Options: proceed with refactor, keep as-is, defer. If declined or deferred, log to `decisions.md` and stop.

### 5. Requirements gathering

Use AskUserQuestion (up to 3 questions): which change categories to apply, any new requirements, refactor depth (targeted vs full rewrite). Log all answers to `temp/<skill-name>/refactor/decisions.md`.

### 6. Refactor agent

Apply approved changes in-place to the skill files. Preserve sections not flagged for change in targeted refactors. Log implementation notes to `decisions.md`. Read `references/refactor-protocol.md` for the prompt template.

## Constraints

- Never modify skill files without explicit user approval
- Write all intermediate artifacts to `temp/<skill-name>/` before any approval gate
- Log all user decisions and implementation choices to `decisions.md`
