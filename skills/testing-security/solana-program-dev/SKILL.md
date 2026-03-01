---
name: solana-program-dev
description: Design, implement, test, and security-review modern Solana programs (native Rust/Anchor/Steel/Pinocchio/Star Frame) and clients (@solana/kit + Codama) with an intermediate-to-advanced workflow.
---

# solana-program-dev (Solana Program Development Skill)

Use this skill when the user asks for help with:
- Solana on-chain program design/implementation (native Rust, Anchor, Steel, Pinocchio, Star Frame).
- PDA/account schema design, CPIs, token integrations, upgradeability/verifiable builds.
- Testing strategy (local validator, Anchor tests, Mollusk), compute/unit optimization.
- Client generation and modern TS/Rust client stacks (@solana/kit, Codama).

This skill assumes intermediate-to-advanced familiarity. Keep responses **concise, structured, and checklist-driven** (not an essay).

## Scope Boundaries
- Ask for missing context (cluster, program id(s), token standard, upgrade authority posture) instead of guessing.
- Do not "sign off" security without an explicit threat model + adversarial test coverage.
- Avoid mainnet deployment/key management guidance unless the user provides their operational constraints.

## Default Output Shape (what to produce)
When building or modifying a program, produce (in order):
1. **Problem restatement** (1-3 bullets) + explicit assumptions.
2. **Instruction spec** (list instructions + accounts + signer/writable + invariants).
3. **Account model** (PDAs, seeds, bumps, ownership, sizing/serialization).
4. **Implementation plan** (small steps, file touch list).
5. **Security checklist hits** (what you validated / what still needs validation).
6. **Testing plan** (unit/integration, negative tests, edge cases).
7. **Notes on DX/tooling** (Anchor vs native Rust vs Steel vs Pinocchio vs Star Frame; Codama/Kit client plan).

If reviewing code, produce (in order):
1. **Findings** (severity + impact + fix)
2. **Patch plan** (minimal diffs)
3. **Regression tests** to add

## Operating Principles
- Treat every account/arg as adversarial input.
- Validate close to where data enters (constraints, owners, addresses, types).
- Design for composability: stable IDLs, predictable accounts, explicit invariants.
- Track compute, binary size, and account locks from day 1.

## Navigate This Skill
- Core mental model: [concepts.md](./concepts.md)
- "If you remember nothing else": [maxims.md](./maxims.md)
- Rust cleanliness/DRY/idioms: [rust-style.md](./rust-style.md)
- Security checklist + common vulns: [security.md](./security.md)
- Dos/don'ts: [dos-donts.md](./dos-donts.md)
- Framework chooser: [frameworks.md](./frameworks.md)
- Modern toolchain: [tooling.md](./tooling.md)
- Built-in + common mainnet programs: [mainnet-programs.md](./mainnet-programs.md)
- Ecosystem snapshot (Jan 2026): [ecosystem-jan-2026.md](./ecosystem-jan-2026.md)
- Cursor rules template: [cursor.cursorrules.template](./cursor.cursorrules.template)
- Templates: [template-program-spec.md](./template-program-spec.md), [template-security-review.md](./template-security-review.md), [template-test-plan.md](./template-test-plan.md)
- Links: [links.md](./links.md)

## Workflow (repeatable)
### 1) Choose the stack (fast decision)
- Default: Anchor (fast iteration, strong IDL/DX, lots of examples).
- Native Rust: minimal deps + maximum control; more boilerplate.
- Steel: performance/binary-size oriented; lower-level patterns.
- Pinocchio: minimal runtime surface + perf focus; requires very disciplined validation.
- Star Frame: trait-based architecture + compile-time patterns; ecosystem may be smaller.
- Client-side:
  - Prefer `@solana/kit` for TS.
  - Prefer Codama for generated clients + avoiding manual serialization.

### 2) Write the spec before code
- List instructions and invariants.
- Define PDAs and their seed namespaces.
- Define which program owns each account and why.
- Define authority model (who can mutate what, under what conditions).

### 3) Implement with "validation first"
- Validate accounts (address/owner/type), then validate authorization (signer/authority), then validate inputs, then mutate state.

### 4) Tests must prove invariants
Minimum:
- happy path per instruction
- all authorization failures
- PDA substitution attempts
- "wrong program id" CPI attempts (where applicable)
- boundary arithmetic tests (overflows, rounding)
- account sizing/rent-exemption + realloc paths (where applicable)
- discriminator/serialization mismatch tests (wrong account type, corrupted data)

### 5) Security and deployment posture
- Have an upgrade authority plan (multisig? timelock? finalize?).
- Plan for verifiable builds and source authenticity.
- Document breaking changes and IDL versioning.

## Response Style Requirements
- Use headings + bullets.
- Short paragraphs only when necessary.
- Code blocks should be minimal and relevant.
- Call out *invariants* explicitly (they are the real "API").
