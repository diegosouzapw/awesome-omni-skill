---
name: atp
description: Create ATP Protocol servers (FastAPI + middleware) and clients. Use when building ATP-protected APIs, adding payment middleware, or writing clients that call ATP endpoints or the settlement service.
---

# ATP Protocol skill – servers and clients

Use this skill when you need to **create or modify ATP (Agent Trade Protocol) servers or clients**. ATP enables agent-to-agent payments on Solana: middleware on the server charges by token usage; the client pays and receives decrypted responses after settlement.

**Documentation index:** For full Claude Code docs (skills, agents, etc.), fetch: https://code.claude.com/docs/llms.txt

---

## 1. Creating an ATP server

### 1.1 Minimal FastAPI server with ATP middleware

- Use **FastAPI** and add **`ATPSettlementMiddleware`** so selected endpoints are paid per use.
- Endpoints must return **usage data** (input/output token counts) in a supported format so the middleware can compute and execute payment.

**Imports and app:**

```python
from fastapi import FastAPI
from fastapi.responses import JSONResponse
from atp.middleware import ATPSettlementMiddleware
from atp.schemas import PaymentToken

app = FastAPI(title="My ATP-Protected API")
```

**Add middleware (required args):**

```python
app.add_middleware(
    ATPSettlementMiddleware,
    allowed_endpoints=["/v1/chat", "/v1/completions"],  # exact path list
    input_cost_per_million_usd=10.0,   # USD per million input tokens
    output_cost_per_million_usd=30.0,  # USD per million output tokens
    recipient_pubkey="YourSolanaWalletPublicKeyHere",  # required; receives payment
    payment_token=PaymentToken.SOL,    # SOL or PaymentToken.USDC
)
```

**Optional middleware args:**

- `wallet_private_key_header`: Header name for wallet key (default `"x-wallet-private-key"`).
- `skip_preflight`: Skip Solana preflight (default `False`).
- `commitment`: Solana commitment (default `"confirmed"`).
- `settlement_service_url`: Facilitator URL (default from `ATP_SETTLEMENT_URL` or `https://facilitator.swarms.world`).
- `fail_on_settlement_error`: If `True`, raise on settlement failure (default `False`).
- `settlement_timeout`: Timeout in seconds (default from env or `300.0`).

**Protected endpoint example (return usage):**

```python
@app.post("/v1/chat")
async def chat(request: dict):
    message = request.get("message", "")
    # Your agent logic
    response_text = "Agent response here"
    return JSONResponse(content={
        "response": response_text,
        "usage": {
            "input_tokens": 100,
            "output_tokens": 50,
            "total_tokens": 150
        }
    })
```

**Supported usage formats (middleware/settlement parses these):**

- **OpenAI:** `usage.prompt_tokens`, `usage.completion_tokens`, `usage.total_tokens`
- **Anthropic:** `usage.input_tokens`, `usage.output_tokens`
- **Google/Gemini:** `usageMetadata.promptTokenCount`, `candidatesTokenCount`, `totalTokenCount`

Paths not in `allowed_endpoints` are not charged (e.g. `/v1/health`).

**Run server:**

```python
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 1.2 Server checklist

- Set `recipient_pubkey` to the Solana wallet that should receive payments.
- Ensure every protected handler returns a body that includes one of the supported usage structures.
- Optional: set `ATP_ENCRYPTION_KEY` (base64 Fernet) for consistent response encryption across restarts; otherwise a key is generated per process.
- For production: use `fail_on_settlement_error=False` and check `atp_settlement_status` on the client; never log or persist wallet private keys.

---

## 2. Creating an ATP client

### 2.1 Client for ATP-protected endpoints

Use **`ATPClient`** to call ATP-protected APIs: it adds the wallet header and can decrypt responses after payment.

**Imports and constructor:**

```python
from atp.client import ATPClient

client = ATPClient(
    wallet_private_key="[1,2,3,...]",  # or base58 string; required for payment
    settlement_service_url="https://facilitator.swarms.world",  # optional
    settlement_timeout=300.0,           # optional
    wallet_private_key_header="x-wallet-private-key",  # optional
    verbose=False,                     # optional, for debugging
)
```

**Making a paid request (POST):**

```python
import asyncio

response = asyncio.run(client.post(
    url="https://api.example.com/v1/chat",
    json={"message": "Hello!"},
    # wallet_private_key=...,  # optional override per request
    # auto_decrypt=True,       # default True
))
print(response["response"])        # agent output
print(response.get("atp_settlement"))   # payment details
print(response.get("atp_settlement_status"))  # e.g. "paid" or "failed"
```

**GET and other methods:**

```python
response = await client.get(url="https://api.example.com/v1/status")
response = await client.request("PUT", url=some_url, json=payload)
```

If `auto_decrypt=True` (default), the client decrypts encrypted response bodies when payment has succeeded.

### 2.2 Client for settlement service (facilitator) only

Use the same **`ATPClient`** for facilitator operations without calling an ATP-protected API.

**Health check (no wallet needed):**

```python
client = ATPClient()
health = asyncio.run(client.health_check())
```

**Parse usage (no wallet needed):**

```python
from atp.client import ATPClient
import asyncio

client = ATPClient()
usage = asyncio.run(client.parse_usage({"usage": {"prompt_tokens": 100, "completion_tokens": 50, "total_tokens": 150}}))
# usage -> e.g. {"input_tokens": 100, "output_tokens": 50, "total_tokens": 150}
```

**Calculate payment (no blockchain call):**

```python
from atp.schemas import PaymentToken

payment = await client.calculate_payment(
    usage={"input_tokens": 1000, "output_tokens": 500},
    input_cost_per_million_usd=10.0,
    output_cost_per_million_usd=30.0,
    recipient_pubkey="RecipientPublicKeyHere",
    payment_token=PaymentToken.SOL,
)
```

**Execute settlement (spends SOL/USDC):**

```python
result = await client.settle(
    usage={"input_tokens": 1000, "output_tokens": 500},
    input_cost_per_million_usd=10.0,
    output_cost_per_million_usd=30.0,
    recipient_pubkey="RecipientPublicKeyHere",
    payment_token=PaymentToken.SOL,
)
# result includes tx_signature, status, etc.
```

### 2.3 Client checklist

- For paid endpoint calls or `settle()`, provide `wallet_private_key` (constructor or per call).
- Wallet key format: JSON array string `"[1,2,3,...]"` or base58 keypair string.
- Check `atp_settlement_status` in responses to handle failures; never log or store private keys.
- Env vars (optional): `ATP_SETTLEMENT_URL`, `ATP_SETTLEMENT_TIMEOUT`, `ATP_WALLET_PRIVATE_KEY`, `ATP_ENDPOINT_URL`.

---

## 3. Project layout and references

- **Server examples:** `examples/server/example.py`, `examples/server/full_flow_example.py`
- **Client examples:** `examples/client/example_request.py`, `examples/client/example_settle.py`, `examples/client/example_health_check.py`
- **API reference:** `atp/middleware.py` (middleware), `atp/client.py` (client), `atp/schemas.py` (`PaymentToken`, config models)
- **Docs:** README.md in repo; full docs at https://docs.swarms.ai/docs/atp/overview

When creating or editing ATP servers or clients, follow the patterns above and use the existing examples and atp package as the source of truth.
