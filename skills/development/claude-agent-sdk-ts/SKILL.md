---
name: claude-agent-sdk-ts
description: Build AI agents with Claude Agent SDK in TypeScript. Covers V1 query() API for batch workflows and V2 session API for interactive apps. Includes tool configuration, hooks, MCP servers, and multi-turn patterns.
---

# TypeScript Claude Agent SDK Builder

Build AI agents using the Claude Agent SDK in TypeScript by referencing code snippets that demonstrate both V1 (`query()`) and V2 (session-based) API patterns.

## How It Works

1. **Choose your API**: V1 for scripts/batch processing, V2 for interactive apps
2. **Browse** the snippet catalog below or in `snippets/`
3. **Copy** the snippets you need into your project
4. **Customize** the copied code for your use case

Snippets are bundled in this skill's `snippets/` directory.

---

## Authentication

The Claude Agent SDK **automatically uses Claude Code's authentication** - no API key configuration required if you're logged into Claude Code.

**How it works:**
1. The SDK looks for OAuth tokens in `~/.claude/` (created when you run `claude login`)
2. Falls back to `ANTHROPIC_API_KEY` environment variable if set
3. No code changes needed - authentication is handled automatically

**Check your auth status:**
```bash
claude auth status
```

**If not authenticated:**
```bash
claude login
```

### Alternative Providers (Bedrock, Vertex AI, Azure)

The SDK also supports cloud provider APIs. Set environment variables:

**Amazon Bedrock:**
```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-west-2
export AWS_PROFILE=your-profile  # Optional: specify named credentials profile
# Or uses default AWS credentials (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, or IAM role)
```

**Google Vertex AI:**
```bash
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=us-east5
export ANTHROPIC_VERTEX_PROJECT_ID=your-project-id
# Uses your Google Cloud credentials
```

**Azure:**
```bash
export CLAUDE_CODE_USE_AZURE=1
export AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
# Uses your Azure credentials
```

> **Note:** If deploying to a server without Claude Code, set `ANTHROPIC_API_KEY` in your environment (or use Bedrock/Vertex/Azure as above).

---

## When to Use Claude Agent SDK

**Use Claude Agent SDK when you want:**
- Quick setup with minimal boilerplate
- Out-of-the-box Claude Code capabilities
- Pre-built tool ecosystem with MCP server integration
- Session management and multi-turn conversations
- Hook system for tool auditing and control

**Do NOT use Claude Agent SDK when you need:**
- Control over temperature, top_p, top_k, or other advanced sampling parameters
- Direct control over the underlying API request/response
- Non-Claude models

---

## Quick Start Decision Trees

### Which API Should I Use?

```
Building a script or batch processor?
  └─> V1 query() API
      - Single-shot execution
      - Simpler async iteration
      - Good for: CLI tools, automation scripts, one-off tasks

Building an interactive application?
  └─> V2 session API
      - Multi-turn conversations with context
      - Session persistence/resume
      - Good for: Chat apps, web services, persistent assistants

Need quick one-shot queries?
  └─> V2 unstable_v2_prompt()
      - Convenience function, returns result directly
      - No message iteration needed
```

### How Should I Configure Tools?

```
Want full Claude Code capabilities?
  └─> Use tools: { type: 'preset', preset: 'claude_code' }

Need specific tool subset?
  └─> Use allowedTools: ['Bash', 'Read', 'Write', ...]

Need custom tools?
  └─> Use createSdkMcpServer() with tool() helper
      Then: mcpServers: { 'my-tools': customServer }
```

### Do I Need Hooks?

```
Need to audit/log tool usage?
  └─> Use PostToolUse hooks

Need to block dangerous operations?
  └─> Use PreToolUse hooks with decision: 'block'

Need to validate tool inputs?
  └─> Use PreToolUse hooks for validation
```

### Building a Chat UI?

```
Need to persist chat history?
  ├─> Want a database → persistence-sqlite
  └─> Want simple files → persistence-jsonl

Need user approval for tool calls?
  └─> Use ui-tool-approval (PreToolUse hook with WebSocket)

Want to show tool call activity?
  └─> Use ui-tool-history (extract events from stream)

Need everything integrated?
  └─> Use ui-websocket-chat (complete example)
```

---

## Phase 1: Research

### 1.1 Understand Your Use Case

Before writing code, understand:
- Is this a one-shot script or multi-turn application?
- What tools does the agent need access to?
- Do you need to control/audit tool usage?
- Does the agent need to persist conversation state?

### 1.2 Browse Available Snippets

Review the snippet catalog below to identify patterns that match your needs:

| Snippet | Description | Best For |
|---------|-------------|----------|
| `v1-basic-query` | Simple query() execution | Scripts, automation |
| `v1-query-tools` | Tool allowlist/blocklist | Controlled tool access |
| `v1-query-system` | Custom system prompt | Agent personality/behavior |
| `v1-query-queue` | AsyncIterable for multi-turn | V1 chat applications |
| `v2-basic-session` | Session with send/stream | Interactive apps |
| `v2-multi-turn` | Multi-turn with context | Chat applications |
| `v2-one-shot` | Quick query, direct result | Simple questions |
| `v2-session-resume` | Persist/resume sessions | Long-running assistants |
| `tools-allowed` | Tool configuration | Security, capability control |
| `tools-custom-mcp` | Custom MCP server (tools only) | Domain-specific tools |
| `tools-registration` | Zod-based tool definitions | Type-safe custom tools |
| `hook-pre-tool` | Pre-execution hooks | Blocking, validation |
| `hook-post-tool` | Post-execution hooks | Logging, transformation |
| `config-settings` | Settings sources | Loading .claude/ configs |
| `config-model` | Model selection | Choosing opus/sonnet/haiku |
| `config-cwd` | Working directory | File operation scope |
| `pattern-message-queue` | Message queue class | V1 multi-turn |
| `pattern-ai-client` | Reusable client wrapper | Application architecture |
| `pattern-websocket` | WebSocket integration | Real-time apps |
| `pattern-subagent` | Task tool orchestration | Multi-agent systems |
| `pattern-task-filesystem` | File-based subagent definitions | Production multi-agent |
| `pattern-self-improving` | Agents that persist their own skills | Evolving capabilities |
| `persistence-sqlite` | SQLite chat storage with session resume | Database persistence |
| `persistence-jsonl` | JSONL file-based chat storage | Lightweight persistence |
| `ui-tool-approval` | PreToolUse hook with WebSocket approval | Human-in-the-loop |
| `ui-tool-history` | Tool event extraction for UI display | Activity tracking |
| `ui-websocket-chat` | Complete WebSocket chat session | Full-featured chat UI |

### 1.3 Check SDK Version

The SDK evolves quickly. Check the latest version:

```bash
npm info @anthropic-ai/claude-agent-sdk version
```

Note: V2 APIs are prefixed with `unstable_v2_` and may change between versions.

---

## Phase 2: Implement

### 2.1 Initialize Project

```bash
mkdir my-agent && cd my-agent
npm init -y
npm install @anthropic-ai/claude-agent-sdk zod
npm install -D typescript @types/node
npx tsc --init
```

Update `tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "Node16",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

Update `package.json`:
```json
{
  "type": "module",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js"
  }
}
```

### 2.2 Add Snippets

Copy snippets from this skill's `snippets/` directory into your project. The snippets are organized by category:

```
snippets/
├── agent-v1/                 # V1 query() API patterns
│   ├── basic-query.ts
│   ├── query-with-tools.ts
│   ├── query-with-system-prompt.ts
│   └── query-with-queue.ts
├── agent-v2/                 # V2 session API patterns
│   ├── basic-session.ts
│   ├── multi-turn.ts
│   ├── one-shot.ts
│   └── session-resume.ts
├── tools/                    # Tool configuration
│   ├── allowed-tools.ts
│   ├── custom-mcp-server.ts
│   └── tool-registration.ts
├── hooks/                    # Pre/Post tool hooks
│   ├── pre-tool-use.ts
│   └── post-tool-use.ts
├── config/                   # Settings and configuration
│   ├── settings-sources.ts
│   ├── model-selection.ts
│   └── working-directory.ts
├── patterns/                 # Advanced patterns
│   ├── message-queue.ts
│   ├── ai-client-wrapper.ts
│   ├── websocket-session.ts
│   └── subagent-task.ts
├── persistence/              # Chat persistence
│   ├── chat-store-sqlite.ts
│   └── chat-store-jsonl.ts
└── ui/                       # UI integration
    ├── tool-approval-hook.ts
    ├── tool-history-stream.ts
    └── websocket-chat-session.ts
```

**Copy snippets directly:**
```bash
cp snippets/agent-v1/basic-query.ts /path/to/my-agent/src/
cp snippets/tools/allowed-tools.ts /path/to/my-agent/src/
```

### 2.3 Customize and Integrate

Each snippet includes:
- Source attribution with documentation links
- Working code ready to customize

Modify the copied code:
1. Update prompts and system messages
2. Configure allowed/disallowed tools
3. Add custom hooks if needed
4. Integrate with your application logic

---

## Phase 3: Test

### 3.1 Build and Run

```bash
npm run build
npm start
```

### 3.2 Debug Output

Add message logging to see what the agent is doing:

```typescript
for await (const message of q) {
  console.log(JSON.stringify(message, null, 2));
}
```

### 3.3 Quality Checklist

- [ ] Agent responds appropriately to prompts
- [ ] Tools are correctly restricted (if applicable)
- [ ] Hooks fire and behave as expected
- [ ] Error handling works for common failures
- [ ] Cost tracking shows expected usage
- [ ] Session persistence works (V2 only)

### 3.4 Common Issues

**Authentication errors / "Unauthorized"**
- Run `claude auth status` to check if logged in
- Run `claude login` if not authenticated
- For servers: set `ANTHROPIC_API_KEY` environment variable

**"Cannot find module '@anthropic-ai/claude-agent-sdk'"**
- Ensure you installed the SDK: `npm install @anthropic-ai/claude-agent-sdk`

**"await using" syntax error**
- Requires TypeScript 5.2+ with `target: "ES2022"` or higher
- Alternative: manually call `session[Symbol.asyncDispose]()` in finally block

**Tool calls fail silently**
- Check `allowedTools` includes the tool being called
- Check `disallowedTools` doesn't block it

**Agent doesn't see .claude/ settings**
- Add `settingSources: ['project', 'local']` to options
- Ensure `cwd` points to directory containing `.claude/`

---

## Available Snippets Catalog

### V1: Query-based API

| Name | Description |
|------|-------------|
| `v1-basic-query` | Simple query() with prompt string, iterate over messages |
| `v1-query-tools` | Configure allowedTools and disallowedTools arrays |
| `v1-query-system` | Custom systemPrompt option for agent behavior |
| `v1-query-queue` | AsyncIterable<SDKUserMessage> input for multi-turn conversations |

### V2: Session-based API

| Name | Description |
|------|-------------|
| `v2-basic-session` | Create session with unstable_v2_createSession, send() and stream() |
| `v2-multi-turn` | Sequential multi-turn conversation with session context retention |
| `v2-one-shot` | One-shot query using unstable_v2_prompt convenience function |
| `v2-session-resume` | Persist and resume sessions with unstable_v2_resumeSession |

### Tool Configuration

| Name | Description |
|------|-------------|
| `tools-allowed` | Whitelist specific tools with allowedTools configuration |
| `tools-custom-mcp` | Create custom MCP server with createSdkMcpServer (tools only) |
| `tools-registration` | Define tools with Zod schemas using tool() helper function |
| `tools-custom-skill` | Load custom skills from .claude/skills/ via Skill tool |
| `tools-mcp-server-config` | Connect to external MCP servers (stdio, Streamable HTTP) |

### Hooks

| Name | Description |
|------|-------------|
| `hook-pre-tool` | PreToolUse hooks to block or modify tool calls before execution |
| `hook-post-tool` | PostToolUse hooks to process or transform tool results |

### Configuration

| Name | Description |
|------|-------------|
| `config-settings` | Configure settingSources array (user, project, local) |
| `config-model` | Model selection options (opus, sonnet, haiku) |
| `config-cwd` | Set working directory (cwd) for file operations |

### Advanced Patterns

| Name | Description |
|------|-------------|
| `pattern-message-queue` | Async MessageQueue class for multi-turn V1 conversations |
| `pattern-ai-client` | Reusable AIClient wrapper class around query() |
| `pattern-websocket` | WebSocket server with SDK session integration |
| `pattern-subagent` | Task tool pattern for subagent orchestration |
| `pattern-task-filesystem` | File-based subagent definitions in .claude/agents/*.md |
| `pattern-self-improving` | Agents that develop, test, and persist their own reusable skills |

### Chat Persistence

| Name | Description |
|------|-------------|
| `persistence-sqlite` | SQLite chat storage with session ID tracking for resume |
| `persistence-jsonl` | JSONL file-based storage, no native dependencies |

### UI Integration

| Name | Description |
|------|-------------|
| `ui-tool-approval` | PreToolUse hook that pauses for WebSocket-based user approval |
| `ui-tool-history` | Extract and format tool events for UI display |
| `ui-websocket-chat` | Complete WebSocket chat session with persistence and approval |

---

## Related Skills

This skill focuses on **Claude agents** that use MCP servers. See also:

| Skill | Use When |
|-------|----------|
| **claude-agent-ui-ts** | Adding React + WebSocket UI on top of your agent with tool approval |
| **mcp-server-ts** | Building MCP servers with tools, resources, prompts, sampling, tasks |
| **mcp-client-ts** | Building custom MCP clients with full protocol support |

**When to use each:**
- `claude-agent-sdk-ts`: Out-of-the-box Claude agent when you don't need sampling, elicitation, or logging
- `claude-agent-ui-ts`: Adding a web UI with real-time chat and tool approval to your agent
- `mcp-client-ts`: Sophisticated modular agents or apps needing full MCP control
- `mcp-server-ts`: Building the MCP servers themselves

**Claude Agent SDK MCP Capabilities** (verified via everything-server test):

| Capability | Sub-capability | Supported | Verification |
|------------|----------------|-----------|--------------|
| tools | | Yes | SDK has MCP tool calling |
| tools | listChanged | Unverified | |
| resources | | Yes | ListMcpResourcesTool, ReadMcpResourceTool |
| resources | subscribe | Unverified | |
| resources | listChanged | Unverified | |
| prompts | | No | No prompt tools in SDK |
| roots | | Yes | get-roots-list tool registered |
| roots | listChanged | Unverified | |
| sampling | | No | trigger-sampling-request NOT registered |
| elicitation | | No | trigger-elicitation-request NOT registered |
| logging | | No | SDK doesn't expose server logs |

For sampling, elicitation, logging, or prompts support, use `mcp-client-ts` to build a custom client.

---

## Reference Files

For deeper guidance, load these reference documents:

- [Agent SDK API Reference](./reference/agent_sdk_api_reference.md) - V1 and V2 API documentation, types, options
- [Agent SDK Patterns](./reference/agent_sdk_patterns.md) - Best practices, when to use V1 vs V2, common patterns

---

## External Resources

- [Claude Agent SDK Repository](https://github.com/anthropics/claude-agent-sdk-typescript) - Official SDK source
- [Claude Agent SDK Documentation](https://docs.anthropic.com/en/docs/claude-code/sdk) - Official documentation
- [Claude Code](https://claude.ai/code) - Claude Code CLI (SDK is based on this)
