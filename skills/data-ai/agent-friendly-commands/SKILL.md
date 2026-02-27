---
name: agent-friendly-commands
description: Use these when you want low-noise lint/test output (good for LLM/CI logs) while staying aligned with repo policy.
---

## Goal

Run the same checks as CI with less ANSI noise and more compact reporters.

## Commands

- Lint (no ANSI): `npm run lint:agent`
- Unit/integration tests (Rust + UI): `npm run test:agent`
- Rust-only tests (nextest, CI profile): `npm run test:rust:agent`
- UI unit tests (Vitest, dot reporter): `npm run test:ui:agent`
- Smoke suite (builds first): `npm run test:smoke:agent`
- Smoke suite (no rebuild): `npm run test:smoke:quick:agent`

## When to use what

- Tight loop on a Rust failure: start with `npm run test:rust:agent`, then widen to `npm run test:agent`.
- Tight loop on a UI unit test: `npm run test:ui:agent -- -t "<substring>"`.
- Tight loop on smoke: `npm run test:smoke:quick:agent -- -t "<substring>"` (only if you already built recently).

## Notes

- These scripts are implemented inline in `package.json` using `bash -lc` to set `NO_COLOR=1`, disable forced color, and use dot reporters for Vitest.
- Repo policy still applies: after code changes, the completion bar is `npm run lint`, `npm test`, `npm run smoke`.
