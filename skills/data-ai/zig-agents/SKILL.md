---
name: zig-agents
description: Patterns and best practices for building AI agents in Zig. Covers tool systems, context management, LLM providers, streaming responses, and session persistence. Use when implementing agent functionality.
---

# Zig Agent Development

## Overview

This skill covers patterns for building AI agents in Zig, including:

- Tool system design and implementation
- Conversation context management
- LLM provider abstractions
- Streaming response handling
- Session persistence
- Agent loop patterns

## Architecture

```text
Agent
├── Config              # API keys, model settings
├── Context             # Conversation history
├── ToolRegistry        # Available tools
├── Provider            # LLM API abstraction
└── Session             # Persistence layer
```

### Agent Loop Pattern

The core agent loop follows this flow:

```text
1. Add user message to context
2. Loop (max N iterations):
   a. Call LLM with context + tools
   b. Add assistant response to context
   c. If tool calls present:
      - Execute each tool
      - Add tool results to context
      - Continue loop
   d. Else: break
3. Save session
```

## Tool System

### Tool Definition

```zig
pub const ToolContext = struct {
    allocator: std.mem.Allocator,
    config: Config,
};

pub const Tool = struct {
    name: []const u8,
    description: []const u8,
    parameters: []const u8,  // JSON Schema
    execute: *const fn (ctx: ToolContext, arguments: []const u8) anyerror![]const u8,
};
```

### Tool Registry

```zig
pub const ToolRegistry = struct {
    tools: std.StringHashMap(Tool),
    allocator: std.mem.Allocator,

    pub fn init(allocator: std.mem.Allocator) ToolRegistry {
        return .{
            .tools = std.StringHashMap(Tool).init(allocator),
            .allocator = allocator,
        };
    }

    pub fn deinit(self: *ToolRegistry) void {
        self.tools.deinit();
    }

    pub fn register(self: *ToolRegistry, tool: Tool) !void {
        try self.tools.put(tool.name, tool);
    }

    pub fn get(self: *ToolRegistry, name: []const u8) ?Tool {
        return self.tools.get(name);
    }
};
```

### Implementing Tools

Tools should:

1. Parse JSON arguments
2. Perform the operation
3. Return a string result (allocated with ctx.allocator)

Example: File reading tool

```zig
pub fn read_file(ctx: ToolContext, arguments: []const u8) ![]const u8 {
    const Args = struct { path: []const u8 };
    const parsed = try std.json.parseFromSlice(Args, ctx.allocator, arguments, .{
        .ignore_unknown_fields = true,
    });
    defer parsed.deinit();

    const file = try std.fs.cwd().openFile(parsed.value.path, .{});
    defer file.close();

    return file.readToEndAlloc(ctx.allocator, 10 * 1024 * 1024);
}
```

Example: Web search tool with HTTP

```zig
pub fn web_search(ctx: ToolContext, arguments: []const u8) ![]const u8 {
    const Args = struct { query: []const u8 };
    const parsed = try std.json.parseFromSlice(Args, ctx.allocator, arguments, .{
        .ignore_unknown_fields = true,
    });
    defer parsed.deinit();

    const api_key = ctx.config.tools.web.search.apiKey orelse {
        return try ctx.allocator.dupe(u8, "Error: Search API key not configured.");
    };

    var client = http.Client.init(ctx.allocator);
    defer client.deinit();

    // URL encode the query
    var encoded = std.io.Writer.Allocating.init(ctx.allocator);
    defer encoded.deinit();
    try std.Uri.Component.formatQuery(.{ .raw = parsed.value.query }, &encoded.writer);

    const url = try std.fmt.allocPrint(ctx.allocator, 
        "https://api.example.com/search?q={s}", .{encoded.written()});
    defer ctx.allocator.free(url);

    const headers = &[_]std.http.Header{
        .{ .name = "Authorization", .value = api_key },
    };

    var response = client.get(url, headers) catch |err| {
        return try std.fmt.allocPrint(ctx.allocator, "Error: {any}", .{err});
    };
    defer response.deinit();

    return try ctx.allocator.dupe(u8, response.body);
}
```

### Tool JSON Schema

Define parameters using JSON Schema for LLM consumption:

```zig
.parameters = 
    \\{"type": "object", "properties": {"path": {"type": "string", "description": "File path to read"}}, "required": ["path"]}
,
```

## Context Management

### Message Structure

```zig
pub const ToolCall = struct {
    id: []const u8,
    function_name: []const u8,
    arguments: []const u8,
};

pub const Message = struct {
    role: []const u8,           // "system", "user", "assistant", "tool"
    content: ?[]const u8 = null,
    tool_call_id: ?[]const u8 = null,
    tool_calls: ?[]ToolCall = null,
};
```

### Context Implementation

```zig
pub const Context = struct {
    messages: std.ArrayListUnmanaged(Message),
    allocator: std.mem.Allocator,

    pub fn init(allocator: std.mem.Allocator) Context {
        return .{
            .messages = .{},
            .allocator = allocator,
        };
    }

    pub fn deinit(self: *Context) void {
        for (self.messages.items) |msg| {
            self.freeMessage(msg);
        }
        self.messages.deinit(self.allocator);
    }

    pub fn add_message(self: *Context, msg: Message) !void {
        // Duplicate all strings to ensure ownership
        const duped = Message{
            .role = try self.allocator.dupe(u8, msg.role),
            .content = if (msg.content) |c| try self.allocator.dupe(u8, c) else null,
            .tool_call_id = if (msg.tool_call_id) |id| try self.allocator.dupe(u8, id) else null,
            .tool_calls = if (msg.tool_calls) |calls| try self.dupeToolCalls(calls) else null,
        };
        try self.messages.append(self.allocator, duped);
    }

    pub fn get_messages(self: *Context) []Message {
        return self.messages.items;
    }

    fn dupeToolCalls(self: *Context, calls: []ToolCall) ![]ToolCall {
        const result = try self.allocator.alloc(ToolCall, calls.len);
        for (calls, 0..) |call, i| {
            result[i] = .{
                .id = try self.allocator.dupe(u8, call.id),
                .function_name = try self.allocator.dupe(u8, call.function_name),
                .arguments = try self.allocator.dupe(u8, call.arguments),
            };
        }
        return result;
    }

    fn freeMessage(self: *Context, msg: Message) void {
        self.allocator.free(msg.role);
        if (msg.content) |c| self.allocator.free(c);
        if (msg.tool_call_id) |id| self.allocator.free(id);
        if (msg.tool_calls) |calls| {
            for (calls) |call| {
                self.allocator.free(call.id);
                self.allocator.free(call.function_name);
                self.allocator.free(call.arguments);
            }
            self.allocator.free(calls);
        }
    }
};
```

## Provider Abstraction

### Provider Interface

```zig
pub const ChatResponse = struct {
    content: ?[]const u8,
    tool_calls: ?[]ToolCall,
    allocator: std.mem.Allocator,

    pub fn deinit(self: *ChatResponse) void {
        if (self.content) |c| self.allocator.free(c);
        if (self.tool_calls) |calls| {
            for (calls) |call| {
                self.allocator.free(call.id);
                self.allocator.free(call.function_name);
                self.allocator.free(call.arguments);
            }
            self.allocator.free(calls);
        }
    }
};
```

### Streaming Pattern

Use a callback for streaming chunks:

```zig
pub fn chatStream(
    self: *Provider,
    messages: []const Message,
    model: []const u8,
    on_chunk: fn ([]const u8) void,
) !ChatResponse {
    // Build request
    var req = try self.client.post(self.api_url, headers, body);
    defer req.deinit();

    // Stream response
    var content = std.ArrayListUnmanaged(u8){};
    var tool_calls = std.ArrayListUnmanaged(ToolCall){};

    while (try req.readChunk()) |chunk| {
        // Parse SSE data
        const data = parseSSE(chunk);
        
        if (data.content) |text| {
            on_chunk(text);  // Stream to user
            try content.appendSlice(self.allocator, text);
        }
        
        if (data.tool_call) |tc| {
            try tool_calls.append(self.allocator, tc);
        }
    }

    return ChatResponse{
        .content = content.toOwnedSlice(self.allocator),
        .tool_calls = if (tool_calls.items.len > 0) 
            tool_calls.toOwnedSlice(self.allocator) 
        else 
            null,
        .allocator = self.allocator,
    };
}
```

### SSE (Server-Sent Events) Parsing

```zig
fn parseSSELine(line: []const u8) ?[]const u8 {
    if (std.mem.startsWith(u8, line, "data: ")) {
        const data = line[6..];
        if (std.mem.eql(u8, data, "[DONE]")) return null;
        return data;
    }
    return null;
}

fn processStreamChunk(chunk: []const u8, buffer: *std.ArrayListUnmanaged(u8)) !?ParsedChunk {
    try buffer.appendSlice(allocator, chunk);
    
    // Find complete lines
    while (std.mem.indexOf(u8, buffer.items, "\n")) |idx| {
        const line = buffer.items[0..idx];
        // Remove processed line from buffer
        std.mem.copyForwards(u8, buffer.items, buffer.items[idx + 1 ..]);
        buffer.shrinkRetainingCapacity(buffer.items.len - idx - 1);
        
        if (parseSSELine(line)) |json_data| {
            return try parseChunkJson(json_data);
        }
    }
    return null;
}
```

## Session Persistence

### JSON-based Session Storage

```zig
const SESSION_DIR = ".sessions";

pub fn save(allocator: std.mem.Allocator, session_id: []const u8, messages: []Message) !void {
    // Ensure directory exists
    std.fs.cwd().makeDir(SESSION_DIR) catch |err| switch (err) {
        error.PathAlreadyExists => {},
        else => return err,
    };

    const path = try std.fmt.allocPrint(allocator, "{s}/{s}.json", .{ SESSION_DIR, session_id });
    defer allocator.free(path);

    const file = try std.fs.cwd().createFile(path, .{});
    defer file.close();

    var out = std.io.Writer.Allocating.init(allocator);
    defer out.deinit();

    try std.json.Stringify.value(messages, .{}, &out.writer);
    try file.writeAll(out.written());
}

pub fn load(allocator: std.mem.Allocator, session_id: []const u8) ![]Message {
    const path = try std.fmt.allocPrint(allocator, "{s}/{s}.json", .{ SESSION_DIR, session_id });
    defer allocator.free(path);

    const file = try std.fs.cwd().openFile(path, .{});
    defer file.close();

    const content = try file.readToEndAlloc(allocator, 10 * 1024 * 1024);
    defer allocator.free(content);

    const parsed = try std.json.parseFromSlice([]Message, allocator, content, .{
        .ignore_unknown_fields = true,
    });
    // Note: caller must free parsed.value elements
    return parsed.value;
}
```

## Agent Implementation

### Full Agent Structure

```zig
pub const Agent = struct {
    config: Config,
    allocator: std.mem.Allocator,
    ctx: Context,
    registry: ToolRegistry,
    session_id: []const u8,

    pub fn init(allocator: std.mem.Allocator, config: Config, session_id: []const u8) Agent {
        var self = Agent{
            .config = config,
            .allocator = allocator,
            .ctx = Context.init(allocator),
            .registry = ToolRegistry.init(allocator),
            .session_id = session_id,
        };

        // Load existing session
        if (session.load(allocator, session_id)) |history| {
            for (history) |msg| {
                self.ctx.add_message(msg) catch {};
            }
            // Free loaded history (context dupes it)
            freeLoadedHistory(allocator, history);
        } else |_| {}

        // Register tools
        self.registerDefaultTools();
        
        return self;
    }

    pub fn deinit(self: *Agent) void {
        self.ctx.deinit();
        self.registry.deinit();
    }

    pub fn run(self: *Agent, message: []const u8) !void {
        try self.ctx.add_message(.{ .role = "user", .content = message });

        var provider = Provider.init(self.allocator, self.config.api_key);
        defer provider.deinit();

        const tool_ctx = ToolContext{
            .allocator = self.allocator,
            .config = self.config,
        };

        var iterations: usize = 0;
        const max_iterations = 5;

        while (iterations < max_iterations) : (iterations += 1) {
            var response = try provider.chatStream(
                self.ctx.get_messages(),
                self.config.model,
                printChunk,
            );
            defer response.deinit();

            try self.ctx.add_message(.{
                .role = "assistant",
                .content = response.content,
                .tool_calls = response.tool_calls,
            });

            if (response.tool_calls) |calls| {
                for (calls) |call| {
                    const result = try self.executeToolCall(tool_ctx, call);
                    defer self.allocator.free(result);

                    try self.ctx.add_message(.{
                        .role = "tool",
                        .content = result,
                        .tool_call_id = call.id,
                    });
                }
                continue;
            }

            break;
        }

        try session.save(self.allocator, self.session_id, self.ctx.get_messages());
    }

    fn executeToolCall(self: *Agent, ctx: ToolContext, call: ToolCall) ![]const u8 {
        if (self.registry.get(call.function_name)) |tool| {
            return tool.execute(ctx, call.arguments);
        }
        return std.fmt.allocPrint(self.allocator, "Error: Tool {s} not found", .{call.function_name});
    }

    fn registerDefaultTools(self: *Agent) void {
        self.registry.register(.{
            .name = "list_files",
            .description = "List files in the current directory",
            .parameters = "{}",
            .execute = tools.list_files,
        }) catch {};
        // ... more tools
    }
};
```

## Error Handling

### Tool Error Patterns

Tools should catch errors and return user-friendly messages:

```zig
pub fn safe_tool(ctx: ToolContext, arguments: []const u8) ![]const u8 {
    const result = innerOperation(ctx, arguments) catch |err| {
        return try std.fmt.allocPrint(ctx.allocator, "Error: {any}", .{err});
    };
    return result;
}
```

### Agent Error Recovery

```zig
if (self.registry.get(call.function_name)) |tool| {
    const result = tool.execute(tool_ctx, call.arguments) catch |err| {
        return try std.fmt.allocPrint(self.allocator, "Tool error: {any}", .{err});
    };
    // ...
} else {
    const error_msg = try std.fmt.allocPrint(
        self.allocator, 
        "Error: Tool {s} not found", 
        .{call.function_name}
    );
    try self.ctx.add_message(.{
        .role = "tool",
        .content = error_msg,
        .tool_call_id = call.id,
    });
}
```

## Testing Agents

### Unit Testing Tools

```zig
const std = @import("std");
const testing = std.testing;

test "list_files returns files" {
    const allocator = testing.allocator;
    const ctx = ToolContext{
        .allocator = allocator,
        .config = Config.default(),
    };

    const result = try list_files(ctx, "{}");
    defer allocator.free(result);

    try testing.expect(result.len > 0);
}
```

### Mocking Providers

```zig
const MockProvider = struct {
    responses: []const []const u8,
    call_count: usize = 0,

    pub fn chatStream(
        self: *MockProvider,
        messages: []const Message,
        model: []const u8,
        on_chunk: fn ([]const u8) void,
    ) !ChatResponse {
        _ = messages;
        _ = model;
        
        const response = self.responses[self.call_count];
        self.call_count += 1;
        
        on_chunk(response);
        
        return ChatResponse{
            .content = response,
            .tool_calls = null,
            .allocator = undefined,  // Mock doesn't need cleanup
        };
    }
};
```

## Best Practices

### 1. Memory Management

- **Always use `defer` for cleanup** immediately after acquisition
- **Use `errdefer` for error-path cleanup**
- **Pass allocators explicitly** to all functions
- **Duplicate strings** when storing in context to ensure ownership

### 2. Tool Design

- **Single responsibility**: One tool, one purpose
- **Clear JSON schemas**: Document all parameters
- **Graceful error handling**: Return error messages, don't crash
- **Limit output size**: Cap response length for LLM consumption

### 3. Context Management

- **Duplicate all strings**: Context owns its data
- **Free on deinit**: Clean up all allocations
- **Session isolation**: Each session is independent

### 4. Streaming

- **Buffer incomplete chunks**: SSE lines may span chunks
- **Handle [DONE] signal**: Mark end of stream
- **Immediate output**: Call on_chunk for real-time display

### 5. Provider Abstraction

- **Interface consistency**: Same signature for all providers
- **Error translation**: Convert HTTP errors to domain errors
- **Timeout handling**: Set reasonable request timeouts

## See Also

- [zig-best-practices/SKILL.md](../zig-best-practices/SKILL.md) - Core Zig patterns
- [zig-best-practices/DEBUGGING.md](../zig-best-practices/DEBUGGING.md) - Memory debugging
