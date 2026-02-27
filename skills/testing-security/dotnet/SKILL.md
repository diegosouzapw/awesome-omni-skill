---
name: dotnet
description: .NET development standards and practices for zero-fabrication, test-driven development with strict quality gates. Use when working on .NET/C# projects that require rigorous testing, real integrations only, and co-located tests.
---

# .NET Development Skill

**Zero-Fabrication | Test-Driven | Zero-Tolerance**

This skill defines the mandatory standards for .NET development. These rules are **non-negotiable** and enforced to ensure consistency, maintainability, and correctness.

## ⚡ The Golden Rules

1.  **Real Tests Only**: No mocks, no fakes. Tests must hit actual systems (DB, File, API).
2.  **Co-located Tests**: Every file must have a `#region Tests` at the end.
3.  **Immutability**: Use `record` types only. No public setters.
4.  **One Class Per File**: File name must match type name exactly.
5.  **Zero Warnings**: Warnings are errors. No exceptions.
6.  **"HOW" vs "WHAT"**: 95% of code in `Common` (Utilities), 5% in Features (Business Logic).
7.  **Document Everything**: XML comments are MANDATORY for ALL public members, **INCLUDING TEST METHODS**.

## 📚 Documentation & Standards

-   **[CODE_RULES.md](skills/CODE_RULES.md)**: Coding standards, naming, style, and prohibited patterns.
-   **[ARCH_RULES.md](skills/ARCH_RULES.md)**: Project structure, file organization, and architectural layers.
-   **[TEST_RULES.md](skills/TEST_RULES.md)**: Testing philosophy, patterns, and mandatory practices.
-   **[SETUP.md](skills/SETUP.md)**: Project initialization, configuration, and templates.
-   **[EXAMPLES.md](skills/EXAMPLES.md)**: Reference guide to the included example files.

## 🛠️ Quick Start

1.  Run the setup commands in **[SETUP.md](skills/SETUP.md)**.
2.  Copy `skills/Directory.Build.props` to your `src` directory.
3.  Follow **[ARCH_RULES.md](skills/ARCH_RULES.md)** for file placement.
4.  Verify compliance with **[CODE_RULES.md](skills/CODE_RULES.md)** and **[TEST_RULES.md](TEST_RULES.md)** before every commit.

**IF YOU VIOLATE THESE RULES, YOU WILL FAIL.**
