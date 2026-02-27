---
name: agent-cc
description: Agent Credit Cards - give your agents single and multiuse credit cards so they can securely make purchases with fixed balances.
version: 1.0.0
---

# Agent Credit Cards

You have access to virtual credit cards via BotWallet.ai. This allows you to securely make purchases online using single-use or multi-use cards with fixed spending limits.

## Setup

If not already configured, run:
```bash
agent-cc onboard
```

## Commands

### List All Cards
```bash
agent-cc list
```

### Get Full Card Details
Retrieve card number, CVV, and expiry to make a purchase:
```bash
agent-cc get <card_token>
```

This returns the full card details needed to fill payment forms. **Note:** Calling `get` does NOT use the card - the card is only "used" when a merchant charges it. For `SINGLE_USE` cards, the card closes after the first successful transaction.

### Create a New Card
```bash
agent-cc create <memo> <type> <limit>
```

**Parameters:**
- `memo`: Description (e.g., "Netflix", "Amazon purchase")
- `type`: `SINGLE_USE` or `UNLOCKED`
- `limit`: Spending limit in dollars (e.g., `20` for $20)

**Examples:**
```bash
agent-cc create "Netflix subscription" SINGLE_USE 20
agent-cc create "Shopping card" UNLOCKED 100
```

### View Transactions
```bash
agent-cc transactions
```

## Card Types

| Type | Behavior | Best For |
|------|----------|----------|
| `SINGLE_USE` | Closes after one transaction | One-time purchases, trials, unknown merchants |
| `UNLOCKED` | Multiple transactions allowed | Recurring payments, trusted merchants |

## Making a Purchase - REQUIRED WORKFLOW

**CRITICAL: You MUST get explicit user confirmation before creating a card or making any purchase.**

### Step 1: Identify the Purchase
Gather all details:
- Item/service being purchased
- Exact price
- Merchant/website

### Step 2: Confirm with User
Before creating any card, show the user:
```
I'm ready to make this purchase:
- Item: [item description]
- Price: $[amount]
- Merchant: [website/company]

Do you want me to proceed?
```

**Wait for explicit confirmation before continuing.**

### Step 3: Create the Card
Only after confirmation:
```bash
agent-cc create "[item] from [merchant]" SINGLE_USE [price + buffer]
```

### Step 4: Get Card Details
```bash
agent-cc get <token_from_step_3>
```

### Step 5: Fill Payment Form
Use browser to fill the checkout form with:
- **Card Number**: The 16-digit card number
- **CVV/CVC**: The 3-digit security code
- **Expiry**: Month and year
- **Name on Card**: "BotWallet Agent" or user's name

### Step 6: Complete Purchase
Submit the form and confirm success.

## Security Best Practices

1. **Always use SINGLE_USE** cards unless multi-use is needed
2. **Set spend limits** slightly above purchase price ($22 for a $19.99 item)
3. **Never create cards** without user confirmation
4. **Use descriptive memos** to track card usage

## Example Scenarios

### One-time Purchase
User: "Buy me this $29.99 item from example.com"

1. Confirm: "I'll purchase this $29.99 item from example.com. Proceed?"
2. After "yes": `agent-cc create "Item from example.com" SINGLE_USE 35`
3. Get details: `agent-cc get <token>`
4. Fill checkout form
5. Submit payment

### Subscription Sign-up
User: "Sign me up for this $9.99/month service"

1. Confirm: "I'll sign up for $9.99/month (recurring). Proceed?"
2. After "yes": `agent-cc create "Monthly subscription" UNLOCKED 15`
3. Get details and complete sign-up

### Just Create a Card
User: "Create a $50 card for later"

1. Ask: "Single-use or multi-use?"
2. Create: `agent-cc create "General use" SINGLE_USE 50`
3. Return card details to user
