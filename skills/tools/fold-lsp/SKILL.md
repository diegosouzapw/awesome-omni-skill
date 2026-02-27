---
name: fold-lsp
description: This skill should be used when working with Scheme files in The Fold codebase. Use it to "get type information", "find definitions", "check for errors", "understand Scheme code", or when editing .ss files. Provides LSP-powered code intelligence for Scheme development.
version: 0.1.0
---

# Fold LSP - Scheme Code Intelligence

## Overview

The Fold includes an LSP (Language Server Protocol) server that provides rich code intelligence for Scheme files. The MCP server exposes these capabilities as tools that can be used to understand and navigate Scheme code.

Use LSP tools for:
- Getting type information and documentation for symbols
- Finding where functions are defined
- Finding all usages of a function or variable
- Checking files for syntax errors and warnings
- Searching for symbols by name

## Available Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `fold_lsp_hover` | Get type info at position | Before modifying a function to understand its signature |
| `fold_lsp_definition` | Go to symbol definition | When needing to read or modify a function's implementation |
| `fold_lsp_references` | Find all usages | Before refactoring to understand impact |
| `fold_lsp_symbols` | Search by name | When looking for a function but unsure of exact location |
| `fold_lsp_diagnostics` | Check for errors | After editing to verify code correctness |
| `fold_lsp_format` | Format code | When code needs consistent formatting |
| `fold_lsp_lookup` | Combined lookup | One-stop info gathering (hover + def + refs) |
| `fold_lsp_status` | Check LSP status | Troubleshooting LSP issues |

## Usage Patterns

### When Editing Scheme Files

1. **Before modifying a function**, use `fold_lsp_hover` to understand:
   - The function's type signature
   - What parameters it expects
   - What it returns

2. **After editing**, use `fold_lsp_diagnostics` to:
   - Verify no syntax errors were introduced
   - Check for unbalanced parentheses
   - Identify any type issues

### When Understanding Code

Use `fold_lsp_lookup` for comprehensive symbol analysis:
- Combines type info, definition location, and all references
- More efficient than calling each tool separately
- Useful for understanding a symbol's role in the codebase

### When Refactoring

1. Use `fold_lsp_references` to find all usages of a symbol
2. Review each usage to understand the impact
3. Make changes incrementally
4. Use `fold_lsp_diagnostics` after each change

### When Searching

Use `fold_lsp_symbols` when:
- Looking for a function by partial name
- Exploring available functions in a module
- Finding where a concept is implemented

## Position Parameters

LSP tools use 0-indexed positions:
- `line`: Line number starting from 0
- `character`: Column position starting from 0

To find the position of a symbol in a file:
1. Read the file content
2. Count lines (0-indexed)
3. Count characters from line start (0-indexed)

## Examples

### Get Type Information

To understand what `vec3-add` does:

```
fold_lsp_hover(file: "lattice/linalg/vec.ss", line: 42, character: 8)
```

Returns type signature and documentation if available.

### Find Definition

To navigate to where `matrix-multiply` is defined:

```
fold_lsp_definition(file: "lattice/linalg/matrix.ss", line: 100, character: 12)
```

Returns file path and line number of definition.

### Check for Errors

After editing a file, verify it's correct:

```
fold_lsp_diagnostics(file: "lattice/linalg/vec.ss")
```

Returns list of errors/warnings or "File looks clean!"

### Search for Symbols

Find all functions related to "parse":

```
fold_lsp_symbols(query: "parse")
```

Returns matching function names with their locations.

## Best Practices

1. **Use proactively**: Check types before modifying unfamiliar code
2. **Verify edits**: Run diagnostics after any .ss file changes
3. **Understand impact**: Check references before renaming or removing
4. **Combine tools**: Use lookup for comprehensive understanding
5. **Trust but verify**: LSP type inference is best-effort, not guaranteed

## Integration with Other Tools

LSP tools complement other The Fold tools:

- **With lattice search** (`lf`, `li`, `le`): LSP provides position-specific info, lattice provides module-level discovery
- **With REPL** (`fold_eval`): LSP shows types, REPL executes code
- **With Read tool**: LSP shows semantics, Read shows raw content

## Troubleshooting

**LSP not responding:**
1. Check status with `fold_lsp_status`
2. The LSP starts automatically on first use
3. If issues persist, restart the MCP server

**No results:**
- Ensure file path is correct (absolute or relative to project root)
- Position must be on a symbol, not whitespace
- Some symbols may not have type information

**Wrong results:**
- LSP uses heuristic type inference, not formal type checking
- Built-in primitives have predefined types
- User-defined functions get inferred types when possible
