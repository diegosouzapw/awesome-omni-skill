---
name: create-expert-skill
description: Create production-ready skills from expert knowledge. Extracts domain expertise and system ontologies, uses scripts for deterministic work, loads knowledge progressively. Use when building skills that must work reliably in production.
version: 2.2
---

# Expert Skill Creation

Transform expert knowledge into production-ready skills that combine domain expertise with system-specific understanding.

## Why Skills Fail in Production

AI assistants fail not because they lack intelligence, but because they lack:

1. **Domain Expertise** — Industry-specific rules, edge cases, unwritten conventions
2. **Ontology Understanding** — How YOUR systems, data structures, and workflows actually work

**Both are required.** Domain knowledge without system context produces generic output. System knowledge without domain expertise produces structurally correct but semantically wrong results.

## Workflow

```
Assess → Discover (Expertise + Ontology) → Design → Create → Refine → Ship
```

## Quick Assessment

**Create a skill when:**
- Used 3+ times (or will be)
- Follows consistent procedure
- Saves >300 tokens per use
- Requires specialized knowledge not in Claude's training
- Must produce trusted output (not "close enough")

**Don't create for:** one-time tasks, basic knowledge Claude already has, rapidly changing content.

## Discovery: Two Streams

### Stream 1: Domain Expertise

Deep knowledge that transcends any specific company:
- Industry standards and their versions
- Professional conventions and best practices
- Edge cases only practitioners know
- Validation rules from specifications

*Example (LEDES validation):* LEDES 98B vs XML 2.0 formats, UTBMS code taxonomy, date format requirements, required vs optional fields.

### Stream 2: Ontology Understanding

How the skill maps to specific systems and organizations:
- Company-specific policies and constraints
- Data structures and identifiers unique to the system
- Cross-references between entities (timekeepers → IDs → rates)
- Workflow states and transitions

*Example (LEDES validation):* Firm-specific timekeeper codes, matter numbering conventions, approved billing rates, outside counsel guideline requirements.

### Discovery Questions

When starting, I'll ask about:
1. **Domain & Purpose** — What problem? What industry standards apply?
2. **Ontology Requirements** — What system-specific structures must the skill understand?
3. **Content Source** — Conversation, docs, specifications, or files to distill from?
4. **Automation Potential** — What can be deterministic (scripts)? What needs interpretation (LLM)?
5. **Complexity Level** — Simple (SKILL.md only), Enhanced (+scripts), or Full (+resources)?

## Skill Architecture

```
skill-name/
├── SKILL.md              # Layer 1: Core (300-500 tokens)
├── scripts/              # Layer 0: Automation (0 tokens to run)
│   └── validate.py
└── resources/            # Layer 2: Details (loaded selectively)
    └── ADVANCED.md
```

**Layer 0** (Scripts): Free execution, structured JSON output
**Layer 1** (SKILL.md): Loaded when triggered - keep lean
**Layer 2** (Resources): Fetched only when specific section needed

## Token Optimization

| Technique | Instead of | Do this | Savings |
|-----------|-----------|---------|---------|
| Scripts | 500 tokens explaining validation | `python scripts/validate.py` | ~450 tokens |
| Reference | Inline schema (200 tokens) | Link to `resources/schema.json` | ~185 tokens |
| Layer 2 | Everything in SKILL.md | Link to `resources/ADVANCED.md` | ~750 tokens |

## Description Formula

`<Action> <Object> for <Purpose>. Use when <Trigger>.`

Example: "Validate billing data for system migration. Use before importing invoices."

## Shipping

When content is finalized:

```bash
python scripts/package_skill.py skill-name 1.0
```

Creates `skill-name-v1.0.zip` with:
- DIRECTORY_STRUCTURE.txt (auto-generated)
- README.md with deployment instructions
- All skill files properly organized

## Templates & Examples

See `resources/templates/` for:
- Minimal skill template
- Enhanced skill template  
- Script template

See `resources/examples/` for domain-specific patterns.

## Quality Checklist

Before shipping:
- [ ] Description <30 tokens
- [ ] SKILL.md <500 tokens (Layer 1)
- [ ] Scripts for deterministic operations
- [ ] Advanced content in resources/ (Layer 2)
- [ ] Version in frontmatter
- [ ] All referenced files exist

---

**Version:** 2.2 | **Target:** <500 tokens Layer 1
