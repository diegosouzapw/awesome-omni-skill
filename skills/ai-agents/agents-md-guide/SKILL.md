---
name: agents-md-guide
description: Guide for using and supporting the AGENTS.md standard in VS Code. Use this when asked about AGENTS.md, custom instructions, or repo-level AI agent configuration.
---

# AGENTS.md Guide for VS Code

This skill provides guidance on implementing and using the `AGENTS.md` standard to provide custom instructions for AI coding agents in VS Code.

## Why AGENTS.md?

- **Standardization**: Reduces fragmentation from proprietary files like `.cursorrules`.
- **Interoperability**: Works across different AI tools (GitHub Copilot, Cursor, etc.).
- **Efficiency**: Saves time by providing structured context (build steps, coding conventions).
- **Consistency**: Ensures AI agents follow project-specific protocols.
- **Open Standard**: Governed by the Agentic AI Foundation (Linux Foundation).

## Mental Model

`AGENTS.md` acts as a centralized **instruction manual** for AI coding agents at the repo root.
- **Flow**: Repo clone → agent scans for `AGENTS.md` → parses sections → applies rules during tasks → outputs aligned code.

## VS Code Configuration

To enable `AGENTS.md` support in VS Code:

1.  **Enable Setting**: Set `chat.useAgentsMdFile` to `true`.
2.  **Nested Files (Experimental)**: Set `chat.useNestedAgentsMdFiles` to `true` for subfolder instructions.

## How to Implement AGENTS.md

1.  **Location**: Place `AGENTS.md` at the root of your repository.
2.  **Structure**: Use clear sections:
    -   `## Environment`: Setup and build instructions.
    -   `## Coding Style`: Linting, formatting, and architectural rules.
    -   `## Testing`: How to run and write tests.
3.  **Keep it Concise**: Avoid overly verbose rules.

## Real-World Scenarios

- **Open-source Maintenance**: AI agents auto-generate PRs following style guides.
- **Enterprise Code Reviews**: Teams use repo-level rules during Copilot-assisted edits.
- **Indie Dev Prototyping**: Automate build and test cycles with tools like Cursor or Codex.

## Survival Kit

- **Day 0**: Clone a repo with `AGENTS.md`; ensure `chat.useAgentsMdFile` is enabled in VS Code.
- **Week 1**: Create a basic `AGENTS.md` in a test repo and iterate on sections.
- **Week 2**: Add nested files if needed using experimental settings.

## Security & Risks

- **No Secrets**: Never include API keys or credentials.
- **Goal Hijacking**: Be aware that instruction files can steer agent behavior. Review instructions before running autonomous tasks in untrusted repos.

## References

- [AGENTS.md Homepage](https://agents.md/)
- [VS Code Custom Instructions Docs](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [GitHub Copilot AGENTS.md Support](https://github.blog/changelog/2025-08-28-copilot-coding-agent-now-supports-agents-md-custom-instructions/)
- [Prompt Security: Risks of AGENTS.md](https://prompt.security/blog/when-your-repo-starts-talking-agents-md-and-agent-goal-hijack-in-vs-code-chat)
