---
name: agent-creation
description: "Standards-compliant agent definitions with templates. Trigger: When creating agent definitions, setting up project agents, or documenting workflows."
license: "Apache 2.0"
metadata:
  version: "1.1"
  type: behavioral
  skills:
    - critical-partner
    - english-writing
---

# Agent Creation

Create project-specific agent definitions (AGENTS.md) with YAML frontmatter and structured markdown. Agents define purpose, skills, workflows, and policies for AI assistants working on a project.

## When to Use

- Creating a new agent definition from scratch
- Setting up project-specific agents
- Documenting agent workflows and responsibilities

Don't use for:

- Creating individual skills (use skill-creation)
- Creating context prompts (use prompt-creation)

---

## Critical Patterns

### ✅ REQUIRED: Select Mode Before Anything Else

Before gathering context or writing any content, determine which mode to use.

**Precedence order:**

1. **Explicit mode** (user specifies) — always respect this
   - Keywords: "interview mode", "no context", "ask me questions"
   - Keywords: "analysis mode", "analyze the project", "read the project"
2. **Auto-detect** (if no mode specified):

```
Is destination project context visible (not the framework itself)?
  YES → Analysis Mode (confirm with user before proceeding)
  NO  → Interview Mode
```

**Auto-detect warning:** If the current working directory appears to be the `ai-agents-skills` framework (e.g., `package.json` shows `"name": "ai-agents-skills"`), do **not** auto-select Analysis Mode. Confirm with the user which project to analyze or switch to Interview Mode.

Signals that destination project context is available:

- `package.json` with a project name different from the framework
- `README.md` describing the destination project
- `src/` directory with application source code

---

### ✅ REQUIRED: Gather Context — Two Modes

#### Interview Mode (no project context available)

Ask these 9 questions in order. Wait for answers before proceeding.

1. What is the primary purpose of this agent?
2. What input will it receive? (files, text, user queries)
3. What is the expected output format?
4. Which skills does it need?
5. Specific workflows, policies, or constraints?
6. Target audience? (developers, end-users, AI assistants)
7. Technologies, frameworks, or tools involved?
8. Project context where this agent operates?
9. Tone or communication style? (formal, casual, technical)

**Do not proceed until all required questions are answered.**

#### Analysis Mode (destination project context is available)

Read files in this order and extract answers to the 9 questions above:

| File | Extract |
|------|---------|
| `package.json` | Project name, dependencies, framework versions |
| `README.md` | Project purpose, architecture overview, tech stack |
| `tsconfig.json` | TypeScript strictness, target, paths |
| `src/` structure | Main patterns, component organization, layers |
| `.github/workflows/` | CI/CD tools, test runners, linting setup |
| `AGENTS.md` (existing) | Current skills, workflows already defined |

After analysis, **only ask the user about gaps** — information that could not be inferred from the files.

Always confirm your findings before writing: "Based on your project, I found: X, Y, Z. Does this look correct?"

---

### ✅ REQUIRED: Frontmatter Structure

```yaml
name: my-project-agent # Required: lowercase-with-hyphens
description: "Development assistant for Project X. Expert in TypeScript, React, MUI."
license: "Apache 2.0"
metadata:
  version: "1.0"
  skills:
    - typescript
    - react
    - critical-partner # Mandatory for ALL agents
    - code-conventions # Mandatory for coding-related agents
    - a11y
---
```

Rules:

- `name`, `description`, `skills` are required
- Always include `critical-partner` in skills
- Use YAML list syntax (`- item`), never `[]`
- Omit empty fields completely

---

### ✅ REQUIRED: Content Sections

After frontmatter, include:

1. **# Agent Name** — Title (e.g., "Alpha Project Agent")
2. **## Purpose** — Clear statement of responsibilities
3. **## How to Use Skills** — Auto-discovery workflow (BEFORE Skills Reference table)
4. **## Skills Reference** — Table: Trigger | Skill | Path (**only if skills are already installed**)
5. **## Project Structure & Skills Storage** — Symlink explanation (AFTER Skills Reference table)
6. **## Supported Stack** (if applicable) — Technologies and versions
7. **## Workflows** (optional) — Feature dev, code review, bug fix flows
8. **## Policies** (optional) — Typing rules, accessibility, version constraints

---

### ✅ REQUIRED: How to Use Skills Section

AGENTS.md must include this section BEFORE the Skills Reference table. It must always include auto-discovery as the primary mechanism, regardless of whether skills are installed.

**Template (copy this into AGENTS.md):**

```markdown
## How to Use Skills (MANDATORY WORKFLOW)

This project has skills installed in your model's skills directory. Follow this protocol for ALL coding tasks:

### Step 1: Discover Available Skills

List the contents of your model's skills directory:

- **Cursor:** `.cursor/skills/`
- **Claude:** `.claude/skills/`
- **Copilot:** `.github/skills/`
- **Gemini:** `.gemini/skills/`
- **Codex:** `.codex/skills/`

Each installed skill has a `SKILL.md` file. Read the `description` field to understand when to use it.

### Step 2: Match Task to Skill

Check the "Skills Reference" table below (if present) for a quick lookup.
If your task is not in the table, or the table is absent, scan `.{model}/skills/` for the most relevant skill.

### Step 3: Read the Skill

**Path format:** `.{model}/skills/{skill-name}/SKILL.md`

Replace `{model}` with your coding agent:

- **Cursor:** `.cursor/skills/typescript/SKILL.md`
- **Claude:** `.claude/skills/typescript/SKILL.md`
- **Copilot:** `.github/skills/typescript/SKILL.md`
- **Gemini:** `.gemini/skills/typescript/SKILL.md`
- **Codex:** `.codex/skills/typescript/SKILL.md`

### Step 4: Read Dependencies

Every skill lists dependencies in its frontmatter (`metadata.skills`). Read each dependency before proceeding.

### Step 5: Apply Patterns

- Follow "Critical Patterns" marked with ✅ REQUIRED
- Use "Decision Tree" for implementation choices
- Reference inline code examples

### Example Workflow

**Task:** "Create TypeScript interface for User model"

1. **Scan** `.{model}/skills/` → find `typescript/SKILL.md`
2. **Read:** `.{model}/skills/typescript/SKILL.md`
3. **Check frontmatter** → Dependencies: `javascript`
4. **Read dependency:** `.{model}/skills/javascript/SKILL.md`
5. **Apply patterns:** Use `interface` (not `type`), PascalCase names, export from `types/` directory
```

**Why auto-discovery matters:**

- Skills Reference table reflects skills known at creation time — new skills installed later are not listed
- Auto-discovery via directory listing always reflects the current installed state
- Both mechanisms together ensure 100% coverage at all times

---

### ✅ REQUIRED: Skills Reference Table (Conditional)

Include this section **only if skills have been identified** — either specified by the user (Interview Mode Q4) or detected from project files (Analysis Mode). If no skills were identified at all, omit this section entirely.

Every identified skill must appear in **both** places:

1. `metadata.skills` in the frontmatter
2. A row in the Skills Reference table

This applies regardless of whether the skills are physically installed yet. The table represents the intended skill set for this agent.

When included, place it AFTER the `How to Use Skills` section:

```markdown
## Skills Reference

**IMPORTANT:** Paths shown are model-agnostic. See "How to Use Skills" above for your model's actual path.
New skills installed after this file was created are auto-discovered via `.{model}/skills/`.

| Trigger                     | Skill                 | Relative Path                                 |
| --------------------------- | --------------------- | --------------------------------------------- |
| TypeScript types/interfaces | typescript            | {model}/skills/typescript/SKILL.md            |
| React components/hooks      | react                 | {model}/skills/react/SKILL.md                 |
| Code review                 | critical-partner      | {model}/skills/critical-partner/SKILL.md      |
```

Rules for the table:

- List every skill identified during context gathering (interview or analysis)
- Use `{model}` placeholders for model-agnostic paths
- Keep formatting compact to save tokens

---

### ✅ REQUIRED: Add "Project Structure & Skills Storage" Section

AGENTS.md must include this section AFTER the Skills Reference table. This explains the symlink architecture for LLMs that struggle with symlink resolution.

**Template (copy this into AGENTS.md):**

````markdown
## Project Structure & Skills Storage

**IMPORTANT FOR LLMs:** Skills use a 3-layer symlink structure:

\```
your-project/
├── .agents/skills/ # Canonical symlinks to framework skills/ (shared across models)
│ ├── react/ → ../../skills/react/
│ ├── typescript/ → ../../skills/typescript/
│ └── ...
├── .claude/skills/ # Claude-specific symlinks to .agents/skills/
│ ├── react/ → ../../.agents/skills/react/
│ └── typescript/ → ../../.agents/skills/typescript/
├── .cursor/skills/ # Cursor-specific symlinks to .agents/skills/
├── .github/skills/ # Copilot-specific symlinks to .agents/skills/
├── .gemini/skills/ # Gemini-specific symlinks to .agents/skills/
├── .codex/skills/ # Codex-specific symlinks to .agents/skills/
└── AGENTS.md # This file
\```

**How to access skills:**

- **Preferred:** Read from `.{model}/skills/<skill-name>/SKILL.md` (your model's directory)
- **If symlinks fail:** Skills are stored in the ai-agents-skills framework installation (referenced via symlinks)
- **Real files location:** All source skills are in the framework's `skills/` directory

**Why 3 layers?**

1. **Layer 1 (framework skills/):** Source of truth maintained by framework
2. **Layer 2 (.agents/skills/):** Canonical shared location in your project
3. **Layer 3 (.{model}/skills/):** Model-specific access for your AI assistant

**Benefits:**

- **Zero duplication:** Skills installed once, available to all 5 AI models
- **Always up-to-date:** Changes propagate instantly via symlinks
- **Token-efficient:** Your AI reads only the skills it needs
````

---

### ❌ NEVER: Skip Mode Selection

Always determine the mode (Interview or Analysis) before gathering any context. Jumping straight to writing the AGENTS.md leads to incomplete or incorrect agent definitions.

### ❌ NEVER: Analyze the Framework as the Destination Project

If running from within `ai-agents-skills`, confirm the destination project before using Analysis Mode. Reading the framework's own `package.json` or `README.md` produces an agent definition for the wrong project.

### ❌ NEVER: Include Skills Reference Table When No Skills Are Installed

An empty table or a table with placeholder rows provides no value and misleads the model. Omit the section entirely if no skills exist yet.

---

## Decision Tree

```
Explicit mode specified by user?
  YES → Use that mode directly
  NO  → Auto-detect:
          Working directory = ai-agents-skills framework?
            YES → Interview Mode (or confirm with user)
            NO  → Destination project files visible?
                    YES → Analysis Mode (confirm findings with user first)
                    NO  → Interview Mode

Interview Mode:
  All 9 questions answered? → NO → Stop: Ask clarifying questions
  → Proceed to create AGENTS.md

Analysis Mode:
  Files read and findings confirmed? → NO → Confirm with user
  All gaps filled? → NO → Ask only about missing information
  → Proceed to create AGENTS.md

Both modes:
  All required skills identified? → NO → Ask: Which skills needed?
  Agent has complex workflows? → YES → Add Workflows section
  Agent has version constraints? → YES → Add Policies section
  Skills installed in project? → YES → Add Skills Reference table
                               → NO  → Omit Skills Reference table
  All referenced skills exist? → NO → Verify paths
  critical-partner in skills? → NO → Must include (mandatory)
```

---

## Workflow

0. **Select mode** → Explicit (user-specified) or auto-detect
1. **Gather context** → Analysis Mode: read project files, infer 9 answers, ask only gaps | Interview Mode: ask all 9 questions
2. **Confirm context** → Show findings to user before writing (both modes)
3. **Create structure** → `mkdir presets/{project-name}` + create `AGENTS.md`
4. **Write frontmatter** → name, description, skills list
5. **Add "How to Use Skills" section** → Complete workflow with auto-discovery (BEFORE Skills Reference table)
6. **Write Skills Reference table** → Only if skills are installed; use `{model}` placeholders
7. **Add "Project Structure & Skills Storage" section** → Complete symlink documentation (AFTER Skills Reference table)
8. **Write content** → Purpose, Supported Stack, Workflows, Policies
9. **Validate** → Run checklist below, verify all skills exist

---

## Example

### Interview Mode

```yaml
name: example-agent
description: "Development assistant for Example Project. TypeScript, React, accessibility."
license: "Apache 2.0"
metadata:
  version: "1.0"
  skills:
    - typescript
    - react
    - critical-partner
    - code-conventions
    - a11y
---
```

````markdown
# Example Project Agent

## Purpose

Primary development assistant ensuring code quality, accessibility, and TypeScript/React best practices.

## How to Use Skills (MANDATORY WORKFLOW)

### Step 1: Discover Available Skills

List your model's skills directory (`.{model}/skills/`) to see all installed skills.
Each skill has a `SKILL.md` with a `description` field showing when to use it.

### Step 2: Match Task to Skill

Check the Skills Reference table below for a quick lookup.
If your task is not listed, scan `.{model}/skills/` for the most relevant skill.

### Step 3: Read the Skill

**Path format:** `.{model}/skills/{skill-name}/SKILL.md`

...

## Skills Reference

**IMPORTANT:** Paths shown are model-agnostic. See "How to Use Skills" above for your model's actual path.
New skills installed after this file was created are auto-discovered via `.{model}/skills/`.

| Trigger                     | Skill                 | Relative Path                                 |
| --------------------------- | --------------------- | --------------------------------------------- |
| TypeScript types/interfaces | typescript            | {model}/skills/typescript/SKILL.md            |
| React components/hooks      | react                 | {model}/skills/react/SKILL.md                 |
| Code review                 | critical-partner      | {model}/skills/critical-partner/SKILL.md      |

## Project Structure & Skills Storage

**IMPORTANT FOR LLMs:** Skills use a 3-layer symlink structure:
...

## Supported Stack

- TypeScript 5.0+, React 18+, Vite

## Policies

- Strict typing (no `any`), keyboard-accessible components, React hooks best practices
````

### Analysis Mode

**Scenario:** User runs the skill from a React + TypeScript project directory.

```
Agent reads: package.json → { "name": "my-store", "dependencies": { "react": "^18", "typescript": "5" } }
Agent reads: README.md → "E-commerce platform built with React, TypeScript, and Redux Toolkit"
Agent reads: tsconfig.json → { "strict": true }
Agent reads: src/ → components/, store/, hooks/, pages/

Agent infers:
  Q1 (purpose): E-commerce development assistant
  Q2 (input): User stories, bug reports, code review requests
  Q3 (output): TypeScript components, Redux slices, hooks
  Q4 (skills): typescript, react, redux-toolkit, critical-partner
  Q7 (tech): React 18, TypeScript 5, Redux Toolkit, Vite

Agent confirms: "Based on your project, I found: React 18 + TypeScript 5 e-commerce app.
  Skills to include: typescript, react, redux-toolkit, critical-partner.
  Does this look correct? Anything to add or change?"

User confirms → Agent creates AGENTS.md
```

---

## Edge Cases

**Agent with 20+ skills:** Group skills in the reference table by category (Framework, Testing, Standards).

**Multiple agents per project:** Each agent should have distinct responsibility. Avoid skill overlap.

**Modifying existing agents:** Re-gather context for changed requirements before updating.

**No skills installed yet:** Omit the Skills Reference table. The auto-discovery section in `How to Use Skills` is sufficient — the model will find skills as they are installed.

**Skills added after creation:** The auto-discovery section handles this automatically. No need to update the AGENTS.md when new skills are installed.

---

## Checklist

- [ ] Mode selected (Interview or Analysis) before gathering context
- [ ] In Analysis Mode: confirmed that analyzed context belongs to the destination project, not the framework
- [ ] Context gathered and confirmed with user (both modes)
- [ ] Directory under `presets/` (lowercase-with-hyphens)
- [ ] `AGENTS.md` with frontmatter: `name`, `description`, `skills`
- [ ] `critical-partner` in skills (mandatory for all)
- [ ] "How to Use Skills" section added BEFORE Skills Reference table, with auto-discovery as Step 1
- [ ] Skills Reference table included ONLY if skills are already installed in the project
- [ ] Skills Reference table uses `{model}` placeholders (model-agnostic paths)
- [ ] "Project Structure & Skills Storage" section added AFTER Skills Reference table
- [ ] All referenced skills exist in `.agents/skills/`
- [ ] Purpose section is clear and actionable
- [ ] Token-efficient (no filler words)
- [ ] Follows english-writing skill guidelines

---

## Resources

- [agents.md spec](https://agents.md/)
- [Agent Skills](https://agentskills.io/)
- [skill-creation](../skill-creation/SKILL.md)
- [critical-partner](../critical-partner/SKILL.md)
