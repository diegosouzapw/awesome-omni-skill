---
name: getblock
description: GetBlock — RPC node and API access for 100+ blockchains; authentication, endpoints, CU pricing, Ethers/JSON-RPC, Yellowstone gRPC, MCP integration.
metadata:
  author: hairy
  version: "2026.2.9"
  source: Generated from https://github.com/GetBlock-io/getblock-docs
---

> Skill based on GetBlock docs, generated 2026-02-09. Source: https://github.com/GetBlock-io/getblock-docs

GetBlock provides plug-and-play node and API access for 100+ chains (Ethereum, BNB Chain, Polygon, Solana, TON, etc.). Authentication is via access token in the endpoint URL; no headers. Shared nodes use Compute Units (CU) for billing; dedicated nodes have custom limits.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Authentication | Access token in URL, no headers; roll/delete if compromised | [core-authentication](references/core-authentication.md) |
| Endpoints | Creating endpoints — protocol, network, full/archive, API type, region | [core-endpoints](references/core-endpoints.md) |
| CU pricing | Compute Units — chain × method multiplier, plan limits | [core-cu-pricing](references/core-cu-pricing.md) |

## Features

### Integration

| Topic | Description | Reference |
|-------|-------------|-----------|
| Ethers.js | Set GetBlock as JsonRpcProvider for Ethereum/EVM | [features-ethers-integration](references/features-ethers-integration.md) |
| JSON-RPC / cURL | Test connection — eth_blockNumber, eth_chainId, eth_getBalance | [features-jsonrpc-curl](references/features-jsonrpc-curl.md) |
| API overview | 100+ chains, JSON-RPC/WS/GraphQL/REST, add-ons (DAS, Firehose, Yellowstone) | [features-api-overview](references/features-api-overview.md) |
| Yellowstone gRPC | Solana real-time streaming (accounts, txs, blocks, slots) | [features-yellowstone-grpc](references/features-yellowstone-grpc.md) |
| MCP with GetBlock | Build MCP server exposing GetBlock Ethereum RPC as tools for AI agents | [features-mcp-getblock](references/features-mcp-getblock.md) |

## External Links

- [GetBlock docs](https://github.com/GetBlock-io/getblock-docs)
- [GetBlock nodes / pricing](https://getblock.io/nodes/)
- [Compute Units](https://getblock.io/pricing/compute-units/)
