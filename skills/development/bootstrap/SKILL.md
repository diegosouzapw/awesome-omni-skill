---
name: bootstrap
description: Smart repo bootstrapping for Agentic flow (AGENTS.md)
---

Please analyze this codebase and create an `AGENTS.md` file, which will be given to future instances of this AI coding agent (like Claude Code, Codex or Gemini Cli) a simple set of rules to operate in this project.

What to add:

1. Commands that will be commonly used, such as how to build, lint, and run tests. Include the necessary commands to develop in this codebase, such as how to run a single test.
2. High-level code architecture and structure so that future instances can be productive more quickly. Focus on the "big picture" architecture that requires reading multiple files to understand

Usage notes:

- If there's already an `AGENTS.md`, suggest improvements to it vs creating a new file.
- When you make the initial `AGENTS.md` do not repeat yourself and do not include obvious instructions like "Provide helpful error messages to users", "Write unit tests for all new utilities", "Never include sensitive information (API keys, tokens) in code or commits"
- Avoid listing every component or file structure that can be easily discovered
- Don't include generic development practices
- If there are Cursor rules (in .cursor/rules/ or .cursorrules), AGENTS.md, GEMINI.md or Copilot rules (in .github/copilot-instructions.md), make sure to include the important parts.
- If there is a README.md, PROJECT.md, make sure to include the important parts.
- Do not make up information such as "Common Development Tasks", "Tips for Development", "Support and Documentation" unless this is expressly included in other files that you read.
- Be sure to prefix the file with the following text:

```
This file provides guidance to AI coding agents like Claude Code (claude.ai/code), Cursor AI, Codex, Gemini CLI, GitHub Copilot, and other AI coding assistants when working with code in this repository.
```

Best practices:

- Good rules are focused, actionable, and scoped.
- Keep rules under 500 lines
- Avoid vague guidance. Write rules like clear internal docs
- Reuse rules when repeating prompts in chat
