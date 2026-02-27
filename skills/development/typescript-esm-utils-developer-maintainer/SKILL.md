---
name: typescript-esm-utils-developer-maintainer
description: skill for developing and maintaining TypeScript utility libraries transpiled to esm. Use this skill when working on TypeScript projects that involve creating reusable utility functions, managing library structure, testing, documentation, and build processes. This includes creating new utilities, refactoring existing code, setting up testing frameworks, configuring build tools, and ensuring type safety.
---

# TypeScript ESM Utils Developer & Maintainer

## Overview

This skill provides comprehensive guidance for developing TypeScript utility libraries transpiled to ESM, including best practices for code organization, testing, documentation, and build configuration.

## Workflow Decision Tree

Use this skill when:
- Creating new TypeScript utility functions or modules
- Maintaining a TypeScript library project
- Maintaining pure cjs source files
- Writing and maintaining tests for utility functions
- Configuring and maintaining build tools (tsc)
- Managing type definitions and exports
- Refactoring utility code for better performance or maintainability

## Core Capabilities

### 1. Project Structure & Setup
- Maintain TypeScript library projects with proper folder structure
  - ./ts is the main source folder
  - ./esm is the main transpilation target folder
- Maintain package.json with appropriate scripts and dependencies
- Set up tsconfig.json for optimal settings between the lib to deliver as esm modules and the tests
- Create proper .gitignore and other config files
- Follow conventional commit messages for git commits
- Maintain a strict linting configuration with ESLint

### 2. Utility Function Development
- Write type-safe utility functions following TypeScript best practices
- Implement proper error handling and edge case management
- Use advanced TypeScript features (generics, conditional types, mapped types)
- Ensure functions are pure and side-effect free when appropriate

### 3. Testing & Quality Assurance
- Make sure no deprecated packages are used
- Set up Bun for unit testing and running scripts
- Write comprehensive test suites with edge cases
- Implement property-based testing where applicable
- Configure test coverage reporting
- **Après toute modification de fichiers par l'agent :** Toujours lancer le linting (`bun run lint`), exécuter les tests (`bun test`), et vérifier qu'il n'y a pas d'erreurs ou d'avertissements TypeScript (`bun run build`). Résoudre immédiatement toute erreur ou avertissement détecté.

### 4. Build & Distribution
- Support continuous ESM transpilation without warning or error
- Generate type definition files (.d.ts)

### 5. Documentation & Maintenance
- Audit the dependencies and make sure no vulnerable package version is used
- Write JSDoc comments for all public APIs
- Maintain changelog and version management
- Implement proper semantic versioning

## Best Practices

### Code Organization
- eslint.config.js is sacred: follow its rules strictly and do not modify it unless explicitly instructed
- Search for existing utilities before creating new ones
- Avoid code duplication by reusing existing utilities
- Look for linting rules and apply them consistently
  - analyse eslint.config.js for coding style rules to follow during development
- Follow strictly the linting rules
  - when coding, avoid eslint-disable comments as much as possible
    - exceptions are:
      - max-params when:
        - a callback or a sorting function with fixed signature
        - a provided or a type definition function with fixed signature
        - mocked and testing functions where parameters are fixed by the implementation or the API that is being mocked
    - @ts-ignore when:
      - unavoidable limitations due to the code being run on different environments (e.g., browser vs node) or node versions
      - unavoidable limitations due to third-party library type definitions
  - use the eslint --fix command to check and automatically fix as many code style issues automatically before analysing the possible remaining errors or warnings
- Group related utilities in feature-based modules
- Use barrel exports (index.ts files) for clean imports
- Implement proper tree-shaking support
- Follow consistent naming conventions
- Keep util functions small and focused, optimize for reusability and tree-shaking
- functions MUST have 0 or 1 parameter only
  - Follow the max-params limit of 1: Avoid functions with multiple parameters unless absolutely necessary
  - If the function requires multiple parameters, use an object as the single parameter to encapsulate them
  - The single parameter can be an optional object to allow multiple named parameters if needed that are all optional only
  - The single parameter is mandatory if at least 1 of its parameters is required for the function to operate correctly
- Do not use ; at the end of statements
  - rewrite the code to avoid needing them when a missing ; would cause a syntax error
- Use single quotes for basic strings
- Use template literals instead of string concatenation
- Prefer using named functions for better stack traces
  - else use arrow functions for anonymous functions
- Use async/await for asynchronous code instead of using callbacks or .then()/.catch()
- Prefer const over let, and never use var
- Use destructuring for objects and arrays except when the value is used only once
- Use optional chaining and nullish coalescing operators for safer property access
- Prefer for...of loops for iterating over arrays
- Avoid using any type, prefer unknown with proper type guards
- Never use typescript enums, prefer explicit reusable constants
- Always use strict equality (=== and !==)
- Never use implicit type coercion, use explicit conversions
- Always use explicit braces for blocks, even for single statements
- Never use break
- Never use continue
- Never use switch statements, prefer lists of declarative states with object literals that can be linked to a specific value or boolean function
- Use PascalCase for class and interface names
- Use camelCase for variable, function, and attribute names
  - Except for globally reusable constants that MUST use UPPER_SNAKE_CASE
- Strictly boolean variable names MUST start with the b prefix (e.g., bIsValid, bHasItems, B_CAN_ARCHIVE_ALERT)
- Use explicit return types for named functions and methods
- Prefer using interfaces for complex object types used in multiple places
- Use += 1 instead of ++
- Use -= 1 instead of --
- 0 or 1 return statements per function only
  - refactor the code to avoid multiple return statements
  - throwing errors is allowed when not on the nominal return path
- Do not use the the forEach method: use mainly for...of loops and loop over GET_OBJECT_ENTRY_LIST or the Array.entries() method when the key/index is used in the loop
  - Use for...index loops in 2 cases only:
    - better performance on arrays of more than 1e6 items
    - when the exit condition is optimized for early exit (e.g., searching for an item until found)
      - use flags or status checks for early exits in the condition for better performance
      - never return in a loop: do not break the single return statement rule

### Type Safety
- Use strict TypeScript settings
- Leverage utility types from TypeScript standard library
- Create custom utility types for complex scenarios
- Ensure 100% type coverage

### Performance
- Write efficient algorithms with O(n) complexity analysis
- Use lazy evaluation where beneficial
- Implement memoization for expensive operations
- Optimize bundle size through tree-shaking when available

### Testing
- Test both happy path and error scenarios driven by edge cases
- Use table-driven tests for multiple inputs
  - All cases MUST have a descriptive test name to display in the test results
- Combine with property-based testing
- Mock external dependencies appropriately
- Maintain high test coverage (>90%)

## Common Patterns

### Utility Function Template
```typescript
/**
 * Description of what the function does
 * @param _ - Description of the provided parameter(s)
 * @returns Description of the return value
 * @example
 * ```typescript
 * const result = utilityFunction(input);
 * ```
 */
export function utilityFunction<T, U, V, W>(_?: { t: T, u?: U, v?: V }): W {
  // Implementation
  return _
}
```
