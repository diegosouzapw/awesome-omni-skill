---
name: workflow-new-plugin
description: Guided workflow for creating a new Volon plugin — ideation, requirements, spec, plan, tasks.
argument-hint: ""plugin-name""
disable-model-invocation: true
---

# Workflow: New Plugin

Drive an agent through the full new-plugin definition lifecycle. Produces a spec
artifact and implementation tasks including the plugin scaffold and `plugin.json`.
Idempotent: skip any stage whose output artifact already exists.
Stage protocols live in `stages/stage-N-*.md`.

---

## Arguments

- `$0` (required): plugin name — quoted string, e.g. `"integrations"`

---

## Step 1 — Preflight

Run: !`cat volon.yaml 2>/dev/null || cat .volon/volon.yaml 2>/dev/null || echo "NO_CONFIG"`

Extract and note:
- `project.name` → used in all artifact frontmatter
- `workflows.new_plugin.enabled` → if `false`: output `workflow-new-plugin is disabled in volon.yaml.` and stop.

If `$0` is absent: output `ERROR: plugin name is required.` and stop.

Invoke `pcc-refresh` (scope=all).
Read `plugins/` directory listing to check for name collision.
If `plugins/<SLUG>/` already exists: output `ERROR: plugin directory plugins/<SLUG>/ already exists.` and stop.

Run: !`date +%Y-%m-%d`
Capture today's date as `TODAY`.

**Derive slug:**
Run: !`echo "$0" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/-\+/-/g' | sed 's/^-\|-$//g'`
Capture result as `SLUG`. Artifact ID = `plugin-<TODAY>-<SLUG>`.

Execute stages in sequence:
- Stage 2: `stages/stage-2-ideation.md`
- Stage 3: `stages/stage-3-requirements.md`
- Stage 4: `stages/stage-4-spec.md`
- Stage 5: `stages/stage-5-plan.md`
- Stage 6: `stages/stage-6-tasks.md`

---

## Step 7 — Finalize

Invoke `/bootstrap-update`.
Ensure `.volon/bootstrap.md` and a history copy exist before completing.
Output `DONE`.

---

## Invariants

- Never create an artifact or plugin directory that already exists — check first, skip if present.
- All artifact frontmatter must use the schema from `docs/03_workflow-contracts.md`.
- All shell commands use `!` prefix.
- Output `DONE` only after Step 7 completes.
