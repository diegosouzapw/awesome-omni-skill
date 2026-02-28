---
name: Skill Builder / Creator
slug: skill-builder
version: 1.0.5
homepage: https://clawic.com/skills/skill-builder
description: Create high-quality skills with modular structure, progressive disclosure, and token-efficient design.
changelog: Added description examples table, security checklist, and improved traps with fixes
metadata: {"clawdbot":{"emoji":"🛠️","requires":{"bins":[]},"os":["linux","darwin","win32"]}}
---

## Setup

On first use, read `setup.md` for integration guidelines.

## When to Use

User wants to create or improve a skill. Agent guides structure, reviews content, and ensures quality.

## Data Storage

If user wants project tracking, create folder in their home directory.
See `memory-template.md` for the template structure.

The agent does NOT create files automatically. Always ask user first.

## Architecture

Skills follow this structure:

```
skill-name/
├── SKILL.md           # Core instructions (SHORT)
├── [topic].md         # On-demand details
└── references/        # Heavy docs (optional)
```

## Quick Reference

| Topic | File |
|-------|------|
| Setup process | `setup.md` |
| Tracking projects | `memory-template.md` |
| Patterns and examples | `patterns.md` |

## Core Rules

### 1. SKILL.md Must Be Short
Target 30-50 lines, max 80. Move details to auxiliary files. Every line must justify its token cost.

### 2. Progressive Disclosure
```
Level 1: Metadata (name + description) — always loaded
Level 2: SKILL.md body — when skill triggers
Level 3: Auxiliary files — on demand
```

### 3. Descriptions Are Critical
One sentence, 15-25 words. Action verb first. Describes capabilities, not triggers.

| ❌ Wrong | ✅ Right |
|----------|----------|
| "Use when user needs PDFs" | "Process, merge, and extract PDF content" |
| "Helper for Docker" | "Build, deploy, and debug Docker containers" |
| "Git guide" | "Manage branches, resolve conflicts, and automate workflows" |

See `patterns.md` for more examples.

### 4. Required Structure
Every skill needs:
- Frontmatter: name, slug, version, description
- `## When to Use` — activation triggers
- `## Core Rules` — 3-7 numbered rules

### 5. Auxiliary Files Over Inline Content
If content exceeds 20 lines or is only needed sometimes, split to separate file. Reference from Quick Reference table.

### 6. No Redundancy
Information lives in ONE place. SKILL.md references files, doesn't duplicate content.

### 7. Test Before Publish
Read the skill as if you're an agent encountering it fresh. Is every instruction clear and necessary?

## Skill Building Traps

| Trap | Why it fails | Fix |
|------|--------------|-----|
| Explaining what X is | Models already know | Explain WHEN and HOW |
| "Use when..." in description | Wastes characters | Action verbs only |
| Keyword lists in description | Looks spammy | One clean sentence |
| Templates inline | Bloats SKILL.md | Separate file |
| Vague "observe" instructions | Gets flagged suspicious | Be specific about what data |
| Undeclared file creation | Security flag | Add Data Storage section |

## Related Skills
Install with `clawhub install <slug>` if user confirms:

- `skill-manager` — manage installed skills
- `skill-update` — update existing skills
- `skill-test` — test skills locally

## Feedback

- If useful: `clawhub star skill-builder`
- Stay updated: `clawhub sync`
