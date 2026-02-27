---
name: copilot-customization-features
description: Use when creating or modifying any GitHub Copilot customization features in VS Code, including custom instructions, AGENTS.md, agent skills, prompt files, custom agents, agent hooks, or AGENTS.md files. Helps select the correct feature type and create, review, or modify it for optimal performance.
---

# Skill Instructions

## Overview

GitHub Copilot provides seven customization features that allow you to tailor how the AI assistant behaves in your workspace. These features enable you to define coding standards, create reusable workflows, build specialized tools, and control how guidance is applied across your codebase. Each feature serves a distinct purpose—from always-on workspace rules to explicit user-invoked prompts—and choosing the right one depends on your specific needs.

This skill helps you:

- Understand the seven feature types and when to use each one
- Select the appropriate feature based on your requirements
- Format and structure each feature type correctly
- Navigate detailed documentation for implementation

## How to Use this Skill

**CRITICAL**: The current file is just an overview, but the files linked have much more information. Once a feature type is selected, you **MUST** read the associated linked file for much more information.

Use the Feature Reference for quick lookup, the Feature Selection Guide to make informed choices, and the linked documentation for comprehensive implementation guidance.

See also the **Global Rules** section for important rules which apply when creating or editing all features.

## Feature Reference

| Feature                 | Usage                            | Selection/Activation                                 | File Location                                   | Documentation                                              |
| ----------------------- | -------------------------------- | ---------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| **Custom Instructions** | Always-on coding standards       | Auto-applied                                         | `.github/copilot-instructions.md`               | [Custom Instructions](./references/custom-instructions.md) |
| **Instructions Files**  | File- or task-specific rules     | Auto-applied by `applyTo` or manual attach           | `.github/instructions/*.instructions.md`        | [Instructions Files](./references/instructions-files.md)   |
| **AGENTS.md**           | Global agent instructions        | Auto-applied                                         | `AGENTS.md` (workspace root or subfolders)      | [AGENTS.md](./references/agents-md.md)                     |
| **Agent Skills**        | On-demand capabilities           | Auto-loaded by `description` match or slash commands | `.github/skills/<name>/SKILL.md`                | [Agent Skills](./references/agent-skills.md)               |
| **Prompt Files**        | Explicit, repeatable tasks       | Run explicitly by user via slash commands            | `.github/prompts/<name>.prompt.md`              | [Prompt Files](./references/prompt-files.md)               |
| **Custom Agents**       | Role-specific tools+instructions | Selected by user or subagent invocation              | `.github/agents/<name>.agent.md`                | [Custom Agents](./references/custom-agents.md)             |
| **Agent Hooks**         | Lifecycle automation             | Auto-fires at lifecycle events                       | `.github/hooks/*.json`, `.claude/settings.json` | [Agent Hooks](./references/agent-hooks.md)                 |

### Feature Purposes

- **Custom Instructions** – Define workspace-wide rules, standards, and conventions that are automatically applied to every chat request in the workspace. Use for guidance that affects all development work.
- **Instructions Files** – Define scoped instructions for specific files, languages, or folders that activate based on file patterns or semantic task matching. Use when rules should apply only to certain contexts.
- **AGENTS.md** – Provide custom instructions intended for all agents in the workspace, designed to be tool-agnostic and work across multiple AI assistants (GitHub Copilot, Claude Code, etc.).
- **Agent Skills** – Provide specialized, reusable capabilities with optional bundled assets (scripts, templates, checklists) that load on demand when requests match the skill description or when invoked via slash commands.
- **Prompt Files** – Provide explicit, reusable prompts for repeatable tasks that users invoke deliberately through slash commands, the command palette, or editor buttons.
- **Custom Agents** – Define specialist assistants with explicit scope, tool restrictions, and constraints for role-specific workflows like planning, code review, or research.
- **Agent Hooks** – Execute custom shell commands deterministically at specific agent lifecycle points (SessionStart, PreToolUse, PostToolUse, etc.) to enforce security policies, automate quality checks, create audit trails, or inject context.

## Feature Selection Guide

### When to Use Each Feature

Use this flow to quickly choose the right feature type:

| Requirement                                                    | Feature Type                                          |
| -------------------------------------------------------------- | ----------------------------------------------------- |
| Should this apply to every request in the workspace?           | Custom instructions or AGENTS.md                      |
| Should this apply only to specific files/folders?              | Instructions files (with `applyTo`)                   |
| Should this load on-demand when relevant?                      | Instructions files (with `description`), Agent Skills |
| Is this a single focused task with inputs?                     | Prompt files                                          |
| Does this need bundled assets (scripts/templates)?             | Agent Skills                                          |
| Does this need tool restrictions or context isolation?         | Custom agents                                         |
| Is this a multi-stage workflow with handoffs?                  | Custom agents                                         |
| Do you need deterministic automation with guaranteed outcomes? | Agent Hooks                                           |
| Do you need to block/modify tool execution before it happens?  | Agent Hooks (PreToolUse event)                        |

**Important:** Only **one** slash command can be used at a time. This includes skills and prompt files invoked via slash commands, so you cannot manually select multiple skills/prompts in a single request.

### Comparing Across Features

Use these criteria when choosing between similar feature types:

### Instructions Files vs Agent Skills

- Does this apply to _most_ work in a domain? → Instructions Files
- Does this apply to _specific_ tasks or workflows? → Agent Skills
- Example: "Always use type hints in Python" → Instructions. "Generate Jest tests with mocks" → Skill.

### Agent Skills vs Prompt Files

- Multi-step workflow with bundled assets (scripts, templates)? → Agent Skills
- Single focused task with parameterized inputs? → Prompt Files
- Example: "Security audit with checklist + scripts" → Skill. "Generate API route from method + path" → Prompt.

### Agent Skills vs Custom Agents

- Same capabilities needed for all steps? → Agent Skills
- Need context isolation (subagent returns single output)? → Custom Agents
- Need different tool restrictions per stage? → Custom Agents
- Example: "Generate tests following team standards" → Skill. "Plan feature (read-only) → Implement → Review (different tools each stage)" → Custom Agents.

### Custom Instructions vs AGENTS.md

- GitHub Copilot-specific standards? → Custom instructions
- Tool-agnostic rules for multiple AI assistants? → AGENTS.md
- Use both when needed; they complement each other.

### Instructions/Skills vs Agent Hooks

- Need to guide agent behavior? → Instructions or Skills
- Need guaranteed deterministic execution? → Agent Hooks
- Need to block operations before they execute? → Agent Hooks (PreToolUse)
- Need to run code at specific lifecycle points? → Agent Hooks
- Example: "Always validate inputs" → Instructions. "Run linter after every file edit" → Hook.

## Global rules

### Naming

- Use lowercase, hyphen-separated names for variable-name features (skills, prompts, agents).
- Avoid generic names (e.g., "testing", "instructions", "standards").
- Use descriptive, action-oriented names that indicate purpose or capability (e.g., `jest-test-generation`, `generate-api-route`, `code-reviewer`).
- Fixed-name features (custom instructions, AGENTS.md) have specific filenames; no variation needed.

### Other Rules

- For Agent Skills, the `description` is the only text used for automatic activation. It must state capability and when to use it.
- For Prompt Files and Custom Agents, the `description` is UI metadata; make it specific and task-focused.
- For Custom Instructions and AGENTS.md, do not use YAML frontmatter.
- Include one concrete example per feature type.

For detailed guidance on phrasing rules, constraints, and instructions, reference the [writing-ai-instructions](../writing-ai-instructions/SKILL.md) skill.

### Validation Checklist

- Name is lowercase and hyphen-separated (skills, prompts, agents)
- File path matches feature type
- Examples are included
