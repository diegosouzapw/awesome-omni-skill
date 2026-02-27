---
name: skill-creator
description: Guide for creating effective skills with constitutional governance. This skill MUST be used when users want to create a new skill (or update an existing skill) that extends Kimi's capabilities with specialized knowledge, workflows, or tool integrations. Triggers on any skill creation, modification, or hardening request. Enforces ANCHOR/REASON/SEAL protocols with F12 Defense, F8 Genius, and F1 Amanah.
version: "v64.1-GAGI-HARDENED"
authority: "Muhammad Arif bin Fazil"
---

# Skill Creator (v64.1-GAGI-HARDENED)

This skill provides guidance for creating effective, constitutionally-governed skills.

## About Skills

Skills are modular, self-contained packages that extend Kimi's capabilities by providing specialized knowledge, workflows, and tools. Think of them as "onboarding guides" for specific domains or tasks—they transform Kimi from a general-purpose agent into a specialized agent equipped with procedural knowledge that no model can fully possess.

### What Skills Provide

1. Specialized workflows - Multi-step procedures for specific domains
2. Tool integrations - Instructions for working with specific file formats or APIs
3. Domain expertise - Company-specific knowledge, schemas, business logic
4. Bundled resources - Scripts, references, and assets for complex and repetitive tasks

---

## ANCHOR Phase — Pre-Flight Environment Check [CRITICAL]

**Constitutional Floor:** F12 Injection Defense (Environmental Validity)

Before ANY skill work begins, validate the execution environment:

```
MANDATORY CHECKS:
├── C0_system_health (mode: "brief")
│   └── Ensure CPU < 80%, RAM available > 500MB
├── C5_config_flags
│   ├── Verify PYTHONPATH includes skill directory
│   ├── Verify .venv is active (if Python skill)
│   └── Check ARIFOS_CONSTITUTIONAL_MODE if applicable
└── C2_fs_inspect (path: target skills directory)
    └── Confirm write permissions exist
```

**ANCHOR Gates:**
- If C5 shows missing `.venv` → **VOID** with: "F12 Defense: Python environment not detected. Run: source .venv/bin/activate"
- If C2 shows permission denied → **VOID** with: "F12 Defense: Cannot write to skills directory"
- If C0 shows critical resource exhaustion → **SABAR** with: "System resources constrained, try again later"

**F12 Defense Rationale:** Prevents "Assumed Environment" failures that waste tokens and break flow.

---

## Core Principles

### Concise is Key

The context window is a public good. Skills share the context window with everything else Kimi needs: system prompt, conversation history, other Skills' metadata, and the actual user request.

**Default assumption: Kimi is already very smart.** Only add context Kimi doesn't already have. Challenge each piece of information: "Does Kimi really need this explanation?" and "Does this paragraph justify its token cost?"

Prefer concise examples over verbose explanations.

### Set Appropriate Degrees of Freedom

Match the level of specificity to the task's fragility and variability:

**High freedom (text-based instructions)**: Use when multiple approaches are valid, decisions depend on context, or heuristics guide the approach.

**Medium freedom (pseudocode or scripts with parameters)**: Use when a preferred pattern exists, some variation is acceptable, or configuration affects behavior.

**Low freedom (specific scripts, few parameters)**: Use when operations are fragile and error-prone, consistency is critical, or a specific sequence must be followed.

Think of Kimi as exploring a path: a narrow bridge with cliffs needs specific guardrails (low freedom), while an open field allows many routes (high freedom).

---

## Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter metadata (required)
│   │   ├── name: (required)
│   │   └── description: (required)
│   └── Markdown instructions (required)
└── Bundled Resources (optional)
    ├── scripts/          - Executable code (Python/Bash/etc.)
    ├── references/       - Documentation intended to be loaded into context as needed
    └── assets/           - Files used in output (templates, icons, fonts, etc.)
```

### SKILL.md (required)

Every SKILL.md consists of:

- **Frontmatter** (YAML): Contains `name` and `description` fields. These are the only fields that Kimi reads to determine when the skill gets used, thus it is very important to be clear and comprehensive in describing what the skill is, and when it should be used.
- **Body** (Markdown): Instructions and guidance for using the skill. Only loaded AFTER the skill triggers (if at all).

### Bundled Resources (optional)

**Scripts (`scripts/`)**: Executable code for tasks that require deterministic reliability or are repeatedly rewritten.

**References (`references/`)**: Documentation and reference material intended to be loaded as needed into context to inform Kimi's process and thinking.

**Assets (`assets/`)**: Files not intended to be loaded into context, but rather used within the output Kimi produces.

---

## Skill Creation Process (Hardened)

Skill creation involves these steps with constitutional governance:

1. **ANCHOR** — Pre-flight environment check (C0, C5) — **F12 Defense**
2. Understand the skill with concrete examples
3. Plan reusable skill contents (scripts, references, assets)
4. Initialize the skill
5. **REASON** — Edit the skill with Plan-Act-Verify loop — **F8 Genius**
6. Package the skill
7. **SEAL** — Session handoff with memory persistence — **F1 Amanah**
8. Iterate based on real usage

---

### Step 1: ANCHOR — Pre-Flight Environment Check [CRITICAL]

Execute C0, C5, C2 checks as defined in ANCHOR Phase above. Do not proceed until SEAL.

---

### Step 2: Understanding the Skill with Concrete Examples

Skip this step only when the skill's usage patterns are already clearly understood.

To create an effective skill, clearly understand concrete examples of how the skill will be used. Ask:

- "What functionality should this skill support?"
- "Can you give examples of how this skill would be used?"
- "What would a user say that should trigger this skill?"

Conclude this step when there is a clear sense of the functionality the skill should support.

---

### Step 3: Planning Reusable Skill Contents

Analyze each example to identify what scripts, references, and assets would be helpful:

- **Scripts**: When the same code is rewritten repeatedly
- **References**: When domain knowledge needs to be referenced
- **Assets**: When templates or boilerplate is reused

---

### Step 4: Initialize the Skill

Create a new skill directory with required `SKILL.md` and optional resource directories. Create only directories you intend to populate.

**Skill Naming:**
- Use lowercase letters, digits, and hyphens only
- Name under 64 characters
- Prefer short, verb-led phrases
- Name folder exactly after skill name

---

### Step 5: REASON — Edit the Skill with Plan-Act-Verify [HARDENED]

**Constitutional Floor:** F8 Genius (G = A × P × X × E² ≥ 0.80)

**REASON Protocol — REQUIRED for:**
- Any edit >50 lines
- Touching >3 files
- Any constitutional floor logic
- Any script that modifies user data

```
PLAN PHASE (Akal + Present):
├── Use sequentialthinking with totalThoughts ≥ 5
├── Include hypothesis: "What should change and why?"
├── Include verification: "How will I know it's correct?"
└── Calculate F8 components: A (clarity), P (relevance), X (exploration), E (effort)

ACT PHASE (Execution):
├── Make the edits
└── Run any included scripts to verify they work

VERIFY PHASE (Validation):
├── C2_fs_inspect — confirm files exist with expected structure
├── C3_log_tail (if scripts were run) — check for errors
└── Self-review: "Does this meet the original hypothesis?"
```

**F8 Genius Gate:**
- If Plan phase shows unclear approach → **SABAR**, ask for clarification
- If Verify phase shows hypothesis failed → rollback and replan
- If G-score would be < 0.80 → **VOID**, request smaller scope

**Stop Shotgun Debugging:** Never try random fixes. Always Plan → Act → Verify.

---

### Step 6: Package the Skill

Validate and package into distributable `.skill` file (zip archive):

1. **Validate**: Check YAML frontmatter, naming conventions, file organization
2. **Package**: Create archive preserving directory structure

```bash
cd <skills-root>
zip -r my-skill.skill my-skill
```

---

### Step 7: SEAL — Session Handoff [CRITICAL]

**Constitutional Floor:** F1 Amanah (Auditability & Continuity)

Before ending the session, persist critical context:

```
SEAL PROTOCOL:
├── memory: create_entities
│   └── Entity: Skill (name, type: "skill", location)
├── memory: add_observations to Skill entity
│   ├── Python version used (e.g., .venv313)
│   ├── Any F12/F11 issues encountered
│   ├── Key architectural decisions made
│   └── Dependencies required (if any)
└── aaa-mcp: vault_seal (if constitutional floors were tested)
    └── Query: "skill-creator session for {skill_name}"
```

**Anti-Amnesia:** The skill location, environment quirks, and gotchas are now persistent. Kimi vNext won't need to rediscover that `.venv313` is the correct Python environment.

---

### Step 8: Iterate

After testing the skill, notice struggles and inefficiencies. Iterate with REASON and SEAL for each significant change.

---

## Constitutional Compliance Matrix

| Step | Floor | Tool | Purpose |
|:---|:---|:---|:---|
| 1 | **F12** Defense | C0, C5, C2 | ANCHOR — Environment validity |
| 5 | **F8** Genius | sequentialthinking | REASON — Plan-Act-Verify |
| 7 | **F1** Amanah | memory, vault_seal | SEAL — Context persistence |

**Skipped floors?** Document why in `memory: add_observations` with tag "constitutional_waiver".

---

**DITEMPA BUKAN DIBERI** — Forged, Not Given.
