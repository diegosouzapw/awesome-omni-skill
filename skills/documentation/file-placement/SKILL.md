---
name: "file-placement"
description: "Activate when creating any summary, report, or output file. Ensures files go to correct directories (summaries/, memory/, stories/, bugs/). Mirrors what summary-file-enforcement hook enforces."
category: "enforcement"
scope: "development"
subcategory: "safeguards"
tags:
  - development
  - enforcement
  - file
  - placement
version: "10.2.14"
author: "Karsten Samaschke"
contact-email: "karsten@vanillacore.net"
website: "https://vanillacore.net"
---

# File Placement Skill

Apply correct file placement rules for all output files.

## Why This Matters

File placement is **enforced by hooks** - violations will be blocked. This skill ensures you understand the rules so your work isn't rejected.

## File Placement Rules

| File Type | Required Directory | Examples |
|-----------|-------------------|----------|
| Summaries | `summaries/` | execution-summary.md, review-summary.md |
| Reports | `summaries/` | analysis-report.md, audit-report.md |
| Stories | `stories/` | STORY-001-feature.md |
| Bugs | `bugs/` | BUG-001-issue.md |
| Memory (exports) | `memory/exports/` | memory/exports/patterns/oauth2.md |
| Documentation | `docs/` | api-docs.md, architecture.md |

## Forbidden Placements

**NEVER place these in the wrong location:**
- Summaries in `docs/` or project root
- Reports in `docs/` or project root
- Memory entries outside `memory/exports/`
- Output files in source directories

## Filename Rules

### ALL-CAPS Restrictions
Only these filenames may be ALL-CAPS:
- README.md, LICENSE, LICENSE.md
- CLAUDE.md, SKILL.md, AGENTS.md
- CHANGELOG.md, CONTRIBUTING.md
- AUTHORS, NOTICE, PATENTS, VERSION
- MAKEFILE, DOCKERFILE, COPYING, COPYRIGHT

**All other files**: Use lowercase-kebab-case
- `execution-summary.md` (correct)
- `EXECUTION-SUMMARY.md` (blocked)

## Hook Enforcement

The `summary-file-enforcement.js` hook will:
1. **Block** files with ALL-CAPS names (except allowlist)
2. **Block** summary/report files outside `summaries/`
3. **Suggest** correct filename/location

## Before Creating Files

Ask yourself:
1. Is this a summary or report? → Put in `summaries/`
2. Is this a memory entry? → Put in `memory/exports/<category>/` (category: architecture, implementation, issues, patterns)
3. Is my filename lowercase-kebab? → If not, fix it
4. Am I using ALL-CAPS? → Only if in allowlist

## Integration with Hooks

This skill provides **guidance** - you understand the rules.
The hook provides **enforcement** - violations are blocked.

Together they ensure consistent file organization even when:
- Context is lost
- Rules are forgotten
- New team members join
