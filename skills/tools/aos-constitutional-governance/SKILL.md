---
name: aos-constitutional-governance
description: Create, sign (Ed25519), verify, and audit an AOS-style immutable constitution for OpenClaw. Use when implementing constitutional governance beneath SOUL.md, designing deny/confirm/allow policy evaluation, generating constitution.yaml + constitution.sig, validating GitTruth attestations, building tamper-evident tool-call logs, or preparing a reference implementation skill/plugin for OpenClaw.
---

# AOS Constitutional Governance (OpenClaw)

Implement AOS-style two-layer governance for tool-using assistants:

- **Immutable Constitution (locked)**: signed + GitTruth-attested policy enforced at runtime.
- **Mutable Identity (flexible)**: persona/workflow guidance (e.g., `SOUL.md`).
- **User Task Intent (ephemeral)**: per-request justification, confirmations, and overrides.

This Skill focuses on **Phase 1** deliverables (spec + signing + verification + audit artifacts) and provides the evaluation algorithm needed for Phase 2 (Gateway enforcement).

**Clarification:** This repository demonstrates a **reference integration** between agent frameworks and constitutional governance concepts. It does **not** grant patent rights or disclose enforcement mechanisms beyond illustrative examples.

## Files produced

- `constitution.yaml` — human-readable policy
- `constitution.c14n.json` — canonical JSON used for hashing/signing
- `constitution.sig.json` — detached Ed25519 signature metadata
- `constitution.attestation.json` — GitTruth attestation metadata (pointer)

## Canonicalization + signing

1. Convert `constitution.yaml` → canonical JSON (sorted keys, normalized scalars).
2. Compute `doc_hash = sha256(c14n_json_bytes)`.
3. Sign `doc_hash` with Ed25519 → `signature`.
4. Commit `constitution.yaml` + `constitution.sig.json` to git.
5. GitTruth attests the commit.
6. Gateway verifies (a) Ed25519 signature over `doc_hash`, and (b) GitTruth attestation for the commit.

Use scripts:
- `scripts/c14n.py` — canonicalize YAML→JSON
- `scripts/sign.py` — Ed25519 sign
- `scripts/verify.py` — verify Ed25519 + (optionally) GitTruth attestation

## Policy evaluation (deny / confirm / allow)

### Decision model

The policy engine returns one of:

- **DENY**: tool call is blocked (constitutional).
- **CONFIRM**: tool call is paused pending explicit user approval (scoped override token).
- **ALLOW**: tool call may execute.

If multiple rules match, select the **most restrictive** decision:

`DENY > CONFIRM > ALLOW`

and merge obligations (logging, disclosure, reflection) from all matched rules.

### Inputs to evaluation

- `tool`: string (e.g., `message.send`, `read`, `exec`)
- `args`: structured args (paths, urls, message text)
- `session`: { kind, label, channel }
- `intent`: user task intent object (may be empty)
- `risk`: derived risk classification (see below)
- `classifications`: derived tags (e.g., `impersonation`, `fraud`) from deterministic matchers

### Output

- `decision`: ALLOW|CONFIRM|DENY
- `reason_code`: stable string
- `obligations`: e.g., disclosure text to append, reflection fields, log requirements
- `override`: if CONFIRM, a scope hash to approve exactly this call

## Risk classification (deterministic)

Risk should be **rules-based and reproducible**, not LLM-judgment.

Use `scripts/risk.py` as a reference implementation.

Risk is computed as:

`risk = max(tool_base_risk(tool), arg_risk(tool,args), data_risk(args), egress_risk(args), user_scope_risk(session,intent))`

with levels: `low < medium < high < critical`.

## Next steps

Phase 2 (Gateway): move the evaluator into the tool router so it is non-bypassable.
