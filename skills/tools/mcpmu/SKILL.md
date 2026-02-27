---
name: mcpmu
description: Install, set up, and manage MCP servers using the mcpmu CLI. Use when the user wants to install mcpmu, register it as an MCP server in Claude Code, add/remove/list MCP servers, manage namespaces, set tool permissions, or expose servers via serve mode.
allowed-tools: Bash(mcpmu *), Bash(brew *), Bash(go install *), Bash(claude mcp *), Bash(which mcpmu), Bash(command -v mcpmu)
---

# mcpmu — MCP Server Manager

mcpmu is a multiplexing MCP server manager. You configure MCP servers once in mcpmu, then expose them as a single unified MCP endpoint to any agent (Claude Code, Codex, Cursor, Windsurf, etc.).

## Installing mcpmu

First check if mcpmu is already installed:
```bash
which mcpmu
```

If not installed, install via Homebrew (preferred) or Go:

**Homebrew (macOS/Linux):**
```bash
brew tap Bigsy/tap && brew install mcpmu
```

**From source (requires Go):**
```bash
go install github.com/Bigsy/mcpmu/cmd/mcpmu@latest
```

## Registering mcpmu as an MCP Server in Claude Code

After installing mcpmu, register it so Claude Code can use all mcpmu-managed servers through a single endpoint:

**Default (all servers, default namespace):**
```bash
claude mcp add mcpmu -- mcpmu serve --stdio
```

**Specific namespace:**
```bash
claude mcp add work -- mcpmu serve --stdio --namespace work
```

**With management tools (lets Claude add/remove servers via MCP):**
```bash
claude mcp add mcpmu -- mcpmu serve --stdio --expose-manager-tools
```

You can verify the registration:
```bash
claude mcp list
```

To remove mcpmu from Claude Code:
```bash
claude mcp remove mcpmu
```

### Scoped registration

Claude Code supports different scopes for MCP server registration:

```bash
claude mcp add mcpmu --scope user -- mcpmu serve --stdio       # available in all projects
claude mcp add mcpmu --scope project -- mcpmu serve --stdio    # this project only
```

## Full Setup Walkthrough

To go from zero to a working mcpmu setup in Claude Code:

1. Install mcpmu: `brew tap Bigsy/tap && brew install mcpmu`
2. Add some MCP servers: `mcpmu add context7 -- npx -y @upstash/context7-mcp`
3. Optionally create a namespace: `mcpmu namespace add work --description "Work tools"`
4. Assign servers to it: `mcpmu namespace assign work context7`
5. Register with Claude Code: `claude mcp add mcpmu -- mcpmu serve --stdio`
6. Restart Claude Code — all mcpmu-managed tools are now available

## Adding Servers

### Stdio servers (local processes)
```bash
mcpmu add <name> -- <command> [args...]
```

Examples:
```bash
mcpmu add context7 -- npx -y @upstash/context7-mcp
mcpmu add filesystem -- npx -y @modelcontextprotocol/server-filesystem /tmp
mcpmu add my-server --env FOO=bar --cwd /path -- ./server --flag
mcpmu add auto-server --autostart -- ./server  # start on app launch
```

### HTTP servers (remote endpoints)
```bash
mcpmu add <name> <url> [flags]
```

Examples:
```bash
mcpmu add atlassian https://mcp.atlassian.com/mcp --scopes read,write
mcpmu add figma https://mcp.figma.com/mcp --bearer-env FIGMA_TOKEN
```

### OAuth login (for HTTP servers that need it)
```bash
mcpmu mcp login <server>
mcpmu mcp login atlassian --scopes read,write
mcpmu mcp logout <server>
```

## Listing and Managing Servers

```bash
mcpmu list              # human-readable list
mcpmu list --json       # JSON output
mcpmu remove <name>     # remove (prompts for confirmation)
mcpmu remove <name> --yes  # skip confirmation
mcpmu rename <old> <new>   # rename (updates namespace/permission refs)
```

## Namespaces

Namespaces group servers into profiles — e.g. work, personal, minimal. The `namespace` subcommand can also be shortened to `ns`.

```bash
mcpmu namespace add <name> --description "desc"
mcpmu namespace list [--json]
mcpmu namespace remove <name> [--yes]
mcpmu namespace assign <namespace> <server>
mcpmu namespace unassign <namespace> <server>
mcpmu namespace default <name>
mcpmu namespace rename <old> <new>
mcpmu namespace set-deny-default <namespace> <true|false>
```

### Common namespace patterns

Create separate profiles:
```bash
mcpmu namespace add work --description "Work servers"
mcpmu namespace add personal --description "Personal projects"
mcpmu namespace assign work atlassian
mcpmu namespace assign work context7
mcpmu namespace assign personal context7
```

Create a minimal namespace that denies all tools by default, then allowlist:
```bash
mcpmu namespace add minimal --description "Lean toolset"
mcpmu namespace set-deny-default minimal true
mcpmu permission set minimal context7 resolve allow
```

## Tool Permissions

Control which tools each server exposes per namespace:

```bash
mcpmu permission list <namespace> [--json]
mcpmu permission set <namespace> <server> <tool> <allow|deny>
mcpmu permission unset <namespace> <server> <tool>
```

Examples:
```bash
mcpmu permission set work atlassian jira_search allow
mcpmu permission set work atlassian confluence_delete deny
```

## Serve Mode

Expose managed servers as a single MCP endpoint:

```bash
mcpmu serve --stdio                          # default namespace
mcpmu serve --stdio --namespace work         # specific namespace
mcpmu serve --stdio -n work --eager          # pre-start all servers
mcpmu serve --stdio --expose-manager-tools   # include mcpmu.* management tools
mcpmu serve --stdio --log-level debug        # verbose logging
```

Flags:
- `-n, --namespace` — namespace to expose
- `--eager` — pre-start all servers (default: lazy/on-demand)
- `--expose-manager-tools` — include mcpmu.* tools in tools/list
- `-l, --log-level` — debug, info, warn, error

## Connecting mcpmu to Other Agents

For Claude Code registration, see "Registering mcpmu as an MCP Server in Claude Code" above.

For other agents:

**Codex:**
```bash
codex mcp add mcpmu -- mcpmu serve --stdio
```

**Any MCP config JSON (Cursor, Windsurf, etc.):**
```json
{
  "mcpmu": {
    "command": "mcpmu",
    "args": ["serve", "--stdio"]
  }
}
```

For a namespace-specific entry:
```json
{
  "work": {
    "command": "mcpmu",
    "args": ["serve", "--stdio", "--namespace", "work"]
  }
}
```

## Interactive TUI

Run `mcpmu` with no arguments to open the terminal UI for visual server management, log monitoring, and namespace switching.

## Config

Config lives at `~/.config/mcpmu/config.json`. All commands support `--config` / `-c` to use a custom config path.

## Shell Completions

```bash
# zsh (Homebrew)
mcpmu completion zsh > "$(brew --prefix)/share/zsh/site-functions/_mcpmu"

# bash
mcpmu completion bash > /etc/bash_completion.d/mcpmu

# fish
mcpmu completion fish > ~/.config/fish/completions/mcpmu.fish
```