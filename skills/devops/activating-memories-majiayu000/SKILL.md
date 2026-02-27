---
name: activating-memories
description: Loads project-specific memories at session start and triggers onboarding for new projects. Use when starting a new coding session or when the user says "activate" or asks about project context.
---

<quick_start>
1) Check `./.{AGENT_NAME}/skills/memories/` for memory skill folders.
   - Each memory is a skill file at `./.{AGENT_NAME}/skills/memories/<name>/SKILL.md`.
   - Example baseline set:
     - `project-overview/SKILL.md`
     - `suggested-commands/SKILL.md`
     - `style-and-conventions/SKILL.md`
     - `task-completion-checklist/SKILL.md`
2) If empty or missing, read `references/onboarding-guide.md` and execute onboarding.
3) Otherwise, read memories and proceed.
4) If you learn new info, update the relevant `<name>/SKILL.md` via `$writing-memories`.
</quick_start>

<decision_points>
- Memories missing/empty -> read `references/onboarding-guide.md` and run onboarding.
</decision_points>

<quality_checklist>
- Each memory should be dense with high-signal, repo-specific facts.
- Prefer concrete evidence (paths, filenames, commands) over vague summaries.
- If a memory is sparse, expand it by scanning the repo and updating it.
</quality_checklist>

<failure_modes>
- `AGENT_NAME` unclear: list directories under `.` and choose the active one (e.g. `.codex/` or `.claude/`).
</failure_modes>
