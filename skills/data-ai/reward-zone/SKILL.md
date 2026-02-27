---
name: reward-zone
description: Send and receive recognition awards on Red Hat Reward Zone. Uses pure HTTP SAML. Use when user says "send reward", "give recognition", "check reward zone points", or "view received awards".
---

# Reward Zone

Interact with rewardzone.redhat.com: send awards, view received/sent, check points. Pure HTTP, no browser.

## Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `action` | string | required | "send", "view_received", "check_points", "view_sent" |
| `recipient` | string | - | Name or email (required for send) |
| `award` | string | "Focus on Team" | Award name |
| `category` | string | "RH Multiplier" | RH Multiplier, Team Advocate, Customer Focus |
| `points` | int | 25 | Points to award |
| `message` | string | - | Recognition message (required for send) |
| `dry_run` | bool | false | Preview without submitting |

## Prerequisites

- Redhatter service on localhost:8009
- Red Hat SSO (Kerberos ID + PIN/token)

## Workflow

### 1. Validate Inputs
- action in ["send", "view_received", "check_points", "view_sent"]
- For send: recipient and message required; points positive int

### 2. Create HTTP Session
- `http_session_create(name="reward_zone", base_url="https://rewardzone.redhat.com")`

### 3. Authenticate
- `http_saml_auth(session="reward_zone")` — pure HTTP SAML
- If not successful → return auth troubleshooting (redhatter, VPN, credentials)

### 4. Dispatch by Action

**check_points:**
- `http_request(session="reward_zone", method="GET", path="/api/v1/NominationPoints/getNominationPoints")`
- Parse and return available points

**view_received:**
- `http_request(session="reward_zone", method="GET", path="/api/v1/Awards/getReceivedAwards")`
- Parse and return awards table (Date, From, Award, Points, Message)

**view_sent:**
- `http_request(session="reward_zone", method="GET", path="/api/v1/Awards/getSentAwards")`
- Parse and return awards table

**send:**
- `http_request(session="reward_zone", method="GET", path="/api/v1/Members/advancedMemberSearch", params='{"searchText": "recipient"}')`
- Parse for pin, name; if not found → return "Recipient not found"
- If dry_run → return preview and exit
- `http_request(session="reward_zone", method="POST", path="/api/v1/Awards/submitNomination", json_body='{"nomineePin": "pin", "awardName": "award", "points": N, "description": "msg", "message": "msg"}')`

### 5. Post-Action
- `memory_session_log("Reward Zone {action}", ...)`

## Key Details

- **No browser:** Pure HTTP SAML flow
- **Auth:** redhatter → `systemctl --user start redhatter`
- **Categories:** RH Multiplier, Team Advocate, Customer Focus
