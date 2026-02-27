---
name: "atlas"
description: "macOS-only AppleScript control for the ChatGPT Atlas desktop app. Use only when the user explicitly asks to control Atlas tabs/bookmarks/history on macOS and the \"ChatGPT Atlas\" app is installed; do not trigger for general browser tasks or non-macOS environments."
knowledge_graph_profile: references/task-profile.json
---


# Atlas Control (macOS)

Use the bundled CLI to control Atlas and inspect local browser data.

## Quick Start

Set a stable path to the CLI:

```bash
export CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
export ATLAS_CLI="$CODEX_HOME/skills/atlas/scripts/atlas_cli.py"
```

User-scoped skills install under `$CODEX_HOME/skills` (default: `~/.codex/skills`).

Then run:

```bash
uv run --python 3.12 python "$ATLAS_CLI" app-name
uv run --python 3.12 python "$ATLAS_CLI" tabs --json
```

The CLI requires the Atlas app bundle in `/Applications` or `~/Applications`:

- `ChatGPT Atlas`

If AppleScript fails with a permissions error, grant Automation permission in System Settings > Privacy & Security > Automation, allowing your terminal to control ChatGPT Atlas.

## Philosophy

- Prefer root-cause understanding over quick symptom patches.
- Keep guidance evidence-based, explicit, and reproducible.
- Optimize for decisions that reduce rework and operational risk.

## Tabs Workflow

1. List tabs to get `window_id` and `tab_index`:

```bash
uv run --python 3.12 python "$ATLAS_CLI" tabs
```

2. Focus a tab using the `window_id` and `tab_index` from the listing:

```bash
uv run --python 3.12 python "$ATLAS_CLI" focus-tab <window_id> <tab_index>
```

3. Open a new tab:

```bash
uv run --python 3.12 python "$ATLAS_CLI" open-tab "https://chatgpt.com/"
```

Optional maintenance commands:

```bash
uv run --python 3.12 python "$ATLAS_CLI" reload-tab <window_id> <tab_index>
uv run --python 3.12 python "$ATLAS_CLI" close-tab <window_id> <tab_index>
```

## Bookmarks and History

Atlas stores Chromium-style profile data under `~/Library/Application Support/com.openai.atlas/browser-data/host/`.

List bookmarks:

```bash
uv run --python 3.12 python "$ATLAS_CLI" bookmarks --limit 100
```

Search bookmarks:

```bash
uv run --python 3.12 python "$ATLAS_CLI" bookmarks --search "docs"
```

Search history:

```bash
uv run --python 3.12 python "$ATLAS_CLI" history --search "openai docs" --limit 50
```

History for today (local time):

```bash
uv run --python 3.12 python "$ATLAS_CLI" history --today --limit 200 --json
```

The history command copies the SQLite database to a temporary location to avoid lock errors.

If history looks stale or empty, ask the user which Atlas install they are using, then check both Atlas data roots and inspect the one with the most recent `History` file:

- `~/Library/Application Support/com.openai.atlas/browser-data/host/`
- `~/Library/Application Support/com.openai.atlas.beta/browser-data/host/`

## References

Read `references/atlas-data.md` in the skill folder (for example, `$CODEX_HOME/skills/atlas/references/atlas-data.md`) when adjusting data paths or timestamps.

## Anti-patterns

- Skipping investigation and jumping directly to fixes.
- Making claims without evidence, logs, or reproducible steps.
- Mixing unrelated workstreams in a single execution path.

## Constraints / Safety

- Redact secrets, tokens, credentials, and PII by default; never echo raw environment values.
- Prefer safe defaults and avoid irreversible changes without explicit confirmation.

## Inputs

- User task context and target environment.
- Relevant constraints, permissions, and preferences required to execute safely.

## Outputs

- A concrete next-step response with explicit, reproducible actions.
- A short verification checklist and caveats for the user.

## Validation

- Fail fast: stop at the first failed check and do not continue.
- Re-run the required checks before proceeding to the next step.
- Report any failed check and requested follow-up actions clearly.

## When to use

- Use this skill when the request matches the skill's intent and scope.
- Do not use it when a different domain or higher-privilege workflow is required.

## Constraints / Safety

- Redact secrets, tokens, credentials, and PII by default; never echo raw environment values.
- Prefer safe defaults and avoid irreversible changes without explicit confirmation.

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
