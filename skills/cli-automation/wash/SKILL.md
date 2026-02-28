---
name: wash
description: Expert in wasmCloud Shell (wash) CLI tool for building, running, and managing WebAssembly components and wasmCloud applications. Use this skill when working with wasmCloud, WebAssembly components, WIT (WebAssembly Interface Types) definitions, or wash commands.
license: Apache-2.0
tags:
  - wasmcloud
  - cli
  - tooling
  - development
  - webassembly
---

# wash - wasmCloud Shell

This skill provides expertise in using the wasmCloud Shell (wash) CLI tool for developing and managing WebAssembly components and wasmCloud applications.

## Prerequisites

- `wash` version 2.0.0 or later (including release candidates like 2.0.0-rc.X)
- Verify version with: `wash --version`

## Important Notes

**Do not use `cargo component` commands** (like `cargo component build`) when working with wasmCloud projects. Always use `wash` commands instead:
- Use `wash build` or `wash dev` for building components
- For plugins, build with `wash build --skip-fetch` (since wasmcloud:wash interface isn't published)

## Core Commands

### wash dev

Use `wash dev` for local development and build loop. This command:
- Automatically rebuilds your component on file changes
- Provides a fast iteration cycle during development
- Watches for changes and recompiles as needed

Example:
```bash
wash dev
```

### wash wit update

Use `wash wit update` when there are mismatched WIT (WebAssembly Interface Types) definitions. This command:
- Updates WIT dependencies to resolve conflicts
- Synchronizes WIT definitions across your project
- Fixes version mismatches in component interfaces

Example:
```bash
wash wit update
```

## Common Workflows

### Starting a New Project

1. Initialize your project with `wash new https://github.com/cosmonic-labs/<TEMPLATE> --name my-project`
2. `cd my-project`
3. Run `wash build` to build the code
4. Make changes to your code

### Fixing WIT Definition Conflicts

If you encounter errors about mismatched WIT definitions:

1. Run `wash wit update` to synchronize definitions
2. Review the updated WIT files
3. Continue with `wash build`

## Additional Resources

- wash is the primary CLI tool for the wasmCloud ecosystem
- It handles component building, testing, and deployment
- Supports both local development and production workflows
