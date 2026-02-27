---
name: promptly-agent-builder
description: Create, update, or fix standalone Swift command line agents built with Swift Package Manager that use Promptly packages (PromptlyKit, PromptlyConsole, PromptlyKitTooling, PromptlySubAgents). Use when asked to scaffold a new Promptly-based agent, change its prompts or command line interface, add or adjust tools or sub agents, or diagnose build and run errors in such agents.
---

# Promptly Agent Builder

## Overview
Build and maintain focused Promptly-based command line agents quickly, using the provided template and Promptly module references.

## Create a new agent
1. Copy the template folder from `assets/standalone-agent-template` into a new project directory.
2. Rename the package and target from `PromptlyAgent` to the new agent name in `Package.swift`.
3. Rename the source folder `Sources/PromptlyAgent` to match the target name.
4. Update `PromptlyAgent.swift` and `PromptSupport.systemPrompt`:
   - Change `commandName`, `abstract`, and the system prompt to the agent purpose.
   - Add or remove command line options as needed.
5. Decide whether to keep tool support:
   - Keep `ToolFactory` and `SubAgentToolFactory` for shell tools and sub agents.
   - Remove them and return `[]` from the tool provider if the agent must run without tools.
6. Build the agent to verify it compiles in the sandboxed environment:
   - `mkdir -p .build/tmp .build/home && TMPDIR="$(pwd)/.build/tmp" HOME="$(pwd)/.build/home" swift build --disable-sandbox`
7. Run the agent with `--task` or `--interactive`.

## Update an existing agent
- Start by reading `Package.swift` and `Sources/<Target>/PromptlyAgent.swift` to understand dependencies and prompt flow.
- Update only the system prompt and command options needed for the new behavior; avoid changing configuration loading unless required.
- If adding tools, keep `toolsFileName`, `includeTools`, and `excludeTools` consistent with the tools configuration file.
- If adding sub agents, keep the supervisor hint insertion so the model sees available sub agents.
- Build the agent to verify it compiles in the sandboxed environment:
  - `mkdir -p .build/tmp .build/home && TMPDIR="$(pwd)/.build/tmp" HOME="$(pwd)/.build/home" swift build --disable-sandbox`

## General guidance
- Prefer a small set of cohesive types over one large `main.swift`, with one type per file when practical.
- Be careful with Swift `@main`: avoid top level code in the same module and avoid `main.swift` conflicts by choosing a clear entry file name.
- Minimize static helper sprawl by using small service or value types so responsibilities are explicit and testable.
- Avoid creating state that is never read and avoid unused closures; validate inputs but keep tools side effect free unless the user requested persistence.
- Provide sensible defaults so users can run without extra command line options; make interactive and verbosity optional toggles.
- Treat tool output and user interface output as separate concerns; add a verbose flag and keep quiet output clean by default.
- Watch `Sendable` and actor boundaries in asynchronous callbacks; avoid capturing `self` in `@Sendable` closures by extracting local values.
- Do not write to disk or alter the environment unless the user requested it; prefer in memory mechanisms.
- When scaffolding tools, include include and exclude filtering and keep tool names stable for prompt instructions.
- Run a build or test check when feasible to catch missing imports or type visibility issues early.

## Prompt guidance
Use these when you are writing or updating the agent system prompt and tool usage instructions.
- Provide a clear workflow with numbered steps and explicit fallback behavior so the agent can proceed when follow up questions are not possible.
- Define a strict output format with required sections and ordering to avoid response drift.
- Specify tool call ordering, including any required tool calls before the assistant outputs text.
- Require plain text tool inputs without labels or code fences to avoid contaminated inputs.
- Explicitly forbid side effects and environment changes unless the user asked for them.
- State when to ask questions versus when to proceed with safe assumptions.
- Describe how to handle tool failures, including which steps are critical and which can be skipped.

## Fix common issues
- Config file not found: verify the `--config-file` path and ensure it exists.
- Authentication failures: confirm the token is present in the environment or keychain for the chosen provider.
- No tool calls: confirm the tools configuration file is present and tools are not excluded by filters.
- Empty assistant output: ensure `PromptConsoleRunner` is used and the system prompt is not empty.

## Resources
- Read `references/promptly-agent-notes.md` for the Promptly types and source references.
- Use `assets/standalone-agent-template` as the base project for new agents.
