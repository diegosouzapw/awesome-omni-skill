---
name: claude-agent-sdk-builder
description: Guide for building agents with the Claude Agent SDK (TypeScript/Node.js). Use when creating SDK-based agents, custom tools, in-code subagents, or production agent applications. Provides templates, patterns, and best practices for agent development.
---

# Claude Agent SDK Builder

Create powerful agents using the Claude Agent SDK with custom tools, programmatic subagents, and production-ready patterns.

## Overview

This skill provides comprehensive guidance for building agents with the Claude Agent SDK (TypeScript/Node.js). Use this skill when:

- Creating new SDK-based agents
- Adding custom tools to extend agent capabilities
- Configuring subagents programmatically (preferred over markdown)
- Building production agent applications with WebSocket streaming
- Implementing hooks for safety and permissions
- Managing multi-turn conversations and sessions

## Prerequisites

To use this skill and build Claude Agent SDK applications, ensure the following are installed:

**Required:**
- **Node.js 18+** or **Bun runtime** - For running TypeScript/JavaScript code
- **npm** or **bun** - Package manager for installing dependencies
- **Git** - For version control and using init-agent-project.sh script

**Optional but Recommended:**
- **TypeScript** - For type checking and better development experience
- **Anthropic API Key** - Required to run agents (get from [console.anthropic.com](https://console.anthropic.com))

**Verify Installation:**
```bash
node --version  # Should be 18.0.0 or higher
npm --version
git --version
```

Or with Bun:
```bash
bun --version
```

## Helper Scripts

This skill includes powerful scripts to automate common tasks:

### init-agent-project.sh - Project Scaffolding

Quickly create a new agent project from templates:

```bash
./scripts/init-agent-project.sh
```

The script will:
- Prompt for project name
- Let you choose a template (basic-agent, custom-tools, in-code-subagent, full-app)
- Copy template files
- Create `.env` and `.gitignore`
- Initialize git repository
- Install dependencies
- Create workspace directory

### generate-tool.ts - Custom Tool Generator

Generate boilerplate for new custom tools:

```bash
bun run scripts/generate-tool.ts
```

Interactive prompts for:
- Tool name and description
- Parameters with types and descriptions
- Generates complete TypeScript code with Zod schemas

### generate-subagent.ts - Subagent Config Generator

Generate in-code subagent configurations:

```bash
bun run scripts/generate-subagent.ts
```

Creates:
- Factory function or class-based subagent
- Proper TypeScript types
- System prompt template
- Tool selection
- Usage examples

### validate-agent-config.ts - Configuration Validator

Validate agent configuration before running:

```bash
bun run scripts/validate-agent-config.ts config.ts
```

Checks for:
- Tool name typos and case sensitivity
- Missing MCP servers
- Invalid tool references
- Subagent configuration errors
- Common mistakes

### add-hooks.ts - Hooks Generator

Generate common hook patterns:

```bash
bun run scripts/add-hooks.ts
```

Available templates:
- File path validation
- Command allowlist
- File type restrictions
- Rate limiting
- Logging hooks
- Environment-based restrictions

## Quick Start Workflow

### Step 1: Choose Your Starting Point

Select the appropriate template based on your needs:

1. **basic-agent** - Simple one-shot agent with basic message handling
2. **custom-tools** - Agent with custom MCP tools
3. **in-code-subagent** - Agent with programmatic subagent configuration
4. **full-app** - Production-ready app with WebSocket, sessions, and persistence

Access templates in `assets/templates/`.

### Step 2: Core SDK Concepts

Before building, understand these core concepts:

**Query Function**
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({ prompt, options })) {
  // Handle messages
}
```

**Message Types**
- `system` - Init, context updates
- `user` - User messages
- `assistant` - AI responses and tool calls
- `result` - Final success/failure with cost

**Multi-Turn Conversations**
- Capture `session_id` from system init message
- Use `resume: sessionId` option for subsequent turns

### Step 3: Implementation Patterns

#### Pattern A: Basic Agent

Copy from `assets/templates/basic-agent/`:

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Your task",
  options: {
    maxTurns: 20,
    model: "sonnet",
    allowedTools: ["Read", "Write", "Edit"]
  }
})) {
  if (message.type === 'assistant') {
    console.log(message.message.content);
  }
}
```

#### Pattern B: Agent with Custom Tools

Copy from `assets/templates/custom-tools/`:

```typescript
import { tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const myServer = createSdkMcpServer({
  name: "mytools",
  version: "1.0.0",
  tools: [
    tool("tool_name", "Description", {
      param: z.string().describe("Parameter description")
    }, async (args) => ({
      content: [{ type: "text", text: "Result" }]
    }))
  ]
});

// Use in query
options: {
  mcpServers: { "mytools": myServer },
  allowedTools: ["mcp__mytools__tool_name"]
}
```

#### Pattern C: Agent with In-Code Subagents (Preferred)

Copy from `assets/templates/in-code-subagent/`:

```typescript
// Define subagents programmatically
function createSearchSubagent() {
  return {
    name: "searcher",
    description: "Finds and analyzes files",
    tools: ["Read", "Grep", "Glob"],
    systemPrompt: "Search specialist instructions...",
    maxTurns: 10
  };
}

// Register with main agent
options: {
  subagents: [createSearchSubagent()],
  allowedTools: ["Task"]  // Allow spawning subagents
}
```

**Why in-code subagents?**
- Type safety - catch errors at compile time
- Dynamic configuration - adjust at runtime
- Better testing - unit test configurations
- Code reuse - share configuration logic

**When to use markdown subagents?**
- Only when user explicitly requests markdown-based configuration
- For non-technical users who need to modify agents

#### Pattern D: Production App

Copy from `assets/templates/full-app/`:

Complete application with:
- WebSocket server for real-time streaming
- Session management for multi-turn conversations
- Database persistence
- Error handling and cleanup
- Built-in web UI

See `assets/templates/full-app/README.md` for details.

## Creating Custom Tools

Custom tools extend agent capabilities with domain-specific functionality. Use `tool()` with Zod schemas to define parameters, then bundle into MCP servers with `createSdkMcpServer()`.

**Key Points:**
- Tool naming format: `mcp__<server-name>__<tool-name>`
- Define parameters with Zod for type safety
- Return format: `{ content: [{ type: "text", text: "..." }] }`

**Quick Example:**
```typescript
const myTool = tool("tool_name", "Description", {
  param: z.string()
}, async (args) => ({
  content: [{ type: "text", text: "Result" }]
}));
```

**Generator:** Use `bun run scripts/generate-tool.ts` to create tool boilerplate.

**Full Guide:** See `references/custom-tools-guide.md` for:
- Complete tool creation examples
- MCP server configuration
- Real-world patterns (email, database, API integration)
- Error handling and testing

## Configuring In-Code Subagents

**Preferred Approach:** Define subagents programmatically in code rather than markdown files for better type safety, testing, and dynamic configuration.

**Key Patterns:**
- **Factory functions** - Reusable config generators
- **Class-based** - For complex subagents with state
- **Dynamic** - Adjust behavior based on runtime conditions

**Spawning:** Main agent uses `Task` tool with `subagent_type` parameter.

**Benefits over Markdown:**
- Type safety (catch errors at compile time)
- Dynamic configuration (adjust at runtime)
- Better testing and code reuse

**Generator:** Use `bun run scripts/generate-subagent.ts` for boilerplate.

**Full Guide:** See `references/in-code-subagents.md` for:
- Factory and class-based patterns
- Spawning and orchestration
- Multi-level subagents
- Context passing and error handling

## Implementing Safety Hooks

Hooks intercept tool execution for validation, permissions, and safety. Use `PreToolUse` hooks to block unsafe operations before they execute.

**Common Use Cases:**
- File path validation (restrict to specific directories)
- Command allowlisting (only safe bash commands)
- Environment-based restrictions (dev vs prod)
- Rate limiting and logging

**Hook Return:**
- Allow: `{ continue: true }`
- Block: `{ decision: 'block', stopReason: '...', continue: false }`

**Generator:** Use `bun run scripts/add-hooks.ts` for common patterns.

**Full Guide:** See `references/hooks-guide.md` for:
- Complete hook examples
- Real-world patterns (workspace safety, file types, rate limiting)
- Testing hooks
- PostToolUse hooks

## Managing Sessions

Multi-turn conversations require capturing and resuming session state. Capture `session_id` from system init messages and use `resume` option for subsequent turns.

**Basic Pattern:**
1. Capture `session_id` from first `system` message
2. Use `resume: sessionId` option for follow-up queries
3. Session persists conversation context

**Production Patterns:**
- Session classes for encapsulation
- WebSocket streaming for real-time updates
- Database persistence for session data
- Cleanup strategies for inactive sessions

**Full Guide:** See `references/session-patterns.md` for:
- Multi-turn conversation patterns
- Streaming to WebSocket, console, or React
- SessionManager class for production
- Error handling and retry patterns

## Reference Documentation

Detailed guides available in `references/`:

- **sdk-api-reference.md** - Complete API reference for query(), options, message types
  - Search: `grep "query()" references/sdk-api-reference.md` for query function details
  - Search: `grep "tool()" references/sdk-api-reference.md` for custom tool creation
  - Search: `grep "ClaudeAgentOptions" references/sdk-api-reference.md` for configuration options
  - Search: `grep "Multi-Turn" references/sdk-api-reference.md` for session management

- **custom-tools-guide.md** - Creating tools, MCP servers, tool patterns
  - Search: `grep "createSdkMcpServer" references/custom-tools-guide.md` for MCP server creation
  - Search: `grep "Zod" references/custom-tools-guide.md` for parameter schema examples
  - Search: `grep "Real-World Examples" references/custom-tools-guide.md` for complete tool examples

- **in-code-subagents.md** - Programmatic subagent configuration (preferred approach)
  - Search: `grep "Factory" references/in-code-subagents.md` for factory pattern examples
  - Search: `grep "Class-Based" references/in-code-subagents.md` for class-based patterns
  - Search: `grep "Spawning" references/in-code-subagents.md` for Task tool delegation

- **hooks-guide.md** - PreToolUse and PostToolUse hooks for safety
  - Search: `grep "PreToolUse" references/hooks-guide.md` for pre-execution hooks
  - Search: `grep "File Path" references/hooks-guide.md` for path validation examples
  - Search: `grep "Command Allowlist" references/hooks-guide.md` for bash restrictions

- **session-patterns.md** - Session management, streaming, multi-turn, production patterns
  - Search: `grep "Multi-Turn" references/session-patterns.md` for conversation patterns
  - Search: `grep "WebSocket" references/session-patterns.md` for streaming examples
  - Search: `grep "SessionManager" references/session-patterns.md` for production patterns

## Template Usage

### Quick Start (Recommended)

Use the automated project scaffolder:

```bash
./scripts/init-agent-project.sh
```

This interactive script will guide you through creating a new project from any template.

### Manual Template Copy

Alternatively, copy templates manually:

#### Basic Agent

```bash
cp -r assets/templates/basic-agent ./my-agent
cd my-agent
npm install
npm start
```

#### Custom Tools

```bash
cp -r assets/templates/custom-tools ./my-agent
cd my-agent
npm install
npm start "Calculate 5 + 3 times 2"
```

#### In-Code Subagents

```bash
cp -r assets/templates/in-code-subagent ./my-agent
cd my-agent
npm install
npm start
```

#### Full Production App

```bash
cp -r assets/templates/full-app ./my-agent
cd my-agent
npm install
npm start
# Open browser to http://localhost:3000
```

## Best Practices

1. **Start simple** - Begin with basic-agent, add complexity as needed
2. **Use in-code subagents** - Prefer programmatic configuration over markdown
3. **Validate with hooks** - Add PreToolUse hooks for safety
4. **Capture session IDs** - Always capture for multi-turn conversations
5. **Handle all message types** - Process system, user, assistant, and result messages
6. **Set appropriate maxTurns** - Prevent runaway execution
7. **Use specific tool allowlists** - Only enable tools the agent needs
8. **Test thoroughly** - Unit test tools, hooks, and configurations
9. **Log extensively** - Log tool calls, sessions, errors for debugging
10. **Monitor costs** - Track total_cost_usd from result messages

## Common Scenarios

### Building a Code Assistant

1. Start with `custom-tools` template
2. Create tools for:
   - Reading codebases
   - Running tests
   - Generating code
3. Add in-code subagents for:
   - Code search
   - Code analysis
   - Documentation generation
4. Add hooks to restrict file operations

### Building a Data Assistant

1. Start with `custom-tools` template
2. Create tools for:
   - Database queries
   - API calls
   - Data processing
3. Add in-code subagents for:
   - Data retrieval
   - Data analysis
   - Report generation

### Building a Production App

1. Start with `full-app` template
2. Customize AIClient with your tools
3. Add authentication to WebSocket
4. Configure database for persistence
5. Add monitoring and logging
6. Deploy with proper security

## Troubleshooting

**Agent not using tools**
- Check tool is in `allowedTools` array
- Verify MCP server is in `mcpServers` object
- Check tool naming: `mcp__<server>__<tool>`

**Multi-turn not working**
- Verify session ID is captured from init message
- Use `resume: sessionId` in options
- Don't recreate session between messages

**Hooks not firing**
- Check matcher regex pattern matches tool name
- Ensure hooks are in correct format
- Verify hook returns proper HookOutput

**WebSocket disconnecting**
- Check for errors in message broadcasting
- Verify session cleanup logic
- Add error handling to WebSocket handlers

## Next Steps

1. Review templates in `assets/templates/`
2. Read detailed guides in `references/`
3. Start with basic-agent and iterate
4. Add custom tools for your use case
5. Configure in-code subagents for specialization
6. Implement hooks for safety
7. Build production app with full-app template

For specific implementation details, consult the reference documentation.
