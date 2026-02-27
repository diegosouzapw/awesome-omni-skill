---
name: agent-creator
description: This skill should be used when the user asks to "create a new agent", "make a subagent", "build an agent for X", "generate agent configuration", or wants to create a new Claude Code sub-agent. Triggers on "agent-creator", "new agent", "subagent", or agent creation requests.
tools: AskUserQuestion, Read, Write, Edit, WebFetch, Glob
---

# Agent Creator Skill

## Purpose

This skill acts as an expert agent architect, generating complete, ready-to-use Claude Code sub-agent configuration files from user descriptions. It gathers requirements through structured questions, fetches current documentation, and produces well-structured agent definitions.

## Workflow

### Step 1: Gather Requirements

Before creating any agent, use `AskUserQuestion` to collect essential information:

```
Question 1: "What is the primary purpose or goal of this agent?"
Header: "Agent Goal"
Options:
- Code analysis/review
- Code generation/writing
- Documentation tasks
- Testing/debugging
- Other (free text)

Question 2: "What domain should this agent be an expert in?"
Header: "Expertise"
Options:
- Frontend development
- Backend development
- DevOps/Infrastructure
- Data/Analytics
- Other (free text)

Question 3: "What tools should this agent have access to?"
Header: "Tools"
MultiSelect: true
Options:
- Read, Grep, Glob (code exploration)
- Edit, Write (code modification)
- Bash (command execution)
- WebFetch (documentation/research)
```

### Step 2: Fetch Documentation

Retrieve up-to-date sub-agent documentation:

```
WebFetch: https://code.claude.com/docs/en/sub-agents
Prompt: "Extract the sub-agent configuration format, available frontmatter fields, and best practices for creating agents"
```

### Step 3: Analyze and Design

Based on gathered requirements:

1. **Devise a name**: Create a concise, descriptive `kebab-case` name (e.g., `api-tester`, `dependency-manager`)
2. **Select a color**: Choose from: red, blue, green, yellow, purple, orange, pink, cyan
3. **Write delegation description**: Craft an action-oriented description stating *when* to use the agent. Include trigger phrases for when users should invoke it. Example: "Use proactively when user says 'review my code' or 'check for bugs'. Specialist for code quality assessment."
4. **Fetch tools available**: Use WebFetch to get the current list of available tools from Claude Code documentation, as tools may change over time
5. **Determine tools needed**: Based on the agent's purpose and the available tools, select the minimal tool set required for the agent's tasks

### Step 4: Construct the Agent File

Generate a complete markdown file following the template in `references/agent-template.md`.

### Step 5: Write the Agent File

Save the generated agent to: `~/.claude/agents/<generated-agent-name>.md`

Use the Write tool to create the file at the correct location.

## Best Practices

- Keep descriptions action-oriented and specific about when the agent should be used
- Include "Use proactively when..." in descriptions for automatic delegation
- Minimize tool access to only what's necessary for the agent's purpose
- Write clear, numbered instructions that guide the agent step-by-step
- Include domain-specific best practices relevant to the agent's expertise
- Define expected output format clearly

## Additional Resources

### Reference Files

- **`references/agent-template.md`** - Complete agent file template with all sections
