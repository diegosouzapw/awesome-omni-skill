---
name: mcp-add
description: Add MCP (Model Context Protocol) servers to AI coding clients using npx. Use when the user wants to configure, add, or set up an MCP server for Claude Code, Claude Desktop, Cursor, VS Code, Windsurf, Continue, Goose, Codex, OpenCode, Gemini CLI, or Copilot CLI.
---

# mcp-add

Add MCP servers to AI coding clients without installation.

## Command

```bash
npx mcp-add [flags]
```

Run without flags for interactive mode, or provide all required flags for non-interactive mode.

## Flags

| Flag | Description |
|------|-------------|
| `--name, -n` | Server name (identifier in config) |
| `--type, -t` | Server type: `stdio`, `http`, or `sse` |
| `--command, -c` | Command to run (stdio only), e.g., `"npx -y @modelcontextprotocol/server-filesystem"` |
| `--url, -u` | Server URL (http/sse only) |
| `--args, -a` | Arguments for stdio command (can repeat) |
| `--env, -e` | Environment variables as `KEY=value` (can repeat) |
| `--headers, -H` | HTTP headers as `Key: value` (can repeat, http/sse only) |
| `--scope, -s` | Config scope: `global` or `project` |
| `--clients` | Target clients (comma-separated or repeated) |

## Supported Clients

`claude desktop` `claude code` `copilot cli` `cursor` `continue` `windsurf` `opencode` `vscode` `goose` `codex` `gemini`

**Note:** Client names with spaces must be quoted: `--clients "claude code"`

## Examples

**Stdio server (filesystem access):**
```bash
npx mcp-add --name filesystem --type stdio \
  --command "npx -y @modelcontextprotocol/server-filesystem /path/to/dir" \
  --scope global --clients "claude code"
```

**Stdio server with separate args:**
```bash
npx mcp-add --name sqlite --type stdio \
  --command "npx -y @anthropic/mcp-server-sqlite" \
  --args "--db" --args "/path/to/db.sqlite" \
  --scope project --clients cursor,vscode
```

**Remote SSE server:**
```bash
npx mcp-add --name my-server --type sse \
  --url "http://localhost:3000/sse" \
  --scope global --clients "claude code"
```

**With environment variables:**
```bash
npx mcp-add --name github --type stdio \
  --command "npx -y @modelcontextprotocol/server-github" \
  --env "GITHUB_TOKEN=ghp_xxx" \
  --scope global --clients "claude code"
```

**Interactive mode (prompts for all options):**
```bash
npx mcp-add
```

## Notes

- Project scope creates config in `.cursor/`, `.vscode/`, etc. in current directory
- Global scope uses platform-specific config locations
- Claude Desktop only supports global scope
- Command strings are split by whitespace; first part becomes command, rest becomes args
