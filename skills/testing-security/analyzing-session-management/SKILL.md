---
name: analyzing-session-management
description: Detects session management vulnerabilities including session fixation, session hijacking, and insecure cookie handling. Use when analyzing authentication sessions, cookie security, or investigating session-related vulnerabilities.
---

# Session Management Detection

## Detection Workflow

1. **Identify session operations**: Find session creation code, locate session validation checks, identify session destruction, map session lifecycle
2. **Analyze session ID generation**: Review session ID generation algorithm, check randomness and entropy, assess predictability, test for collision resistance
3. **Check transmission security**: Verify SSL/TLS usage, check for session ID in URLs, assess cookie security flags, review transmission methods
4. **Assess session lifecycle**: Verify session expiration, check logout behavior, assess session invalidation, review concurrent session handling

## Key Patterns

- Session fixation: predictable session IDs, session IDs not regenerated after login, accepting attacker-provided session IDs, weak session ID generation
- Session hijacking: session IDs exposed in URLs, session IDs transmitted insecurely, missing SSL/TLS, weak session ID entropy
- Session timeout issues: missing session expiration, excessive session timeout, no session invalidation on logout, persistent sessions across devices
- Cookie security: missing HttpOnly flag, missing Secure flag, cookie accessible via JavaScript, cookie path/domain misconfiguration

## Output Format

Report with: id, type, subtype, severity, confidence, location, vulnerability, session_generation (method, predictability, entropy), attack_scenario, bypass_steps, exploitable, impact, mitigation.

## Severity Guidelines

- **CRITICAL**: Session fixation allowing account takeover
- **HIGH**: Session hijacking with weak session IDs
- **MEDIUM**: Excessive session timeout or missing logout
- **LOW**: Minor cookie security issues

## See Also

- `patterns.md` - Detailed detection patterns and exploitation scenarios
- `examples.md` - Example analysis cases and code samples
- `references.md` - CWE references and mitigation strategies