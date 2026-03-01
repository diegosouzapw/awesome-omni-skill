---
name: reflect
description: Session reflection — captures knowledge and self-improvement findings,
  then auto-applies them as Cursor rules or skills. Analyzes what was learned
  AND what went wrong. Use when user says "reflect", "self-review", "co można
  poprawić", "przeanalizuj sesję", "zapamiętaj", or invokes /reflect. Also
  called automatically by the wrap-up skill.
---

# Reflect

Analyze the full conversation in three passes:

1. **Knowledge capture** — what was learned that should persist.
2. **Self-improvement** — what went wrong or could be better.
3. **Company knowledge candidate** — could any finding benefit the whole org?

If the session was short or routine with nothing notable in any pass,
say "Nothing to reflect on" and stop.

**Auto-apply all actionable findings immediately** — do not ask for
approval on each one. Apply changes, then present a summary.

---

## Pass 1 · Knowledge capture

Review what was learned during the session. For each finding, decide
where it belongs using the placement guide below.

### Memory placement guide (Cursor)

| Destination | When to use | Format |
|-------------|-------------|--------|
| `.cursor/rules/topic.mdc` | Permanent project conventions, architecture decisions, coding standards | MDC with `alwaysApply: true` |
| `.cursor/rules/topic.mdc` (scoped) | Rules for specific files or areas | MDC with `glob:` pattern |
| `.cursor/skills/name/SKILL.md` | Reusable multi-step workflows | Skill directory |
| Existing rule or skill file | Small additions, quirks, insights | Append to the most relevant file |

### Decision framework

- Permanent project convention? → `.cursor/rules/` with `alwaysApply: true`.
- Scoped to specific files or directories? → `.cursor/rules/` with `glob:`.
- Reusable multi-step workflow? → `.cursor/skills/` (new or update).
- Small insight or quirk? → Append to existing rule or skill.
- Already documented somewhere? → Don't duplicate; skip.

---

## Pass 2 · Self-improvement

Analyze the conversation for things that went wrong or could be better.

### Finding categories

- **Skill gap** — things that took multiple attempts, were done wrong,
  or required user correction.
- **Friction** — repeated manual steps, things the user had to ask for
  explicitly that should have been automatic.
- **Knowledge** — facts about the project, user preferences, or setup
  that were unknown but should have been.
- **Automation** — repetitive patterns that could become skills, rules,
  or scripts.

### Root cause — łańcuch przyczynowy

Dla każdego znalezionego problemu zadaj pytanie:

> Jaki wcześniejszy krok lub brakujący guard zapobiegłby
> całej tej sytuacji?

Jeśli odpowiedź prowadzi do brakującej reguły, skilla lub
checklisty — to jest właściwy finding do utrwalenia, a nie
sam objaw techniczny.

Technika: cofnij się od objawu po łańcuchu zdarzeń aż do
pierwszej decyzji, która mogła pójść inaczej. Utrwal tę
decyzję, nie naprawę objawu.

### Action types

| Action | Target |
|--------|--------|
| New/update rule | `.cursor/rules/topic.mdc` |
| New/update skill | `.cursor/skills/name/SKILL.md` |
| New/update script | `.cursor/skills/name/scripts/` |

When creating a new rule, choose the correct scope:

- `alwaysApply: true` — permanent project-wide convention.
- `glob:` pattern — scoped to specific files/directories.

---

## Pass 3 · Company knowledge candidate (JMB.Agents)

After Pass 1 and 2, review all findings (applied and no-action) and
evaluate whether any of them could benefit the whole organization.

### Evaluation criteria

For each finding ask:

1. **Is it project-specific or universal?** If the convention, pattern,
   or workflow would help another JMB Lab team → candidate.
2. **Does it already exist in JMB.Agents?** Read `catalog.json`
   (via `jmb-agents-lock.json` → `sourcePath`). If the finding is
   already covered → skip.
3. **Is it mature enough?** A finding from a single session might be
   premature. If the finding has been re-discovered in 2+ sessions
   or is a well-known industry practice → candidate.

### Actions

| Verdict | Action |
|---------|--------|
| Strong candidate | Propose contribution: "This finding could benefit the org. Run `Contribute` via JMB.Agents skill." |
| Maybe | Note it: "Potential JMB.Agents candidate — revisit after next session." |
| Project-only | No action (already handled in Pass 1/2). |

Do **not** auto-contribute. Present the recommendation and let the user
decide whether to run the Contribute operation.

### Output

Add a third section to the output format:

```
JMB.Agents candidates:

5. 🏢 Strong: "Always use dedicated hooks for global listeners"
   → Propose rule: react-global-listeners
   → Run: Contribute rule react-global-listeners to JMB.Agents

6. 🔄 Maybe: "RSC split breaks E2E timing assumptions"
   → Revisit after next session
```

---

## Output format

Present all findings (from both passes) in two sections — applied first,
then no-action:

```
Findings (applied):

1. ✅ Knowledge: discovered convention for field ordering in forms
   → [Rule] Created .cursor/rules/form-field-ordering.mdc

2. ✅ Skill gap: label consistency was missed in 3 places
   → [Rule] Created .cursor/rules/label-consistency.mdc

3. ✅ Automation: onboarding field reorder is a repeated pattern
   → [Skill] Created .cursor/skills/reorder-form-fields/SKILL.md

---
No action needed:

4. Knowledge: Playwright needs --headed for photo upload tests
   Already documented in e2e-smoke-test SKILL.md
```
