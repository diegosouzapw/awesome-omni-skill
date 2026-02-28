---
name: tator-trader
description: "Execute crypto trades using natural language via Tator's AI trading API. Use when: buying tokens, selling tokens, swapping, bridging cross-chain, sending tokens, wrapping/unwrapping ETH, opening perp positions, betting on prediction markets, launching tokens, registering blockchain names, or managing yield positions. Triggers: 'buy token', 'sell token', 'swap X for Y', 'bridge to', 'send tokens', 'open long', 'open short', 'bet on', 'launch token', 'register name', 'deposit yield', 'wrap ETH'. Supports 24 chains. Returns UNSIGNED transactions — you sign and broadcast. Costs $0.20 USDC per request via x402. Recommended wallet integration: Sponge (SPONGE_API_KEY) or AgentWallet (AGENTWALLET_API_TOKEN) — no raw private keys needed. This skill is a transaction builder and never accesses your private keys or tokens."
---

# Tator AI Trading API

Trade on 24 chains using natural language. Send a prompt like "Buy 0.1 ETH worth of PEPE on Base" — Tator returns UNSIGNED transactions for you to review, sign, and broadcast. $0.20 USDC per request via x402. Tator never touches your keys.

## Quick Reference

| Situation | Action |
|-----------|--------|
| User wants to buy/sell/swap a token | Build prompt with amount + token + chain, call Tator |
| User wants to bridge tokens | Include source chain, destination chain, amount in prompt |
| User wants to open a leveraged position | Include leverage, direction, collateral amount, protocol |
| User gives a vague prompt ("buy some crypto") | Ask for specifics: which token, how much, which chain |
| Response has `type: "transaction"` | **Verify each TX** (check `to`, `value`, `chainId`), then sign and broadcast IN ORDER |
| Response has `type: "error"` | Show error message to user, suggest fixes |
| Response has `type: "info"` | Display the information to user |
| Multiple transactions returned | Execute sequentially — wait for each to confirm before sending next |
| Unknown/new token | Use contract address instead of name in prompt |
| User hasn't scanned the token yet | **Scan first** with `quickintel-scan` skill ($0.03) before buying |

## Endpoint

`POST https://x402.quickintel.io/v1/tator/prompt`

```json
{
  "prompt": "Buy 0.1 ETH worth of PEPE on Base",
  "walletAddress": "0xYourPublicWalletAddress",
  "provider": "your-agent-name"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `prompt` | Yes | Natural language trading instruction |
| `walletAddress` | Yes | Your PUBLIC wallet address (receives tokens, signs TX) |
| `provider` | Yes | Your agent/app identifier |
| `async` | No | Returns job ID to poll. Default: `false` |
| `chain` | No | Preferred chain (e.g., "base", "ethereum") |
| `slippage` | No | Slippage tolerance %. Default: `1` |

**Cost:** $0.20 USDC (200000 atomic units) on any of 14 payment networks. Base recommended for lowest fees.

> **⚠️ NEVER paste private keys or seed phrases into prompts.** Tator only needs your PUBLIC wallet address. The `walletAddress` field is your public address — the one you'd share to receive tokens.

## Writing Good Prompts

Better prompts = better results. You pay $0.20 regardless of outcome.

| Good | Bad |
|------|-----|
| "Buy 0.1 ETH worth of PEPE on Base" | "Buy some crypto" |
| "Swap 100 USDC for ETH on Arbitrum" | "Swap tokens" |
| "Bridge 50 USDC from Base to Arbitrum" | "Bridge my tokens" |
| "Open 5x long on ETH with 100 USDC on Avantis" | "Go long ETH" |
| "Buy 0x1234...abcd on Base" (contract address for obscure tokens) | "Get me that new meme coin" |

**Tips:** Always include chain. Specify amounts. Use contract addresses for obscure tokens. For bridging, include source AND destination chain.

## Capabilities

| Category | Operations | Example |
|----------|-----------|---------|
| **Trading** | Buy, sell, swap | "Swap 100 USDC for ETH on Arbitrum" |
| **Transfers** | Send, wrap, unwrap, burn | "Send 50 USDC to 0x1234..." |
| **Bridging** | Cross-chain via Relay, LiFi, GasZip, deBridge | "Bridge 100 USDC from Base to Arbitrum" |
| **Perps** | Long/short via Avantis (Base) | "Open 5x long on ETH with 100 USDC" |
| **Prediction Markets** | Bet via Myriad | "Bet $10 on YES for 'Will ETH hit 5k?'" |
| **Token Launch** | Clanker (Base, Ethereum, Arbitrum, Unichain), Flaunch (Base), Pump.fun (Solana) | "Launch MYTOKEN with symbol MTK on Clanker" |
| **Names** | Basenames, MegaETH, Somnia | "Register myname.base" |
| **Yield** | Aave, Morpho, Compound, Yearn | "Deposit 1000 USDC into Aave on Base" |

## Supported Chains (24)

ethereum, base, arbitrum, optimism, polygon, avalanche, bsc, linea, sonic, berachain, abstract, unichain, ink, soneium, ronin, worldchain, sei, hyperevm, katana, somnia, plasma, monad, megaeth, solana

Use exact chain names as shown (e.g., `"base"` not `"Base"`, `"bsc"` not `"binance"`).

## Handling Responses

### Transaction Response (Most Common)

```json
{
  "type": "transaction",
  "transactions": [
    {
      "to": "0xContractAddress",
      "data": "0xCalldata...",
      "value": "100000000000000000",
      "chainId": 8453,
      "description": "Buy PEPE with 0.1 ETH on Base"
    }
  ],
  "message": "Transaction ready. Sign and broadcast to complete."
}
```

**Before signing every transaction, verify:**
1. **`to` address** — Should be a known DEX router, bridge contract, or the address YOU specified
2. **`value` field** — Should match the ETH/native amount from your prompt (for ERC-20 swaps this is usually `"0"`)
3. **`chainId`** — Should match the chain you requested
4. **`description`** — Should align with your original prompt
5. **For approvals** (`0x095ea7b3` selector) — Spender should be a known protocol contract

If anything looks wrong, **do not sign**. Ask Tator again with a more specific prompt or verify the contract on a block explorer.

### Error Response

```json
{
  "type": "error",
  "message": "Insufficient balance. You have 0.05 ETH but need 0.1 ETH.",
  "code": "INSUFFICIENT_BALANCE"
}
```

| Code | Fix |
|------|-----|
| `INSUFFICIENT_BALANCE` | Check token balance before calling |
| `UNSUPPORTED_CHAIN` | Use a supported chain from the list |
| `TOKEN_NOT_FOUND` | Use contract address instead of name |
| `INVALID_PROMPT` | Be more specific |
| `SLIPPAGE_TOO_HIGH` | Reduce trade size or increase slippage tolerance |

### Info Response

Returned for information queries (prices, balances). Display the `message` to the user.

## Wallet Integration

> **⚠️ Wallet Security:** Tator does NOT require your private key. The x402 payment uses YOUR agent's existing wallet. Use a **managed wallet service** to avoid raw key exposure entirely.

| Your setup | Use this | Key exposure |
|-----------|----------|-------------|
| Using Sponge Wallet | **Pattern A** below (recommended) | ✅ No raw keys |
| Using AgentWallet (frames.ag) | **Pattern B** below | ✅ No raw keys |
| Using Lobster.cash / Crossmint | See `references/REFERENCE.md` | ✅ No raw keys |
| Using Vincent Wallet | See `references/REFERENCE.md` | ✅ No raw keys |
| Need programmatic signing (viem, ethers, @x402/fetch, Solana) | See `references/REFERENCE.md` | ⚠️ Advanced — read security notes |
| Not sure / no wallet | Start with **Pattern A** (Sponge) | ✅ No raw keys |

### Pattern A: Sponge Wallet (Recommended — No Raw Keys)

```bash
curl -sS -X POST "https://api.wallet.paysponge.com/api/x402/fetch" \
  -H "Authorization: Bearer $SPONGE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://x402.quickintel.io/v1/tator/prompt",
    "method": "POST",
    "body": {
      "prompt": "Buy 0.1 ETH worth of PEPE on Base",
      "walletAddress": "0xYourSpongeWalletAddress",
      "provider": "sponge-agent"
    },
    "preferred_chain": "base"
  }'
```

Sponge handles the 402 payment flow automatically. You still verify and sign the returned trade transactions. Requires: `SPONGE_API_KEY` env var.

### Pattern B: AgentWallet (No Raw Keys)

```javascript
const response = await fetch('https://frames.ag/api/wallets/{username}/actions/x402/fetch', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.AGENTWALLET_API_TOKEN}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    url: 'https://x402.quickintel.io/v1/tator/prompt',
    method: 'POST',
    body: {
      prompt: 'Swap 100 USDC for ETH on Base',
      walletAddress: agentWalletAddress,
      provider: 'my-agent'
    }
  })
});
const result = await response.json();

// Verify and broadcast via AgentWallet
if (result.type === 'transaction') {
  for (const tx of result.transactions) {
    // VERIFY before signing — check to, value, chainId, description
    const broadcast = await fetch(
      'https://frames.ag/api/wallets/{username}/actions/send-transaction',
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${process.env.AGENTWALLET_API_TOKEN}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          chainId: tx.chainId, to: tx.to, data: tx.data, value: tx.value
        })
      }
    );
    console.log(`TX sent: ${(await broadcast.json()).hash}`);
  }
}
```

> For programmatic signing patterns (@x402/fetch, viem, ethers.js, Solana, Vincent), see `references/REFERENCE.md`. These require a dedicated hot wallet with minimal funds — never use your main wallet.

## Async Mode

For long-running operations, add `"async": true` to get a job ID:

```json
{ "type": "pending", "jobId": "job_abc123", "message": "Poll for result." }
```

Poll (FREE — no payment): `GET https://x402.quickintel.io/v1/tator/jobs/job_abc123`

## Scan Before You Buy

**Always scan unknown tokens before trading.** Use the `quickintel-scan` skill ($0.03) to check for honeypots, scams, and rug pull risks:

```javascript
// 1. Scan token ($0.03)
const scan = await scanToken(chain, tokenAddress);
if (scan.tokenDynamicDetails.is_Honeypot) throw new Error('HONEYPOT');
if (scan.quickiAudit.has_Scams) throw new Error('SCAM');

// 2. Only then trade ($0.20)
const trade = await callTator(`Buy 0.1 ETH worth of ${tokenAddress} on ${chain}`, wallet);
```

Total cost: $0.23 for a scanned, verified trade. See `references/REFERENCE.md` for the complete scan-then-buy example.

## Security Model

```
YOUR SIDE                          TATOR'S SIDE
─────────────                      ─────────────
• Private keys (never shared)      • Interprets your prompt
• Transaction review (before sign) • Constructs unsigned calldata
• Signing decision (your wallet)   • Returns TXs for your review
• Broadcasting (you submit)        • Charges $0.20 via x402

Tator NEVER receives your private key.
Tator CANNOT execute without your signature.
The worst Tator can do is return a bad transaction.
The worst YOU can do is sign it without checking.
```

## Payment Networks (14)

Pay on any of these networks: Base (recommended), Ethereum, Arbitrum, Optimism, Polygon, Avalanche, Unichain, Linea, Sonic, HyperEVM, Ink, Monad, MegaETH (USDM), Solana.

See `references/REFERENCE.md` for full USDC/USDM addresses per chain.

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| `402 Payment Required` | No payment header | Ensure wallet configured with $0.20+ USDC |
| `402 Signature verification failed` | Bad payload structure | Check references/REFERENCE.md common mistakes |
| `INSUFFICIENT_BALANCE` | Can't afford trade | Check balance first |
| `TOKEN_NOT_FOUND` | Unknown token | Use contract address |

## Important Notes

- **Payment charged regardless of outcome.** Use `payment-identifier` extension for safe retries (see references/REFERENCE.md).
- **Multi-TX requires sequential execution.** Approvals must confirm before swaps.
- **Bridge times vary.** 30 seconds to 30 minutes depending on protocol.
- **Perps have liquidation risk.** Leverage trading can lose your collateral.
- **Scan results are point-in-time.** Re-scan periodically for held tokens.

## Discovery Endpoint

Query accepted payments and schemas: `GET https://x402.quickintel.io/accepted`

## Cross-Reference

- **Token security scanning** → `quickintel-scan` skill ($0.03/scan)
- **Deep x402 implementation, all wallet patterns, transaction verification** → `references/REFERENCE.md`
- **Chain details, RPCs, block explorers** → `references/chains.md`

## About

Tator's endpoint (`x402.quickintel.io`) is operated by **Quick Intel LLC**, a registered US-based cryptocurrency security company. Over 50 million token scans processed. APIs power DexTools, DexScreener, and Tator Trader. Operational since April 2023.

- Tator Docs: https://docs.quickintel.io/tator
- x402 Protocol: https://www.x402.org
- Support: https://t.me/tatortrader