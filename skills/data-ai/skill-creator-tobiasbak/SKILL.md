---
name: skill-creator
description: "Specialized skill for creating new OpenCode agent skills. Detailed instructions on triggers, structure, and best practices. Triggers: 'create a skill', 'make a new skill', 'add a skill', 'new skill'."
license: MIT
compatibility: opencode
---

# skill-creator

This skill guides you through creating high-quality, reusable OpenCode agent skills.

## Best Practices

1.  **Naming**: Use `lowercase-alphanumeric-with-hyphens`. The name MUST match the directory name.
2.  **Description & Triggers**:
    *   Be precise. The agent uses this description to decide when to load the skill.
    *   **CRITICAL**: Always include common "catchphrases" or triggers in the description to help the agent identify when to use the skill.
    *   *Example description*: "Skill for analyzing logs. Triggers: 'check the logs', 'what happened in the logs', 'analyze errors'."
3.  **Frontmatter**: Always include `name` and `description`. `license` and `compatibility` are recommended.

... (rest of best practices)

## Steps to Create a Skill

1.  **Identify the behavior**: Define exactly what reusable logic you want to encapsulate.
2.  **Fetch Documentation**: Use the `webfetch` tool to read `https://opencode.ai/docs/skills/` to ensure the skill is created using the most up-to-date syntax and rules.
3.  **Determine Location**: Use the `question` tool to ask the user if the skill should be created **locally** (project-level) or **globally** (user-level).
    *   **Local**: `.opencode/skill/` or `.claude/skills/` (default to `.claude/skills/` if the project uses it).
    *   **Global**: `~/.config/opencode/skill/` or `~/.claude/skills/` (default to `~/.claude/skills/`).

3.  **Scaffold**:
    *   Create the skill directory in the chosen location.
    *   Create a `references/` subfolder if complex documentation is needed.
4.  **Initialize**: Use the `SKILL_TEMPLATE.md` to create the new `SKILL.md`. 
5.  **Draft Instructions**: Write the "how-to" for the agent. Use headers for clarity (e.g., `## Steps`, `## Guidelines`).
6.  **Verify**: Ensure the frontmatter follows YAML standards and the name follows the regex `^[a-z0-9]+(-[a-z0-9]+)*$`.

## Templates
Use the standard template from this skill's `references/` directory as a base.
