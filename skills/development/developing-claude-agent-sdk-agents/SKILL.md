---
name: developing-claude-agent-sdk-agents
description: "Build AI agents with the Claude Agent SDK (TypeScript/Python). Covers creating agents, custom tools, hooks, subagents, MCP integration, permissions, sessions, and deployment. Use when building, reviewing, debugging, or deploying SDK-based agents. Invoke PROACTIVELY when user mentions Agent SDK, claude-agent-sdk, ClaudeSDKClient, query(), or building autonomous agents."
---

<objective>
Build production-ready AI agents using the Claude Agent SDK. This skill provides comprehensive guidance for the full agent development lifecycle: creating agents, adding custom tools, implementing hooks, configuring permissions, managing sessions, and deploying to production.
</objective>

<essential_principles>
**Core SDK Concepts**

The Claude Agent SDK enables building autonomous AI agents that can:
- Execute multi-turn conversations with tool use
- Read, write, and edit files in a working directory
- Run shell commands
- Integrate with external services via MCP
- Spawn specialized subagents

**1. Streaming is Primary**

The SDK operates as a **streaming API**. You iterate over messages as they're generated:

```typescript
// TypeScript
for await (const message of query({ prompt: "...", options })) {
  // Handle each message type
}
```

```python
# Python
async for message in query(prompt="...", options=options):
    # Handle each message type
```

**2. System Prompt is Empty by Default**

The SDK uses an **empty system prompt** by default. To get Claude Code's full capabilities:

```typescript
systemPrompt: { type: "preset", preset: "claude_code" }
```

**3. Settings Sources Must Be Explicit**

CLAUDE.md, skills, and slash commands are NOT loaded by default. You must specify:

```typescript
settingSources: ["user", "project"]  // TypeScript
setting_sources=["user", "project"]  # Python
```

**4. Permission Modes Control Tool Access**

Choose permission mode based on your use case:
- `default` - Interactive approval
- `acceptEdits` - Auto-approve file changes
- `bypassPermissions` - Skip all permission checks (use with care)

**5. MCP Tools Require Streaming Input**

Custom MCP tools require streaming input mode (async generator), not simple strings.

</essential_principles>

<quick_start>
**Get Started in 60 Seconds**

**Install:**
```bash
npm install @anthropic-ai/claude-agent-sdk  # TypeScript
pip install claude-agent-sdk                 # Python
```

**Basic agent:**
```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const msg of query({
  prompt: "Read package.json and summarize the dependencies",
  options: {
    systemPrompt: { type: "preset", preset: "claude_code" },
    allowedTools: ["Read", "Grep", "Glob"],
    maxTurns: 5
  }
})) {
  if (msg.type === "result") console.log(msg.result);
}
```

**For detailed guidance, select a workflow below.**
</quick_start>

<intake>
**What would you like to do?**

1. Build a new agent from scratch
2. Add custom tools (MCP servers)
3. Implement hooks for behavior modification
4. Configure permissions and security
5. Deploy an agent to production
6. Debug an agent issue
7. Something else

**Then read the matching workflow from `workflows/` and follow it.**
</intake>

<routing>
| Response | Workflow |
|----------|----------|
| 1, "new", "create", "build", "start" | `workflows/build-new-agent.md` |
| 2, "tool", "tools", "mcp", "custom" | `workflows/add-custom-tools.md` |
| 3, "hook", "hooks", "lifecycle", "callback" | `workflows/implement-hooks.md` |
| 4, "permission", "security", "sandbox" | `workflows/configure-permissions.md` |
| 5, "deploy", "host", "production", "ship" | `workflows/deploy-agent.md` |
| 6, "debug", "fix", "error", "broken" | Read `references/debugging.md`, then diagnose |
| 7, other | Clarify, then select workflow or references |

**After reading the workflow, follow it exactly.**
</routing>

<verification_loop>
**After Every Change**

```bash
# TypeScript
npx tsc --noEmit  # Type check
npm test          # Run tests

# Python
python -m mypy .  # Type check
pytest            # Run tests
```

Test the agent interactively:
```bash
npx tsx my-agent.ts    # TypeScript
python my_agent.py     # Python
```

Report to the user:
- "Type check: OK/X errors"
- "Tests: X pass, Y fail"
- "Agent runs and responds correctly"
</verification_loop>

<reference_index>
**Domain Knowledge**

All in `references/`:

**Core:**
- sdk-overview.md - Architecture, installation, message types, API comparison
- built-in-tools.md - Read, Write, Edit, Bash, Glob, Grep, MCP tools

**Features:**
- hooks.md - Lifecycle hooks, callbacks, patterns
- subagents.md - Creating and spawning subagents
- mcp-and-custom-tools.md - MCP servers, custom tool definition
- permissions.md - Permission modes, tool restrictions, canUseTool
- sessions.md - Resuming, forking, context management

**Advanced:**
- patterns-and-features.md - Streaming modes, structured outputs, checkpointing
- filesystem-features.md - Skills, slash commands, CLAUDE.md, plugins
- hosting-and-deployment.md - Docker, sandboxing, production patterns
- debugging.md - Common issues and solutions
</reference_index>

<workflows_index>
**Workflows**

All in `workflows/`:

| File | Purpose |
|------|---------|
| build-new-agent.md | Create new agent from scratch |
| add-custom-tools.md | Add MCP tools to extend capabilities |
| implement-hooks.md | Add lifecycle hooks |
| configure-permissions.md | Set up security and permissions |
| deploy-agent.md | Deploy to production |
</workflows_index>

<success_criteria>
A well-built SDK agent:
- Uses streaming API correctly
- Handles all message types appropriately
- Has proper error handling
- Uses appropriate permission mode
- Includes timeout and maxTurns limits
- Has been tested with real queries
- Is type-safe (TypeScript) or type-hinted (Python)
</success_criteria>
