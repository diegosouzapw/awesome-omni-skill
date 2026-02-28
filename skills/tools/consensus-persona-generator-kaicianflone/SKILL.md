---
name: consensus-persona-generator
description: Generate and persist reusable persona panels (persona_set artifacts) for consensus decision workflows. This skill initializes lightweight multi-agent disagreement with weighted reputations so downstream guards can make auditable, policy-governed decisions.
homepage: https://github.com/kaicianflone/consensus-persona-generator
source: https://github.com/kaicianflone/consensus-persona-generator
metadata:
  {"openclaw": {"requires": {"bins": ["node", "tsx"]}}}
---

# consensus-persona-generator

`consensus-persona-generator` is the entrypoint for evaluator diversity in the Consensus.Tools ecosystem.

## What this skill does

- creates N distinct decision personas (role, bias, risk posture, voting style)
- assigns initial reputation spread for weighted arbitration
- persists a versioned `persona_set` artifact to board state
- reuses compatible persona sets when possible to reduce churn

## Why this matters

Most agent pipelines fail because one model self-approves its own output. This skill injects structured disagreement first, so later guards operate over explicit multi-perspective review.

## Ecosystem role

Stack position:

`consensus-tools -> consensus-interact pattern -> consensus-persona-generator -> domain guards`

- **consensus-tools**: board/job/submission ledger substrate
- **consensus-interact**: board-native orchestration contract
- **persona-generator**: lightweight multi-agent initialization layer

## Inputs / outputs (automation-friendly)

- strict JSON input contract (`board_id`, `task_context`, `n_personas`, etc.)
- strict JSON output with `persona_set_id`, `personas[]`, and board write refs
- deterministic/replayable behavior where feasible

## Typical use cases

- bootstrap evaluators for email/publish/support/merge/action guards
- regenerate persona cohorts by domain or risk profile
- establish reusable governance personas for long-running automation


## Runtime, credentials, and network behavior

- runtime binaries: `node`, `tsx`
- network calls: none in the guard decision path itself
- conditional network behavior: if a run needs persona generation and your persona-generator backend uses an external LLM, that backend may perform outbound API calls
- credentials: `OPENAI_API_KEY` (or equivalent provider key) may be required **only** for persona generation in LLM-backed setups; if `persona_set_id` is provided, guards can run without LLM credentials
- filesystem writes: board/state artifacts under the configured consensus state path

## Dependency trust model

- `consensus-guard-core` and `consensus-persona-generator` are first-party consensus packages
- versions are semver-pinned in `package.json` for reproducible installs
- this skill does not request host-wide privileges and does not mutate other skills

## Quick start

```bash
node --import tsx run.js --input ./examples/persona-input.json
```

## Tool-call integration

This skill is wired to the consensus-interact contract boundary (via shared consensus-guard-core wrappers where applicable):
- readBoardPolicy
- getLatestPersonaSet / getPersonaSet
- writeArtifact / writeDecision
- idempotent decision lookup

This keeps board orchestration standardized across skills.

## Invoke Contract

This skill exposes a canonical entrypoint:

- `invoke(input, opts?) -> Promise<OutputJson | ErrorJson>`

`invoke()` starts the guard flow, which then executes persona evaluation and consensus-interact-contract board operations (via shared guard-core wrappers where applicable).
