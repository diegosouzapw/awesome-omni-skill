---
name: instruction-creator
description: 'Create and manage high-quality custom instruction files for GitHub Copilot. Use when you need to define new project-specific guidelines, workflows, or coding standards in the instructions/ directory.'
---

# Instruction Creator

This skill guides the creation of effective custom instruction files that help GitHub Copilot follow project conventions and domain-specific logic.

## Workflow

1.  **Define Scope**: Identify the purpose and the files the instructions should apply to (e.g., `**/*.ts`).
2.  **Initialize**: Use the `scripts/init_instruction.py` script to create the boilerplate.
    ```bash
    python skills/instruction-creator/scripts/init_instruction.py "my-new-instruction" --description "Description here" --applyTo "**/*.ext"
    ```
3.  **Draft Content**: Follow the [guidelines](references/guidelines.md) to fill in the sections.
4.  **Use Templates**: Refer to [templates](references/templates.md) for standard structures.
5.  **Semantic Linking**: Ensure you link to relevant Skills in the `Workflow` section.
    - Example: `To perform [Task], execute the [Skill Name](skills/<skill-name>/SKILL.md).`
6.  **Validate**: Test the instructions with Copilot to ensure they are clear and actionable.

## Core Principles

-   **Policy Maker**: Instructions define the "What" and "How" (decision logic and standards).
-   **The 5 Sections**: Effective instructions include: Overview, Tech Stack, Structure, Guidelines, and Resources.
-   **Concise & focused**: Avoid fluff; use imperative language. Limit files to ~500 lines.
-   **Actionable**: Provide concrete "Good" and "Bad" code examples and clear steps.
-   **Linked**: Connect instructions to the skills that execute them and internal automation scripts.

## Resources

-   **[Guidelines](references/guidelines.md)**: Detailed rules for frontmatter, style, and linking.
-   **[Templates](references/templates.md)**: Ready-to-use markdown structures.
-   **`scripts/init_instruction.py`**: Automation for creating new files.
