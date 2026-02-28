---
name: skill-sync-checker
description: Detects content drift between skill files and their source documents. Helps maintain skills that are derived from other documentation by comparing content and flagging outdated sections.
version: 0.1.0
author: reskill
tags:
  - maintenance
  - sync
  - skills
  - documentation
metadata:
  globs: "README.md"
---

# Skill Sync Checker

A utility skill that detects when skill content has drifted from its source documents. Any skill file that includes a `<!-- source: path/to/file -->` marker can be checked for freshness and updated accordingly.

## When to Use This Skill

### User-Initiated (Agent Requested)

Use this skill when the user:

- Asks to check if a skill's content is up to date
- Has modified source documentation and wants to sync derived skills
- Is preparing a skill for publishing and wants to verify freshness
- Asks "is this skill still accurate?" or "has the source changed?"
- Performs routine maintenance on skills

### Auto-Triggered (via Cursor globs)

This skill is automatically injected into context when the user edits `README.md` (configurable via `.cursor/rules/skill-sync-checker.mdc` globs).

When auto-triggered, follow this lightweight flow:

1. Scan `skills/**/SKILL.md` for `<!-- source: ... -->` markers referencing the file being edited
2. If a match is found, **briefly remind** the user that a skill depends on this file and may need syncing
3. Do **not** automatically run the full detection workflow or modify any files — just notify

Example notification:

```
Note: skills/reskill-usage/SKILL.md is derived from README.md (last synced: 2026-02-12).
If your changes affect CLI commands, options, or usage examples, the skill may need updating.
Run a sync check when you're done editing.
```

## Source Marker Convention

Skill files that are derived from other documents should include source markers as HTML comments at the top of the file (before or after the YAML frontmatter):

```markdown
<!-- source: README.md -->
<!-- synced: 2026-02-11 -->

---
name: my-skill
description: ...
---

# My Skill
...
```

### Marker Format

| Marker                    | Required | Description                                          |
| ------------------------- | -------- | ---------------------------------------------------- |
| `<!-- source: <path> -->` | Yes      | Relative path to the source file (from project root) |
| `<!-- synced: <date> -->` | No       | ISO date of last sync (YYYY-MM-DD)                   |

A file can have **multiple source markers** if it derives from several documents:

```markdown
<!-- source: README.md -->
<!-- source: docs/cli-spec.md -->
<!-- synced: 2026-02-11 -->
```

### Where to Place Markers

- Place source markers at the very top of the file
- If the file has YAML frontmatter (`---`), markers can go before or after it
- Markers are HTML comments and will not render in the skill content

## Detection Workflow

When asked to check a skill for sync status, follow these steps:

### Step 1: Find Source Markers

Scan all `.md` files in the target skill directory. For each file, look for `<!-- source: ... -->` comments. Files without source markers are original content and can be skipped.

### Step 2: Read Source Files

For each source marker, read the referenced file. If the source file does not exist, report it as an error — the source may have been moved or deleted.

### Step 3: Compare Content

Compare the skill file against its source document(s). Focus on **structural and factual differences**, not formatting:

**Key things to check:**
- Commands, options, or features mentioned in the source but missing from the skill
- Commands, options, or features in the skill that no longer exist in the source
- Changed default values, paths, or configuration formats
- New sections in the source that should be reflected in the skill
- Deprecated or removed functionality still mentioned in the skill

**Ignore:**
- Minor wording differences (the skill may rephrase for agent consumption)
- Formatting differences (tables vs lists, heading levels)
- Content the skill intentionally omits (internal details, development docs)
- Order of sections

### Step 4: Report Results

Present findings in a clear summary:

```
Sync Check: skills/reskill-usage/SKILL.md
Source: README.md
Last synced: 2026-02-11

Status: ⚠ Out of sync

Differences found:
  1. New command `find` added in source (missing from skill)
  2. Option `--registry` added to `install` command
  3. New agent "Gemini" added to Multi-Agent Support table

Recommendation: Update the skill to reflect these changes.
```

If everything is in sync:

```
Sync Check: skills/reskill-usage/SKILL.md
Source: README.md
Last synced: 2026-02-11

Status: ✓ In sync

No significant differences found.
```

If the `<!-- synced: ... -->` marker is absent, report "Last synced: unknown":

```
Sync Check: skills/example/SKILL.md
Source: docs/example-spec.md
Last synced: unknown

Status: ⚠ Out of sync
...
```

## Update Workflow

When differences are detected and the user confirms they want to update:

### Step 1: Show Differences

Present each difference with the relevant content from the source file so the user can review what changed.

### Step 2: Update Skill Content

After user confirmation, update the skill file to reflect the source changes. Preserve the skill's structure and agent-specific content (When to Use, Workflows, Troubleshooting) — only update the sections that correspond to source material.

### Step 3: Update Synced Date

Update the `<!-- synced: ... -->` marker to today's date:

```markdown
<!-- synced: 2026-02-12 -->
```

## Detecting Potentially Relevant New Documents

When checking a skill, also scan the project for new documentation that might be relevant but is not yet tracked:

### Heuristic Rules

| File Pattern                            | Likely Relevant          | Action                   |
| --------------------------------------- | ------------------------ | ------------------------ |
| `*-spec.md`, `*-summary.md`             | Yes — user-facing specs  | Suggest adding as source |
| `README.md`, `README.*.md`              | Yes — user-facing docs   | Suggest adding as source |
| `API.md`, `docs/*.md`                   | Yes — user-facing docs   | Suggest adding as source |
| `CHANGELOG.md`                          | No — auto-generated      | Skip                     |
| `CONTRIBUTING.md`                       | No — contributor guide   | Skip                     |
| `*-design.md`                           | No — internal design     | Skip                     |
| `*-plan.md`, `*-implementation-plan.md` | No — internal plans      | Skip                     |
| `*-migration-plan.md`                   | No — internal migration  | Skip                     |
| `*-adaptation.md`                       | No — internal adaptation | Skip                     |

When a potentially relevant new document is found, notify the user:

```
New document detected: docs/new-feature-spec.md
This looks like a user-facing spec that might be relevant to the skill.
Would you like to review it for inclusion?
```

**Never automatically add new sources or modify skill content without user confirmation.**

## Example: Checking reskill-usage

Here is a complete example of checking the `reskill-usage` skill:

```
User: "Check if the reskill-usage skill is up to date"

Agent steps:
1. Read skills/reskill-usage/SKILL.md
2. Find marker: <!-- source: README.md -->
3. Read README.md
4. Compare commands table, source formats, configuration, etc.
5. Report: "The skill is missing the new `find` command that was
   added to README.md. Would you like me to update it?"
```

After user confirms:

```
Agent steps:
1. Add `find` command to the Commands table in SKILL.md
2. Update <!-- synced: 2026-02-12 -->
3. Report: "Updated SKILL.md — added `find` command, synced date
   updated to 2026-02-12"
```

## Batch Checking

To check all skills in a project at once:

```
User: "Check all skills for sync status"

Agent steps:
1. Find all SKILL.md files in the project (skills/, .cursor/skills/, etc.)
2. For each file with source markers, run the detection workflow
3. Present a summary of all results:

   Sync Check Summary:
     ✓ skills/find-skills/SKILL.md — no source markers (original content)
     ⚠ skills/reskill-usage/SKILL.md — 2 differences found (source: README.md)
     ✓ skills/skill-sync-checker/SKILL.md — no source markers (original content)
```

## Limitations

- This skill guides the agent through a manual comparison process — it does not run automated diffing tools
- The agent uses judgment to determine what constitutes a "significant" difference
- Source markers are a convention, not enforced — files without markers are simply skipped
- The heuristic rules for new document detection are suggestions, not definitive classifications
