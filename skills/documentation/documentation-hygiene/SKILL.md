---
name: Documentation Hygiene
description: This skill should be used when the user asks to perform "documentation hygiene", "update docs for change", "mark historical docs", "add deprecation headers", "documentation migration", or when making significant changes that affect multiple documentation files (like license migrations, tier changes, or API changes).
version: 1.0.0
---

# Documentation Hygiene Skill

Systematically update documentation when making significant changes to prevent AI agents and developers from referencing outdated information.

---

## Core Principle

**Never delete historical documentation.** Instead, mark it clearly as outdated while preserving valuable context. Future AI agents read CLAUDE.md first, so prominent warnings there prevent confusion.

---

## Workflow

### Step 1: Gather Change Information

Before starting, collect from the user:

1. **What changed?** (e.g., "License changed from Apache-2.0 to Elastic License 2.0")
2. **Search terms for OLD state** (e.g., "Apache-2.0", "three-tier", "unlimited free")
3. **Authoritative reference for NEW state** (e.g., "ADR-013, ADR-017")

Ask if unclear:
```
To perform documentation hygiene, I need:
1. A summary of the change being made
2. Keywords that identify the OLD/outdated state
3. The authoritative document(s) for the NEW state

What change are we documenting?
```

### Step 2: Search for Affected Documentation

Search the codebase for references to the OLD state:

```bash
# Search for old terms in markdown files
grep -r "OLD_TERM" docs/ --include="*.md"
grep -r "OLD_TERM" packages/*/README.md
grep -r "OLD_TERM" CLAUDE.md
```

### Step 3: Categorize Files

Present a categorized plan to the user:

| Category | Treatment | Examples |
|----------|-----------|----------|
| **CLAUDE.md** | Add prominent warning section at top | Project root CLAUDE.md |
| **ADRs** | Update to reflect current state, add revision history | docs/adr/*.md |
| **Package READMEs** | Update to reflect current state | packages/*/README.md |
| **Current Reference Docs** | Update to reflect current state | docs/architecture/*.md |
| **Historical/Retro Docs** | Add deprecation header with links | docs/retros/*.md, docs/strategy/*.md |
| **Legal Docs** | Flag for legal review | docs/legal/*.md |

### Step 4: Get User Approval

Present the plan:

```markdown
## Documentation Hygiene Plan

**Change:** [Summary of change]

### Files to Update (X files)

**CLAUDE.md** (1 file)
- [ ] Add warning section about change

**ADRs** (X files)
- [ ] docs/adr/XXX.md - Update for new state
- [ ] docs/adr/YYY.md - Create new ADR if needed

**Current Reference Docs** (X files)
- [ ] file1.md - Update license reference
- [ ] file2.md - Update tier information

**Historical Docs** (X files) - Add deprecation headers
- [ ] docs/retros/phase-X.md
- [ ] docs/strategy/analysis.md

**Legal Docs** (X files) - Flag for legal review
- [ ] docs/legal/TERMS.md

Shall I proceed with this plan?
```

### Step 5: Execute Updates

Apply updates in this order:

1. **CLAUDE.md first** - Most important for AI agents
2. **ADRs** - Authoritative technical decisions
3. **Package READMEs** - Public-facing documentation
4. **Current reference docs** - Architecture, publishing guides
5. **Historical docs** - Retros, strategy docs
6. **Legal docs** - Flag only, don't modify without legal approval

---

## Templates

### CLAUDE.md Warning Section

Add after the first major section (usually after project overview or critical notices):

```markdown
## [EMOJI] IMPORTANT: [Change Title] ([Date])

**As of [Date], [summary of change].**

> **CRITICAL FOR AI AGENTS**: Documentation written before [Date] may reference the old [X]. **IGNORE** any references to:
> - "[Old term 1]"
> - "[Old term 2]"
> - "[Old term 3]"

### Current State

[Brief description of current state with key facts]

| Key | Value |
|-----|-------|
| [Fact 1] | [Value 1] |
| [Fact 2] | [Value 2] |

**Authoritative References:**
- [ADR-XXX: Title](path/to/adr) - [Description]
- [ADR-YYY: Title](path/to/adr) - [Description]
```

### Deprecation Header for Historical Docs

```markdown
> [WARNING_EMOJI] **HISTORICAL DOCUMENT**
>
> This [document type] documents [Phase/Period] ([Date]). [What changed]:
> - [Change 1]
> - [Change 2]
>
> See [ADR-XXX](path/to/adr) and [ADR-YYY](path/to/adr) for current [topic].
```

### Legal Review Flag

```markdown
> [WARNING_EMOJI] **LEGAL REVIEW REQUIRED**
>
> **This document requires legal review to reflect the [Date] [change type].**
>
> **Changes needed:**
> - Section X.X: [Specific change needed]
> - Section Y.Y: [Specific change needed]
>
> **See:** [ADR-XXX](path/to/adr)
>
> **Tracking Issue:** [Issue ID] (to be created for legal review)
```

### ADR Revision History

Add after the Status/Date/Deciders header:

```markdown
## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | [Original Date] | Original [description] |
| 1.1 | [Update Date] | **[Summary of changes]** |
```

---

## Best Practices

### 1. CLAUDE.md is Priority One

AI agents read CLAUDE.md first. A prominent warning there prevents:
- Agents citing outdated license information
- Agents referencing old pricing/tier structures
- Agents using deprecated patterns

### 2. Never Delete Historical Docs

Historical documents provide valuable context:
- Why decisions were made
- What was tried before
- Evolution of the project

Instead of deletion, add clear deprecation headers.

### 3. ADRs Are the Source of Truth

Architecture Decision Records should always reflect the current state:
- Add revision history for significant changes
- Cross-reference related ADRs
- Create new ADRs for major architectural changes

### 4. Consistent Visual Indicators

Use consistent emoji and formatting:
- `> [WARNING_EMOJI]` for deprecation notices
- **Bold** for critical terms
- Tables for structured comparisons
- Links to authoritative sources

### 5. Category-Specific Treatment

| Doc Type | Action | Rationale |
|----------|--------|-----------|
| CLAUDE.md | Update immediately | AI agents read first |
| ADRs | Update with revision history | Authoritative reference |
| READMEs | Update | Public-facing, affects perception |
| Reference docs | Update | Current guidance |
| Retros/Strategy | Deprecation header | Historical context valuable |
| Legal | Flag for review | Requires legal expertise |

---

## Environment Variables

This skill does not require any environment variables.

---

## Example Usage

### License Migration

**User:** "We changed from Apache-2.0 to Elastic License 2.0. Run documentation hygiene."

**Agent workflow:**
1. Search for: `Apache-2.0`, `Apache License`, `open source`, `free tier unlimited`
2. Categorize ~20 files found
3. Present plan to user
4. Update CLAUDE.md with license warning
5. Update ADRs with revision history
6. Update package READMEs
7. Add deprecation headers to historical docs
8. Flag TERMS.md for legal review

### Tier Structure Change

**User:** "We added an Individual tier. Update docs."

**Agent workflow:**
1. Search for: `three-tier`, `Community/Team/Enterprise`, pricing references
2. Categorize affected files
3. Present plan to user
4. Update CLAUDE.md with new tier table
5. Update ADRs with new tier information
6. Add deprecation headers to old strategy docs

---

## Related Skills

- **governance** - For code review and standards compliance
- **issue-tracker** - For creating tracking issues for legal review (Linear, Jira, GitHub Issues, etc.)
