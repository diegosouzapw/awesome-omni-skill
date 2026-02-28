---
name: skill-creator
description: How to create new agent skills following the open standard (agentskills.io). Use when the user wants to create, scaffold, or design a new skill for any AI coding agent.
---

# Skill Creator

Meta-skill for creating new agent skills that follow the [agentskills.io](https://agentskills.io/specification) open standard. Works across Claude Code, OpenAI Codex, Google Gemini CLI, Cursor, and Windsurf.

## Input
- A skill idea from the user (name, purpose, or general description)

## Output
- A complete skill directory with `SKILL.md` and optional subdirectories
- Placed in `.agent/skills/<skill-name>/`

---

## Process

### Phase 1: Interview (Understand the Skill)

Ask these questions **one at a time**. Wait for each answer before continuing.

1. **What should this skill do?** (1-2 sentence summary)
2. **When should an agent use it?** (trigger conditions — be specific)
3. **What inputs does it need?** (files, user answers, config, etc.)
4. **What does it produce?** (deliverables, files, decisions)
5. **Does it have phases/steps?** (break it down if yes)
6. **Does it need templates, scripts, or reference docs?** (optional directories)

After all questions are answered, confirm with the user:
> "Here's what I understood: [summary]. Shall I create it?"

### Phase 2: Scaffold (Generate the Skill)

Create the directory and `SKILL.md` using the template at `templates/SKILL_TEMPLATE.md`.

```
.agent/skills/<skill-name>/
└── SKILL.md              # Required — use the template
```

#### Frontmatter Rules (from agentskills.io spec)

| Field | Required | Rules |
|-------|----------|-------|
| `name` | ✅ | kebab-case, 1-64 chars, lowercase only, must match directory name |
| `description` | ✅ | 1-1024 chars, describe what it does AND when to use it |
| `license` | Optional | Short name or reference to LICENSE file |
| `metadata` | Optional | Key-value pairs (author, version, etc.) |
| `compatibility` | Optional | Environment requirements (tools, OS, etc.) |

#### Name Validation

```
✅ Valid:   code-review, sprint-planner, github-flow
❌ Invalid: Code-Review (uppercase), -my-skill (leading hyphen), my--skill (double hyphen)
```

#### Body Structure (our convention)

Follow this pattern — consistent with our existing skills:

```markdown
# Skill Name

Brief description.

## Input
- What the skill needs

## Output
- What the skill produces

## Process

### Phase 1: [Name]
1. Step 1
2. Step 2

### Phase 2: [Name]
...

## Rules
1. **ALWAYS** ...
2. **NEVER** ...
```

### Phase 3: Enrich (Add Optional Directories)

Based on the interview answers, add subdirectories as needed:

| Directory | When to Use | Examples |
|-----------|------------|---------|
| `templates/` | Skill produces files from a template | `TEMPLATE.md`, `CONFIG_TEMPLATE.yaml` |
| `references/` | Skill needs detailed docs not loaded by default | Guides, specs, glossaries |
| `scripts/` | Skill runs executable code | Python, Bash, Node.js scripts |
| `assets/` | Skill uses static resources | Images, fonts, data files |

```
.agent/skills/<skill-name>/
├── SKILL.md
├── templates/          # Optional
│   └── TEMPLATE.md
├── references/         # Optional
│   └── GUIDE.md
├── scripts/            # Optional
│   └── helper.py
└── assets/             # Optional
    └── config.json
```

> **Important:** Keep `SKILL.md` under 500 lines (< 5000 tokens). Move detailed content to `references/`.

### Phase 4: Validate (Completeness Checklist)

Before delivering, verify:

- □ `name` field is kebab-case and matches directory name?
- □ `description` field clearly states what + when?
- □ `SKILL.md` body has Input, Output, Process, and Rules sections?
- □ Body is under 500 lines?
- □ All referenced templates/scripts exist?
- □ No hardcoded paths — skill is portable?
- □ Rules section has at least 2 guardrails?
- □ Each phase produces something verifiable?

---

## Progressive Disclosure (How Agents Load Skills)

Skills are loaded in 3 tiers to save context:

```
Tier 1: Discovery  → name + description only (~100 tokens)
Tier 2: Activation → Full SKILL.md body (< 5000 tokens)
Tier 3: Resources  → scripts/, references/, assets/ (on demand)
```

This means:
- **Description is critical** — it's the ONLY thing agents see at startup
- **Keep SKILL.md focused** — move extras to references/
- **Templates are loaded only when used** — don't inline them

---

## Cross-Platform Compatibility

Skills created with this tool work across all major platforms:

| Platform | Skill Location | Notes |
|----------|---------------|-------|
| **This project** | `.agent/skills/` | ✅ Primary location |
| Claude Code | `.claude/skills/` | Copy skill directory |
| OpenAI Codex | `.codex/skills/` | Copy skill directory |
| Gemini CLI | `.gemini/skills/` | Copy skill directory |
| Cursor | `.cursor/skills/` | Via Agent Skills standard |
| Windsurf | `.windsurf/skills/` | Via Agent Skills standard |

> For cross-platform details, see `references/CROSS_PLATFORM_GUIDE.md`.

---

## Rules
1. **ALWAYS** ask the user before creating — never assume skill requirements
2. **ALWAYS** use kebab-case for directory and `name` field
3. **ALWAYS** write descriptions that include BOTH what + when
4. **ALWAYS** include a Rules section with guardrails
5. **NEVER** exceed 500 lines in `SKILL.md` — use references/ for overflow
6. **NEVER** hardcode absolute paths in skill content
7. **NEVER** skip the validation checklist
8. **ALWAYS** use English for all skill content
