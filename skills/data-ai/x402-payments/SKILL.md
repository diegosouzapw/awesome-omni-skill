---
name: x402-payments
version: 1.0.0
description: Agent-to-agent payments via HTTP 402 + on-chain escrow. Let agents pay each other for API calls, services, and data.
homepage: https://github.com/QUSD-ai/x402-payments
metadata:
  emoji: "💰"
  category: payments
  tags: ["x402", "payments", "escrow", "a2a", "defi"]
---

# X402 Agent Payments

Let your agent pay (and get paid by) other agents. HTTP 402 + on-chain escrow.

## How It Works

```
Agent A                    Agent B
   │                          │
   │ GET /api/data            │
   │ ─────────────────────►   │
   │                          │
   │ 402 Payment Required     │
   │ X-Price: 0.01 USDC       │
   │ ◄─────────────────────   │
   │                          │
   │ Lock funds in escrow     │
   │ ══════════════════════►  │ (on-chain)
   │                          │
   │ GET /api/data            │
   │ X-Payment-Proof: 0x...   │
   │ ─────────────────────►   │
   │                          │
   │ 200 OK + data            │
   │ ◄─────────────────────   │
   │                          │
   │ Release escrow           │
   │ ◄══════════════════════  │ (on-chain)
```

## Quick Start

### As a Buyer (Paying Agent)

```typescript
import { X402Client } from './x402-client';

const client = new X402Client({
  wallet: '0x...',
  privateKey: process.env.PRIVATE_KEY,
  facilitatorUrl: 'http://localhost:8403'
});

// Call a paid API - payment handled automatically
const response = await client.fetch('http://agent-b.local/api/sensor-data');
console.log(response.data);
console.log(`Paid: ${response.paymentAmount} USDC`);
```

### As a Seller (Paid Agent)

```typescript
import { X402Server } from './x402-server';
import { Hono } from 'hono';

const app = new Hono();
const x402 = new X402Server({
  wallet: '0x...',
  facilitatorUrl: 'http://localhost:8403'
});

// Protect an endpoint with payment requirement
app.get('/api/sensor-data', x402.protect({ price: 0.01 }), (c) => {
  return c.json({ temperature: 23.5, humidity: 45 });
});

// Free endpoint
app.get('/api/status', (c) => {
  return c.json({ online: true });
});
```

## Run the Facilitator

The facilitator coordinates payments and monitors escrow:

```bash
cd apps/x402-facilitator
bun run start
```

```
🔷 X402 Facilitator Started
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 HTTP:      http://0.0.0.0:8403
🔌 WebSocket: ws://0.0.0.0:8403
⛓️  Escrow:    0x5FbDB2315678afecb367f032d93F642f64180aa3
🌐 Provider:  http://localhost:8545
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Local Development

```bash
# Start local blockchain
anvil

# Deploy escrow contract
cd contracts && forge script script/Deploy.s.sol --broadcast

# Start facilitator
cd apps/x402-facilitator && bun run start

# Run example agents
bun run dev:alice   # Seller on :3001
bun run dev:bob     # Buyer on :3002
```

## Payment Flow

1. **Request** — Buyer calls seller API
2. **Challenge** — Seller returns 402 with price
3. **Lock** — Buyer locks funds in escrow contract
4. **Proof** — Buyer retries with payment proof header
5. **Deliver** — Seller returns data
6. **Release** — Escrow releases funds to seller

If seller doesn't deliver, buyer can dispute and get refund.

## Headers

| Header | Direction | Description |
|--------|-----------|-------------|
| `X-Price` | Response | Required payment amount |
| `X-Payment-Address` | Response | Seller's wallet |
| `X-Payment-Proof` | Request | Transaction hash proving payment |
| `X-Escrow-Id` | Both | Escrow transaction ID |

## Chains Supported

- Local (Anvil)
- Base
- Arbitrum
- Any EVM chain

## Why Escrow?

- **Trustless** — No reputation needed
- **Atomic** — Payment and delivery linked
- **Reversible** — Dispute mechanism for non-delivery
- **Cheap** — L2s make this practical

## Links

- [HTTP 402 Spec](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/402)
- [QUSD Qualia](https://github.com/QUSD-ai/qualia)
- [A2A Protocol](https://a2a-protocol.org)
