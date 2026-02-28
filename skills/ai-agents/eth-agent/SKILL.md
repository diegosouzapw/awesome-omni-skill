---
name: eth-agent
description: Expert knowledge for using the eth-agent library - the simplest, safest way for AI agents to send stablecoins on Ethereum. Use when writing code that interacts with Ethereum, sends tokens, or builds AI agent payment systems.
argument-hint: [task description]
---

# eth-agent Library Expert

You are an expert in the **eth-agent** library - a TypeScript library that provides the simplest, safest way for AI agents to interact with Ethereum and send stablecoins.

## Core Philosophy

- **Simplicity**: Send USDC in one line, not 15+
- **Safety First**: Built-in spending limits, human approval, address policies
- **AI-Centric**: Structured errors with suggestions, predictable APIs
- **Human-in-the-Loop**: Easy integration with approval workflows

## Installation

```bash
npm install @lambdaclass/eth-agent
```

## Quick Start

```typescript
import { AgentWallet } from '@lambdaclass/eth-agent';

const wallet = AgentWallet.create({
  privateKey: process.env.ETH_PRIVATE_KEY,
  rpcUrl: 'https://eth.llamarpc.com',
});

// Send stablecoins (human-readable amounts, no decimals needed)
await wallet.sendUSDC({ to: 'alice.eth', amount: '100' });
await wallet.sendUSDT({ to: 'bob.eth', amount: '50.25' });

// Send ETH
await wallet.send({ to: 'alice.eth', amount: '0.1 ETH' });
```

## Stablecoins

The library has first-class support for major stablecoins across chains:

```typescript
import { USDC, USDT, USDS, DAI, PYUSD, FRAX } from '@lambdaclass/eth-agent';

// Each token has: symbol, name, decimals, and addresses per chain
// Amounts are always human-readable strings - NO manual decimal handling

await wallet.sendUSDC({ to: 'alice.eth', amount: '100' });     // Sends 100 USDC
await wallet.sendUSDT({ to: 'bob.eth', amount: '50.25' });     // Sends 50.25 USDT

// Generic stablecoin method
await wallet.sendStablecoin({ token: DAI, to: 'carol.eth', amount: '1000' });

// Check balances (returns formatted string like "1,234.56")
const balance = await wallet.getStablecoinBalance(USDC);
const allBalances = await wallet.getStablecoinBalances(); // All stablecoins
```

### Supported Chains for Stablecoins

USDC, USDT, DAI are available on: Ethereum, Arbitrum, Optimism, Base, Polygon, Avalanche

## Safety Features

### Spending Limits

```typescript
const wallet = AgentWallet.create({
  privateKey: KEY,
  rpcUrl: URL,
  limits: {
    perTransaction: '100',      // Max per single tx (in token units or ETH)
    perHour: '500',             // Hourly spending cap
    perDay: '2000',             // Daily spending cap
    emergencyStopBelow: '10',   // Halt if balance drops below this
  },
});

// Check current limits and usage
const limits = await wallet.getLimits();
console.log(limits.daily.remaining); // How much left today
```

### Human Approval

```typescript
const wallet = AgentWallet.create({
  privateKey: KEY,
  rpcUrl: URL,
  approvalConfig: {
    requireApprovalWhen: {
      amountExceeds: '50',        // Amounts over $50 need approval
      recipientIsNew: true,       // New recipients need approval
      recipientNotInTrusted: true // Non-trusted addresses need approval
    },
    trustedAddresses: ['0x...', 'alice.eth'],
  },
  onApprovalRequired: async (request) => {
    // Integrate with Slack, email, or UI
    console.log(`Approval needed: ${request.summary}`);
    return await askHumanForApproval(request);
  },
});
```

### Address Policies

```typescript
const wallet = AgentWallet.create({
  privateKey: KEY,
  rpcUrl: URL,
  addressPolicy: {
    mode: 'allowlist',  // or 'blocklist'
    addresses: ['0x...', 'alice.eth', 'bob.eth'],
  },
});
```

## Error Handling

### Safe Methods (Result Type)

All methods have `safe*` variants that return Result types instead of throwing:

```typescript
import { isOk, isErr, matchResult } from '@lambdaclass/eth-agent';

const result = await wallet.safeSendUSDC({ to: 'alice.eth', amount: '100' });

if (isOk(result)) {
  console.log(`Success! TX: ${result.value.hash}`);
} else {
  console.log(`Error: ${result.error.code}`);
  console.log(`Suggestion: ${result.error.suggestion}`);
}
```

### Pattern Matching

```typescript
const message = matchResult(result)
  .ok(r => `Sent! TX: ${r.hash}`)
  .errWith({ code: 'INSUFFICIENT_FUNDS' }, e => `Need more: ${e.details.shortage}`)
  .errWith({ code: 'DAILY_LIMIT_EXCEEDED' }, () => 'Wait until tomorrow')
  .errWith({ code: 'APPROVAL_REQUIRED' }, () => 'Human approval needed')
  .err(e => e.suggestion)
  .run();
```

### Structured Errors

All errors have:
- `code`: Machine-readable (e.g., `DAILY_LIMIT_EXCEEDED`)
- `message`: Human-readable description
- `suggestion`: Recovery action
- `retryable`: Boolean
- `retryAfter`: Milliseconds to wait (if applicable)

## Transaction Preview

Preview transactions before sending:

```typescript
const preview = await wallet.preview({
  to: 'alice.eth',
  amount: '0.5 ETH',
});

console.log(preview.canExecute);           // boolean
console.log(preview.costs.total.eth);      // Total cost including gas
console.log(preview.costs.gas.eth);        // Gas cost
console.log(preview.blockers);             // Array of reasons if can't execute
console.log(preview.warnings);             // Non-blocking warnings
```

## Smart Accounts (Gasless Transactions)

For gasless and batch operations using ERC-4337:

```typescript
import { SmartAgentWallet, createRemotePaymaster } from '@lambdaclass/eth-agent';

const smartWallet = SmartAgentWallet.create({
  privateKey: KEY,
  rpcUrl: URL,
  bundlerUrl: 'https://bundler.example.com',
  paymaster: createRemotePaymaster({ url: PAYMASTER_URL }),
});

// Send without needing ETH for gas
await smartWallet.sendUSDCGasless({ to: 'alice.eth', amount: '100' });

// Batch multiple transfers in one transaction
await smartWallet.sendStablecoinBatch({
  token: USDC,
  transfers: [
    { to: 'alice.eth', amount: '50' },
    { to: 'bob.eth', amount: '30' },
    { to: 'carol.eth', amount: '20' },
  ],
});
```

## Payment Watching

Monitor incoming payments:

```typescript
import { USDC, USDT } from '@lambdaclass/eth-agent';

// Use wallet's built-in payment watching methods
const watcher = wallet.onStablecoinReceived((payment) => {
  console.log(`Received ${payment.formattedAmount} ${payment.token.symbol}`);
}, { tokens: [USDC, USDT] });

// Stop watching when done
watcher.stop();

// Or wait for a specific payment
const payment = await wallet.waitForPayment({
  token: USDC,
  minAmount: '100',  // Human-readable
  timeout: 60000,    // 60 seconds
});
```

## Cross-Chain Bridging

Bridge stablecoins between chains using the unified BridgeRouter:

```typescript
import { USDC } from '@lambdaclass/eth-agent';

// Simple one-liner - auto-selects best bridge
const result = await wallet.bridge({
  token: USDC,
  amount: '100',
  destinationChainId: 42161,  // Arbitrum
});

console.log(result.trackingId);  // Use for status tracking
console.log(result.summary);     // Human-readable summary
```

### Route Preferences

Control how the router selects bridges:

```typescript
// Prefer speed over cost
const fast = await wallet.bridge({
  token: USDC,
  amount: '500',
  destinationChainId: 8453,  // Base
  preference: {
    priority: 'speed',
    maxSlippageBps: 50,  // Max 0.5% slippage
  },
});

// Force specific protocol (bypasses auto-selection)
const viaCCTP = await wallet.bridge({
  token: USDC,
  amount: '1000',
  destinationChainId: 10,  // Optimism
  protocol: 'CCTP',
});
```

### Compare Routes Before Bridging

```typescript
const routes = await wallet.compareBridgeRoutes({
  token: USDC,
  amount: '1000',
  destinationChainId: 8453,
});

console.log(routes.recommendation.reason);  // "CCTP: lowest fees ($0)"
for (const quote of routes.quotes) {
  console.log(`${quote.protocol}: ${quote.fee.totalUSD} USD fee`);
}
```

### Preview Bridge with Validation

```typescript
const preview = await wallet.previewBridgeWithRouter({
  token: USDC,
  amount: '1000',
  destinationChainId: 42161,
});

if (preview.canBridge) {
  console.log(`Ready to bridge. Fee: $${preview.quote?.fee.totalUSD}`);
  console.log(`Needs approval: ${preview.needsApproval}`);
} else {
  console.log('Cannot bridge:', preview.blockers.join(', '));
}
```

### Track Bridge Status

```typescript
// After initiating a bridge
const result = await wallet.bridge({ ... });

// Use tracking ID to check status (works across protocols)
const status = await wallet.getBridgeStatusByTrackingId(result.trackingId);
console.log(`Progress: ${status.progress}%`);
console.log(`Message: ${status.message}`);

// Wait for completion
const attestation = await wallet.waitForBridgeByTrackingId(result.trackingId);
console.log('Bridge completed!');
```

### Safe Bridge (Result Type)

```typescript
const result = await wallet.safeBridge({
  token: USDC,
  amount: '100',
  destinationChainId: 42161,
});

if (isOk(result)) {
  console.log(`Success! Tracking: ${result.value.trackingId}`);
} else {
  console.log(`Error: ${result.error.code}`);
  console.log(`Suggestion: ${result.error.suggestion}`);
}
```

### Supported Bridge Protocols

| Protocol | Tokens | Speed | Fees | Notes |
|----------|--------|-------|------|-------|
| CCTP (Circle) | USDC | 10-20 min | $0 | No slippage, 1:1 burn/mint |
| Stargate | USDC, USDT | 5-15 min | ~0.06% | Has slippage |
| Across | USDC, USDT | 2-5 min | Variable | Instant delivery |

### Legacy USDC-Only Method

For direct CCTP bridging without route selection:

```typescript
// Direct CCTP bridge (backward compatible)
const result = await wallet.bridgeUSDC({
  amount: '100',
  destinationChainId: 42161,
});

// Check status
const status = await wallet.getBridgeStatus(result.messageHash);
```

## AI Framework Integration

### Anthropic (Claude)

```typescript
import { AgentWallet } from '@lambdaclass/eth-agent';
import { anthropicTools } from '@lambdaclass/eth-agent/integrations';
import Anthropic from '@anthropic-ai/sdk';

const wallet = AgentWallet.create({ privateKey: KEY, rpcUrl: URL });
const tools = anthropicTools(wallet);

const response = await client.messages.create({
  model: 'claude-sonnet-4-20250514',
  tools: tools.definitions,
  messages: [{ role: 'user', content: 'Send 10 USDC to alice.eth' }],
});

// Execute tool calls
for (const block of response.content) {
  if (block.type === 'tool_use') {
    const result = await tools.execute(block.name, block.input);
  }
}
```

### OpenAI

```typescript
import { openaiTools } from '@lambdaclass/eth-agent/integrations';

const tools = openaiTools(wallet);
// Use with OpenAI function calling
```

### LangChain

```typescript
import { langchainTools } from '@lambdaclass/eth-agent/integrations';

const tools = langchainTools(wallet);
// Use with LangChain agents
```

## Key Imports

```typescript
// Core
import { AgentWallet, SmartAgentWallet } from '@lambdaclass/eth-agent';

// Stablecoins
import { USDC, USDT, USDS, DAI, PYUSD, FRAX, STABLECOINS } from '@lambdaclass/eth-agent';

// Result types
import { ok, err, isOk, isErr, matchResult, unwrap } from '@lambdaclass/eth-agent';

// Units
import { ETH, GWEI, WEI, parseUnits, formatUnits } from '@lambdaclass/eth-agent';

// AI integrations (subpath import)
import { anthropicTools, openaiTools, langchainTools } from '@lambdaclass/eth-agent/integrations';

// Payment watching (use wallet methods instead)
// wallet.onStablecoinReceived() and wallet.waitForPayment()

// Smart accounts
import { createRemotePaymaster, createVerifyingPaymaster } from '@lambdaclass/eth-agent';

// Bridging (use wallet.bridge() for simple cases, BridgeRouter for advanced)
import { BridgeRouter, type RoutePreference } from '@lambdaclass/eth-agent';
```

## Common Patterns

### 1. Simple Payment Agent

```typescript
const wallet = AgentWallet.create({
  privateKey: process.env.ETH_PRIVATE_KEY,
  rpcUrl: process.env.RPC_URL,
  limits: { perTransaction: '100', perDay: '1000' },
});

async function payUser(recipient: string, amount: string) {
  const result = await wallet.safeSendUSDC({ to: recipient, amount });
  if (isOk(result)) {
    return { success: true, txHash: result.value.hash };
  }
  return { success: false, error: result.error.suggestion };
}
```

### 2. Approval-Gated Payments

```typescript
const wallet = AgentWallet.create({
  privateKey: KEY,
  rpcUrl: URL,
  approvalConfig: {
    requireApprovalWhen: { amountExceeds: '50' },
  },
  onApprovalRequired: async (req) => {
    // Post to Slack and wait for response
    return await slackApprovalFlow(req.summary);
  },
});
```

### 3. Multi-Recipient Batch

```typescript
const smartWallet = SmartAgentWallet.create({ ... });

await smartWallet.sendStablecoinBatch({
  token: USDC,
  transfers: recipients.map(r => ({ to: r.address, amount: r.amount })),
});
```

### 4. Cross-Chain Payment with Bridge

```typescript
async function sendCrossChain(
  recipient: string,
  amount: string,
  destChainId: number
) {
  // Preview first to check feasibility
  const preview = await wallet.previewBridgeWithRouter({
    token: USDC,
    amount,
    destinationChainId: destChainId,
    recipient,
  });

  if (!preview.canBridge) {
    return { success: false, error: preview.blockers.join(', ') };
  }

  // Execute bridge
  const result = await wallet.safeBridge({
    token: USDC,
    amount,
    destinationChainId: destChainId,
    recipient,
  });

  if (isOk(result)) {
    return {
      success: true,
      trackingId: result.value.trackingId,
      summary: result.value.summary,
    };
  }
  return { success: false, error: result.error.suggestion };
}
```

## Task: $ARGUMENTS

Based on the above knowledge, help with the requested task.
