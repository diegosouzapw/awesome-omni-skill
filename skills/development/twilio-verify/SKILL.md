---
name: twilio-verify
cluster: twilio
description: "Verify: 2FA SMS/voice/email, TOTP, phone verification, Verify Guard fraud prevention, SNA"
tags: ["2fa","otp","verify","twilio"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: false
scope: general
model_hint: claude-sonnet
embedding_hint: "twilio verify 2fa otp totp phone verification fraud sna"
---

# twilio-verify

## Purpose

Enable OpenClaw to implement and operate Twilio Verify (V2) in production: SMS/voice/email OTP, TOTP, custom channels, phone verification, Verify Fraud Guard (risk scoring + blocking), and Silent Network Authentication (SNA) where available. This skill focuses on:

- Building a **reliable verification pipeline** (send → check → enforce) with **rate limits**, **fraud controls**, and **observability**.
- Integrating Verify with **Programmable Messaging/Voice**, **SendGrid**, and **webhook-driven** status/telemetry.
- Handling **real Twilio failure modes** (carrier filtering, invalid E.164, auth errors, rate limits) with deterministic remediation.
- Operating at scale: **cost controls**, **regional routing**, **idempotency**, and **abuse prevention**.

---

## Prerequisites

### Accounts & Twilio resources

- Twilio account with:
  - **Account SID** (`AC...`)
  - **Auth Token** (or API Key + Secret)
  - A **Verify Service SID** (`VA...`) created in Twilio Console → Verify → Services
- If using SMS/Voice:
  - A verified **Messaging Service** (recommended) or phone number(s)
  - If US A2P: **10DLC** registration completed for your brand/campaign (or use toll-free/short code as appropriate)
- If using email channel:
  - SendGrid account + verified sender domain, or Twilio Verify Email channel configuration (depending on your setup)
- If using SNA:
  - SNA availability depends on region/carrier and Twilio enablement; confirm in Console and with Twilio support.

### Local tooling (exact versions)

- Node.js **20.11.1** (LTS) or **18.19.1** (LTS)
- Python **3.12.2** (if using Python examples)
- Twilio helper libraries:
  - `twilio` npm package **4.22.0**
  - `twilio` Python package **9.0.5**
- HTTP tooling:
  - `curl` **8.5.0**
  - `jq` **1.7**
- Optional (recommended):
  - `ngrok` **3.13.1** for webhook testing
  - `openssl` **3.0.13** for signature verification utilities

### Auth setup (recommended patterns)

Prefer **API Key** auth over Auth Token for server-side apps.

- Create API Key in Twilio Console → Account → API keys & tokens:
  - API Key SID (`SK...`)
  - API Key Secret (store once)
- Store secrets in a secret manager (AWS Secrets Manager, GCP Secret Manager, Vault). Do not commit to repo.

Environment variables expected by examples:

- `TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
- `TWILIO_AUTH_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` (if using Auth Token)
- `TWILIO_API_KEY_SID=SKxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
- `TWILIO_API_KEY_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
- `TWILIO_VERIFY_SERVICE_SID=VAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

---

## Core Concepts

### Verify Service

A Verify Service (`VA...`) is the policy boundary for:

- Channels enabled (sms, call, email, whatsapp, push, custom)
- Code length, locale templates, TTL, rate limits
- Fraud Guard configuration (risk thresholds, blocking)
- Webhooks (status callbacks, events)

Treat a Verify Service as an environment-scoped resource:
- `VA_prod...` for production
- `VA_staging...` for staging
- Separate services for different products/tenants only if policy differs materially.

### Verification vs Verification Check

- **Verification**: sending a challenge (OTP) to a destination (phone/email) via a channel.
- **Verification Check**: validating the user-provided code (or other factor) against the verification attempt.

Your application should:
1. Create verification (send)
2. Accept user input
3. Create verification check (verify)
4. Enforce outcome (issue session token, mark phone verified, etc.)

### Channels

Common channels:
- `sms`: OTP via SMS
- `call`: OTP via voice call (TwiML-driven by Twilio)
- `email`: OTP via email (Verify email channel or custom)
- `totp`: time-based one-time password (app-based)
- `whatsapp`: WhatsApp OTP (requires WhatsApp enablement)
- `custom`: your own delivery mechanism (push, in-app, etc.)

Channel selection should be policy-driven:
- Default to `sms`
- Offer `call` fallback for deliverability
- Offer `email` for account recovery or when phone is unavailable
- Offer `totp` for high-assurance accounts

### E.164 normalization

Twilio expects phone numbers in **E.164** format: `+14155552671`.

Do not accept raw user input directly. Normalize and validate:
- Use libphonenumber (Node: `google-libphonenumber` or `libphonenumber-js`)
- Store canonical E.164 in DB
- Reject ambiguous numbers early

### Rate limiting & abuse controls

Verify has built-in rate limiting, but you should also implement:
- Per-IP and per-identity throttles (Redis token bucket)
- Device fingerprinting / risk scoring
- Cooldowns after failed attempts
- CAPTCHA gating for suspicious traffic

### Fraud Guard (Verify Fraud Guard)

Fraud Guard helps detect:
- SIM swap risk
- High-risk destinations
- Traffic anomalies

Integrate Fraud Guard decisions into your auth flow:
- Block high-risk verifications
- Step-up to stronger factor (TOTP) if medium risk
- Log risk signals for incident response

### Webhooks & eventing

Use webhooks for:
- Verification status events
- Delivery outcomes (for messaging/voice)
- Audit trails and analytics

Design webhooks as:
- Idempotent (dedupe by event SID)
- Authenticated (Twilio signature validation)
- Retry-safe (Twilio retries on non-2xx)

### Silent Network Authentication (SNA)

SNA verifies a user’s phone number via carrier network signals without OTP entry (where supported). Treat it as:
- A **step-up** or **frictionless** verification path
- Not universally available; implement fallback to OTP
- Subject to carrier/region constraints and privacy requirements

---

## Installation & Setup

### Official Python SDK — Verify

**Repository:** https://github.com/twilio/twilio-python  
**PyPI:** `pip install twilio` · **Supported:** Python 3.7–3.13

```python
from twilio.rest import Client
client = Client()

SERVICE_SID = os.environ["TWILIO_VERIFY_SERVICE_SID"]

# Start verification (SMS / WhatsApp / email / TOTP)
verification = client.verify.v2.services(SERVICE_SID) \
    .verifications.create(to="+15558675309", channel="sms")
print(verification.status)

# Check code
check = client.verify.v2.services(SERVICE_SID) \
    .verification_checks.create(to="+15558675309", code="123456")
print(check.status)  # "approved" | "pending"
```

Source: [twilio/twilio-python — verify](https://github.com/twilio/twilio-python/blob/main/twilio/rest/verify/)

### Ubuntu 22.04 LTS (x86_64)

```bash
sudo apt-get update
sudo apt-get install -y curl jq ca-certificates gnupg

# Node.js 20.x (NodeSource)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

node -v   # v20.11.1 (or later 20.x)
npm -v

# Python 3.12 (deadsnakes PPA)
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install -y python3.12 python3.12-venv python3.12-dev
python3.12 --version
```

### Fedora 39 (x86_64)

```bash
sudo dnf install -y curl jq nodejs python3.12 python3.12-devel
node -v
python3.12 --version
```

### macOS 14 (Sonoma) — Intel & Apple Silicon

```bash
brew update
brew install node@20 python@3.12 jq curl openssl@3

# Ensure PATH includes brew Node/Python
node -v
python3.12 --version
```

### Project dependencies (Node.js)

```bash
mkdir -p verify-service && cd verify-service
npm init -y
npm install twilio@4.22.0 express@4.18.3 pino@9.0.0 zod@3.22.4
npm install --save-dev tsx@4.7.1 typescript@5.3.3 @types/express@4.17.21
```

### Project dependencies (Python)

```bash
mkdir -p verify-service-py && cd verify-service-py
python3.12 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip==24.0
pip install twilio==9.0.5 fastapi==0.109.2 uvicorn==0.27.1 pydantic==2.6.1
```

### Environment configuration

Create a local env file (do not commit):

- Node: `/etc/openclaw/twilio-verify.env` (production) or `./.env` (local)
- Python: same

Example (local):

```bash
cat > ./.env <<'EOF'
TWILIO_ACCOUNT_SID=AC2f7c2c6b2d2f4a1b9a0b3c1d2e3f4a5
TWILIO_API_KEY_SID=YOUR_API_KEY_SID
TWILIO_API_KEY_SECRET=9b2c3d4e5f60718293a4b5c6d7e8f9a0
TWILIO_VERIFY_SERVICE_SID=VA0a1b2c3d4e5f60718293a4b5c6d7e8f
TWILIO_AUTH_TOKEN=use_api_key_in_prod_if_possible
APP_ENV=local
EOF
```

Load it:

```bash
set -a
source ./.env
set +a
```

---

## Key Capabilities

### Send OTP via SMS/Voice/Email (Verify V2)

Core operation: create a Verification.

- SMS:
  - Best default for consumer sign-in
  - Watch for carrier filtering and A2P compliance
- Voice:
  - Fallback when SMS fails
  - Ensure user experience: language/voice, repeat code, DTMF handling if needed
- Email:
  - Useful for account recovery or when phone is not available
  - Ensure SPF/DKIM/DMARC alignment if using SendGrid/custom email

Node example (send):

```ts
// src/send.ts
import twilio from "twilio";

const accountSid = process.env.TWILIO_ACCOUNT_SID!;
const apiKeySid = process.env.TWILIO_API_KEY_SID!;
const apiKeySecret = process.env.TWILIO_API_KEY_SECRET!;
const verifyServiceSid = process.env.TWILIO_VERIFY_SERVICE_SID!;

const client = twilio(apiKeySid, apiKeySecret, { accountSid });

export async function sendOtp(to: string, channel: "sms" | "call" | "email") {
  const verification = await client.verify.v2
    .services(verifyServiceSid)
    .verifications.create({
      to,
      channel,
      locale: "en",
    });

  return {
    sid: verification.sid,
    status: verification.status, // "pending"
    to: verification.to,
    channel: verification.channel,
  };
}
```

### Check OTP (Verification Check)

Create a Verification Check with the user-provided code.

Node example (check):

```ts
// src/check.ts
import twilio from "twilio";

const accountSid = process.env.TWILIO_ACCOUNT_SID!;
const apiKeySid = process.env.TWILIO_API_KEY_SID!;
const apiKeySecret = process.env.TWILIO_API_KEY_SECRET!;
const verifyServiceSid = process.env.TWILIO_VERIFY_SERVICE_SID!;

const client = twilio(apiKeySid, apiKeySecret, { accountSid });

export async function checkOtp(to: string, code: string) {
  const check = await client.verify.v2
    .services(verifyServiceSid)
    .verificationChecks.create({ to, code });

  return {
    sid: check.sid,
    status: check.status, // "approved" or "pending"/"canceled"
    valid: check.valid,
  };
}
```

Enforcement rule (typical):
- Accept only `status === "approved"` and `valid === true`
- On failure, increment local counters and apply cooldowns

### TOTP enrollment and verification

Use Verify TOTP for app-based codes. Typical flow:
1. Create a TOTP factor (enrollment)
2. Display QR code / secret to user
3. Verify initial code
4. Store factor SID and bind to user

Note: Twilio Verify TOTP APIs are part of Verify V2 “Entities/Factors” model. Ensure your account has access and you’re using the correct endpoints.

Operational guidance:
- Treat factor SIDs as secrets (they’re identifiers, but still sensitive)
- Allow multiple factors per user (device migration)
- Provide recovery codes outside Twilio (your system)

### Custom channels (email/push/in-app)

Use Verify “custom” channel when you deliver the code yourself but want Twilio to manage:
- Code generation
- TTL
- Attempt limits
- Verification checks

Pattern:
- Request verification with `channel=custom`
- Twilio returns a code (or you fetch it via API depending on configuration)
- Deliver via your channel (push, in-app)
- Verify via Verification Check

This is useful when:
- You have an existing push infrastructure
- You want consistent policy enforcement across channels

### Fraud Guard integration

Use Fraud Guard signals to:
- Block verification attempts to high-risk destinations
- Require step-up (TOTP) for medium risk
- Alert on spikes per ASN/country

Implementation pattern:
- On “send verification” request:
  - Evaluate risk (Twilio + your own)
  - If blocked: return 403 with generic message
  - Else: proceed

### Rate limiting and throttling

Combine:
- Twilio Verify service rate limits
- Application-level throttles

Recommended minimums:
- Per IP: 5 sends / 10 minutes
- Per destination: 3 sends / 10 minutes
- Per identity (user id): 5 checks / 10 minutes
- Global circuit breaker on Twilio 5xx spikes

### Webhook-driven observability

Use:
- Verify event webhooks (where configured)
- Messaging status callbacks for SMS delivery outcomes (if using Messaging)
- Voice status callbacks for call outcomes

Store:
- verification SID
- destination hash (HMAC)
- channel
- status transitions
- error codes

---

## Command Reference

This section assumes direct REST usage via `curl` and helper library usage. Twilio does not provide an official “twilio verify” CLI with full parity; use REST calls or helper SDKs.

### REST: Create a Verification (send OTP)

Endpoint:
- `POST https://verify.twilio.com/v2/Services/{ServiceSid}/Verifications`

Auth:
- Basic auth with API Key SID/Secret (preferred) or Account SID/Auth Token

Flags/fields (important):
- `To` (string): destination (`+14155552671` or email)
- `Channel` (string): `sms|call|email|whatsapp|custom`
- `Locale` (string): e.g. `en`, `es`, `fr`
- `ChannelConfiguration.*` (object): channel-specific config (varies)
- `CustomFriendlyName` (string): label for logs/UX
- `RateLimits.*` (object): service-level overrides (if enabled)
- `RiskCheck` / Fraud Guard fields (if enabled; account-dependent)

Example (SMS):

```bash
curl -sS -X POST "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" \
  --data-urlencode "To=+14155552671" \
  --data-urlencode "Channel=sms" \
  --data-urlencode "Locale=en"
```

Example (Voice call):

```bash
curl -sS -X POST "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" \
  --data-urlencode "To=+14155552671" \
  --data-urlencode "Channel=call" \
  --data-urlencode "Locale=en"
```

Example (Email):

```bash
curl -sS -X POST "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" \
  --data-urlencode "To=alice@example.com" \
  --data-urlencode "Channel=email" \
  --data-urlencode "Locale=en"
```

### REST: Create a Verification Check (validate OTP)

Endpoint:
- `POST https://verify.twilio.com/v2/Services/{ServiceSid}/VerificationCheck`

Fields:
- `To` (string): same destination used for send
- `Code` (string): user-provided OTP
- `VerificationSid` (string): optional in some flows; prefer `To+Code` unless you store SID

Example:

```bash
curl -sS -X POST "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/VerificationCheck" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" \
  --data-urlencode "To=+14155552671" \
  --data-urlencode "Code=123456"
```

### REST: List Verifications (audit/debug)

Endpoint:
- `GET https://verify.twilio.com/v2/Services/{ServiceSid}/Verifications`

Query params (common):
- `To` (string)
- `Status` (string): `pending|approved|canceled`
- `Channel` (string)
- `DateCreated` (date filter; Twilio-style)
- `PageSize` (int): up to 1000 depending on endpoint

Example:

```bash
curl -sS "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications?To=%2B14155552671&PageSize=50" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" | jq .
```

### REST: Fetch a Verification by SID

Endpoint:
- `GET https://verify.twilio.com/v2/Services/{ServiceSid}/Verifications/{VerificationSid}`

```bash
curl -sS "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications/VExxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" | jq .
```

### REST: Cancel a Verification (invalidate)

Endpoint:
- `POST https://verify.twilio.com/v2/Services/{ServiceSid}/Verifications/{VerificationSid}` with `Status=canceled`

```bash
curl -sS -X POST "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications/VExxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" \
  --data-urlencode "Status=canceled"
```

### REST: Verify Service configuration (read)

Endpoint:
- `GET https://verify.twilio.com/v2/Services/{ServiceSid}`

```bash
curl -sS "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" | jq .
```

### Twilio Node helper library: client initialization options

`twilio(apiKeySid, apiKeySecret, { accountSid, region, edge, logLevel })`

Important options:
- `accountSid`: required when using API Key auth
- `region`: e.g. `us1`, `ie1`, `au1` (data residency/latency)
- `edge`: e.g. `ashburn`, `dublin` (latency optimization)
- `logLevel`: `debug|info|warn|error` (avoid debug in prod)

Example:

```ts
import twilio from "twilio";
const client = twilio(process.env.TWILIO_API_KEY_SID!, process.env.TWILIO_API_KEY_SECRET!, {
  accountSid: process.env.TWILIO_ACCOUNT_SID!,
  region: "us1",
  edge: "ashburn",
});
```

---

## Configuration Reference

### OpenClaw skill config (example)

Path:
- `/etc/openclaw/skills/twilio/twilio-verify.toml`

```toml
# /etc/openclaw/skills/twilio/twilio-verify.toml
[twilio]
account_sid = "AC2f7c2c6b2d2f4a1b9a0b3c1d2e3f4a5"
auth_mode = "api_key" # "api_key" | "auth_token"
api_key_sid_env = "TWILIO_API_KEY_SID"
api_key_secret_env = "TWILIO_API_KEY_SECRET"
auth_token_env = "TWILIO_AUTH_TOKEN"
region = "us1"
edge = "ashburn"

[verify]
service_sid = "VA0a1b2c3d4e5f60718293a4b5c6d7e8f"
default_channel = "sms"
fallback_channels = ["call", "email"]
default_locale = "en"
code_ttl_seconds = 600

[rate_limits]
# App-level throttles (in addition to Twilio)
send_per_ip_per_10m = 5
send_per_to_per_10m = 3
check_per_identity_per_10m = 5
cooldown_seconds_after_failed_check = 60

[fraud_guard]
enabled = true
block_on_high_risk = true
step_up_on_medium_risk = true
step_up_channel = "totp"

[logging]
redact_fields = ["to", "email", "code", "auth_token", "api_key_secret"]
log_level = "info"
```

### Node service config (example)

Path:
- `./config/verify.config.json`

```json
{
  "twilio": {
    "region": "us1",
    "edge": "ashburn"
  },
  "verify": {
    "serviceSid": "VA0a1b2c3d4e5f60718293a4b5c6d7e8f",
    "defaultLocale": "en",
    "channels": ["sms", "call", "email"]
  },
  "security": {
    "hmacKeyEnv": "VERIFY_DESTINATION_HMAC_KEY",
    "webhookAuthTokenEnv": "TWILIO_AUTH_TOKEN"
  }
}
```

### systemd unit (production)

Path:
- `/etc/systemd/system/openclaw-verify.service`

```ini
[Unit]
Description=OpenClaw Verify Gateway
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=openclaw
Group=openclaw
EnvironmentFile=/etc/openclaw/twilio-verify.env
WorkingDirectory=/opt/openclaw/verify-gateway
ExecStart=/usr/bin/node /opt/openclaw/verify-gateway/dist/server.js
Restart=on-failure
RestartSec=2
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/var/lib/openclaw /var/log/openclaw
AmbientCapabilities=
CapabilityBoundingSet=
LockPersonality=true
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
```

---

## Integration Patterns

### Compose with Programmable Messaging status callbacks

Even when using Verify, you may need delivery telemetry. Pattern:
- Use Verify for OTP generation and checking
- Use Messaging status callbacks for delivery outcomes (if your setup routes via Messaging Service)
- Correlate by `to` hash + timestamp window + verification SID

Pipeline:
1. `POST /verify/send` → Twilio Verify creates verification
2. Twilio sends SMS
3. Messaging status callback hits `/webhooks/sms-status`
4. Store `MessageSid`, `MessageStatus`, `ErrorCode` (e.g., `30003`)
5. If repeated failures, auto-switch to voice/email

### Compose with Voice IVR fallback

If SMS fails:
- Offer voice call OTP
- If voice fails, route to agent or require TOTP

If you already have IVR state machines:
- Keep Verify call OTP separate from IVR calls
- Use IVR only for support flows; Verify call is optimized for OTP

### Compose with SendGrid for custom email channel

If you need branded email beyond Verify templates:
- Use Verify `custom` channel to generate code
- Send email via SendGrid dynamic templates
- Verify via Verification Check

SendGrid dynamic template example (Handlebars):

```json
{
  "personalizations": [
    {
      "to": [{ "email": "alice@example.com" }],
      "dynamic_template_data": {
        "code": "123456",
        "ttl_minutes": 10
      }
    }
  ],
  "from": { "email": "no-reply@example.com", "name": "Example Security" },
  "template_id": "d-13b8f94f2b2a4c4f9a8d0a1b2c3d4e5f"
}
```

### CI/CD: smoke test Verify in staging

In pipeline:
- Deploy staging
- Run a smoke test that:
  - Sends OTP to a test phone (or uses test credentials)
  - Checks OTP using a controlled channel (Twilio test credentials do not send real SMS)
- Gate production deploy on:
  - Twilio API auth success
  - Verify service reachable
  - Webhook endpoint signature verification passes

### Data model integration

Store:
- `user_id`
- `destination_e164` (encrypted at rest)
- `destination_hash` (HMAC for logs)
- `verify_service_sid`
- `last_verification_sid`
- `verified_at`
- `failed_attempts`
- `cooldown_until`

Do not store OTP codes.

---

## Error Handling & Troubleshooting

Handle Twilio errors by **code**, not by string matching, but include exact messages for operator recognition.

### 1) 20003 — Authentication Error

**Message:**
`Twilio could not authenticate the request. Please check your credentials.`

**Root causes:**
- Wrong API Key Secret/Auth Token
- Using API Key without `accountSid` in SDK init
- Clock skew rarely affects auth but can affect signature validation

**Fix:**
- Verify env vars and secret manager values
- For Node SDK with API Key: pass `{ accountSid }`
- Rotate API key if leaked

### 2) 20429 — Too Many Requests

**Message:**
`Rate limit exceeded`

**Root causes:**
- Verify service rate limits hit
- Burst traffic (bot attack)
- Repeated resend loops in client

**Fix:**
- Implement app-level throttles and exponential backoff
- Add resend cooldown UI (e.g., 30–60s)
- Consider separate Verify Services for distinct traffic classes only if policy differs

### 3) 21211 — Invalid 'To' Phone Number

**Message:**
`The 'To' number +1415555 is not a valid phone number.`

**Root causes:**
- Not E.164
- User typed local format without country
- Bad parsing/normalization

**Fix:**
- Normalize with libphonenumber
- Require country selection or infer from user profile
- Reject early with actionable UX

### 4) 30003 — Unreachable destination handset / carrier filtering

**Message (Messaging):**
`Unreachable destination handset`

**Root causes:**
- Carrier filtering (A2P issues, content filtering)
- Number inactive/out of coverage
- Wrong destination type (landline for SMS)

**Fix:**
- Offer voice fallback
- Ensure 10DLC compliance and correct sender type (toll-free/short code)
- Use Messaging Service with geo-matching and proper sender pools

### 5) 60200 — Invalid parameter (Verify)

**Message:**
`Invalid parameter: Channel`

**Root causes:**
- Unsupported channel string
- Channel not enabled for the Verify Service

**Fix:**
- Validate channel enum in API layer
- Ensure service configuration includes the channel
- Use separate service if policy differs

### 6) 60203 — Max check attempts reached / verification blocked

**Message:**
`Max check attempts reached`

**Root causes:**
- User repeatedly entered wrong code
- Attack on a destination

**Fix:**
- Enforce cooldown and require resend after lockout
- Add bot mitigation
- Alert on spikes per destination hash

### 7) 60202 — Verification expired

**Message:**
`Verification expired`

**Root causes:**
- User waited beyond TTL
- Delivery delays (carrier)
- Client clock confusion (UX)

**Fix:**
- Increase TTL if justified (tradeoff: security)
- Improve UX: show countdown and resend option
- Prefer voice fallback for delayed SMS

### 8) Webhook signature validation failure

**Typical log:**
`Error: Twilio Request Validation Failed.`

**Root causes:**
- Using wrong Auth Token for validation
- URL mismatch (ngrok URL changed, missing query string)
- Reverse proxy rewriting host/path

**Fix:**
- Validate against the exact public URL Twilio calls
- Preserve original URL in proxy (`X-Forwarded-Host`, `X-Forwarded-Proto`)
- Keep Auth Token consistent; rotate carefully

### 9) 11200 — HTTP retrieval failure (Voice/TwiML)

**Message (Voice debugger):**
`HTTP retrieval failure`

**Root causes:**
- Twilio cannot reach your webhook (firewall, DNS)
- TLS misconfiguration
- Slow response > timeout

**Fix:**
- Ensure public HTTPS endpoint
- Reduce latency; respond within a few seconds
- Add health checks and multi-region ingress

### 10) 21610 — STOP / opt-out (Messaging)

**Message:**
`Attempt to send to unsubscribed recipient`

**Root causes:**
- User replied STOP to your sender
- You are reusing a sender pool without opt-out awareness

**Fix:**
- Respect opt-out; do not attempt further SMS
- Offer voice/email/TOTP alternatives
- Maintain suppression list keyed by destination

---

## Security Hardening

### Secrets management

- Store Twilio API Key Secret/Auth Token in a secret manager.
- Rotate API keys quarterly or after incidents.
- Use least privilege: separate API keys per environment.

### Webhook validation (mandatory)

Validate Twilio signatures for any inbound webhook.

Node example:

```ts
import twilio from "twilio";
import type { Request, Response } from "express";

export function validateTwilioWebhook(req: Request, res: Response, next: Function) {
  const authToken = process.env.TWILIO_AUTH_TOKEN!;
  const signature = req.header("X-Twilio-Signature") || "";
  const url = `${req.protocol}://${req.get("host")}${req.originalUrl}`;

  const isValid = twilio.validateRequest(authToken, signature, url, req.body);
  if (!isValid) return res.status(403).send("Forbidden");
  next();
}
```

Operational notes:
- If behind a proxy, set `app.set('trust proxy', true)` and reconstruct URL using forwarded headers.
- Ensure body parsing preserves raw body if required by your framework; some setups need raw body for validation.

### PII handling

- Treat phone numbers and emails as PII.
- Log only:
  - HMAC(destination) with a rotation-capable key
  - last 2 digits for debugging (optional)
- Encrypt destination at rest (KMS envelope encryption).

### CIS-aligned host hardening (high-level pointers)

- CIS Ubuntu Linux 22.04 LTS Benchmark:
  - Disable password SSH auth; enforce key-based
  - Enable automatic security updates
  - Restrict outbound egress from app hosts to Twilio endpoints only where feasible
- systemd sandboxing (see unit file above)
- Run as non-root, read-only filesystem where possible

### Abuse prevention

- Require proof-of-work / CAPTCHA for suspicious send attempts.
- Block disposable email domains for email channel (policy-dependent).
- Add ASN/country anomaly detection.

---

## Performance Tuning

### Reduce Twilio API latency with region/edge

Set `region` and `edge` in SDK init.

Expected impact:
- Typical p50 improvement: 30–80ms depending on proximity
- p95 improvement: 50–150ms in cross-region deployments

Measure:
- Instrument `sendOtp` and `checkOtp` durations
- Compare before/after with same traffic

### Connection reuse and timeouts

- Use keep-alive HTTP agents (Node) to reduce TLS handshake overhead.
- Set sane timeouts:
  - connect timeout: 2s
  - request timeout: 5s (send), 5s (check)
- Implement retries only for safe failure modes (network errors, 5xx). Do not retry on 4xx.

### Cache normalization results

Phone parsing can be expensive at scale.
- Cache E.164 normalization per raw input for short TTL (e.g., 10 minutes) keyed by `(raw, defaultCountry)`.

### Avoid resend loops

Client UX:
- Disable resend button for 30 seconds
- Show countdown
- Backoff on repeated failures

This reduces:
- Twilio costs
- 20429 rate limits
- Carrier filtering risk

---

## Advanced Topics

### Idempotency strategy

Twilio Verify “send” is not inherently idempotent across repeated calls. Implement app-level idempotency:

- Compute key: `sha256(user_id + destination + channel + floor(now/30s))`
- Store in Redis with TTL 60s
- If key exists, return existing verification SID/status

This prevents accidental double-sends from:
- mobile retries
- double-clicks
- network timeouts

### Multi-channel fallback policy

Implement deterministic fallback:
1. SMS
2. If SMS delivery fails with `30003` or no delivery within 20s → Voice
3. If voice fails → Email or TOTP enrollment prompt

Do not automatically fallback without user consent in some jurisdictions; ensure compliance.

### Handling landlines and VoIP

- Some numbers are not SMS-capable.
- Use a carrier lookup (Twilio Lookup API) to detect line type:
  - If landline: skip SMS, offer voice/email
  - If VoIP: consider higher fraud risk; step-up factor

### Internationalization

- Set `Locale` based on user preference.
- Ensure templates exist for target locales.
- For voice, ensure correct language/voice selection (if using custom voice flows).

### Verify + account linking

When verifying phone for account linking:
- Require authenticated session before allowing phone change verification
- Enforce re-authentication for sensitive changes
- Prevent “phone takeover” by requiring existing factor confirmation

### SNA fallback design

If SNA is enabled:
- Attempt SNA first for eligible devices/networks
- If unavailable/failed:
  - fallback to SMS/voice
- Log SNA eligibility and failure reasons for tuning

---

## Usage Examples

### Scenario 1: Sign-in OTP via SMS with resend cooldown (Node + Express)

```ts
// src/server.ts
import express from "express";
import twilio from "twilio";
import { z } from "zod";
import pino from "pino";

const log = pino({ level: process.env.LOG_LEVEL || "info" });
const app = express();
app.use(express.json());

const client = twilio(process.env.TWILIO_API_KEY_SID!, process.env.TWILIO_API_KEY_SECRET!, {
  accountSid: process.env.TWILIO_ACCOUNT_SID!,
  region: "us1",
  edge: "ashburn",
});

const serviceSid = process.env.TWILIO_VERIFY_SERVICE_SID!;

const SendSchema = z.object({
  to: z.string().min(5),
  channel: z.enum(["sms", "call", "email"]).default("sms"),
});

const CheckSchema = z.object({
  to: z.string().min(5),
  code: z.string().min(4).max(10),
});

app.post("/verify/send", async (req, res) => {
  const { to, channel } = SendSchema.parse(req.body);

  // TODO: enforce app-level rate limits here (Redis token bucket)
  const v = await client.verify.v2.services(serviceSid).verifications.create({
    to,
    channel,
    locale: "en",
  });

  log.info({ verificationSid: v.sid, channel: v.channel }, "verify_send");
  res.json({ sid: v.sid, status: v.status });
});

app.post("/verify/check", async (req, res) => {
  const { to, code } = CheckSchema.parse(req.body);

  const c = await client.verify.v2.services(serviceSid).verificationChecks.create({ to, code });

  log.info({ checkSid: c.sid, status: c.status, valid: c.valid }, "verify_check");

  if (c.status === "approved" && c.valid) {
    // Issue session token, mark verified, etc.
    return res.json({ ok: true });
  }
  return res.status(401).json({ ok: false });
});

app.listen(3000, () => log.info("listening on :3000"));
```

Run:

```bash
npx tsx src/server.ts
curl -sS -X POST http://localhost:3000/verify/send -H 'content-type: application/json' \
  -d '{"to":"+14155552671","channel":"sms"}' | jq .
```

### Scenario 2: Voice fallback after SMS failure (policy-driven)

Pseudo-logic:

```ts
type DeliverySignal = { smsFailed: boolean; smsTimedOut: boolean };

function chooseChannel(signal: DeliverySignal) {
  if (signal.smsFailed || signal.smsTimedOut) return "call";
  return "sms";
}
```

Operationally:
- Use Messaging status callbacks to detect `failed` with `30003`
- Or time out after 20 seconds without `delivered` (not always available for SMS)
- Offer user a “Call me instead” option

### Scenario 3: Email OTP using custom channel + SendGrid template

1) Request custom verification:

```bash
curl -sS -X POST "https://verify.twilio.com/v2/Services/${TWILIO_VERIFY_SERVICE_SID}/Verifications" \
  -u "${TWILIO_API_KEY_SID}:${TWILIO_API_KEY_SECRET}" \
  --data-urlencode "To=alice@example.com" \
  --data-urlencode "Channel=custom" | jq .
```

2) Deliver code via SendGrid (your app sends email).
3) Check code via Verify `VerificationCheck`.

### Scenario 4: TOTP enrollment for high-risk accounts

Flow:
- User signs in with password
- Risk engine flags medium/high risk
- Require TOTP enrollment:
  - Create factor
  - Verify initial code
  - Store factor SID
- On future sign-ins, require TOTP check

Key production detail:
- Provide recovery path (support + identity proofing)
- Allow multiple devices

### Scenario 5: Phone verification for profile changes (step-up)

When user changes phone number:
- Require existing session + re-auth
- Send OTP to new number
- Only after approval:
  - update phone in DB
  - mark verified_at
- Prevent swapping to already-verified number owned by another account unless policy allows

### Scenario 6: Webhook endpoint with signature validation and idempotency

```ts
import express from "express";
import twilio from "twilio";
import crypto from "crypto";

const app = express();

// For some frameworks you may need raw body; adjust accordingly.
app.use(express.urlencoded({ extended: false }));

const seen = new Set<string>(); // replace with Redis in prod

app.post("/webhooks/verify-events", (req, res) => {
  const authToken = process.env.TWILIO_AUTH_TOKEN!;
  const signature = req.header("X-Twilio-Signature") || "";
  const url = `${req.protocol}://${req.get("host")}${req.originalUrl}`;

  const ok = twilio.validateRequest(authToken, signature, url, req.body);
  if (!ok) return res.status(403).send("Forbidden");

  const eventSid = req.body.Sid || req.body.EventSid || "";
  const dedupeKey = crypto.createHash("sha256").update(eventSid).digest("hex");
  if (seen.has(dedupeKey)) return res.status(200).send("ok");
  seen.add(dedupeKey);

  // Persist event, update metrics, etc.
  return res.status(200).send("ok");
});
```

---

## Quick Reference

| Task | Command / API | Key flags/fields |
|---|---|---|
| Send OTP | `POST /v2/Services/{VA}/Verifications` | `To`, `Channel`, `Locale` |
| Check OTP | `POST /v2/Services/{VA}/VerificationCheck` | `To`, `Code` |
| List verifications | `GET /v2/Services/{VA}/Verifications` | `To`, `Status`, `Channel`, `PageSize` |
| Fetch verification | `GET /v2/Services/{VA}/Verifications/{VE}` | n/a |
| Cancel verification | `POST /v2/Services/{VA}/Verifications/{VE}` | `Status=canceled` |
| Auth (preferred) | API Key | `SK...` + secret + `accountSid` |
| Common errors | Twilio codes | `20003`, `20429`, `21211`, `30003`, `60202`, `60203` |
| Webhook security | Signature validation | `X-Twilio-Signature`, exact URL |

---

## Graph Relationships

### DEPENDS_ON

- `twilio-core-auth` (Account SID + API Key/Auth Token handling)
- `twilio-webhooks` (signature validation, retry/idempotency patterns)
- `pii-handling` (redaction, encryption at rest)
- `rate-limiting` (Redis token bucket / leaky bucket)

### COMPOSES

- `twilio-messaging` (delivery telemetry, STOP handling, 10DLC considerations)
- `twilio-voice` (voice fallback, call status callbacks)
- `sendgrid-transactional` (custom email channel delivery, bounce handling)
- `studio-flows` (optional orchestration for complex verification journeys)

### SIMILAR_TO

- `auth-otp-generic` (OTP flows without Twilio-managed policy)
- `firebase-phone-auth` (phone verification managed by another provider)
- `okta-verify` (factor-based verification with enterprise IAM)
