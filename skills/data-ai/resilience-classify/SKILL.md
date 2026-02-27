---
name: resilience-classify
description: Research and classify stablecoins for resilience sub-factor overrides (chainRisk, collateralQuality, custodyModel). Run after types/defaults are implemented to identify coins needing explicit overrides.
---

# Resilience Classification Skill

Identify stablecoins where the default inference (from backing + governance) is incorrect and apply overrides.

## When to Invoke

- After the resilience types and default inference are implemented
- When a new stablecoin is added to the tracker
- When auditing resilience scores for accuracy

## Process

### Step 1 — Identify candidates

Read all coins from `src/lib/stablecoins.ts`. For each, apply the default inference rules (see `inferResilienceDefaults()` in `src/lib/report-cards.ts`). Flag coins where the default is likely wrong based on:

- `collateral` text containing keywords: "Solana", "tBTC", "WBTC", "delta-neutral", "perpetual", "CEX", "off-exchange", "Copper", "Ceffu", "Fireblocks", "bridged"
- `pegMechanism` text containing: "Solana", "Bitcoin L2", "not Ethereum", "Tron"
- `contracts[]` listing only non-Ethereum chains
- `backing` = `crypto-backed` but collateral text mentions RWAs, bridges, or exotic strategies
- Coins on this known-override list: HYUSD, USDe, meUSD, USDD, sUSD (Synthetix), USDJ

### Step 2 — Research each candidate

For each flagged coin, in parallel:
- `WebFetch` official docs for collateral composition, custody arrangement, and chain architecture
- `WebSearch` for `"{coin name}" stablecoin collateral custody chain` to find independent analysis
- Cross-reference with existing `collateral` and `pegMechanism` text fields

### Step 3 — Classify

For each coin, determine the correct tier:

| Sub-factor | Question | Tiers |
|---|---|---|
| `chainRisk` | Where does the core protocol live and where is collateral held? | `ethereum` (100), `stage1-l2` (66), `established-alt-l1` (33), `unproven` (0) |
| `collateralQuality` | What are the trust assumptions in backing assets? | `native` (100), `eth-lst` (66), `alt-lst-bridged-or-mixed` (33), `exotic` (0) |
| `custodyModel` | Who holds the collateral and can it be verified on-chain? | `onchain` (100), `institutional` (50), `cex` (0) |

**Classification rules:**
- **chainRisk**: Based on where the protocol's smart contracts and collateral vaults live, NOT where the token is bridged to
- **collateralQuality**: For mixed collateral, use the tier of the riskiest significant component (>15% of backing). Stablecoin portions don't count here (handled by dependency risk)
- **custodyModel**: If ANY significant portion is held off-chain by a non-institutional custodian, classify as `cex`
- When uncertain between two tiers, choose the riskier (lower score) tier

### Step 4 — Present findings

For each coin needing an override, present:

```
## {Name} ({Symbol}) — ID: {id}

### Default inference
- chainRisk: {inferred} — {correct/wrong because...}
- collateralQuality: {inferred} — {correct/wrong because...}
- custodyModel: {inferred} — {correct/wrong because...}

### Proposed overrides
- {field}: {value} — {justification with source URL}

### No override needed
- {fields where default is correct}
```

### Step 5 — Apply

After user approval, edit `src/lib/stablecoins.ts` to add only the override fields that differ from defaults. Example:

```typescript
usd("123", "Example", "EX", "crypto-backed", "decentralized", {
  // ... existing fields ...
  chainRisk: "established-alt-l1",
  collateralQuality: "alt-lst-bridged",
}),
```

Run `npm run build` to verify.
