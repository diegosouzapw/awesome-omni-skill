---
name: celestia
description: Celestia data availability node—bridge/full/light types, headers, shares, DAS, state API, P2P discovery, and fraud proofs.
metadata:
  author: hairy
  version: "2026.2.24"
  source: Generated from https://github.com/celestiaorg/celestia-node, scripts at https://github.com/antfu/skills
---

> Skill based on celestia-node, generated from `sources/celestia`. Doc path: `sources/celestia/README.md`, `sources/celestia/docs/adr/`, and package `doc.go` (header, share, das, state).

Celestia-node is the Go implementation of Celestia’s data availability (DA) node types: **bridge**, **full**, and **light**. The DA network wraps celestia-core by consuming or producing ExtendedHeaders and making block data available via share sampling (DAS). Use this skill to operate nodes, query headers/shares/state, submit PayForBlob and other transactions, and reason about P2P discovery and fraud proofs.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Node types | Bridge, full, light—roles, DAS, and CLI init/start | [core-node-types](references/core-node-types.md) |
| Headers | ExtendedHeader, header service flow, sync, Header module API | [core-headers](references/core-headers.md) |
| Shares and DAS | GetSharesByNamespace, availability, DASer, Shares/DAS module | [core-shares-and-das](references/core-shares-and-das.md) |
| State and transactions | StateModule, SubmitTx, SubmitPayForBlob, Transfer, staking, Accessor | [core-state-and-txs](references/core-state-and-txs.md) |

## Features

### Public API and discovery

| Topic | Description | Reference |
|-------|-------------|-----------|
| Public API | Module-centric API (Header, Shares, P2P, Node, DAS, State, Fraud, Metrics) | [features-public-api](references/features-public-api.md) |
| P2P discovery | Full-node discovery, advertising, bridge/light behavior, tagging | [features-p2p-discovery](references/features-p2p-discovery.md) |
| Fraud proofs | BEFP, subscribe/verify, storage, fraud sync, halting | [features-fraud-proofs](references/features-fraud-proofs.md) |

## Best Practices

| Topic | Description | Reference |
|-------|-------------|-----------|
| State verification | Verifying balance (and state) against header AppHash with Merkle proofs | [best-practices-state-verification](references/best-practices-state-verification.md) |
