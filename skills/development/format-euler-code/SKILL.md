---
name: format-euler-code
description: Enforce the Euler repository C++ formatting and style rules from AGENTS.md. Use when writing, editing, or reviewing Euler solution code so it matches required includes, types, layout, namespaces, and I/O conventions.
---

# Format Euler Code

## Overview

Apply the Euler C++ code-style rules so solutions are consistent, readable, and compliant. Focus on formatting, structure, and required conventions while preserving correctness.

## Guidelines

- Keep helper functions and structs above `main()`; keep `main()` short and focused on orchestration and output.
- Use a single translation unit; avoid extra headers unless the problem demands it.
- List only required standard headers explicitly; avoid mega-headers.
- Qualify all standard library usage with `std::`; do not use `using namespace std;`.
- Use `static_cast<...>` for conversions; avoid C-style casts.
- Prefer fixed-width integers (`std::uint64_t`, `std::int64_t`) and keep constants `const` or `constexpr` near use.
- Mark file-scope helpers as `static` (or `static inline` for small helpers).
- Prefer `std::vector`/`std::array`; `reserve()` or pre-size when size is known; use `std::size_t` for indexing by size.
- Use clear loops with early-continue/early-return; avoid unnecessary recursion.
- Use `std::ios::sync_with_stdio(false); std::cin.tie(nullptr);` when reading input.
- End final output with `std::endl`.
- Keep comments short and only for non-obvious math or logic; use ASCII only; never reference AGENTS instructions.
- Use `<primesieve.hpp>` for prime generation when needed; use Boost multiprecision only when bounds require it.

## Quick Workflow

1. Scan the file for style violations: includes, namespaces, helper placement, types, casts, and I/O setup.
2. Apply the minimal edits needed to conform to guidelines without changing logic.
3. Re-check for unnecessary headers, incorrect types, or missing `static` helpers.
4. Confirm the final output line ends with `std::endl`.
5. Confirm that the program output remains unchanged compared to prior to the code updates.
