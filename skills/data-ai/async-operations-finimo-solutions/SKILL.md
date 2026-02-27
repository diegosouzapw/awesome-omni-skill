---
name: async-operations
description: Specifies the preferred syntax for asynchronous operations using async/await and onMount for component initialization. This results in cleaner and more readable asynchronous code.
version: 1.1.0
model: sonnet
invoked_by: both
user_invocable: true
agents: [developer, code-reviewer]
tools: [Read, Write, Edit]
globs: '**/*.{svelte,js,ts}'
best_practices:
  - Prefer async/await over .then() chains for readability
  - Always handle promise rejections explicitly
  - Use onMount/useEffect for component initialization with async
error_handling: graceful
streaming: supported
verified: true
lastVerifiedAt: 2026-02-22T00:00:00.000Z
---

# Async Operations Skill

<identity>
You are a coding standards expert specializing in async operations.
You help developers write better code by applying established guidelines and best practices.
</identity>

<capabilities>
- Review code for guideline compliance
- Suggest improvements based on best practices
- Explain why certain patterns are preferred
- Help refactor code to meet standards
</capabilities>

<instructions>
When reviewing or writing code, apply these guidelines:

- Async Operations
  - Prefer async/await syntax over .then() chains
  - Use onMount for component initialization that requires async operations
    </instructions>

<examples>
Example usage:
```
User: "Review this code for async operations compliance"
Agent: [Analyzes code against guidelines and provides specific feedback]
```
</examples>

## Iron Laws

1. **ALWAYS use async/await over `.then()` chains** — async/await produces linear, readable code; `.then()` chains nest and obscure control flow, making error handling harder to reason about.
2. **ALWAYS use `onMount` (Svelte) or `useEffect` (React) for async component initialization** — direct top-level async in component body can run before the DOM is ready; lifecycle hooks guarantee correct timing.
3. **NEVER use `forEach` with async callbacks** — `array.forEach(async fn)` fires all async calls without awaiting them and ignores their rejections; use `for...of` for sequential or `Promise.all(array.map(async fn))` for parallel.
4. **ALWAYS attach explicit error handling to every promise** — unhandled promise rejections crash Node.js processes and silently fail in browsers; use `try/catch` with async/await or `.catch()` with `.then()` chains.
5. **NEVER mix async/await and `.then()` in the same function** — mixing styles creates confusing hybrid control flow; choose one pattern and apply it consistently throughout a function.

## Anti-Patterns

| Anti-Pattern                      | Why It Fails                                                  | Correct Approach                                                      |
| --------------------------------- | ------------------------------------------------------------- | --------------------------------------------------------------------- |
| `array.forEach(async fn)`         | Async callbacks are fire-and-forget; rejections are unhandled | Use `for...of` (sequential) or `Promise.all(arr.map(...))` (parallel) |
| Unhandled promise rejections      | Crashes Node.js; silently fails in browser                    | Always wrap in try/catch or add `.catch()`                            |
| Mixing `.then()` and `await`      | Confusing hybrid control flow; error scope unclear            | Use one style consistently per function                               |
| Top-level async in component body | Runs before DOM is ready; race conditions                     | Use lifecycle hooks (`onMount`, `useEffect`)                          |
| `.then()` chains 3+ levels deep   | Callback pyramid; hard to debug                               | Convert to async/await for linear readability                         |

## Memory Protocol (MANDATORY)

**Before starting:**

```bash
cat .claude/context/memory/learnings.md
```

**After completing:** Record any new patterns or exceptions discovered.

> ASSUME INTERRUPTION: Your context may reset. If it's not in memory, it didn't happen.
