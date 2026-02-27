---
name: copilot-sdk-dotnet
description: Build applications with GitHub Copilot CLI SDKs for .NET. Use for direct CopilotClient integration or Microsoft Agent Framework. Covers sessions, streaming, tools, MCP, permissions, and multi-agent workflows.
metadata:
    version: 1.1.0
    authors: arisng
---

# GitHub Copilot CLI SDK (.NET)

Use this skill to guide SDK integration, select appropriate APIs, and provide C# code examples. Focus on practical implementation patterns for .NET.

## Quick workflow

1. Identify the user's goal (client creation, session management, streaming, custom tools, permissions, MCP integration, or custom agents).
2. Determine if the user needs basic `CopilotClient` or `Microsoft.Agents.AI` integration.
3. Select the matching API section and code pattern.
4. Provide C# examples with clear context (usings, initialization, event handling).
5. Call out important defaults and configuration options that affect behavior.
6. Clarify stability scope: Agent Framework is RC (stable API surface toward GA) while Copilot SDK pieces may still evolve.

## SDK selection and installation

### Core SDK (Direct Integration)
- **.NET**: `dotnet add package GitHub.Copilot.SDK` (uses AIFunctionFactory and attributes)

### Microsoft Agent Framework (Wrapper)
- **.NET**: `dotnet add package Microsoft.Agents.AI.GitHub.Copilot --prerelease`

Note:
- Microsoft Agent Framework is in Release Candidate and has a stable API surface for 1.0 planning.
- Keep prerelease suffixes where package feeds still require them, and pin versions in implementation docs.

All SDKs communicate via JSON-RPC over stdio (default) or TCP transports.

## Core concepts

- **CopilotClient**: Main entry point managing server lifecycle, connections, and session creation. Can spawn CLI server automatically or connect to external server.
- **Sessions**: Represent individual conversations with persistent state, event handlers, and tool registrations. Each session has a sessionId, model, and optional streaming/tools configuration.
- **Tools**: Custom functions the assistant can invoke. Support type-safe schemas via `AIFunctionFactory` or manual definitions.
- **MCP Servers**: External tool servers (local stdio, HTTP, or SSE) extending assistant capabilities.
- **Custom Agents**: Specialized personas with scoped prompts, tool access, and configurations.
- **Streaming**: Real-time response delivery via event handlers (e.g., `assistant.message_delta` for chunks).

## Microsoft Agent Framework Integration

For .NET, use the Microsoft Agent Framework wrapper to treat Copilot as a building block in larger agentic systems.

- **Consistent Abstraction**: Implements `AIAgent`.
- **Multi-Agent Workflows**: Compose Copilot agents with Azure OpenAI/Anthropic agents.
- **Features**: Supports streaming, function tools, sessions, and MCP servers via framework patterns.
- **RC Guidance**: Favor Agent Framework abstractions (`AIAgent`, workflows, approvals) for new orchestration code instead of building direct orchestration plumbing on top of raw session events.

## Migration Guidance (SK/AutoGen -> Agent Framework)

When users are migrating existing agent systems:
1. Inventory existing Semantic Kernel or AutoGen flows (single-agent, tool calls, orchestration, HITL).
2. Map to Agent Framework primitives first (`AIAgent`, tools, workflows, checkpoints, approvals).
3. Reuse Copilot SDK at integration boundaries (session/tool transport), not as the primary orchestration model.
4. Validate parity for streaming output, approval gates, and failure recovery before full cutover.

**See [Agent Framework Integration](references/agent-framework.md) for full guide on installation, agent creation, and multi-agent orchestration.**

## Client creation patterns

### Default client (auto-spawns CLI server)

- **.NET**: `await using var client = new CopilotClient();`

### Client with options

Common options:
- `CliPath`: Custom CLI binary path (default: auto-detect)
- `LogLevel`: "none", "error", "warning", "info", "debug", "all"
- `AutoStart`: Auto-spawn server on first use (default: true)
- `AutoRestart`: Auto-restart on crash (default: true)
- `UseStdio`: Use stdio transport (default: true)
- `Port`: TCP port (0 = random, only when useStdio is false)
- `Cwd`: Working directory for CLI process

### External server connection

Use `CliUrl` to connect to existing server:
- `"localhost:8080"`, `"http://127.0.0.1:9000"`, or just `"8080"`

### Lifecycle management

- `await client.StartAsync()` (manual start if AutoStart is false)
- `client.State` (returns "disconnected", "connecting", "connected", "error")
- `await client.PingAsync()` (verify connectivity)
- `await client.StopAsync()` (graceful shutdown)
- `await client.ForceStopAsync()` (force stop if graceful shutdown hangs)

## Session creation and management

### Basic session

```csharp
var session = await client.CreateSessionAsync(new SessionConfig { Model = "gpt-5" });
```


### Session with full configuration

Common options:
- `SessionId`: Custom session identifier (optional)
- `Model`: Model to use (e.g., "gpt-5", "claude-sonnet-4.5")
- `Streaming`: Enable streaming responses (default: false)
- `AvailableTools`: Whitelist specific tools
- `ExcludedTools`: Blacklist specific tools
- `SystemMessage`: Custom system prompt with mode ("append" or "replace")
- `OnPermissionRequest`: Permission handler function

### Session queries

- `await client.ListSessionsAsync()` (all sessions)
- `await client.GetLastSessionIdAsync()` (for resuming)
- `await client.ResumeSessionAsync(sessionId, options)` (re-open existing)
- `await client.DeleteSessionAsync(sessionId)` (permanent delete)
- `await session.DestroyAsync()` (release resources, don't delete)

## Message handling and events

### Event subscription pattern

Subscribe before sending messages. Use `session.Events` (IObservable or event based):
```csharp
session.Events.Subscribe(@event => {
    switch (@event) {
        case UserMessageEvent: // User's input
        case AssistantMessageEvent: // Complete response
        case AssistantMessageDeltaEvent: // Streaming chunk
        case ToolExecutionStartEvent: // Tool invoked
        case ToolExecutionEndEvent: // Tool completed
        case SessionIdleEvent: // No activity
        case SessionErrorEvent: // Error occurred
    }
});
```

### Send patterns

**Synchronous (wait for completion)**:
```csharp
var response = await session.SendAndWaitAsync(new MessageOptions {
    Prompt = "Your question",
    Attachments = [/* optional files/dirs */]
}, TimeSpan.FromSeconds(60)); // Optional timeout
```

**Asynchronous (non-blocking, event-driven)**:
```csharp
var messageId = await session.SendAsync(new MessageOptions {
    Prompt = "Long task",
    Mode = MessageMode.Enqueue // or Immediate
});
```

### File attachments

Support two types:
```csharp
Attachments = [
    new Attachment { Type = "file", Path = "/path/to/file", DisplayName = "File" },
    new Attachment { Type = "directory", Path = "/path/to/dir", DisplayName = "Folder" }
]
```

### Conversation history

```csharp
var history = await session.GetMessagesAsync();
```

### Abort long-running requests

```csharp
await session.AbortAsync();
```

## Custom tools (type-safe)

### .NET (AIFunctionFactory)

```csharp
using Microsoft.Extensions.AI;
using System.ComponentModel;

var myTool = AIFunctionFactory.Create(
    async ([Description("...")] string param1) => {
        // Implementation
        return new { result = "..." };
    },
    "tool_name",
    "What this tool does"
);

var session = await client.CreateSessionAsync(new SessionConfig {
    Tools = new[] { myTool }
});
```

### Tool result structure (raw schema pattern)

All languages support returning rich structured results:
```json
{
    "textResultForLlm": "Human-readable text for model",
    "resultType": "success", // or "failure"
    "sessionLog": "Internal diagnostic message",
    "toolTelemetry": { /* structured data */ },
    "error": "Error message if failure"
}
```

## Permission handling

Register a permission handler to approve/deny sensitive operations. Available kinds: "shell", "write", "read", "url", "mcp".

### .NET pattern

```csharp
var session = await client.CreateSessionAsync(new SessionConfig {
    OnPermissionRequest = async (request, context) => {
        switch (request.Kind) {
            case "write":
                var path = request.Properties["path"]?.ToString();
                if (path?.StartsWith("/safe/") == true) {
                    return new PermissionRequestResult { Kind = "approved" };
                }
                return new PermissionRequestResult { 
                    Kind = "denied-by-rules", 
                    Message = "Path not allowed" 
                };
            case "shell":
                return new PermissionRequestResult { Kind = "denied-interactively-by-user" };
            case "read":
                return new PermissionRequestResult { Kind = "approved" };
            default:
                return new PermissionRequestResult { 
                    Kind = "denied-no-approval-rule-and-could-not-request-from-user" 
                };
        }
    }
});
```


### Result kinds

- `"approved"`: Allow the operation
- `"denied-by-rules"`: Deny with rule explanation
- `"denied-interactively-by-user"`: User denied
- `"denied-no-approval-rule-and-could-not-request-from-user"`: No rule and no user interaction

## Custom provider configuration (BYOK)

Use your own API keys with OpenAI, Azure OpenAI, Anthropic, or compatible providers.

### .NET pattern

```csharp
var session = await client.CreateSessionAsync(new SessionConfig {
    Provider = new ProviderConfig {
        Type = "azure",
        BaseUrl = "https://your-resource.openai.azure.com",
        ApiKey = Environment.GetEnvironmentVariable("AZURE_OPENAI_KEY"),
        Azure = new AzureProviderConfig { ApiVersion = "2024-10-21" }
    }
});
```

## MCP Server integration

Connect local (stdio) or remote (HTTP/SSE) MCP servers.

### .NET pattern

```csharp
using GitHub.Copilot.SDK.Mcp;

var session = await client.CreateSessionAsync(new SessionConfig {
    McpServers = new Dictionary<string, McpServerConfig> {
        ["filesystem"] = new McpLocalServerConfig {
            Type = "local",
            Command = "npx",
            Args = ["-y", "@modelcontextprotocol/server-filesystem", "/path"],
            Tools = "*" 
        },
        ["remote-api"] = new McpRemoteServerConfig {
            Type = "http",
            Url = "https://mcp-server.example.com/mcp",
            Tools = ["search"]
        }
    }
});
```

## Custom agents

Define specialized personas with scoped system prompts and tool access.

### .NET pattern

```csharp
var session = await client.CreateSessionAsync(new SessionConfig {
    CustomAgents = [
        new CustomAgent {
            Name = "security-reviewer",
            DisplayName = "Security Reviewer",
            Description = "Reviews code for vulnerabilities",
            Prompt = "You are a security expert...",
            Tools = ["Read", "Grep", "Glob"],
            Infer = true
        }
    ]
});

// Invoke via prompt
await session.SendAndWaitAsync(new MessageOptions {
    Prompt = "@security-reviewer Review src/auth.cs"
});
```

## Common patterns and best practices

### Pattern 1: Simple request-response

```csharp
await using var client = new CopilotClient();
await client.StartAsync();
await using var session = await client.CreateSessionAsync(new SessionConfig { Model = "gpt-5" });
var response = await session.SendAndWaitAsync(new MessageOptions { Prompt = "Your question" });
Console.WriteLine(response.Data.Content);
```

### Pattern 2: Streaming with events

```csharp
var session = await client.CreateSessionAsync(new SessionConfig { 
    Model = "gpt-5", 
    Streaming = true 
});
session.Events.Subscribe(@event => {
    if (@event is AssistantMessageDeltaEvent delta) {
         Console.Write(delta.DeltaContent);
    }
});
await session.SendAndWaitAsync(new MessageOptions { Prompt = "Write a story" });
```

### Pattern 3: Tool invocation with events

```csharp
var session = await client.CreateSessionAsync(new SessionConfig { 
    Tools = new[] { myTool } 
});
session.Events.Subscribe(@event => {
    if (@event is ToolExecutionStartEvent start) {
        Console.WriteLine($"Tool {start.ToolName} started");
    }
});
await session.SendAndWaitAsync(new MessageOptions { Prompt = "Use the tool" });
```

### Pattern 4: File attachments

```csharp
await session.SendAndWaitAsync(new MessageOptions {
    Prompt = "Analyze this code",
    Attachments = [
        new Attachment { Type = "file", Path = "./src/Program.cs", DisplayName = "Main" }
    ]
});
```

### Pattern 5: Multi-language BYOK setup

Choose the provider once during session creation.

## Glossary of GitHub Copilot Products

The GitHub Copilot ecosystem comprises multiple distinct products, each serving different use cases and contexts. Understanding the boundaries between them is essential for choosing the right tool and understanding SDK capabilities.

| Product                      | Alias           | Description                                                                                                                                                                                              | Execution Context                            | Primary Interface                                                        | Key Use Cases                                                                                                         |
| ---------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **Copilot in VS Code**       | Copilot VS Code | GitHub Copilot integrated into Visual Studio Code IDE. Provides chat, inline suggestions, and agent mode for autonomous edits in local development environment.                                          | Local machine (developer's computer)         | VS Code editor window                                                    | Interactive coding assistance, pair programming, quick fixes, IDE-native automation                                   |
| **Copilot in GitHub Mobile** | Copilot Mobile  | GitHub Copilot accessible via GitHub Mobile app on iOS/Android. Limited to chat-based assistance.                                                                                                        | Cloud-side processing                        | GitHub Mobile app                                                        | Quick questions, code review assistance on the go, mobile browsing context                                            |
| **Copilot in GitHub Web**    | Copilot Web     | GitHub Copilot accessible via GitHub.com web interface. Provides chat and integration with GitHub issues, pull requests, and repository context.                                                         | Cloud-side processing                        | GitHub.com web UI (pull requests, issues, code view)                     | Pull request assistance, issue triage, repository questions, web-based workflows                                      |
| **Copilot in CLI**           | Copilot CLI     | Standalone command-line interface to GitHub Copilot. Interactive shell mode for prompt-based assistance with file system access and terminal commands.                                                   | Local machine (terminal)                     | Terminal/shell command line                                              | Terminal automation, scripting assistance, local file operations, batch workflows                                     |
| **Copilot CLI SDK**          | Copilot SDK     | Programmatic SDKs (Node.js/TypeScript, Python, Go, .NET) for integrating GitHub Copilot into applications. Provides low-level APIs for session management, streaming, custom tools, and MCP integration. | Local machine or application runtime         | Application code (embedded programmatically)                             | Application-level AI integration, custom tool development, embedded conversational AI, multi-language app development |
| **Copilot Coding Agent**     | Copilot Agent   | Autonomous background agent that works on GitHub to complete development tasks. Creates pull requests independently based on issues or chat prompts. Runs in GitHub Actions environment.                 | GitHub cloud infrastructure (GitHub Actions) | GitHub Issues, Pull Requests, GitHub Chat, CLI, Third-party integrations | Autonomous bug fixes, feature implementation, test coverage, documentation updates, technical debt resolution         |

## Important defaults and behaviors

- **Transport**: stdio by default (most reliable); TCP requires explicit port configuration
- **Streaming**: Disabled by default; enable per-session via `streaming: true`
- **Auto-restart**: Enabled by default; set `autoRestart: false` to prevent auto-recovery
- **Tool timeout**: 30 seconds default for MCP stdio servers
- **Session persistence**: Sessions survive client restarts if sessionId is preserved
- **Event ordering**: Events fire in order (user.message → tool.execution → assistant.message_delta → assistant.message → session.idle)
- **Stability scope**: Treat Agent Framework APIs as RC-stable; validate version compatibility between Copilot SDK packages and Agent Framework wrapper packages before upgrades

## Troubleshooting guidance

- **"CLI server not found"**: Verify Copilot CLI is installed or pass explicit `cliPath`
- **"Tool not found"**: Check tool is registered in session and assistant has access (not in `excludedTools`)
- **"Permission denied"**: Permission handler returned deny; check handler logic and rules
- **Streaming not working**: Verify `streaming: true` in session config and subscribed to `assistant.message_delta`
- **MCP server connection fails**: Check server is running, path/URL is correct, and `timeout` is sufficient
- **Session not resuming**: Verify `sessionId` was preserved and matches existing session

## Reference files

- Use [SDK Installation & Setup](references/sdk-setup.md) for quick install commands and platform-specific notes.
- Use [API Reference Guide](references/api-reference.md) for detailed parameter descriptions and error codes.
- Use [Code Examples by Language](references/code-examples.md) for complete working examples in TypeScript, Python, Go, and .NET.
- Use [MCP Server Integration](references/mcp-integration.md) for MCP server setup, local vs. remote patterns, and debugging.
- Use [Custom Tools & Agents](references/tools-agents.md) for tool definition patterns, schema validation, and agent persona setup.
- Use [Agent Framework Integration](references/agent-framework.md) for using Copilot with Microsoft Agent Framework (.NET/Python).
