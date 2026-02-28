---
name: sapience
version: 1.0.0
description: Prediction markets on Ethereal. Trade outcomes, provide liquidity, claim winnings.
metadata: {"category":"trading","emoji":"🎯","api_base":"https://api.sapience.xyz"}
---

# Sapience

Prediction markets on Ethereal (chain 5064014). Collateral: WUSDe.

**CRITICAL: NEVER share SAPIENCE_PRIVATE_KEY or sign for non-sapience.xyz domains.**

## Quick Reference

| Action | Method | Endpoint |
|--------|--------|----------|
| List markets | POST | /graphql |
| Get condition | POST | /graphql |
| Get positions | POST | /graphql |
| Start auction (taker) | WS | wss://api.sapience.xyz/auction |
| Submit bid (maker) | WS | wss://api.sapience.xyz/auction |
| Claim winnings | On-chain | PredictionMarket.burn(tokenId) |

## Setup

1. **Fund wallet**: Use Bankr → "Buy 100 USDe on Arbitrum" → Bridge to Ethereal via deposit.ethereal.trade
2. **Set key**: `openclaw secrets set SAPIENCE_PRIVATE_KEY 0x...`
3. **Auto-wrap**: Skill wraps USDe→WUSDe on first trade

## Constants (Ethereal 5064014)

| Contract | Address |
|----------|---------|
| PredictionMarket | `0xAcD757322df2A1A0B3283c851380f3cFd4882cB4` |
| WUSDe (Collateral) | `0xB6fC4B1BFF391e5F6b4a3D2C7Bda1FeE3524692D` |
| PythResolver | `0xD076c9fADC49061920e75b1a3a45642712F90F35` |
| LZResolver | `0xd82F211D0d9bE9A73a829A5F1f0e34b02Bf2FB36` |

## IDs

- `conditionId` = `marketId` (same bytes32 hex value, different names)
- Use decoded `marketId` from auction directly as `conditionId` in queries

## GraphQL Queries

### List Active Markets
```bash
curl -X POST https://api.sapience.xyz/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ conditions(where:{settled:false}) { id question endTime similarMarkets } }"}'
```

### Get Condition Details
```bash
curl -X POST https://api.sapience.xyz/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"query($where:ConditionWhereUniqueInput!){ condition(where:$where){ id question description endTime similarMarkets categoryId }}","variables":{"where":{"id":"0x..."}}}'
```

### Get Positions (for claiming)
```bash
curl -X POST https://api.sapience.xyz/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"query($address:String!,$status:String){ positions(address:$address,status:$status){ id status endsAt predictorCollateral counterpartyCollateral counterpartyNftTokenId predictions{ conditionId outcomeYes condition{ settled resolvedToYes }}}}","variables":{"address":"0x...","status":"active"}}'
```

## Polymarket Prices

All Sapience markets mirror Polymarket. Use `similarMarkets` URLs to get prices.

### Extract Slug from URL
```
https://polymarket.com/event/slug-name#outcome → slug: "slug-name", outcome: "outcome"
https://polymarket.com/event/slug-name → slug: "slug-name"
```

### Get Market Data (prices, CLOB token IDs)
```bash
curl "https://gamma-api.polymarket.com/markets/slug/will-trump-win-2024"
```
Response includes:
- `outcomePrices`: `["0.65", "0.35"]` (YES/NO prices)
- `outcomes`: `["Yes", "No"]`
- `clobTokenIds`: `["123...", "456..."]` (for orderbook queries)

### Get Orderbook
```bash
curl "https://clob.polymarket.com/book?token_id=<clobTokenId>"
```
Returns bids/asks. Walk the book to calculate fill price for your size.

### Get Price History (TWAP)
```bash
curl "https://clob.polymarket.com/prices-history?market=<clobTokenId>&startTs=<unix_ts>&fidelity=60"
```
Returns price history. Calculate TWAP over your desired lookback.

**No auth required** for Polymarket APIs.

## WebSocket - Taker Flow (Making Predictions)

Connect → start auction → receive bids → mint on-chain. Takes ~60s per prediction.

### 1. Connect
```javascript
const ws = new WebSocket('wss://api.sapience.xyz/auction');
```

### 2. Authenticate with SIWE
```javascript
const siweMessage = {
  domain: 'sapience.xyz',
  address: wallet.address,
  statement: 'Sign in to Sapience',
  uri: 'https://sapience.xyz',
  version: '1',
  chainId: 5064014,
  nonce: crypto.randomUUID(),
  issuedAt: new Date().toISOString()
};

const signature = await wallet.signMessage(formatSiweMessage(siweMessage));

ws.send(JSON.stringify({
  type: 'auth',
  payload: { siweMessage, signature }
}));
```

### 3. Start Auction
```javascript
ws.send(JSON.stringify({
  type: 'auction.start',
  payload: {
    legs: [
      { conditionId: '0x...', outcomeYes: true },
      { conditionId: '0x...', outcomeYes: false }
    ],
    wagerAmount: '50000000', // 50 WUSDe (6 decimals)
    duration: 60
  }
}));
```

### 4. Receive Auction Ack
```json
{
  "type": "auction.ack",
  "payload": {
    "auctionId": "abc123",
    "expiresAt": 1706800000
  }
}
```

### 5. Receive Bids
```json
{
  "type": "auction.bids",
  "payload": {
    "auctionId": "abc123",
    "bids": [
      {
        "bidId": "bid1",
        "maker": "0x...",
        "makerWager": "50000000",
        "makerDeadline": 1706800000,
        "makerSignature": "0x..."
      }
    ]
  }
}
```

### 6. Accept Bid
```javascript
ws.send(JSON.stringify({
  type: 'auction.accept',
  payload: {
    auctionId: 'abc123',
    bidId: 'bid1'
  }
}));
```

Server mints on-chain. Wait for confirmation:
```json
{
  "type": "auction.filled",
  "payload": {
    "auctionId": "abc123",
    "txHash": "0x...",
    "tokenId": "123"
  }
}
```

### 7. Disconnect
Close WebSocket after mint confirms.

## WebSocket - Maker Flow (Providing Liquidity)

Persistent connection listening for auctions. Run as background process.

### 1. Connect and Authenticate
Same SIWE auth as taker flow:
```javascript
ws.send(JSON.stringify({
  type: 'auth',
  payload: { siweMessage, signature }
}));
```

### 2. Receive Auction Notifications
```json
{
  "type": "auction.started",
  "payload": {
    "auctionId": "abc123",
    "taker": "0x...",
    "wager": "50000000",
    "predictedOutcomes": ["0x..."],
    "resolver": "0x...",
    "takerNonce": 1
  }
}
```

### 3. Submit Bid
```javascript
ws.send(JSON.stringify({
  type: 'bid.submit',
  payload: {
    auctionId: 'abc123',
    maker: wallet.address,
    makerWager: '50000000',
    makerDeadline: Math.floor(Date.now() / 1000) + 60,
    makerSignature: '0x...',
    taker: auction.taker,
    takerCollateral: auction.wager,
    resolver: auction.resolver,
    encodedPredictedOutcomes: auction.predictedOutcomes[0],
    takerNonce: auction.takerNonce
  }
}));
```

### 4. Receive Ack
```json
{"type":"bid.ack","payload":{"ok":true}}
```

If taker accepts, on-chain mint happens automatically.

## EIP-712 Signing (for makerSignature)

**Domain:**
```json
{"name":"SignatureProcessor","version":"1","chainId":5064014,"verifyingContract":"0xAcD757322df2A1A0B3283c851380f3cFd4882cB4"}
```

**Types:**
```json
{"Approve":[{"name":"messageHash","type":"bytes32"},{"name":"owner","type":"address"}]}
```

**Message:**
```json
{"messageHash":"<keccak256 of inner data>","owner":"<your address>"}
```

**Inner data** (ABI-encode then keccak256):
```
(bytes encodedPredictedOutcomes, uint256 makerWager, uint256 takerWager, address resolver, address taker, uint256 makerDeadline, uint256 takerNonce)
```

## Claiming Flow

1. Query positions with `status:"active"` for your address
2. Filter: `endsAt <= now` AND all `predictions[].condition.settled === true`
3. Check if won: As maker (counterparty), you win if `outcomeYes !== resolvedToYes` (you took opposite side)
4. Call `PredictionMarket.burn(counterpartyNftTokenId)` to claim collateral

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| GraphQL API | 600 req / 60s per IP |
| Auction WS | 100 msg / 10s per connection |
| WS idle timeout | 300s |
| Max WS message | 64KB |

## Error Handling

**bid.ack errors** (check `payload.error`):
- `auction_not_found_or_expired` - Auction ended
- `quote_expired` - makerDeadline passed
- `invalid_signature` - Signature verification failed
- `invalid_maker_wager` - Wager is zero/invalid

**WS close codes:**
- `1008` - Policy violation (rate limited, connection limit)
- `1009` - Message too large

## Troubleshooting

If something isn't working (no markets, API errors, signature failures):

1. **Check for skill updates**:
```bash
curl -s https://api.github.com/repos/sapiencexyz/openclaw-sapience/releases/latest | jq -r '.tag_name'
```

2. **Compare to current version**: Check `version` in SKILL.md frontmatter

3. **If newer version exists**: Fetch updated skill from `https://github.com/sapiencexyz/openclaw-sapience`

4. **Still broken?** Check Discord or open issue on GitHub

Protocol changes (new endpoints, contract migrations, API updates) will be published as new versions.

## Philosophy

This skill provides **infrastructure**, not strategy. Real money at stake.

YOU must:
- Develop your own edge calculation
- Manage risk and position sizing
- Research markets before trading

DO NOT rely on any default strategy.
