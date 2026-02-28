---
name: Resolvio
description: Interact with Resolvio - an ENS (Ethereum Name Service) resolution API. Resolve ENS names to addresses, perform reverse lookups, and fetch profile data.
---

# Resolvio API Skill

This skill enables AI agents to interact with **Resolvio**, an ENS (Ethereum Name Service) resolution API.

## What is Resolvio?

Resolvio is a REST API dedicated to **ENS resolution only**. It provides:
- **Forward resolution**: ENS name → addresses, texts, contenthash
- **Reverse resolution**: Ethereum address → primary ENS name
- **Multi-chain support**: ETH, BTC, SOL, OP, Base, ARB addresses via coin types
- **Profile data**: Complete ENS profiles with all records

## Production API

- **Base URL**: `https://resolvio.namespace.ninja`
- **Documentation**: https://docs.namespace.ninja/api-reference/resolvio/
- **Swagger**: https://resolvio.namespace.ninja/api/docs

## Core Capabilities

### 1. Get ENS Profile

Fetch complete ENS profile including addresses, text records, and contenthash.

**Endpoint**: `GET /ens/v1/profile/:name`

**Example**:
```bash
curl https://resolvio.namespace.ninja/ens/v1/profile/vitalik.eth
```

**Response**:
```json
{
  "name": "vitalik.eth",
  "addresses": [
    { "coin": 60, "name": "eth", "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045" }
  ],
  "texts": [
    { "key": "avatar", "value": "..." },
    { "key": "com.twitter", "value": "..." }
  ],
  "contenthash": "ipfs://...",
  "resolver": "0x..."
}
```

### 2. Get Specific Addresses

Fetch cryptocurrency addresses for specific coin types.

**Endpoint**: `GET /ens/v1/addresses/:name?coins[]=60&coins[]=0`

**Common Coin Types**:
- `60` = Ethereum (ETH)
- `0` = Bitcoin (BTC)
- `501` = Solana (SOL)
- `2147483658` = Optimism (OP)
- `2147492101` = Base
- `2147525809` = Arbitrum (ARB)

**Example**:
```bash
curl "https://resolvio.namespace.ninja/ens/v1/addresses/vitalik.eth?coins[]=60&coins[]=501"
```

### 3. Get Text Records

Fetch specific text records (social links, avatar, etc.).

**Endpoint**: `GET /ens/v1/texts/:name?keys[]=avatar&keys[]=com.twitter`

**Common Text Keys**:
- `avatar` - Profile picture
- `description` - Bio/description
- `com.twitter` - Twitter handle
- `com.github` - GitHub username
- `com.discord` - Discord username
- `url` - Website URL
- `email` - Email address

**Example**:
```bash
curl "https://resolvio.namespace.ninja/ens/v1/texts/vitalik.eth?keys[]=avatar&keys[]=com.twitter"
```

### 4. Reverse Resolution (Bulk)

Convert multiple Ethereum addresses to their primary ENS names.

**Endpoint**: `POST /ens/v1/reverse/bulk`

**Example**:
```bash
curl -X POST https://resolvio.namespace.ninja/ens/v1/reverse/bulk \
  -H "Content-Type: application/json" \
  -d '{
    "addresses": [
      "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "0x225f137127d9067788314bc7fcc1f36746a3c3B5"
    ]
  }'
```

**Response**:
```json
{
  "result": [
    {
      "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
      "name": "vitalik.eth",
      "hasReverseRecord": true
    }
  ]
}
```

### 5. Reverse Resolution (Single)

Resolve a single Ethereum address to its primary ENS name.

**Endpoint**: `POST /ens/v1/reverse/single/:address`

**Example**:
```bash
curl -X POST https://resolvio.namespace.ninja/ens/v1/reverse/single/0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

## Common Use Cases

### Use Case 1: Display User Profile
When you need to show a complete ENS profile:
```
GET /ens/v1/profile/vitalik.eth
```

### Use Case 2: Get Ethereum Address from Name
When you need to resolve a name to an ETH address:
```
GET /ens/v1/addresses/vitalik.eth?coins[]=60
```
Extract `addresses[0].address` from response.

### Use Case 3: Show ENS Name for Address
When you have an address and want to display the ENS name:
```
POST /ens/v1/reverse/single/0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

### Use Case 4: Get Social Links
When you want to fetch social media handles:
```
GET /ens/v1/texts/vitalik.eth?keys[]=com.twitter&keys[]=com.github&keys[]=url
```

### Use Case 5: Multi-Chain Address Resolution
When you need addresses for multiple blockchains:
```
GET /ens/v1/addresses/vitalik.eth?coins[]=60&coins[]=0&coins[]=501
```
Returns ETH, BTC, and SOL addresses.

### Use Case 6: Dashboard UX - Display Human-Readable Names
When building a dashboard that shows multiple addresses and you want to display ENS names for better UX:
```
POST /ens/v1/reverse/bulk
{
  "addresses": [
    "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    "0x225f137127d9067788314bc7fcc1f36746a3c3B5",
    ...
  ]
}
```
**Tip**: Always use bulk resolution instead of individual calls when displaying multiple addresses - it's much more efficient and reduces API load.

## Query Parameters

### Cache Control
Add `noCache=true` to any GET endpoint to bypass cache:
```
GET /ens/v1/profile/vitalik.eth?noCache=true
```

### Chain ID (for reverse resolution)
Specify chain for reverse resolution (defaults to mainnet):
```json
{
  "addresses": ["0x..."],
  "chain_id": 1
}
```

## API Limitations

### Bulk Resolution Limits
- **Maximum addresses per request**: 20 (hard limit enforced)
- **Error on excess**: Returns 400 error if >20 addresses provided
- **Best practice**: For >20 addresses, split into batches of 20

### Rate Limiting
- **Current status**: No rate limiting enforced
- **Fair use policy**: Please avoid excessive requests that may impact service availability
- **Future**: Rate limits may be added; will be documented when implemented

### Caching
- **Default cache TTL**: 5 minutes
- **Bypass cache**: Use `noCache=true` query parameter
- **Cache type**: In-memory (resets on service restart)

### Chain Support
- **Forward resolution**: Ethereum mainnet only (other chains via coin types)
- **Reverse resolution**: Ethereum mainnet (chain_id: 1)
- **Multi-chain addresses**: Via coin type parameter (see coin types list)

### Known Limitations
- ENS-only (other naming services not yet supported)
- No WebSocket/real-time updates
- No API key authentication (public access)
- No persistent storage (cache is ephemeral)

## Error Handling

- **404**: ENS name doesn't exist or has no resolver
- **400**: Invalid request (malformed address, invalid coin type)
- **Empty arrays**: ENS name exists but no records set for requested types

## Tips for Agents

1. **Always use production URL** for live queries: `https://resolvio.namespace.ninja`
2. **Default coin type is 60 (ETH)** - if user asks for "the address", use coin 60
3. **Cache is enabled by default** - responses are cached for 5 minutes
4. **Bulk operations are more efficient** - use `/reverse/bulk` when resolving multiple addresses
5. **Check `hasReverseRecord`** in reverse resolution - not all addresses have ENS names set

## Full API Reference

For complete API documentation with all parameters and responses, visit:
- **Docs**: https://docs.namespace.ninja/api-reference/resolvio/
- **Swagger**: https://resolvio.namespace.ninja/api/docs
- **OpenAPI**: https://resolvio.namespace.ninja/api/openapi.json
