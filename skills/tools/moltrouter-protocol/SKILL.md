---
name: moltrouter-protocol
description: "Design, implement, or document Moltrouter Protocol (MRP) for agent-native discovery, capability manifests, negotiation flows, routing graphs, and evidence/provenance. Use when creating MRP specs, adding endpoints or schemas, drafting manifests, or generating curl/npm-style usage examples for agent-to-agent tooling."
---

# Moltrouter Protocol Skill

Follow this workflow to design or extend MRP artifacts for agent-native discovery and routing.

## 1) Clarify scope and target
- Identify whether the request is for **spec**, **service implementation**, **client tooling**, or **registry**.
- Confirm the transport target (HTTP/2, QUIC, libp2p) and required content types.

## 2) Use the protocol reference
- Use the core spec in `references/protocol.md` for message envelopes, content types, and routing concepts.
- Use `references/schemas.md` for capability manifests, offers, and evidence structures.
- Use `references/endpoints.md` for discovery and negotiation endpoints plus curl examples.
- Use `references/auth.md`, `references/payments.md`, and `references/errors.md` for auth, payment intents, and error handling.
- Use `references/jobs.md`, `references/registry.md`, `references/task-graph.md`, and `references/rate-limits.md` for async flows, discovery, routing, and quotas.

## 3) Produce artifacts in a standard format
- Emit JSON schemas or manifest examples when asked to define a capability.
- Include negotiation constraints (policy, budget, proof requirements) explicitly.
- Provide agent-usable CLI examples (`curl` or `npx`) when requested.

## 4) Validate consistency
- Ensure message types and fields align with the envelope in `references/protocol.md`.
- Ensure endpoints link back to the discovery flow and offer/execute lifecycle.

## 5) Keep outputs agent-native
- Prefer declarative structures (intent, capabilities, proofs) over prose.
- Include provenance and evidence fields whenever output represents an execution.
