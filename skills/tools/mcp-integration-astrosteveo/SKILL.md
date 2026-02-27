---
name: mcp-integration
description: This skill should be used when the user asks to "add MCP server", "integrate MCP", "configure MCP in plugin", "use .mcp.json", "set up Model Context Protocol", "connect external service", mentions "${CLAUDE_PLUGIN_ROOT} with MCP", or discusses MCP server types (SSE, stdio, HTTP, WebSocket). Provides comprehensive guidance for integrating Model Context Protocol servers into Claude Code plugins for external tool and service integration.
---

# MCP Integration for Claude Code Plugins

## Overview

Model Context Protocol (MCP) enables Claude Code plugins to integrate with external services and APIs by providing structured tool access. Use MCP integration to expose external service capabilities as tools within Claude Code.

**Key capabilities:**
- Connect to external services (databases, APIs, file systems)
- Provide 10+ related tools from a single service
- Handle OAuth and complex authentication flows
- Bundle MCP servers with plugins for automatic setup

> **Transport Status (2026):** HTTP (streamable) is the recommended remote transport. SSE is being phased out in favor of HTTP. stdio remains the standard for local servers.

## MCP Server Configuration Methods

Plugins can bundle MCP servers in two ways:

### Method 1: Dedicated .mcp.json (Recommended)

Create `.mcp.json` at plugin root:

```json
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

**Benefits:**
- Clear separation of concerns
- Easier to maintain
- Better for multiple servers

### Method 2: Inline in plugin.json

Add `mcpServers` field to plugin.json:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

**Benefits:**
- Single configuration file
- Good for simple single-server plugins

## MCP Server Types

### stdio (Local Process)

Execute local MCP servers as child processes. Best for local tools and custom servers.

**Configuration:**
```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"],
    "env": {
      "LOG_LEVEL": "debug"
    }
  }
}
```

**Use cases:**
- File system access
- Local database connections
- Custom MCP servers
- NPM-packaged MCP servers

**Process management:**
- Claude Code spawns and manages the process
- Communicates via stdin/stdout
- Terminates when Claude Code exits

### HTTP (REST API)

> **Recommended** for remote MCP servers. HTTP (streamable) is the primary remote transport.

Connect to RESTful MCP servers with token or OAuth authentication.

**Configuration:**
```json
{
  "api-service": {
    "type": "http",
    "url": "https://api.example.com/mcp",
    "headers": {
      "Authorization": "Bearer ${API_TOKEN}"
    }
  }
}
```

HTTP now supports OAuth flows in addition to token auth.

**Use cases:**
- REST API-based MCP servers
- Token-based or OAuth authentication
- Custom API backends
- Stateless interactions

### SSE (Server-Sent Events)

> **Note:** SSE transport is being phased out. For new remote integrations, prefer HTTP (streamable) transport. Existing SSE configurations continue to work.

Connect to hosted MCP servers with OAuth support. Best for cloud services.

**Configuration:**
```json
{
  "asana": {
    "type": "sse",
    "url": "https://mcp.asana.com/sse"
  }
}
```

**Use cases:**
- Official hosted MCP servers (Asana, GitHub, etc.)
- Cloud services with MCP endpoints
- OAuth-based authentication
- No local installation needed

**Authentication:**
- OAuth flows handled automatically
- User prompted on first use
- Tokens managed by Claude Code

### WebSocket (Real-time)

Connect to WebSocket MCP servers for real-time bidirectional communication.

**Configuration:**
```json
{
  "realtime-service": {
    "type": "ws",
    "url": "wss://mcp.example.com/ws",
    "headers": {
      "Authorization": "Bearer ${TOKEN}"
    }
  }
}
```

**Use cases:**
- Real-time data streaming
- Persistent connections
- Push notifications from server
- Low-latency requirements

## Environment Variable Expansion

All MCP configurations support environment variable substitution:

**${CLAUDE_PLUGIN_ROOT}** - Plugin directory (always use for portability):
```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/servers/my-server"
}
```

**User environment variables** - From user's shell:
```json
{
  "env": {
    "API_KEY": "${MY_API_KEY}",
    "DATABASE_URL": "${DB_URL}"
  }
}
```

**Best practice:** Document all required environment variables in plugin README.

## MCP Tool Naming

When MCP servers provide tools, they're automatically prefixed:

**Format:** `mcp__plugin_<plugin-name>_<server-name>__<tool-name>`

**Example:**
- Plugin: `asana`
- Server: `asana`
- Tool: `create_task`
- **Full name:** `mcp__plugin_asana_asana__asana_create_task`

### Using MCP Tools in Commands

Pre-allow specific MCP tools in command frontmatter:

```markdown
---
allowed-tools: [
  "mcp__plugin_asana_asana__asana_create_task",
  "mcp__plugin_asana_asana__asana_search_tasks"
]
---
```

**Wildcard (use sparingly):**
```markdown
---
allowed-tools: ["mcp__plugin_asana_asana__*"]
---
```

**Best practice:** Pre-allow specific tools, not wildcards, for security.

## MCP Tool Search

When plugins provide many MCP tools (more than 10% of context window), Claude Code automatically enables tool search to find relevant tools efficiently.

**Auto-enabled:** When tool count exceeds threshold
**Manual control:** Set `ENABLE_TOOL_SEARCH=1` or `ENABLE_TOOL_SEARCH=0` environment variable

**Implications for plugin developers:**
- Write clear, descriptive tool names and descriptions
- Tools with better descriptions are found more reliably
- Large tool sets benefit from logical naming conventions

## MCP Prompts as Commands

MCP servers can expose prompts that appear as slash commands in Claude Code:

**Format:** `/mcp__servername__promptname`

**Example:** If server `mytools` exposes a prompt `analyze-code`:
```
/mcp__mytools__analyze-code
```

**For plugin developers:** If your MCP server exposes prompts, document the available prompt commands in your README.

## MCP Resources

MCP servers can expose resources accessible via @ syntax:

**Format:** `@server:protocol://resource/path`

**Example:**
```
@mydb:postgres://tables/users
@docs:file://api-reference.md
```

**For plugin developers:** Document available resources and their URI formats in your README.

## Dynamic Tool Updates

MCP servers can notify Claude Code when their tool list changes via `list_changed` notifications. This allows tools to be added or removed without restarting Claude Code.

**For server developers:** Implement the `list_changed` notification in your MCP server to support dynamic tool registration.

## Lifecycle Management

**Automatic startup:**
- MCP servers start when plugin enables
- Connection established before first tool use
- Restart required for configuration changes

**Lifecycle:**
1. Plugin loads
2. MCP configuration parsed
3. Server process started (stdio) or connection established (SSE/HTTP/WS)
4. Tools discovered and registered
5. Tools available as `mcp__plugin_...__...`

**Viewing servers:**
Use `/mcp` command to see all servers including plugin-provided ones.

## Authentication Patterns

### OAuth (SSE/HTTP)

OAuth handled automatically by Claude Code:

```json
{
  "type": "sse",
  "url": "https://mcp.example.com/sse"
}
```

User authenticates in browser on first use. No additional configuration needed.

### Token-Based (Headers)

Static or environment variable tokens:

```json
{
  "type": "http",
  "url": "https://api.example.com",
  "headers": {
    "Authorization": "Bearer ${API_TOKEN}"
  }
}
```

Document required environment variables in README.

### OAuth CLI Flags

For custom OAuth configurations, use CLI flags when adding MCP servers:

```bash
claude mcp add my-server --transport http \
  --url https://api.example.com/mcp \
  --client-id "${CLIENT_ID}" \
  --client-secret "${CLIENT_SECRET}"
```

These flags set custom OAuth client credentials for MCP servers that don't use the default OAuth discovery.

### Environment Variables (stdio)

Pass configuration to MCP server:

```json
{
  "command": "python",
  "args": ["-m", "my_mcp_server"],
  "env": {
    "DATABASE_URL": "${DB_URL}",
    "API_KEY": "${API_KEY}",
    "LOG_LEVEL": "info"
  }
}
```

## Integration Patterns

### Pattern 1: Simple Tool Wrapper

Commands use MCP tools with user interaction:

```markdown
# Command: create-item.md
---
allowed-tools: ["mcp__plugin_name_server__create_item"]
---

Steps:
1. Gather item details from user
2. Use mcp__plugin_name_server__create_item
3. Confirm creation
```

**Use for:** Adding validation or preprocessing before MCP calls.

### Pattern 2: Autonomous Agent

Agents use MCP tools autonomously:

```markdown
# Agent: data-analyzer.md

Analysis Process:
1. Query data via mcp__plugin_db_server__query
2. Process and analyze results
3. Generate insights report
```

**Use for:** Multi-step MCP workflows without user interaction.

### Pattern 3: Multi-Server Plugin

Integrate multiple MCP servers:

```json
{
  "github": {
    "type": "sse",
    "url": "https://mcp.github.com/sse"
  },
  "jira": {
    "type": "sse",
    "url": "https://mcp.jira.com/sse"
  }
}
```

**Use for:** Workflows spanning multiple services.

## Security Best Practices

### Use HTTPS/WSS

Always use secure connections:

```json
✅ "url": "https://mcp.example.com/sse"
❌ "url": "http://mcp.example.com/sse"
```

### Token Management

**DO:**
- ✅ Use environment variables for tokens
- ✅ Document required env vars in README
- ✅ Let OAuth flow handle authentication

**DON'T:**
- ❌ Hardcode tokens in configuration
- ❌ Commit tokens to git
- ❌ Share tokens in documentation

### Permission Scoping

Pre-allow only necessary MCP tools:

```markdown
✅ allowed-tools: [
  "mcp__plugin_api_server__read_data",
  "mcp__plugin_api_server__create_item"
]

❌ allowed-tools: ["mcp__plugin_api_server__*"]
```

## Error Handling

### Connection Failures

Handle MCP server unavailability:
- Provide fallback behavior in commands
- Inform user of connection issues
- Check server URL and configuration

### Tool Call Errors

Handle failed MCP operations:
- Validate inputs before calling MCP tools
- Provide clear error messages
- Check rate limiting and quotas

### Configuration Errors

Validate MCP configuration:
- Test server connectivity during development
- Validate JSON syntax
- Check required environment variables

## Performance Considerations

### Lazy Loading

MCP servers connect on-demand:
- Not all servers connect at startup
- First tool use triggers connection
- Connection pooling managed automatically

### Batching

Batch similar requests when possible:

```
# Good: Single query with filters
tasks = search_tasks(project="X", assignee="me", limit=50)

# Avoid: Many individual queries
for id in task_ids:
    task = get_task(id)
```

## Testing MCP Integration

### Local Testing

1. Configure MCP server in `.mcp.json`
2. Install plugin locally (`.claude-plugin/`)
3. Run `/mcp` to verify server appears
4. Test tool calls in commands
5. Check `claude --debug` logs for connection issues

### Validation Checklist

- [ ] MCP configuration is valid JSON
- [ ] Server URL is correct and accessible
- [ ] Required environment variables documented
- [ ] Tools appear in `/mcp` output
- [ ] Authentication works (OAuth or tokens)
- [ ] Tool calls succeed from commands
- [ ] Error cases handled gracefully

## Debugging

### Enable Debug Logging

```bash
claude --debug
```

Look for:
- MCP server connection attempts
- Tool discovery logs
- Authentication flows
- Tool call errors

### Common Issues

**Server not connecting:**
- Check URL is correct
- Verify server is running (stdio)
- Check network connectivity
- Review authentication configuration

**Tools not available:**
- Verify server connected successfully
- Check tool names match exactly
- Run `/mcp` to see available tools
- Restart Claude Code after config changes

**Authentication failing:**
- Clear cached auth tokens
- Re-authenticate
- Check token scopes and permissions
- Verify environment variables set

## Exposing Claude Code as MCP Server

Use `claude mcp serve` to expose Claude Code itself as an MCP server, allowing other tools or editors to use Claude Code's capabilities:

```bash
claude mcp serve
```

This is useful for integrating Claude Code into existing MCP-compatible workflows.

## Importing from Claude Desktop

Import MCP server configurations from Claude Desktop:

```bash
claude mcp add-from-claude-desktop
```

This copies MCP server configurations from Claude Desktop to Claude Code, avoiding duplicate setup.

## Enterprise and Managed MCP

Organizations can manage MCP server configurations centrally:

**Managed configuration:** `managed-mcp.json` deployed by IT
**Allow/deny lists:**
- `allowedMcpServers` — Only these servers can be used
- `deniedMcpServers` — These servers are blocked

**Plugin developer guidance:**
- Document enterprise compatibility in your README
- Provide server names that IT teams can add to allow lists
- Support configuration via environment variables for enterprise environments
- Handle gracefully when servers are blocked by managed policy

## MCP Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MAX_MCP_OUTPUT_TOKENS` | 25,000 | Maximum tokens in MCP tool output |
| `MCP_TIMEOUT` | 60s | Connection timeout for MCP servers |
| `ENABLE_TOOL_SEARCH` | auto | Force enable/disable tool search (1/0) |

**For plugin developers:** If your MCP server returns large outputs, document `MAX_MCP_OUTPUT_TOKENS` in your README.

## Quick Reference

### MCP Server Types

| Type | Transport | Best For | Auth | Status |
|------|-----------|----------|------|--------|
| stdio | Process | Local tools, custom servers | Env vars | Stable |
| HTTP | REST | Remote servers, APIs | OAuth/Tokens | Recommended |
| SSE | HTTP/SSE | Hosted services | OAuth | Phasing out |
| ws | WebSocket | Real-time, streaming | Tokens | Stable |

### Configuration Checklist

- [ ] Server type specified (stdio/SSE/HTTP/ws)
- [ ] Type-specific fields complete (command or url)
- [ ] Authentication configured
- [ ] Environment variables documented
- [ ] HTTPS/WSS used (not HTTP/WS)
- [ ] ${CLAUDE_PLUGIN_ROOT} used for paths

### Best Practices

**DO:**
- ✅ Use ${CLAUDE_PLUGIN_ROOT} for portable paths
- ✅ Document required environment variables
- ✅ Use secure connections (HTTPS/WSS)
- ✅ Pre-allow specific MCP tools in commands
- ✅ Test MCP integration before publishing
- ✅ Handle connection and tool errors gracefully

**DON'T:**
- ❌ Hardcode absolute paths
- ❌ Commit credentials to git
- ❌ Use HTTP instead of HTTPS
- ❌ Pre-allow all tools with wildcards
- ❌ Skip error handling
- ❌ Forget to document setup

## Additional Resources

### Reference Files

For detailed information, consult:

- **`references/server-types.md`** - Deep dive on each server type
- **`references/authentication.md`** - Authentication patterns and OAuth
- **`references/tool-usage.md`** - Using MCP tools in commands and agents
- **`references/enterprise-mcp.md`** - Enterprise MCP configuration guide

### Example Configurations

Working examples in `examples/`:

- **`stdio-server.json`** - Local stdio MCP server
- **`http-oauth-server.json`** - HTTP server with OAuth (recommended)
- **`sse-server.json`** - Hosted SSE server with OAuth (phasing out)
- **`http-server.json`** - REST API with token auth

### External Resources

- **Official MCP Docs**: https://modelcontextprotocol.io/
- **Claude Code MCP Docs**: https://docs.claude.com/en/docs/claude-code/mcp
- **MCP SDK**: @modelcontextprotocol/sdk
- **Testing**: Use `claude --debug` and `/mcp` command

## Implementation Workflow

To add MCP integration to a plugin:

1. Choose MCP server type (stdio, SSE, HTTP, ws)
2. Create `.mcp.json` at plugin root with configuration
3. Use ${CLAUDE_PLUGIN_ROOT} for all file references
4. Document required environment variables in README
5. Test locally with `/mcp` command
6. Pre-allow MCP tools in relevant commands
7. Handle authentication (OAuth or tokens)
8. Test error cases (connection failures, auth errors)
9. Document MCP integration in plugin README

Focus on stdio for custom/local servers, HTTP for remote services (preferred over SSE).
