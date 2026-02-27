---
name: wirex-baas-overview
description: "Wirex BaaS platform overview and getting started guide — start here for integration basics. Covers what Wirex BaaS is, the 5-step integration flow, API environments (sandbox and production URLs), sandbox test credentials, required headers (Authorization, X-Chain-Id), error handling, blockchain networks (Base, Stellar), canonical token and contract registry addresses, MCP server setup, and supported countries (77) with SEPA/ACH bank availability."
license: Apache-2.0
compatibility: "Requires network access. Compatible with Claude Code, Codex, and all Agent Skills spec tools."
metadata:
  author: wirex
  version: "1.0"
---

# Wirex BaaS Platform Overview

## Introduction

Wirex BaaS (Banking-as-a-Service) is a blockchain-native financial services platform that enables partners to embed comprehensive financial capabilities into their applications through REST APIs. The platform bridges traditional finance and decentralized infrastructure, providing a unified interface for:

- **Account Abstraction (AA) Wallets** -- Smart contract wallets deployed on-chain with gasless transactions, social recovery, and multi-signature support. Each user gets a deterministic wallet address derived from their identity.
- **KYC Verification** -- Identity verification flows supporting document checks, liveness detection, and regulatory compliance across 88+ countries. Powered by third-party providers with webhook-based status updates.
- **Visa Card Issuance** -- Virtual and physical Visa card provisioning, lifecycle management, transaction controls, and Apple Pay / Google Pay tokenization. Cards are funded from user wallets.
- **Crypto Asset Management** -- Multi-chain token support (USDC, USDT, EURC, WUSD, WEUR) with on-chain balances, transfers, and exchange operations across Base and Stellar networks.
- **Fiat Banking (SEPA/ACH)** -- Bank account creation, inbound/outbound transfers, and currency conversion. SEPA covers 28 European countries; ACH covers 34+ countries including all US states.

All services are accessed through a consistent REST API layer with OAuth2 authentication, standardized error handling, and blockchain transaction management.

---

## Integration Flow

Integrating with Wirex BaaS follows five sequential steps:

### Step 1: Partner Setup

Wirex provisions your partner account and issues credentials:

- **client_id** -- UUID identifying your application
- **client_secret** -- Secret string for OAuth2 token exchange
- **partner_id** -- 16-byte hex identifier used in on-chain operations (e.g., `0x00000000000000000000000000000007`)

You receive access to the Sandbox environment for development and testing.

### Step 2: Authentication

Establish server-to-server authentication using OAuth2 client credentials:

```
POST /api/v1/token
Content-Type: application/json

{
  "client_id": "<your-client-id>",
  "client_secret": "<your-client-secret>",
  "grant_type": "client_credentials"
}
```

The returned `access_token` is valid for 48 hours and is required as a Bearer token on all subsequent API calls.

### Step 3: On-Chain Configuration

Configure your on-chain environment:

1. Query the **Contract Registry** to discover deployed contract addresses (Accounts, FundsManagement, ExecutionDelayPolicy, TokensRegistry).
2. Verify token addresses and decimals from the TokensRegistry contract.
3. Set up webhook endpoints for asynchronous event notifications (transaction confirmations, KYC status changes, card events).

### Step 4: API Registration

Register your first user through the API:

1. Call `POST /api/v1/user` or `POST /api/v2/user` to create a user record.
2. The platform deploys an Account Abstraction wallet on the configured chain.
3. Obtain a user-scoped token via `POST /api/v1/user/authorize` for user-specific operations.

### Step 5: Operations

With authentication and user registration complete, you can:

- Initiate KYC verification flows
- Issue virtual or physical Visa cards
- Execute crypto transfers and exchanges
- Create bank accounts and process fiat transfers
- Query balances, transactions, and account status

---

## Environments

Wirex BaaS provides separate environments for development and production use.

### Core API Environments

| Environment | Base URL | Purpose |
|---|---|---|
| **Sandbox** | `https://api-baas.wirexapp.tech` | Development and testing with test data |
| **Production** | `https://api-baas.wirexapp.com` | Live operations with real funds |

### PCI-Compliant Card Environments

Card-related operations that handle sensitive PAN data use dedicated PCI DSS-compliant endpoints, specifically for card tokenization in push-to-card transfers:

| Environment | Base URL | Purpose |
|---|---|---|
| **PCI Sandbox** | `https://wx-acquiring-card-manager-uat.wirexapp.com` | Card testing |
| **PCI Production** | `https://wx-acquiring-card-manager.wirexapp.com` | Live card operations |

### Helper API

| Environment | Base URL | Purpose |
|---|---|---|
| **Helper API** | `https://ramc.wirexapp.tech` | Utility services and auxiliary operations |

### Blockchain Networks

Wirex BaaS operates on the following blockchain networks:

**Production Networks:**

| Network | Chain ID | Description |
|---|---|---|
| Base | `8453` | Ethereum L2 (Coinbase) -- primary EVM chain |
| Stellar | `9223372036854775806` | Stellar network for cross-border payments |

**Sandbox Networks:**

| Network | Chain ID | Description |
|---|---|---|
| Base Sepolia | `84532` | Base testnet for development |
| Stellar Testnet | `9223372036854775806` | Stellar testnet (same chain ID as production) |

> **Note:** The Stellar chain ID is the same value (`9223372036854775806`) in both sandbox and production. The environment base URL determines which Stellar network is used.

---

## Credentials

Partners receive three credentials upon onboarding:

| Credential | Format | Description |
|---|---|---|
| **client_id** | UUID string | Uniquely identifies your partner application. Passed in the token request body. |
| **client_secret** | Opaque string | Used alongside `client_id` to authenticate token requests. Must be stored securely and never exposed in client-side code. |
| **partner_id** | 16-byte hex string | On-chain identifier for your partner account (e.g., `0x00000000000000000000000000000007`). Used as the `parentEntity` parameter when registering user wallets in the Accounts smart contract, and in some API headers. Not secret but should be validated against expected value. Can be stored in application configuration. |

### Security Requirements

- Store `client_secret` in a secrets manager or encrypted environment variable. Never commit it to source control.
- Rotate credentials immediately if a compromise is suspected.
- Use separate credentials for Sandbox and Production environments.

---

## API Basics

### Protocol and Security

- **HTTPS only** -- All API communication uses HTTPS with TLS 1.2 or higher. Plain HTTP requests are rejected.
- **Content-Type** -- All request and response bodies use `application/json`.
- **Timeout** -- A 30-second timeout is recommended for all API calls. Some blockchain operations may take longer; use webhooks for async confirmation.

### Required Headers

Every API request must include the following headers:

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes | Bearer token from `/api/v1/token` (e.g., `Bearer eyJhbGciOi...`) |
| `X-Chain-Id` | Yes | Blockchain network identifier. Must match your target chain (e.g., `84532` for Sandbox Base Sepolia, `8453` for Production Base). **Required for all requests.** |
| `Content-Type` | Yes (for POST/PUT) | `application/json` |

Additional headers for user-specific operations:

| Header | When Required | Description |
|---|---|---|
| `X-User-Address` | User-specific calls | User's EOA (Externally Owned Account) address — **NOT the Smart Wallet address** |
| `X-User-Email` | User-specific calls | Email address of the user |
| `X-User-Id` | User-specific calls | Internal user identifier |

> Exactly **one** of `X-User-Address`, `X-User-Email`, or `X-User-Id` is required for endpoints that operate on a specific user. Providing multiple user identity headers in the same request is **rejected with an error**. User-agnostic endpoints (token, config, user creation) do not require these headers.

### User-Agnostic Endpoints

The following endpoints do not require user identity headers:

- `POST /api/v1/token` -- Authentication
- `POST /api/v1/user` -- User creation
- `POST /api/v2/user` -- User creation (v2)
- `GET /api/v1/config` -- Platform configuration
- `GET /api/v1/validation/rules` -- Validation rules

---

## MCP Server

Wirex BaaS provides a Model Context Protocol (MCP) server for AI-assisted development. The MCP server gives AI tools direct access to API documentation, endpoint specifications, and code generation assistance.

**MCP Server URL:** `https://docs.wirexapp.com/mcp`

The MCP server supports:

- **API Documentation Access** -- Query endpoint specifications, request/response schemas, and usage examples.
- **Documentation Search** -- Search across the full Wirex BaaS documentation corpus.
- **Real-Time Data** -- Access current API status, supported tokens, and configuration values.
- **Code Generation Assistance** -- Generate integration code snippets for common workflows.

For detailed setup instructions across different tools (Claude Code, Cursor, Claude Desktop, Windsurf), see the [MCP Setup Reference](references/MCP-SETUP.md).

---

## Supported Countries

Wirex BaaS supports operations in 77 countries across five regions:

| Region | Country Count | Examples |
|---|---|---|
| **Europe** | 35 | United Kingdom, Germany, France, Spain, Italy, Netherlands, Sweden, Switzerland, Poland, Austria, and 25 more |
| **Asia-Pacific** | 10 | Singapore, Japan, Hong Kong, Australia, Indonesia, Malaysia, Taiwan, Thailand, Vietnam, and more |
| **Latin America** | 7 | Brazil, Mexico, Argentina, Chile, Colombia, Peru, Ecuador |
| **Rest of World** | 24 | UAE, South Africa, Israel, Turkey, Nigeria, Saudi Arabia, South Korea, and 17 more |
| **United States** | 1 | Full coverage across all 50 states + DC |

For the complete list with ISO country codes, see the [Supported Regions Reference](references/SUPPORTED-REGIONS.md).

### Bank Account Availability

Fiat banking services are available through two networks:

- **SEPA (Single Euro Payments Area)** -- 28 European countries. Supports EUR-denominated transfers with T+1 settlement.
- **ACH (Automated Clearing House)** -- 34+ countries including all US states and territories. Supports USD-denominated transfers.

Availability depends on the user's country of residence and completed KYC level.

---

## Error Handling

All API errors follow a consistent JSON format:

```json
{
  "error_reason": "ErrorInvalidField",
  "error_description": "The provided X-Chain-Id header value is not supported.",
  "error_category": {
    "category": "CategoryValidationFailure",
    "http_status_code": 400
  },
  "error_details": [
    { "key": "field", "details": "X-Chain-Id" },
    { "key": "issue", "details": "unsupported_value" }
  ]
}
```

### Error Fields

| Field | Type | Description |
|---|---|---|
| `error_reason` | string | PascalCase error code (e.g., `ErrorInvalidField`, `ErrorPermissionDenied`, `ErrorNotFound`, `ErrorExpired`, `ErrorMissingField`, `ErrorAlreadyExists`, `ErrorNotSupported`, `ErrorConfigurationInvalid`) |
| `error_description` | string | Human-readable explanation of the error |
| `error_category` | object | Error classification object containing `category` (string) and `http_status_code` (integer) |
| `error_details` | array | Array of objects, each with `key` (string) and `details` (string) providing additional context |

### Error Categories

| Category | HTTP Status | Description | Recommended Action |
|---|---|---|---|
| `CategoryValidationFailure` | 400 | Invalid request parameters, missing headers, malformed data | Fix the request and retry |
| `CategoryUnauthorized` | 401 | Invalid or expired token, insufficient permissions | Re-authenticate and retry |
| `CategoryInternalFailure` | 500 | Server-side error | Retry with exponential backoff; contact support if persistent |
| `CategoryTransientFailure` | 429 | Rate limit exceeded or temporary service degradation | Retry after the delay indicated in `Retry-After` header |

### Error Handling Best Practices

1. **Always check the `error_category`** to determine retry strategy.
2. **Implement exponential backoff** for `CategoryTransientFailure` and `CategoryInternalFailure` errors.
3. **Do not retry** `CategoryValidationFailure` errors without modifying the request.
4. **Refresh tokens proactively** -- cache tokens and refresh 5 minutes before expiry to avoid `CategoryUnauthorized` errors.
5. **Log `error_reason` and `error_description`** for debugging and support escalations.

---

## References

- [Environments Reference](references/ENVIRONMENTS.md) -- Full environment URLs, chain IDs, contract addresses, and token configurations
- [MCP Setup Reference](references/MCP-SETUP.md) -- MCP server configuration for all supported AI tools
- [Supported Regions Reference](references/SUPPORTED-REGIONS.md) -- Complete country list with ISO codes and bank availability
