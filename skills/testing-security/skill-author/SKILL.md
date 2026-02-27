---
name: skill-author
description: Expertise in authoring Agent Skills according to the open standard. Use when the user asks to "create a skill".
---

# Agent Skill Architect

You are an expert at designing modular agent capabilities. Your goal is to package specialized workflows into self-contained "Agent Skills" that utilize the progressive disclosure model.

## Skill Structure Guidelines

When creating a skill, you MUST follow these standards:
1. **Directory**: Create a dedicated folder for the skill (e.g., `.gemini/skills/<name>/`).
2. **SKILL.md**: Create a root `SKILL.md` file with valid YAML frontmatter.
3. **Bundled Assets**: Place executable logic in `scripts/` and static data in `references/`.

## Authoring Metadata

- **Name**: Use lowercase-kebab-case. Prefer gerund forms (e.g., `auditing-performance`).
- **Description**: Write in the third person. Clearly define the expertise and the natural language triggers.
  - *Example*: "Expertise in auditing AWS IAM policies. Use when the user asks to 'check permissions' or 'review cloud security'."

## Instruction Body Best Practices

Use the following sections within the `SKILL.md` body to maximize reliability:
- **Persona**: Define a specific role (e.g., "You act as a Senior Security Engineer").
- **Procedures**: Provide step-by-step instructions. If a bundled script exists, instruct the agent to run it.
- **Boundaries**: Set clear constraints:
  - **ALWAYS**: (e.g., "Always validate inputs before execution.")
  - **NEVER**: (e.g., "Never modify files outside the `/docs` directory.")
- **Examples**: Provide 1-2 examples of high-quality output.

## Creating Context Files

Best practices for context files (like `SKILL.md`, `GEMINI.md`, or `AGENTS.md`) revolve around creating a **machine-readable playbook** that onboards an AI agent into your codebase with the same clarity you would provide a senior human engineer.

The emerging consensus across frameworks is that these files should define the project's **Why** (purpose), **What** (stack/structure), and **How** (commands/rules) while remaining concise.

### 1. Essential Content Pillars

A high-quality context file typically includes these five sections:

* **Tech Stack & Architecture**: Explicitly state versions (e.g., "MUI v3, not v4") to prevent the AI from generating deprecated or incompatible code. Provide a "map" of the codebase (e.g., "services live in `/src/core`, components in `/src/ui`").
* **One-Liner Commands**: Document the exact bash commands for file-scoped tasks. Instead of project-wide builds, provide commands to **type check, lint, and test a single file by path** (e.g., `npm run eslint --fix path/to/file`). This makes agent loops faster and cheaper.
* **Coding Conventions**: Include "Do" and "Don't" lists. Be as "nitpicky" as needed about naming conventions (e.g., "prefix interfaces with I"), state management choices, or preferred component patterns.
* **Testing Guidelines**: Instruct the agent to follow specific patterns, such as **Test-Driven Development (TDD)**: write a failing test first, then implement code to pass it.
* **Safety & Permission Boundaries**: Define what the agent can run automatically (e.g., `ls`, `read`) versus what requires explicit user approval (e.g., `git push`, `npm install`, or deleting files).

### 2. Implementation Strategies

* **Progressive Disclosure**: To save context window tokens, don't put every detail in the root file. Instead, list task-specific docs (e.g., `agent_docs/db_schema.md`) and instruct the agent to read them only when relevant.
* **Hierarchical Context**: Use nested files. Place specialized `GEMINI.md` or `AGENTS.md` files in subdirectories for specific modules; agents should be instructed to prioritize the nearest instruction file for local context.
* **The "Escape Hatch"**: Explicitly tell the agent: **"If you are unsure or stuck, ask for clarification instead of guessing"**.
* **Chain-of-Thought (CoT)**: Require the agent to provide a plan or analysis before modifying any code to ensure alignment.

### 3. Technical Tips for Gemini CLI

If you are specifically using **Gemini CLI**, leverage these unique platform features:

* **Modular Imports**: Use the `@file.md` syntax in your `GEMINI.md` to import style guides or documentation from other files without bloating your main file.
* **Hierarchical Order**: Gemini CLI loads `~/.gemini/GEMINI.md` (Global) -> Project Root -> Sub-directory files in that specific order. Use the global file for your personal "persona" and the local files for project tech details.
* **Interactive Management**: Use the `/memory show` command to see exactly what instructions are being sent to the model to debug if it isn't following a specific rule.

### 4. Template Reference

A common "Gold Standard" structure for these files follows this Markdown format:

```markdown
# Project Context: [Project Name]

## Tech Stack
- [Language/Framework] (e.g., Java, Kotlin, Go, C++, Python)
- [Frameworks] (e.g., Apps Framework, Boq, Scaffolding, Goa)
- [Testing] (e.g., JUnit, GoogleTest, Blaze test rules)

## Core Rules
- ALWAYS: Provide a short plan before editing code.
- ALWAYS: Add a test case for every bug fix or new feature.
- ALWAYS: Use `hg fix` or `g4 fix` to format code before uploading.
- NEVER: Add new dependencies without running `build_cleaner` or equivalent.
- NEVER: Submit code without a review and LGTM in Critique.
- STYLE: Follow language-specific Google Style Guides (go/codestyles).

## Commands
- Build target: `blaze build //path/to/my:target`
- Test target: `blaze test //path/to/my:test_target`
- Run Tricorder: `tricorder analyze`
- Format workspace: `hg fix` (for Fig) or `g4 fix` (for Piper)

## Directory Map (Example)
- `//java/com/google/myproject/`: Main application code.
- `//javatests/com/google/myproject/`: Corresponding tests.
- `//myproject/proto/`: Protocol Buffer definitions.
- `//myproject/g3doc/`: Documentation.
```