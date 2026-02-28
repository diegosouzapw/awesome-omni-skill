---
name: codegen-sync
description: Verifies TypeScript codegen is in sync with Elixir protocol definitions. Use this skill after modifying Elixir structs, protocol messages, or RPC types in lib/opal/ to regenerate and verify the TypeScript client SDK types.
globs: "{lib/opal/rpc/**,lib/opal/agent/**,cli/src/sdk/**}"
---

# Codegen Sync Skill

This project uses code generation to keep TypeScript SDK types in sync with Elixir protocol definitions. When Elixir types change, the TypeScript side must be regenerated.

## When to act

- After modifying any Elixir structs, message types, or RPC definitions in `lib/opal/`.
- After adding or changing tool schemas in `lib/opal/tool/`.
- When CI fails with a codegen drift error.
- When the user asks to sync or regenerate types.

## Commands

```bash
# Run codegen (Elixir -> TypeScript)
mix run scripts/codegen_ts.exs

# Verify codegen is up to date (what CI runs)
mix run scripts/codegen_ts.exs --check
```

## Workflow

1. After modifying Elixir protocol types, run `mix run scripts/codegen_ts.exs`.
2. Review the generated diff in `cli/src/sdk/` to verify correctness.
3. Include the regenerated files in the same commit as the Elixir changes.

## Rules

1. **Never hand-edit generated files.** Always regenerate from the Elixir source.
2. If codegen output looks wrong, fix the Elixir definitions or the codegen script (`scripts/codegen_ts.exs`), not the output.
3. Always commit codegen output alongside the source changes — never in a separate commit.
