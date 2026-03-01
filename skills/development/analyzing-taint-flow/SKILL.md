---
name: analyzing-taint-flow
description: Tracks untrusted input propagation from sources to sinks in binary code to identify injection vulnerabilities. Use when analyzing data flow, tracing user input to dangerous functions, or detecting command/SQL injection.
---

# Taint Analysis

## Detection Workflow

1. **Identify sources**: Find recv, read, getenv, fgets, scanf, argv (input functions)
2. **Identify sinks**: Find system, popen, strcpy, sprintf, execve, malloc (dangerous functions)
3. **Find taint paths**: Use `xrefs_to` to trace from sources to sinks
4. **Analyze sanitization**: Check for input validation, length checks, character filtering, encoding/escaping
5. **Assess risk**: Determine reachability, check if attacker controls critical parts, evaluate exploitability

## Key Patterns

- Direct command injection: recv() -> buffer -> sprintf(cmd, "echo %s", buffer) -> system(cmd)
- Path traversal: fgets() -> filename -> fopen(filename, "r")
- Buffer overflow via tainted size: recv() -> size_buffer -> atoi(size_buffer) -> malloc(size)

## Output Format

Report taint paths with: source (function, address, context), sink (function, address, context), path (list of functions), sanitizers_found, is_vulnerable, confidence, vulnerability_type.

## Severity Guidelines

- **CRITICAL**: Direct injection with no sanitization (command injection, SQL injection)
- **HIGH**: Path traversal, buffer overflow via tainted size
- **MEDIUM**: Potential injection with partial sanitization
- **LOW**: Tainted data with limited impact

## See Also

- `patterns.md` - Detailed detection patterns and exploitation scenarios
- `examples.md` - Example analysis cases and code samples
- `references.md` - CWE references and mitigation strategies