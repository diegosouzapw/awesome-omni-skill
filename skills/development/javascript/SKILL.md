---
name: javascript
description: Best practices for JavaScript/TypeScript development including modern ES6+ patterns, error handling, and performance optimization.
globs: ["**/*.js", "**/*.mjs", "**/*.cjs", "**/*.jsx", "**/*.ts", "**/*.tsx"]
priority: 100
tags: ["language"]
---

# JavaScript/TypeScript Best Practices

## Code Style
- Use const by default, let when reassignment is needed
- Use template literals for string interpolation
- Prefer arrow functions for callbacks
- Use destructuring for object/array access
- Use async/await over .then() chains
- Use semicolons consistently (pick a style)

## Error Handling
- Always handle promise rejections with .catch() or try/catch
- Use try/catch with async/await
- Provide meaningful error messages with context
- Don't swallow errors silently
- Create custom error classes for domain errors

## Modern Patterns
- Use optional chaining (?.) and nullish coalescing (??)
- Use spread operator for shallow object/array copies
- Use Map/Set for complex data structures
- Prefer for...of over forEach for iterables
- Use Object.entries/keys/values for object iteration
- Use Array.from() for array-like conversions

## TypeScript Specific
- Prefer interfaces over type aliases for objects
- Use strict mode in tsconfig.json
- Avoid 'any' type - use 'unknown' if needed
- Use generics for reusable type-safe code
- Use discriminated unions for type narrowing
- Leverage const assertions for literal types

## Performance
- Avoid creating functions inside loops
- Use object pools for frequent allocations
- Debounce/throttle expensive operations
- Use Web Workers for CPU-intensive tasks
