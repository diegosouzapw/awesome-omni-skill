---
name: creating-agents
description: 'Create and review agent definition files (agents.md) that give AI coding agents a clear persona, project knowledge, executable commands, code style examples, and explicit boundaries. Use when a user asks to create an agent, define an agent persona, write an agents.md file, set up a custom Copilot agent, review an existing agent definition, or improve agent quality. Covers the six core areas: commands, testing, project structure, code style, git workflow, and boundaries.'
license: MIT
---

# Creating Agents

## Overview

This skill provides capabilities for creating and reviewing agent definition files (`agents.md` / `agent.md`) that transform a general-purpose AI assistant into a focused specialist. Agent definitions give an AI coding agent a specific persona, project knowledge, executable commands, and explicit boundaries.

A well-written agent definition follows the principle: **specific beats vague**. "You are a helpful coding assistant" fails. "You are a test engineer who writes tests for React components, follows these examples, and never modifies source code" succeeds.

This skill is informed by analysis of over 2,500 `agents.md` files across public repositories ([source](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)).

## Capabilities

| Capability | Action | Description |
|------------|--------|-------------|
| Create | `actions/create.md` | Generate a new agent definition with persona, commands, and boundaries |
| Review | `actions/review.md` | Analyse an existing agent definition for quality and completeness |

## Standards

This skill bundles the following standards in `standards/`:

| Standard | File | Description |
|----------|------|-------------|
| Agent Structure | `agent-structure.md` | Required sections and organisation for agent definitions |
| Persona | `persona.md` | Writing effective agent personas and role definitions |
| Boundaries | `boundaries.md` | Defining always-do, ask-first, and never-do rules |
| Commands and Tools | `commands-and-tools.md` | Documenting executable commands agents can run |
| Code Style | `code-style.md` | Providing code examples and style guidance to agents |
| Checklist | `checklist.md` | Consolidated compliance and quality checklist |

## Principles

### 1. Specific Beats Vague

Every successful agent definition gives the agent a clear, narrow job. State the exact role, tech stack with versions, file paths, and commands. Ambiguity leads to unpredictable behavior.

### 2. Show, Don't Tell

One real code snippet showing your preferred style beats three paragraphs describing it. Provide concrete examples of good output — naming conventions, error handling patterns, test structures.

### 3. Commands Early, Boundaries Clear

Put executable commands (`npm test`, `pytest -v`, `cargo build`) near the top of the agent definition. Agents reference these often. Define boundaries using a three-tier system: ✅ Always do, ⚠️ Ask first, 🚫 Never do.

### 4. Cover the Six Core Areas

The best agent definitions address six areas:

1. **Commands** — Executable commands with flags and options
2. **Testing** — Test framework, commands, and coverage expectations
3. **Project structure** — File layout and what lives where
4. **Code style** — Naming, patterns, and concrete examples
5. **Git workflow** — Branch naming, commit messages, PR process
6. **Boundaries** — What the agent must never touch

### 5. Start Small, Iterate

Begin with a minimal agent definition for one specific task. Test it with real work. Add detail when the agent makes mistakes. The best agent definitions grow through iteration, not upfront planning.

## Usage

1. Load this skill manifest
2. Identify the required capability (create or review)
3. Load the bundled standards from `standards/`
4. Execute the action following `actions/<capability>.md`

## References

- [How to write a great agents.md](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) — Lessons from over 2,500 repositories
- [GitHub Copilot Custom Agents](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot) — Official documentation
- [Agent Skills Specification](https://agentskills.io/specification) — Open format for agent capabilities
