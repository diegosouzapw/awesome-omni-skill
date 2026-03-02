---
name: trustra-escrow
version: 1.0.0
description: Escrow as a Service for AI agents. Create trustless USDC escrow transactions on Solana.
homepage: https://trustra.xyz
metadata: {"emoji":"🔐","category":"payments","api_base":"https://api.trustra.xyz/api/v2"}
---

# Trustra Escrow 🔐

Trustless USDC escrow for agent-to-agent transactions on Solana.

## I Want To BUY Something (Pay Someone)

```bash
# 1. Register (once)
python register.py --name "My Agent"

# 2. Check your balance
python balance.py

# 3. Create escrow with seller's wallet
python escrow_create.py <SELLER_WALLET> <AMOUNT> -d "Payment for service"

# 4. Pay into escrow (funds held until delivery)
python escrow_pay.py <ESCROW_ID>

# 5. Wait for seller to deliver, then confirm to release funds
python escrow_confirm.py <ESCROW_ID>
```

**If there's a problem:** `python escrow_dispute.py <ESCROW_ID> --reason "Issue description"`

## I Want To SELL Something (Receive Payment)

```bash
# 1. Register (once)
python register.py --name "My Agent"

# 2. Share your wallet address with buyer
python balance.py  # Shows your wallet address

# 3. Wait for buyer to create & pay escrow
python escrow_list.py --status paid

# 4. After delivering service/product, mark as delivered (12h after payment)
python escrow_deliver.py <ESCROW_ID>

# 5. Wait for buyer to confirm (or 7 days auto-release)
python escrow_withdraw.py <ESCROW_ID>  # After 7 days if no response
```

## Quick Reference

| Action | Command |
|--------|---------|
| Register | `python register.py --name "Agent Name"` |
| Balance | `python balance.py` |
| Create escrow | `python escrow_create.py <WALLET> <AMOUNT> [-d "desc"]` |
| Pay escrow | `python escrow_pay.py <ID>` |
| List escrows | `python escrow_list.py [--status STATUS]` |
| Mark delivered | `python escrow_deliver.py <ID>` (seller) |
| Confirm release | `python escrow_confirm.py <ID>` (buyer) |
| Dispute | `python escrow_dispute.py <ID> --reason "..."` |
| Cancel | `python escrow_cancel.py <ID>` (buyer, before delivery) |
| Withdraw | `python escrow_withdraw.py <ID>` (seller, after 7d) |
| Export key | `python export_key.py` |

## Escrow Flow

```
BUYER creates escrow → BUYER pays → (12h wait) → SELLER delivers → BUYER confirms
                                                                 ↘ Funds released to SELLER

If problem: Either party can DISPUTE → Trustra resolves
If no response: SELLER can WITHDRAW after 7 days
```

## Escrow Statuses

| Status | Who acts next? |
|--------|----------------|
| `created` | Buyer pays |
| `paid` | Seller delivers (after 12h wait) |
| `delivered` | Buyer confirms (or wait 7d) |
| `completed` | Done - funds released |
| `disputed` | Trustra team resolves |
| `canceled` | Escrow canceled |
| `withdrawn` | Seller got funds after 7d |

## Time Constraints

| Constraint | Duration | Purpose |
|------------|----------|---------|
| Cancel window | 12 hours | Buyer can cancel within 12h after paying |
| Seller deliver | After 12h | Seller can only mark delivered after cancel window |
| Auto-release | 7 days | Seller can withdraw if buyer doesn't respond |

## Setup (one-time)

```bash
python register.py --name "My Agent"
```

Creates a managed wallet + API key stored in `credentials.json`. Fund wallet with SOL (for tx fees) and USDC to use escrows.

## Errors

| Error | Fix |
|-------|-----|
| `No API key found` | Run `register.py` |
| `Escrow not found` | Wrong ID or you're not buyer/seller |
| `Invalid status` | Check `escrow_list.py` for current status |
| `CancelDurationNotEnded` | Wait 12 hours after payment to mark delivered |
| `Too early to withdraw` | Wait 7 days after delivery |

## Credentials

```json
{
  "api_key": "trustra_sk_...",
  "wallet_address": "7xKXtg..."
}
```

Never share your API key.
