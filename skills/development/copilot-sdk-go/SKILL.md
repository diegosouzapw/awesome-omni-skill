---
name: copilot-sdk-go
description: Expert guidance for using the GitHub Copilot CLI SDK with Go, including API reference, best practices, and common usage patterns.
---

# GitHub Copilot SDK for Go - Skill Guide

This skill provides comprehensive guidance for using the GitHub Copilot CLI SDK with Go programming language.

## Reference Documentation Locations

When working with the Copilot SDK for Go, always refer to these documentation sources:

### Core SDK Documentation
- **Main README**: [./reference/go/README.md](./reference/go/README.md) - Complete API reference, installation, and quick start
- **Type Definitions**: [./reference/go/types.go](./reference/go/types.go) - All type definitions and interfaces
- **Client Implementation**: [./reference/go/client.go](./reference/go/client.go) - Client implementation details
- **Session Management**: [./reference/go/session.go](./reference/go/session.go) - Session handling code

### Cookbook & Examples
- **Cookbook Index**: [./reference/cookbook/README.md](./reference/cookbook/README.md) - Overview of all cookbooks
- **Go Cookbook**: [./reference/cookbook/go/README.md](./reference/cookbook/go/README.md) - Go-specific recipes index

#### Go Recipes (with runnable examples)
- **Error Handling**: [./reference/cookbook/go/error-handling.md](./reference/cookbook/go/error-handling.md)
  - Example code: [./reference/cookbook/go/recipe/error-handling.go](./reference/cookbook/go/recipe/error-handling.go)
- **Multiple Sessions**: [./reference/cookbook/go/multiple-sessions.md](./reference/cookbook/go/multiple-sessions.md)
  - Example code: [./reference/cookbook/go/recipe/multiple-sessions.go](./reference/cookbook/go/recipe/multiple-sessions.go)
- **Managing Local Files**: [./reference/cookbook/go/managing-local-files.md](./reference/cookbook/go/managing-local-files.md)
  - Example code: [./reference/cookbook/go/recipe/managing-local-files.go](./reference/cookbook/go/recipe/managing-local-files.go)
- **PR Visualization**: [./reference/cookbook/go/pr-visualization.md](./reference/cookbook/go/pr-visualization.md)
  - Example code: [./reference/cookbook/go/recipe/pr-visualization.go](./reference/cookbook/go/recipe/pr-visualization.go)
- **Persisting Sessions**: [./reference/cookbook/go/persisting-sessions.md](./reference/cookbook/go/persisting-sessions.md)
  - Example code: [./reference/cookbook/go/recipe/persisting-sessions.go](./reference/cookbook/go/recipe/persisting-sessions.go)

### Test Examples
- **E2E Tests**: [./reference/go/e2e/](./reference/go/e2e/) - Comprehensive end-to-end test examples

## Installation

```bash
go get github.com/github/copilot-sdk/go
```

## Core Concepts

### 1. Client Creation and Lifecycle

```go
import copilot "github.com/github/copilot-sdk/go"

// Create client with default options
client := copilot.NewClient(nil)

// Or with custom options
client := copilot.NewClient(&copilot.ClientOptions{
    LogLevel:    "error",
    UseStdio:    true,
    AutoStart:   copilot.Bool(true),
    AutoRestart: copilot.Bool(true),
})

// Always start the client
if err := client.Start(); err != nil {
    log.Fatal(err)
}
defer client.Stop()
```

**Client Options:**
- `CLIPath`: Path to CLI executable (default: "copilot")
- `CLIUrl`: Connect to existing server (format: "localhost:8080" or just "8080")
- `UseStdio`: Use stdio transport (default: true)
- `Port`: Server port for TCP mode
- `LogLevel`: Log level (default: "info")
- `AutoStart`: Auto-start server (default: true, use `Bool(false)` to disable)
- `AutoRestart`: Auto-restart on crash (default: true)
- `GithubToken`: GitHub token for auth
- `UseLoggedInUser`: Use logged-in user auth (default: true)

### 2. Session Management

```go
// Create a new session
session, err := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-5",  // Required when using custom provider
    SessionID: "my-custom-id",  // Optional custom ID
    Streaming: true,  // Enable streaming responses
})
if err != nil {
    log.Fatal(err)
}
defer session.Destroy()

// Resume an existing session
session, err := client.ResumeSession("session-id")

// List all sessions
sessions, err := client.ListSessions()

// Delete a session
err := client.DeleteSession("session-id")
```

**SessionConfig Options:**
- `Model`: Model to use ("gpt-5", "claude-sonnet-4.5", etc.) - Required with custom provider
- `SessionID`: Custom session ID
- `Tools`: Custom tools ([]Tool)
- `SystemMessage`: System message config
- `Provider`: Custom API provider (BYOK)
- `Streaming`: Enable streaming deltas
- `InfiniteSessions`: Auto-compaction config
- `OnUserInputRequest`: Handler for ask_user tool
- `Hooks`: Session lifecycle hooks
- `MCPServers`: MCP server configurations
- `CustomAgents`: Custom agent configs

### 3. Sending Messages and Handling Events

```go
// Set up event handler
done := make(chan bool)
session.On(func(event copilot.SessionEvent) {
    switch event.Type {
    case "assistant.message":
        if event.Data.Content != nil {
            fmt.Println(*event.Data.Content)
        }
    case "assistant.message_delta":
        // Streaming chunk
        if event.Data.DeltaContent != nil {
            fmt.Print(*event.Data.DeltaContent)
        }
    case "session.idle":
        close(done)
    }
})

// Send a message
_, err = session.Send(copilot.MessageOptions{
    Prompt: "What is 2+2?",
    Attachments: []copilot.Attachment{
        {Type: "file", Path: "/path/to/file.txt"},
    },
})

<-done  // Wait for completion
```

### 4. Custom Tools

Using `DefineTool` (recommended - type-safe with auto schema):

```go
type LookupParams struct {
    ID string `json:"id" jsonschema:"Issue identifier"`
}

lookupTool := copilot.DefineTool("lookup_issue",
    "Fetch issue details",
    func(params LookupParams, inv copilot.ToolInvocation) (any, error) {
        // params is automatically unmarshaled
        return fetchIssue(params.ID)
    })

session, _ := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-5",
    Tools: []copilot.Tool{lookupTool},
})
```

Using Tool struct directly (more control):

```go
tool := copilot.Tool{
    Name:        "my_tool",
    Description: "Tool description",
    Parameters: map[string]interface{}{
        "type": "object",
        "properties": map[string]interface{}{
            "param": map[string]interface{}{
                "type": "string",
                "description": "Parameter description",
            },
        },
        "required": []string{"param"},
    },
    Handler: func(inv copilot.ToolInvocation) (copilot.ToolResult, error) {
        args := inv.Arguments.(map[string]interface{})
        // Process args
        return copilot.ToolResult{
            TextResultForLLM: "result",
            ResultType:       "success",
        }, nil
    },
}
```

### 5. Custom Providers (BYOK)

**Ollama (local):**
```go
session, err := client.CreateSession(&copilot.SessionConfig{
    Model: "deepseek-coder-v2:16b",  // Required!
    Provider: &copilot.ProviderConfig{
        Type:    "openai",
        BaseURL: "http://localhost:11434/v1",
        // APIKey not needed for Ollama
    },
})
```

**Azure OpenAI:**
```go
session, err := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-4",
    Provider: &copilot.ProviderConfig{
        Type:    "azure",  // Must be "azure", not "openai"!
        BaseURL: "https://my-resource.openai.azure.com",
        APIKey:  os.Getenv("AZURE_OPENAI_KEY"),
        Azure: &copilot.AzureProviderOptions{
            APIVersion: "2024-10-21",
        },
    },
})
```

**Custom OpenAI-compatible:**
```go
Provider: &copilot.ProviderConfig{
    Type:    "openai",
    BaseURL: "https://my-api.example.com/v1",
    APIKey:  os.Getenv("MY_API_KEY"),
}
```

### 6. Streaming Responses

```go
session, err := client.CreateSession(&copilot.SessionConfig{
    Model:     "gpt-5",
    Streaming: true,  // Enable streaming
})

session.On(func(event copilot.SessionEvent) {
    if event.Type == "assistant.message_delta" {
        // Incremental chunks
        if event.Data.DeltaContent != nil {
            fmt.Print(*event.Data.DeltaContent)
        }
    } else if event.Type == "assistant.message" {
        // Final complete message
        fmt.Println("\nComplete!")
    }
})
```

### 7. Infinite Sessions (Auto-compaction)

```go
// Default: enabled with default thresholds
session, _ := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-5",
})

// Custom thresholds
session, _ := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-5",
    InfiniteSessions: &copilot.InfiniteSessionConfig{
        Enabled:                       copilot.Bool(true),
        BackgroundCompactionThreshold: copilot.Float64(0.80),  // 80%
        BufferExhaustionThreshold:     copilot.Float64(0.95),  // 95%
    },
})

// Disable
InfiniteSessions: &copilot.InfiniteSessionConfig{
    Enabled: copilot.Bool(false),
}

// Access workspace path
fmt.Println(session.WorkspacePath())
// => ~/.copilot/session-state/{sessionId}/
```

### 8. User Input Requests (ask_user tool)

```go
session, err := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-5",
    OnUserInputRequest: func(req copilot.UserInputRequest, inv copilot.UserInputInvocation) (copilot.UserInputResponse, error) {
        fmt.Printf("Question: %s\n", req.Question)
        if len(req.Choices) > 0 {
            fmt.Printf("Choices: %v\n", req.Choices)
        }

        // Get user input (implement your UI logic here)
        answer := getUserInput()

        return copilot.UserInputResponse{
            Answer:      answer,
            WasFreeform: len(req.Choices) == 0,
        }, nil
    },
})
```

### 9. Session Hooks

```go
session, err := client.CreateSession(&copilot.SessionConfig{
    Model: "gpt-5",
    Hooks: &copilot.SessionHooks{
        OnPreToolUse: func(input copilot.PreToolUseHookInput, inv copilot.HookInvocation) (*copilot.PreToolUseHookOutput, error) {
            fmt.Printf("About to run: %s\n", input.ToolName)
            return &copilot.PreToolUseHookOutput{
                PermissionDecision: "allow",  // "allow", "deny", "ask"
                ModifiedArgs:       input.ToolArgs,
                AdditionalContext:  "Extra context",
            }, nil
        },
        OnPostToolUse: func(input copilot.PostToolUseHookInput, inv copilot.HookInvocation) (*copilot.PostToolUseHookOutput, error) {
            fmt.Printf("Tool completed: %s\n", input.ToolName)
            return &copilot.PostToolUseHookOutput{
                AdditionalContext: "Post-execution notes",
            }, nil
        },
        OnSessionStart: func(input copilot.SessionStartHookInput, inv copilot.HookInvocation) (*copilot.SessionStartHookOutput, error) {
            fmt.Printf("Session started: %s\n", input.Source)  // "startup", "resume", "new"
            return nil, nil
        },
        OnSessionEnd: func(input copilot.SessionEndHookInput, inv copilot.HookInvocation) (*copilot.SessionEndHookOutput, error) {
            fmt.Printf("Session ended: %s\n", input.Reason)
            return nil, nil
        },
        OnErrorOccurred: func(input copilot.ErrorOccurredHookInput, inv copilot.HookInvocation) (*copilot.ErrorOccurredHookOutput, error) {
            return &copilot.ErrorOccurredHookOutput{
                ErrorHandling: "retry",  // "retry", "skip", "abort"
            }, nil
        },
    },
})
```

## Best Practices

### Error Handling
1. Always use `defer client.Stop()` and `defer session.Destroy()`
2. Handle connection errors (CLI might not be installed)
3. Set appropriate timeouts for long-running requests
4. Use context for cancellation
5. Wrap errors with `fmt.Errorf` and `%w` for error chains

**Example:**
```go
func doWork() error {
    client := copilot.NewClient(nil)
    if err := client.Start(); err != nil {
        return fmt.Errorf("failed to start: %w", err)
    }
    defer client.Stop()

    session, err := client.CreateSession(&copilot.SessionConfig{Model: "gpt-5"})
    if err != nil {
        return fmt.Errorf("failed to create session: %w", err)
    }
    defer session.Destroy()

    // Work with session...
    return nil
}
```

### Multiple Sessions
- One session per user in multi-user apps
- Separate sessions for different tasks
- Use custom session IDs for easy tracking
- Clean up sessions with `DeleteSession()` when done

### Streaming
- Enable `Streaming: true` for real-time responses
- Handle `assistant.message_delta` for incremental updates
- Handle `assistant.reasoning_delta` for model reasoning (if supported)
- Final `assistant.message` always contains complete content

### Image Support
```go
_, err = session.Send(copilot.MessageOptions{
    Prompt: "What's in this image?",
    Attachments: []copilot.Attachment{
        {Type: "file", Path: "/path/to/image.jpg"},
    },
})
```

## Common Patterns

### Pattern 1: Simple Question-Answer
```go
client := copilot.NewClient(nil)
client.Start()
defer client.Stop()

session, _ := client.CreateSession(&copilot.SessionConfig{Model: "gpt-5"})
defer session.Destroy()

done := make(chan string)
session.On(func(event copilot.SessionEvent) {
    if event.Type == "assistant.message" && event.Data.Content != nil {
        done <- *event.Data.Content
    }
})

session.Send(copilot.MessageOptions{Prompt: "What is 2+2?"})
fmt.Println(<-done)
```

### Pattern 2: Multiple Sessions
```go
session1, _ := client.CreateSession(&copilot.SessionConfig{
    SessionID: "python-help",
    Model:     "gpt-5",
})
session2, _ := client.CreateSession(&copilot.SessionConfig{
    SessionID: "go-help",
    Model:     "gpt-5",
})

session1.Send(copilot.MessageOptions{Prompt: "Python question"})
session2.Send(copilot.MessageOptions{Prompt: "Go question"})
```

### Pattern 3: Custom Tool with Type Safety
```go
type GitStatusParams struct {
    Repo string `json:"repo" jsonschema:"Repository path"`
}

gitStatus := copilot.DefineTool("git_status", "Get git status",
    func(params GitStatusParams, inv copilot.ToolInvocation) (any, error) {
        output, err := exec.Command("git", "-C", params.Repo, "status").Output()
        return string(output), err
    })
```

### Pattern 4: Graceful Shutdown
```go
sigChan := make(chan os.Signal, 1)
signal.Notify(sigChan, os.Interrupt, syscall.SIGTERM)

go func() {
    <-sigChan
    fmt.Println("\nShutting down...")
    client.Stop()
    os.Exit(0)
}()
```

## Helper Functions

- `Bool(v bool) *bool` - Create bool pointers for options
- `Float64(v float64) *float64` - Create float64 pointers for thresholds

```go
AutoStart: copilot.Bool(false)
BackgroundCompactionThreshold: copilot.Float64(0.80)
```

## Transport Modes

- **stdio (default)**: Stdin/stdout pipes - recommended for most cases
- **TCP**: Socket communication - useful for distributed scenarios

```go
// stdio (default)
client := copilot.NewClient(nil)

// TCP with custom port
client := copilot.NewClient(&copilot.ClientOptions{
    UseStdio: false,
    Port:     8080,
})

// Connect to existing server
client := copilot.NewClient(&copilot.ClientOptions{
    CLIUrl: "localhost:8080",
})
```

## Environment Variables

- `COPILOT_CLI_PATH`: Path to Copilot CLI executable

## Troubleshooting

### CLI not found
```go
if err := client.Start(); err != nil {
    var execErr *exec.Error
    if errors.As(err, &execErr) {
        log.Fatal("Copilot CLI not found. Please install it.")
    }
}
```

### Connection timeout
Use context with timeout when sending messages.

### Custom provider not working
- Ensure `Model` is specified (required with custom providers)
- For Azure, use `Type: "azure"`, not `"openai"`
- BaseURL should be just the host, no `/v1` path for Azure

## When to Use What

| Use Case | Solution |
|----------|----------|
| Simple chat | Basic session with default config |
| Real-time responses | `Streaming: true` |
| Long conversations | Infinite sessions (default) |
| Custom functionality | Define tools with `DefineTool` |
| Local models | Custom provider with Ollama |
| Multi-user app | Multiple sessions with custom IDs |
| Permission control | Session hooks with `OnPreToolUse` |
| User interaction | `OnUserInputRequest` handler |
| Different models | Multiple sessions with different configs |

## Additional Resources

- Full E2E test examples: `./reference/go/e2e/`
- All type definitions: `./reference/go/types.go`
- Client internals: `./reference/go/client.go`
- Session internals: `./reference/go/session.go`

## Quick Reference

```go
// Essential imports
import copilot "github.com/github/copilot-sdk/go"

// Client lifecycle
client := copilot.NewClient(nil)
client.Start()
defer client.Stop()

// Session lifecycle
session, _ := client.CreateSession(&copilot.SessionConfig{Model: "gpt-5"})
defer session.Destroy()

// Event handling
session.On(func(event copilot.SessionEvent) { /* ... */ })

// Send message
session.Send(copilot.MessageOptions{Prompt: "Hello"})

// Custom tool
copilot.DefineTool("name", "desc", func(params Type, inv copilot.ToolInvocation) (any, error) { /* ... */ })
```
