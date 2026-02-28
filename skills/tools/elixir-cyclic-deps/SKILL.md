---
name: elixir-cyclic-deps
description: Detects and removes cyclic compile-time dependencies in Elixir codebases using mix xref, with minimal code changes. Use when the user explicitly asks to check for cycles, remove cyclic dependencies, or fix xref cycle failures. Requires Elixir 1.19 or higher for accurate results.
---

# Elixir Cyclic Dependencies

## When to Use

Apply this skill only when the user **explicitly asks** to check or remove cyclic dependencies (e.g. "check for cycles", "remove cyclic dependencies", "fix xref cycles").

## Prerequisite: Elixir Version

Cycle detection is correct only on **Elixir 1.19 or higher**. Before running any cycle checks:

1. Verify version: `elixir -v` or inside the project run `mix run -e 'IO.inspect(System.version())'`.
2. If the version is below 1.19, stop and inform the user that upgrading to Elixir 1.19+ is required for reliable cycle detection.

## Detecting Cycles

Run both commands from the project root. Start with a relaxed threshold, then tighten:

```bash
mix xref graph --format cycles --label compile-connected --fail-above 3
mix xref graph --format cycles --label compile --fail-above 3
```

- **compile-connected**: cycles in the compile-time dependency graph (modules that compile in a cycle).
- **compile**: same graph, different label; both should be run.
- **--fail-above N**: exit code fails when cycle count is above N. Use `3` initially, goal is **0**.

To list cycles without failing (for inspection), omit `--fail-above` or set it high:

```bash
mix xref graph --format cycles --label compile-connected
mix xref graph --format cycles --label compile
```

**Goal**: reach `--fail-above 0` for both commands (no cycles).

## Workflow

1. **Check Elixir version** (must be ≥ 1.19).
2. **Establish baseline**: run both xref commands with `--fail-above 3` (or current project setting). Note cycle count and which modules appear in cycles.
3. **Inspect cycles**: run without `--fail-above` to see full cycle output; identify the smallest set of edges (module A → module B) that, if removed, break cycles.
4. **Plan minimal changeset**:
   - Prefer breaking a single dependency (e.g. move shared code to a new module both can depend on) over large refactors.
   - **Helper modules** are allowed if they yield the smallest change: extract shared logic into a new module that the cycle participants depend on, so the cycle is broken.
   - Avoid duplication; follow existing repository guidelines (e.g. AGENTS.md, .cursor rules) and prior patterns in the codebase.
5. **Implement**: make the chosen change, then re-run both xref commands with `--fail-above 0` (or gradually lower from 3 → 0).
6. **Verify**: `mix compile --warnings-as-errors`, tests, and any project lint (e.g. `mix lint` or `mix cyclecheck` if defined).

## Minimal-Change Strategies

| Strategy                  | When to use                                                                                                                        |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Extract helper module** | Two or more modules in a cycle need the same logic; extract it to a new module that has no dependency back on the cycle.           |
| **Move code down**        | Move a function from module A to module B so that A no longer depends on B (or vice versa), breaking the cycle.                    |
| **Invert dependency**     | If A uses B and B uses A, see if one usage can be replaced by a callback, option, or data structure so only one direction remains. |
| **Split module**          | One module has two distinct responsibilities and participates in two cycles; split into two modules to break cycles.               |

Choose the option that removes cycles with the **least** code churn and no new duplication.

## Project Integration

If the project has a `cyclecheck` (or similar) task in `mix.exs`, use it after changes:

```bash
mix cyclecheck
```

Ensure it runs both xref commands with the target `--fail-above 0` once cycles are cleared.

## Summary Checklist

- [ ] Elixir version ≥ 1.19 confirmed
- [ ] Both `compile-connected` and `compile` xref cycle commands run
- [ ] Goal: `--fail-above 0` for both
- [ ] Minimal changeset (helper modules OK)
- [ ] No code duplication; follows repo guidelines
- [ ] `mix compile`, tests, and lint pass after changes
