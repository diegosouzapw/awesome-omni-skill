---
name: analyzing-protocols
description: Analyzes network protocol implementations to identify parsing vulnerabilities, state machine issues, and protocol-level security problems. Use when analyzing network servers, protocol handlers, or investigating protocol implementation bugs.
---

# Protocol Analysis

## Detection Workflow

1. **Identify protocol handlers**: Find network socket handlers, locate packet parsing functions, identify protocol state machines, map protocol message types
2. **Analyze protocol messages**: Identify message formats, parse message fields, check field validation, assess length handling
3. **Trace protocol state**: Map state transitions, identify state variables, check state machine completeness, assess state corruption potential
4. **Assess security**: Check authentication and authorization, verify input validation, assess exploitability, identify protocol-level attacks

## Key Patterns

- Protocol parsing vulnerabilities: buffer overflows in packet parsing, integer overflow in length fields, format string in protocol handling, command injection in protocol commands
- State machine issues: out-of-order message handling, missing state transitions, state corruption attacks, protocol state confusion
- Authentication flaws: weak authentication mechanisms, missing authentication on critical operations, authentication bypass opportunities, session management issues
- Protocol logic errors: race conditions in protocol handling, TOCTOU in protocol operations, improper input validation, protocol downgrade attacks

## Output Format

Report with: id, type, subtype, severity, confidence, location, protocol, vulnerability, packet_field (name, offset, size), overflow_scenario, attack_vector, exploitable, impact, mitigation.

## Severity Guidelines

- **CRITICAL**: Remote code execution via protocol vulnerability
- **HIGH**: Remote DoS or information disclosure
- **MEDIUM**: Local protocol issues
- **LOW**: Minor protocol bugs

## See Also

- `patterns.md` - Detailed detection patterns and exploitation scenarios
- `examples.md` - Example analysis cases and code samples
- `references.md` - CWE references and mitigation strategies