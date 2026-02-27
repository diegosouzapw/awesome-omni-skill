---
name: criador-skills
description: Helper skill to create new agent skills following the standard structure. Use this when you want to define a new capability or workflow for the agent.
---

# Skill Creator

This skill guides you through the process of creating a new agent skill.

## When to use this skill
- When the user wants to add a new capability to the agent.
- When the user wants to standardize a workflow (e.g., "deployment", "testing", "documentation").
- When you need to scaffold a new skill folder structure.

## Process

### 1. Gather Information
Before creating any files, ask the user for the following information if not already provided:
1.  **Skill Name**: A unique identifier in kebab-case (e.g., `my-new-skill`, `database-migration`).
2.  **Description**: A concise summary of what the skill does (goes into the YAML frontmatter).
3.  **Context/Role**: What role does the agent play when using this skill? (e.g., "You are a QA engineer...").
4.  **Instructions**: Specific steps, rules, or checklists the agent should follow.
5.  **Resources**: Any specific scripts, templates, or strict schemas needed?

### 2. Validation
- Ensure the name is URL-friendly (kebab-case).
- Ensure the directory doesn't already exist in `.agent/skills/`.

### 3. Execution
Perform the following actions:
1.  Create the directory: `.agent/skills/<skill-name>/`.
2.  Create the `SKILL.md` file at `.agent/skills/<skill-name>/SKILL.md`.
3.  (Optional) Create subdirectories `resources/`, `scripts/`, or `examples/` if the user mentioned them.

### 4. Template for SKILL.md
Use the following template for the new file:

```markdown
---
name: <skill-name>
description: <description>
---

# <Title Case Skill Name>

## When to use this skill
- <Add bullet points based on description>
- <Add scenarios where this skill is useful>

## Context
<Context/Role information>

## Instructions
<Detailed instructions provided by the user>
```

### 5. Final Confirmation
Inform the user that the skill has been created and is ready to be used.

## Example Interaction
**User**: "Create a skill for code review."
**Agent**: "I can help with that. What should be the specific focus of this code review skill? Do you have a preferred checklist?"
**User**: "It should focus on security and performance. Use the OWASP guidelines."
**Agent**: (Proceeds to create `.agent/skills/code-review/SKILL.md` with the relevant context).
