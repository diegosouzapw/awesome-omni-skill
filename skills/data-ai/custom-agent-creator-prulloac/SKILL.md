---
name: custom-agent-creator
description: Skill to create custom agents for VS Code Copilot or OpenCode, helping users configure and generate agent files with proper formatting and configurations. Use when users want to create specialized AI assistants for VS Code Copilot (.agent.md files) or OpenCode (JSON/markdown agent configs) with specific tools, prompts, models, and behaviors. If the user is not specific about the target platform, ask them to specify Copilot or OpenCode.
---

# Custom Agent Creator

## Overview

This skill guides the creation of custom agents for either VS Code Copilot or OpenCode. It helps users define agent properties like name, description, tools, models, and permissions, then generates the appropriate configuration file in the correct format.

## Workflow

1. **Determine Target Platform**: If not specified, ask the user whether they want to create an agent for Copilot or OpenCode.

2. **Gather Agent Details**: Collect information about the agent:
   - Name
   - Description
   - Tools and permissions
   - Model (optional)
   - Additional configuration options

3. **Generate Configuration**: Create the agent file following the platform's specifications.

## Copilot Agents

Custom agents for VS Code Copilot are defined in `.agent.md` files with YAML frontmatter.

**File Structure**:
- Frontmatter: YAML with fields like `description`, `name`, `tools`, `model`, `handoffs`
- Body: Markdown instructions for the agent

**Common Fields**:
- `description`: Brief description shown in chat input
- `tools`: List of available tools
- `model`: AI model to use
- `handoffs`: Suggested next steps for workflow transitions

See [references/copilot-agents.md](references/copilot-agents.md) for complete documentation.

## OpenCode Agents

Agents for OpenCode can be defined in JSON config or Markdown files.

**Configuration Options**:
- `description`: Required description
- `mode`: "primary" or "subagent"
- `model`: Model identifier
- `tools`: Tool permissions (true/false)
- `permissions`: Granular control over actions
- `temperature`: Response creativity (0.0-1.0)

See [references/opencode-agents.md](references/opencode-agents.md) for complete documentation.

## Resources

### references/
- `copilot-agents.md`: Full documentation for Copilot custom agents
- `opencode-agents.md`: Full documentation for OpenCode agents

### assets/
- `copilot-template.agent.md`: Template for Copilot agent files
- `opencode-template.md`: Template for OpenCode agent files
