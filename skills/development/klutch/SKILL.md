---
name: klutch
description: Klutch Agentic Credit Card OpenClaw Skill. Manage virtual cards, transactions, and automated spending patterns.
metadata:
  openclaw:
    emoji: 💳
    requires:
      env: []
      optional_env:
        - KLUTCH_CLIENT_ID
        - KLUTCH_SECRET_KEY
        - KLUTCH_API_KEY
        - KLUTCH_API_SECRET
        - KLUTCH_1PASSWORD_ITEM
    install:
      - id: pip
        kind: pip
        requirements: requirements.txt
---

# Klutch Skill

OpenClaw skill for Klutch programmable credit card API integration.

## Overview

This skill provides a command-line interface for accessing Klutch credit card data through their GraphQL API. It supports viewing card information, transaction history, spending categories, and spending analysis.

## Prerequisites

1. **Klutch Account**: Active Klutch credit card account
2. **API Credentials**: Client ID and Secret Key from Klutch developer portal
3. **Python 3.10+**: Required for running the scripts

## Configuration

### Environment Variables

Set your Klutch API credentials:

```bash
# Option 1: Direct credentials
export KLUTCH_CLIENT_ID="your-client-id"
export KLUTCH_SECRET_KEY="your-secret-key"

# Option 2: 1Password CLI integration (requires 'op' CLI)
export KLUTCH_1PASSWORD_ITEM="Klutch API Credential"
```

### Configuration File

The skill stores configuration and session tokens in `~/.config/klutch/`:

```bash
~/.config/klutch/
├── config.json      # User preferences
└── token.json       # Cached session token (auto-managed)
```

### Configuration Options

Edit `~/.config/klutch/config.json` to customize:

```json
{
  "api": {
    "endpoint": "https://graphql.klutchcard.com/graphql",
    "timeout": 30
  }
}
```

## Commands Reference

### Balance

```bash
# Check card information
python scripts/klutch.py balance

# Example output:
{
  "cards": [
    {
      "id": "crd_xxx",
      "name": "Martin Kessler",
      "status": "ACTIVE"
    }
  ]
}
```

### Transactions

```bash
# List recent transactions (last 30 days)
python scripts/klutch.py transactions

# Limit results
python scripts/klutch.py transactions --limit 25

# Example output:
{
  "transactions": [
    {
      "id": "txn_xxx",
      "amount": -100.0,
      "merchantName": "Checking",
      "transactionStatus": "SETTLED"
    }
  ]
}
```

### Card Management

#### List Cards

```bash
python scripts/klutch.py card list
```

#### View Categories

```bash
python scripts/klutch.py card categories
```

#### View Spending by Category

```bash
python scripts/klutch.py card spending
```

### Configuration Management

```bash
# Get configuration value
python scripts/klutch.py config get api.timeout

# Set configuration value
python scripts/klutch.py config set api.timeout 60

# View all configuration
python scripts/klutch.py config get
```

## API Endpoints

The skill connects to Klutch's GraphQL API:

| Environment | Endpoint |
|-------------|----------|
| Production | `https://graphql.klutchcard.com/graphql` |
| Sandbox | `https://sandbox.klutchcard.com/graphql` |

## Authentication Flow

The skill uses Klutch's session token authentication:

1. **Initial Request**: Sends `createSessionToken` mutation with Client ID and Secret Key
2. **Token Caching**: Stores the JWT session token in `~/.config/klutch/token.json`
3. **Subsequent Requests**: Uses cached token until it expires
4. **Auto-Refresh**: Creates a new session token when the cached one fails

## Hypothetical Agent Use Cases

The Klutch skill enables agents to handle their own budget or provide personal finance assistance.

*   **Sub-Agent Budgeting**: Create a virtual card for a sub-agent to pay for its own usage (AWS, OpenAI) with a hard limit.
*   **Budget Guardrails**: Monitor spending categories (e.g., 'FOOD') and alert the user if they exceed a monthly budget.
*   **Transaction Alerts**: Watch for specific merchants or unusual activity and notify the user immediately.
*   **Expense Summary**: Summarize monthly spending and categorize transactions for personal journaling.

## Error Handling

The skill handles common error scenarios:

- **Authentication failures**: Prompts to verify credentials
- **Session expiration**: Automatically creates a new session token
- **Network errors**: Clear error messages with retry suggestions
- **GraphQL errors**: Detailed error messages from the API

## Integration with OpenClaw

### Using from OpenClaw Sessions

```bash
# OpenClaw can invoke the skill directly
klutch balance
klutch transactions --limit 5
klutch card list
```

## Troubleshooting

### Authentication Issues

If you receive authentication errors:
1. Verify your credentials with `python scripts/klutch.py config get`
2. Delete `~/.config/klutch/token.json` to force re-authentication
3. Check that your API credentials are correct

### Session Token Issues

Force token refresh:
```bash
rm ~/.config/klutch/token.json
```

## Security Notes

- Never commit credentials to version control
- The skill stores tokens in `~/.config/klutch/token.json`
- Session tokens are refreshed automatically when needed
