---
name: agent-creation
description: "Standards-compliant agent definitions with templates. Trigger: When creating agent definitions, setting up project agents, or documenting workflows."
license: "Apache 2.0"
metadata:
  version: "1.0"
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

### ✅ REQUIRED: Gather Context First (9 Questions)

Before creating an agent, gather answers to:

1. What is the primary purpose of this agent?
2. What input will it receive? (files, text, user queries)
3. What is the expected output format?
4. Which skills does it need?
5. Specific workflows, policies, or constraints?
6. Target audience? (developers, end-users, AI assistants)
7. Technologies, frameworks, or tools involved?
8. Project context where this agent operates?
9. Tone or communication style? (formal, casual, technical)

**Do not proceed until you have sufficient context.**

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

### ✅ REQUIRED: Content Sections

After frontmatter, include:

1. **# Agent Name** - Title (e.g., "Alpha Project Agent")
2. **## Purpose** - Clear statement of responsibilities
3. **## Skills Reference** - Table: Trigger | Skill | Path

- **Avoid unnecessary whitespace in markdown tables.** Cells and separators should be compact to save tokens and improve readability.

4. **## Supported Stack** (if applicable) - Technologies and versions
5. **## Workflows** (optional) - Feature dev, code review, bug fix flows
6. **## Policies** (optional) - Typing rules, accessibility, version constraints

### ✅ REQUIRED: Add "How to Use Skills" Section

AGENTS.md must include this section BEFORE the Skills Reference table. This is the "push context" that ensures AI models understand how to discover and use skills.

**Template (copy this into AGENTS.md):**

```markdown
## How to Use Skills (MANDATORY WORKFLOW)

This project has skills installed in your model's skills directory. Follow this protocol for ALL coding tasks:

### Step 1: Find the Trigger

Check the "Skills Reference" table below. Match your task to the "Trigger" column.

### Step 2: Read the Skill

**Path format:** `.{model}/skills/{skill-name}/SKILL.md`

Replace `{model}` with your coding agent:

- **Cursor:** `.cursor/skills/typescript/SKILL.md`
- **Claude:** `.claude/skills/typescript/SKILL.md`
- **Copilot:** `.github/skills/typescript/SKILL.md`
- **Gemini:** `.gemini/skills/typescript/SKILL.md`
- **Codex:** `.codex/skills/typescript/SKILL.md`

### Step 3: Read Dependencies

Every skill lists dependencies in its frontmatter (`metadata.skills`). Read each dependency skill before proceeding.

**Example:** `react` skill depends on: `code-conventions`, `a11y`, `typescript`, `javascript`, `architecture-patterns`, `humanizer`

You must read all 6 skills.

### Step 4: Apply Patterns

- Follow "Critical Patterns" marked with ✅ REQUIRED
- Use "Decision Tree" for implementation choices
- Reference inline code examples

### Example Workflow

**Task:** "Create TypeScript interface for User model"

1. **Check table below** → Trigger: "TypeScript types/interfaces" → Skill: `typescript`
2. **Read:** `.{model}/skills/typescript/SKILL.md`
3. **Check frontmatter** → Dependencies: `code-conventions`, `javascript`
4. **Read dependencies:**
   - `.{model}/skills/code-conventions/SKILL.md`
   - `.{model}/skills/javascript/SKILL.md`
5. **Apply patterns:** Use `interface` (not `type`), PascalCase names, export from `types/` directory
```

**Why this matters:**

- Vercel research shows AGENTS.md "push context" achieves 100% success rate vs on-demand discovery
- All models (Cursor, Claude, Copilot, Gemini, Codex) auto-discover skills from `.{model}/skills/` directories
- Complete workflow in AGENTS.md ensures consistent interpretation

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

**Why this matters:**

- Some LLMs (Cursor, Claude, etc.) struggle resolving multiple levels of symlinks
- Explicit documentation ensures any AI assistant can find and read skills
- Fallback instructions prevent skill access failures

### ❌ NEVER: Skip Context Gathering

Don't create an agent without answering the 9 context questions. Incomplete context leads to vague, unhelpful agents.

---

## Decision Tree

```
Context gathered (9 questions)? → NO → Stop: Ask clarifying questions
All required skills identified? → NO → Ask: Which skills needed?
Agent has complex workflows? → YES → Add Workflows section
Agent has version constraints? → YES → Add Policies section with versions
All referenced skills exist in skills/? → NO → Stop: Verify paths
critical-partner in skills? → NO → Stop: Must include (mandatory)
```

---

## Workflow

1. **Gather context** → Ask 9 questions, understand project needs
2. **Create structure** → `mkdir presets/{project-name}` + create `AGENTS.md`
3. **Write frontmatter** → name, description, skills list
4. **Add "How to Use Skills" section** → Complete workflow from template above (BEFORE Skills Reference table)
5. **Write Skills Reference table** → Use `{model}` placeholders for model-agnostic paths
6. **Add "Project Structure & Skills Storage" section** → Complete symlink documentation from template above (AFTER Skills Reference table)
7. **Write content** → Purpose, Supported Stack, Workflows, Policies
8. **Validate** → Run checklist below, verify all skills exist

---

## Example

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

## Skills Reference

**IMPORTANT:** Paths shown are model-agnostic. See "How to Use Skills" above for your model's actual path.

| Trigger                     | Skill                 | Relative Path                                 |
| --------------------------- | --------------------- | --------------------------------------------- |
| TypeScript types/interfaces | typescript            | {model}/skills/typescript/SKILL.md            |
| React components/hooks      | react                 | {model}/skills/react/SKILL.md                 |
| Code review                 | critical-partner      | {model}/skills/critical-partner/SKILL.md      |

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

## Supported Stack

- TypeScript 5.0+, React 18+, Vite

## Policies

- Strict typing (no `any`), keyboard-accessible components, React hooks best practices
````

---

## Edge Cases

**Agent with 20+ skills:** Group skills in the reference table by category (Framework, Testing, Standards).

**Multiple agents per project:** Each agent should have distinct responsibility. Avoid skill overlap.

**Modifying existing agents:** Re-gather context for changed requirements before updating.

---

## Checklist

- [ ] Context gathered (9 questions answered)
- [ ] Directory under `presets/` (lowercase-with-hyphens)
- [ ] `AGENTS.md` with frontmatter: `name`, `description`, `skills`
- [ ] `critical-partner` in skills (mandatory for all)
- [ ] "How to Use Skills" section added BEFORE Skills Reference table
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
