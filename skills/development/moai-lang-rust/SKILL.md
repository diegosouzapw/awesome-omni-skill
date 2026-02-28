---
name: moai-lang-rust
description: Rust specialist for async services, safe systems programming, and production tooling with Tokio, Axum, SQLx, and clippy-driven quality checks. Use when building or refactoring Rust code.
license: Apache-2.0
compatibility: Designed for Claude Code
allowed-tools: Read, Grep, Glob, Bash
user-invocable: false
metadata:
  version: "2.0.0"
  category: "language"
  status: "active"
  updated: "2026-02-22"
---

# Rust Development (Lean)

## When to use

- `.rs` files and `Cargo.toml`
- Async services, CLI tools, systems components
- Ownership/lifetime, trait, or performance-sensitive work

## Defaults

- Prefer explicit error types at library boundaries.
- Keep unsafe usage isolated and documented.
- Use clippy and rustfmt as non-optional quality gates.
- Favor simple ownership flows over complex lifetimes when possible.

## Quick workflow

1. Identify crate boundaries and public APIs.
2. Implement minimal change with clear types/errors.
3. Add/update tests for changed behavior.
4. Run fmt, clippy, and tests.

## Commands

- Format: `cargo fmt --all`
- Lint: `cargo clippy --all-targets --all-features -- -D warnings`
- Test: `cargo test --all-features`
- Build release: `cargo build --release`

## Implementation checklist

- `Result<T, E>` and `?` used consistently.
- Avoid panics in non-test code paths unless explicitly fatal.
- Serialization contracts versioned when exposed externally.
- Concurrency primitives chosen deliberately (`Mutex`, `RwLock`, channels, semaphore).

## Validation checklist

- `cargo fmt` clean.
- `cargo clippy` clean.
- Tests pass.
- New dependencies justified.

## References

- `references/reference.md` - compact guidance and docs links
- `references/examples.md` - concise service/test patterns
- `../AGENT_SKILL_SPEC.md` - shared Anthropic/Copilot alignment
