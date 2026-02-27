---
name: claude-agent-sdk
description: "Build production AI agents using the Claude Agent SDK (TypeScript/Python). Use this skill when the user asks about: (1) Building AI agents with Claude, (2) Using the @anthropic-ai/claude-agent-sdk or claude-agent-sdk packages, (3) Implementing agent features like tools, hooks, subagents, MCP integration, or sessions, (4) Migrating from Claude Code SDK to Agent SDK, (5) Configuring permissions, budgets, or custom tool access."
---

# Claude Agent SDK

Build AI agents that autonomously read files, run commands, search the web, edit code, and more. The Agent SDK provides the same tools, agent loop, and context management that power Claude Code, programmable in Python and TypeScript.

## Installation

### Prerequisites
Install Claude Code runtime first:
```bash
# macOS/Linux/WSL
curl -fsSL https://claude.ai/install.sh | bash

# or via npm
npm install -g @anthropic-ai/claude-code
```

### SDK Installation
```bash
# TypeScript
npm install @anthropic-ai/claude-agent-sdk

# Python
pip install claude-agent-sdk
```

### API Key
```bash
export ANTHROPIC_API_KEY=your-api-key
```

Alternative authentication:
- **Amazon Bedrock**: `CLAUDE_CODE_USE_BEDROCK=1`
- **Google Vertex AI**: `CLAUDE_CODE_USE_VERTEX=1`
- **Microsoft Foundry**: `CLAUDE_CODE_USE_FOUNDRY=1`

## Basic Usage

### TypeScript
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Find and fix the bug in auth.py",
  options: { allowedTools: ["Read", "Edit", "Bash"] }
})) {
  if ("result" in message) console.log(message.result);
}
```

### Python
```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"])
    ):
        if hasattr(message, "result"):
            print(message.result)

asyncio.run(main())
```

## Built-in Tools

| Tool | Description |
|------|-------------|
| **Read** | Read any file in the working directory |
| **Write** | Create new files |
| **Edit** | Make precise edits to existing files |
| **Bash** | Run terminal commands, scripts, git operations |
| **Glob** | Find files by pattern (`**/*.ts`, `src/**/*.py`) |
| **Grep** | Search file contents with regex |
| **WebSearch** | Search the web for current information |
| **WebFetch** | Fetch and parse web page content |
| **Task** | Spawn subagents for specialized tasks |

## Key Features

For detailed API reference and examples:
- **TypeScript**: See [references/typescript-api.md](references/typescript-api.md)
- **Python**: See [references/python-api.md](references/python-api.md)

### Permission Modes
- `default`: Standard permission checks
- `acceptEdits`: Automatically approve file edits
- `bypassPermissions`: Skip all permission checks (use with caution)

### Sessions
Resume conversations with full context:
```typescript
// Capture session ID from init message
if (message.type === 'system' && message.subtype === 'init') {
  sessionId = message.session_id;
}

// Resume later
for await (const message of query({
  prompt: "Continue where we left off",
  options: { resume: sessionId }
})) { ... }
```

### MCP Integration
Connect external tools via Model Context Protocol:
```typescript
const response = query({
  prompt: "Open example.com and describe what you see",
  options: {
    mcpServers: {
      playwright: { command: "npx", args: ["@playwright/mcp@latest"] }
    }
  }
});
```

### Subagents
Spawn specialized agents for focused tasks:
```typescript
const response = query({
  prompt: "Use the code-reviewer agent to review this codebase",
  options: {
    allowedTools: ["Read", "Glob", "Grep", "Task"],
    agents: {
      "code-reviewer": {
        description: "Expert code reviewer for quality and security reviews.",
        prompt: "Analyze code quality and suggest improvements.",
        tools: ["Read", "Glob", "Grep"]
      }
    }
  }
});
```

### Hooks
Run custom code at key points in the agent lifecycle:
- `PreToolUse`, `PostToolUse`
- `Stop`, `SessionStart`, `SessionEnd`
- `UserPromptSubmit`

### Budget Control
```typescript
const response = query({
  prompt: "Analyze the codebase",
  options: {
    maxBudgetUsd: 2.5  // Stop if cost exceeds $2.50
  }
});
```

### Project Skills
Load custom skills from `.claude/commands/`:
```typescript
const response = query({
  prompt: "Review the pull request using available skills",
  options: {
    settingSources: ["project"]  // Loads skills from .claude/commands/
  }
});
```

## Message Types

The SDK streams various message types:
- `system` (subtype `init`): Session initialization with `session_id`
- `assistant`: Claude's text responses and tool calls
- `user`: Tool results and user input
- `result`: Final response with execution stats
- `error`: Error information

## Available Models

- `claude-opus-4-5-20251101` - Most capable
- `claude-sonnet-4-20250514` - Balanced (default)
- `claude-sonnet-4-5-20250929` - Latest Sonnet
- `claude-haiku-4-5-20251001` - Fastest

## SDK vs Client SDK

The **Agent SDK** handles tool loops autonomously:
```typescript
// Agent SDK: Claude handles tools autonomously
for await (const message of query({ prompt: "Fix the bug in auth.py" })) {
  console.log(message);
}
```

The **Client SDK** requires you to implement tool execution:
```typescript
// Client SDK: You implement the tool loop
let response = await client.messages.create({...});
while (response.stop_reason === "tool_use") {
  const result = yourToolExecutor(response.tool_use);
  response = await client.messages.create({ tool_result: result, ... });
}
```

## Resources

- [TypeScript SDK GitHub](https://github.com/anthropics/claude-agent-sdk-typescript)
- [Python SDK GitHub](https://github.com/anthropics/claude-agent-sdk-python)
- [Example Agents](https://github.com/anthropics/claude-agent-sdk-demos)
