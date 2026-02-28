---
name: cursor-desktop-guide
description: Guide for using Cursor desktop features including AGENTS.md, Rules, Skills, Commands, Subagents, MCP, and Hooks. Use when the user asks about Cursor configuration, features, or how to set up the IDE environment.
---

# Cursor Desktop Guide

## Context Architecture

Understand the priority of context loading to effectively manage the agent's knowledge:

1. **Team Rules** (Highest priority)
2. **Project Rules** (`.cursor/rules/`)
3. **User Rules** (Cursor Settings)
4. **AGENTS.md** (Project root and subdirectories)
5. **Skills** (Automatically discovered)

## Core Components

### AGENTS.md

- **Purpose**: Global instructions and context for the project.
- **Best Practice**: Keep compact (<100 lines). Use nested `AGENTS.md` for subdirectories.
- **Location**: Project root (`AGENTS.md`) or subdirectories (e.g., `projects/foo/AGENTS.md`).

### Project Rules (`.cursor/rules/`)

- **Purpose**: Specific, versioned rules.
- **Format**: `.md` or `.mdc` (with YAML frontmatter).
- **Types**:
  - **Always Apply**: `alwaysApply: true`
  - **Apply Intelligently**: `alwaysApply: false` (Agent decides based on description)
  - **Apply to Specific Files**: `globs: ["pattern"]`
  - **Apply Manually**: `@rule-name` mention

### Skills (`.cursor/skills/`)

- **Purpose**: Portable, versioned packages for domain-specific workflows.
- **Structure**: Directory with `SKILL.md`, `scripts/`, `assets/`, `references/`.
- **Usage**: Multi-step workflows requiring resources or scripts.

### Commands (`.cursor/commands/`)

- **Purpose**: One-off actions triggered by `/command` in chat.
- **Format**: Simple Markdown files.

### Subagents

- **Purpose**: Isolated AI assistants for heavy tasks or parallel execution.
- **Usage**: Use `Task` tool.
- **When to use**: Context isolation needed, parallel work, or specialized long-running tasks.
- **Cost**: Each subagent has its own context window (token overhead).

### MCP Servers & Integrations

- **Local**: Use MCP for stateful connections (e.g., Slack) or CLI tools via `Shell` for request-response (e.g., Jira, GitLab).
- **Cloud Agents**: Not used in this system (require remote repo).

### Hooks (`.cursor/hooks.json`)

- **Purpose**: Scripts triggered on agent events (e.g., `preToolUse`, `afterFileEdit`).
- **Types**: Command-based (shell script) or Prompt-based (LLM evaluation).

## Best Practices

1. **AGENTS.md** is the entry point, not an encyclopedia.
2. Use **Rules** with globs for file formats (e.g., ADRs).
3. Use **Skills** for complex workflows (reviews, reports).
4. Use **Commands** for simple rituals (`/morning`).
5. Use **Subagents** only when context isolation is strictly necessary.
6. Use **`.cursorignore`** to exclude archives and cache from Semantic Search.
7. Use **Max Mode** (1M tokens) for heavy workflows.

## References

- Full documentation: [references/REFERENCE.md](references/REFERENCE.md)
- Online up-to-date documentation: [https://www.cursor.com/docs](https://www.cursor.com/docs)
