---
name: security-review
description: Scan code changes for security vulnerabilities including OWASP Top 10, hardcoded secrets, injection flaws, and authentication issues. Use after implementation to catch security problems before marking a task complete.
metadata:
  author: task-factory
  version: "1.0"
  type: follow-up
  hooks: pre-planning,pre,post
---

# Security Review

Perform a security review of all code changes you just made for this task.

## What to check

1. **Hardcoded secrets**: API keys, passwords, tokens, connection strings in source code
2. **Injection**: SQL injection, command injection, XSS via unsanitized user input
3. **Input validation**: Is all external input validated at system boundaries?
4. **Authentication & authorization**: Are sensitive endpoints protected? Are permissions checked?
5. **Data exposure**: Are error messages leaking internal details? Are logs safe?
6. **Dependencies**: Any known-vulnerable packages or insecure configurations?
7. **Cryptography**: Weak algorithms, hardcoded IVs, missing encryption for sensitive data?
8. **Race conditions**: TOCTOU bugs, shared mutable state without synchronization?

## Instructions

- Review each file you changed or created with a security mindset
- If you find vulnerabilities, fix them immediately
- For secrets found in code, replace with environment variables
- For injection flaws, add parameterized queries or input sanitization
- After fixing, verify the fix doesn't break functionality
- Summarize findings with severity (critical / high / medium / low)
