---
name: devtrail-adr
description: Create an ADR (Architecture Decision Record) for significant architectural decisions. Requires human review.
---

# DevTrail ADR Skill

Create an Architecture Decision Record (ADR) for significant architectural decisions.

> **Note**: ADRs created by AI agents are marked as `draft` and `review_required: true` by default.

## Instructions

Use this skill for major architectural decisions that affect the system structure, technology stack, or design patterns.

### 1. Gather Context

```bash
# Get current date
date +%Y-%m-%d

# Get recent changes for context
git diff --stat HEAD~1 2>/dev/null || git diff --stat

# Check for related ADRs
ls .devtrail/02-design/decisions/ADR-*.md 2>/dev/null | tail -5
```

### 2. Confirm with User

**Always confirm before creating:**

```
╔══════════════════════════════════════════════════════════════════╗
║  DevTrail ADR                                                    ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                   ║
║  ⚠️  ADRs require human review after creation.                    ║
║                                                                   ║
║  📋 An ADR documents architectural decisions that affect:         ║
║     • System structure                                            ║
║     • Technology choices                                          ║
║     • Design patterns                                             ║
║     • Integration approaches                                      ║
║                                                                   ║
║  Please provide:                                                  ║
║  1. Decision title (what architectural decision)                  ║
║  2. Context (why is this decision needed)                         ║
║  3. The decision and alternatives considered                      ║
║                                                                   ║
╚══════════════════════════════════════════════════════════════════╝
```

### 3. Determine Sequence Number

```bash
# Count existing ADRs for today
ls .devtrail/02-design/decisions/ADR-$(date +%Y-%m-%d)-*.md 2>/dev/null | wc -l
```

### 4. Check Language and Load Template

Read `.devtrail/config.yml` for language setting:
- `en` (default): `.devtrail/templates/TEMPLATE-ADR.md`
- `es`: `.devtrail/templates/i18n/es/TEMPLATE-ADR.md`

### 5. Create Document

Fill template with:
- `id`: ADR-YYYY-MM-DD-NNN
- `title`: Architectural decision title
- `status`: **draft** (always for AI-created ADRs)
- `created`: Current date
- `updated`: Current date
- `agent`: gemini-cli-v1.0
- `confidence`: based on research done
- `review_required`: **true** (always for ADRs)
- `risk_level`: minimum `medium` for architectural decisions

**Key sections to fill:**
- Status: Note that this was created by AI agent
- Context: Technical and business context, forces at play
- Decision: The architectural decision with justification
- Alternatives Considered: Other options with pros/cons/why not
- Consequences: Positive, negative, neutral
- Affected Components: Table of impacted parts
- Implementation Plan: High-level steps
- Success Metrics: How to validate the decision

Save to: `.devtrail/02-design/decisions/ADR-YYYY-MM-DD-NNN-description.md`

### 6. Report Result

```
⚠️  ADR created (requires human review):
   .devtrail/02-design/decisions/ADR-YYYY-MM-DD-NNN-description.md

   Status: draft
   Review Required: YES

DevTrail: Created ADR-YYYY-MM-DD-NNN-description.md (review required)
```

## Examples of Architectural Decisions

- Use PostgreSQL over MongoDB for persistence
- Adopt microservices vs. monolith architecture  
- Choose REST vs. GraphQL for API
- Select authentication strategy (JWT, OAuth, etc.)
- Define module boundaries and dependencies
- Establish caching strategy
