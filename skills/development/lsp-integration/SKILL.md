---
name: lsp-integration
description: This skill should be used when the user asks to "add LSP server", "configure language server", "set up LSP in plugin", "add code intelligence", "integrate language server protocol", "use pyright-lsp", "use typescript-lsp", "use rust-lsp", "socket transport", "initializationOptions", mentions LSP servers, or discusses extensionToLanguage mappings. Provides guidance for integrating Language Server Protocol servers into Claude Code plugins for enhanced code intelligence.
---

# LSP Integration for Claude Code Plugins

## Overview

Language Server Protocol (LSP) servers provide code intelligence features like go-to-definition, find references, and hover information. Claude Code plugins can bundle or configure LSP servers to enhance Claude's understanding of code.

**Key capabilities:**

- Enable go-to-definition for code navigation
- Find all references to symbols
- Get hover information and documentation
- Support language-specific features (completions, diagnostics)

## LSP Server Configuration

Plugins can provide LSP servers in the plugin manifest:

### Basic Configuration

```json
{
  "name": "my-plugin",
  "lspServers": {
    "python": {
      "command": "pyright-langserver",
      "args": ["--stdio"],
      "extensionToLanguage": {
        ".py": "python",
        ".pyi": "python"
      }
    }
  }
}
```

### Separate File Configuration

LSP servers can also be configured in a separate `.lsp.json` file at the plugin root:

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

Reference this file in `plugin.json`:

```json
{
  "name": "my-plugin",
  "lspServers": "./.lsp.json"
}
```

### Configuration Fields

**command** (required): The LSP server executable

**args** (optional): Command-line arguments for the server

**extensionToLanguage** (required): Maps file extensions to language IDs

```json
{
  "extensionToLanguage": {
    ".py": "python",
    ".pyi": "python",
    ".pyw": "python"
  }
}
```

**env** (optional): Environment variables for the server process

```json
{
  "env": {
    "PYTHONPATH": "${CLAUDE_PLUGIN_ROOT}/lib"
  }
}
```

**transport** (optional): Communication transport - `stdio` (default) or `socket`

```json
{
  "lspServers": {
    "dart": {
      "transport": "socket",
      "command": "dart",
      "args": ["language-server", "--port", "8123"],
      "extensionToLanguage": { ".dart": "dart" }
    }
  }
}
```

Socket transport connects to the server via TCP port instead of stdin/stdout.

**initializationOptions** (optional): Options passed to the server during LSP initialization

```json
{
  "initializationOptions": {
    "typescript": {
      "tsdk": "./node_modules/typescript/lib"
    },
    "diagnostics": true,
    "formatting": { "tabSize": 2 }
  }
}
```

**settings** (optional): Settings passed via `workspace/didChangeConfiguration`

**workspaceFolder** (optional): Workspace folder path for the server

**startupTimeout** (optional): Maximum time to wait for server startup in milliseconds

**shutdownTimeout** (optional): Maximum time to wait for graceful shutdown in milliseconds

**restartOnCrash** (optional): Whether to automatically restart the server if it crashes

**maxRestarts** (optional): Maximum number of restart attempts before giving up

## What Claude Gains from LSP

When an LSP plugin is installed and its language server binary is available, Claude gains two key capabilities:

### Automatic Diagnostics

After every file edit Claude makes, the language server analyzes the changes and reports errors and warnings back automatically. Claude sees type errors, missing imports, and syntax issues without needing to run a compiler or linter. If Claude introduces an error, it notices and fixes the issue in the same turn.

### Code Navigation

Claude can use the language server to:

- Jump to definitions
- Find all references to a symbol
- Get type information on hover
- List symbols in a file
- Find implementations of interfaces
- Trace call hierarchies

These operations give Claude more precise navigation than grep-based search.

## Pre-built LSP Plugins

Claude Code provides official LSP plugins for common languages. Install from the marketplace:

| Language   | Plugin              | Binary Required              |
| ---------- | ------------------- | ---------------------------- |
| C/C++      | `clangd-lsp`        | `clangd`                     |
| C#         | `csharp-lsp`        | `csharp-ls`                  |
| Go         | `gopls-lsp`         | `gopls`                      |
| Java       | `jdtls-lsp`         | `jdtls`                      |
| Kotlin     | `kotlin-lsp`        | `kotlin-language-server`     |
| Lua        | `lua-lsp`           | `lua-language-server`        |
| PHP        | `php-lsp`           | `intelephense`               |
| Python     | `pyright-lsp`       | `pyright-langserver`         |
| Rust       | `rust-analyzer-lsp` | `rust-analyzer`              |
| Swift      | `swift-lsp`         | `sourcekit-lsp`              |
| TypeScript | `typescript-lsp`    | `typescript-language-server` |

Install the language server binary first, then install the plugin:

```bash
# Example: Python
pip install pyright  # or: npm install -g pyright
claude /install-plugin pyright-lsp
```

**Troubleshooting**: If you see `Executable not found in $PATH` in the `/plugin` Errors tab, install the required binary from the table above.

## Creating Custom LSP Integration

### Step 1: Choose or Build LSP Server

Options:

1. **Use existing LSP server** - Most languages have official or community servers
2. **Bundle with plugin** - Include server binary in plugin
3. **Require user installation** - Document server installation in README

### Step 2: Configure in plugin.json

```json
{
  "name": "go-lsp",
  "version": "1.0.0",
  "description": "Go language server integration",
  "lspServers": {
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {
        ".go": "go",
        ".mod": "go.mod"
      }
    }
  }
}
```

### Step 3: Bundle Server (Optional)

For self-contained plugins, bundle the server:

```
my-lsp-plugin/
├── .claude-plugin/
│   └── plugin.json
└── servers/
    └── my-lsp-server
```

Use `${CLAUDE_PLUGIN_ROOT}` for the command path:

```json
{
  "lspServers": {
    "mylang": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/my-lsp-server",
      "args": ["--stdio"]
    }
  }
}
```

### Step 4: Document Requirements

In your plugin README:

- List required external dependencies
- Provide installation instructions
- Note supported language versions
- Describe available features

## Extension to Language Mapping

The `extensionToLanguage` field maps file extensions to LSP language identifiers:

### Common Mappings

```json
{
  "extensionToLanguage": {
    ".py": "python",
    ".js": "javascript",
    ".ts": "typescript",
    ".jsx": "javascriptreact",
    ".tsx": "typescriptreact",
    ".rs": "rust",
    ".go": "go",
    ".java": "java",
    ".rb": "ruby",
    ".php": "php",
    ".c": "c",
    ".cpp": "cpp",
    ".h": "c",
    ".hpp": "cpp",
    ".cs": "csharp"
  }
}
```

### Multiple Extensions

A single language can have multiple extensions:

```json
{
  "extensionToLanguage": {
    ".ts": "typescript",
    ".mts": "typescript",
    ".cts": "typescript",
    ".d.ts": "typescript"
  }
}
```

## LSP Server Lifecycle

### Startup

LSP servers start automatically when:

1. Claude Code session begins
2. Plugin with LSP server is enabled
3. User opens a file matching configured extensions

### Communication

- Uses stdio for client-server communication
- Follows LSP specification for messages
- Claude Code manages the connection

### Shutdown

Servers terminate when:

- Claude Code session ends
- Plugin is disabled
- Server crashes (auto-restart may occur)

## Best Practices

### Performance

1. **Lazy initialization** - Servers start when needed, not at session start
2. **Minimal configuration** - Only enable features you need
3. **Resource limits** - Consider memory/CPU impact of servers

### Compatibility

1. **Check LSP version** - Ensure server supports required protocol version
2. **Test cross-platform** - Verify on macOS, Linux, Windows
3. **Handle missing servers** - Gracefully degrade if server not installed

### Documentation

1. **List prerequisites** - External tools, versions required
2. **Provide setup guide** - Step-by-step installation
3. **Document features** - Which LSP capabilities are supported

## Troubleshooting

### Server Not Starting

**Check:**

- Command path is correct
- Server is installed and executable
- Required dependencies are available
- `${CLAUDE_PLUGIN_ROOT}` is used for bundled servers

### No Code Intelligence

**Check:**

- File extension matches `extensionToLanguage` mapping
- Language ID is correct for the server
- Server supports the requested feature

### Debug Mode

Enable debug logging:

```bash
claude --debug
```

Look for:

- LSP server startup messages
- Communication logs
- Error responses

## Quick Reference

### Minimal LSP Configuration

```json
{
  "lspServers": {
    "language": {
      "command": "server-command",
      "extensionToLanguage": {
        ".ext": "language-id"
      }
    }
  }
}
```

### Full LSP Configuration

```json
{
  "lspServers": {
    "language": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/lsp-server",
      "args": ["--stdio", "--log-level", "warn"],
      "extensionToLanguage": {
        ".ext1": "language",
        ".ext2": "language"
      },
      "env": {
        "CONFIG_PATH": "${CLAUDE_PLUGIN_ROOT}/config"
      },
      "transport": "stdio",
      "initializationOptions": {},
      "settings": {},
      "workspaceFolder": ".",
      "startupTimeout": 10000,
      "shutdownTimeout": 5000,
      "restartOnCrash": true,
      "maxRestarts": 3
    }
  }
}
```

### Best Practices Summary

**DO:**

- Use `${CLAUDE_PLUGIN_ROOT}` for bundled server paths
- Map all relevant file extensions
- Document external dependencies
- Test on multiple platforms
- Handle server unavailability gracefully

**DON'T:**

- Hardcode absolute paths
- Assume servers are pre-installed
- Bundle large binaries without consideration
- Ignore server startup errors

## Additional Resources

### Reference Files

For detailed information, consult:

- **`references/popular-lsp-servers.md`** - Curated list of LSP servers by language with installation commands
- **`references/lsp-capabilities.md`** - LSP protocol capabilities and what they enable

### Examples

- **`examples/minimal-lsp-plugin/`** - Complete directory structure for a minimal LSP plugin
- **`examples/lsp-json-configs.md`** - Various `.lsp.json` configuration patterns

### External Resources

- **LSP Specification**: <https://microsoft.github.io/language-server-protocol/>
- **Claude Code Plugins Reference**: <https://docs.anthropic.com/en/docs/claude-code/plugins-reference>
- **Language Server List**: <https://langserver.org/>
