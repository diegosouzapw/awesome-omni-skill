---
name: anthropic-token-refresh
description: Automatically refresh Anthropic Claude setup-token before expiration using browser automation. Use when: (1) Setting up auto token refresh for Claude Max/Pro subscription, (2) Token keeps expiring and causing OpenClaw to stop responding, (3) Want to maintain continuous Claude API access without manual intervention.
---

# Anthropic Token Auto-Refresh Skill

This skill enables fully automatic refresh of Anthropic Claude setup-token using browser automation, ensuring your OpenClaw never goes offline due to token expiration.

## Problem

Claude `setup-token` (used for Max/Pro subscriptions) expires periodically and requires manual refresh:
1. Run `claude setup-token`
2. Browser opens authorization page
3. Click "Authorize" button
4. Copy verification code
5. Paste into CLI
6. Update OpenClaw config

This skill automates the entire process!

## Solution Overview

```
Daily Check → Token expires in < 30 days?
                    ↓ Yes
         Start `claude setup-token`
                    ↓
         Browser opens auth URL
                    ↓
         Auto-click "Authorize" button
                    ↓
         Extract verification code
                    ↓
         Auto-input code to CLI
                    ↓
         Get new token
                    ↓
         Update auth-profiles.json
                    ↓
         ✅ Token refreshed!
```

## Prerequisites

1. **Claude CLI installed and logged in**
   ```bash
   claude --version
   ```

2. **OpenClaw browser profile logged into Claude**
   - Run: `openclaw browser start --profile openclaw`
   - Navigate to https://claude.ai and log in
   - This is a one-time setup

3. **jq installed** (for JSON processing)
   ```bash
   brew install jq
   ```

## Setup

### Step 1: One-time Browser Login

The OpenClaw browser needs to be logged into Claude (separate from your regular browser):

```bash
# Start OpenClaw browser
openclaw browser start --profile openclaw

# Navigate to Claude login (via OpenClaw or manually)
# Log in with your Claude account
```

### Step 2: Set Up Cron Jobs

Create two scheduled tasks:

**1. Token Refresh Check (Daily)**
```json
{
  "name": "token-refresh-check",
  "schedule": {"kind": "every", "everyMs": 86400000},
  "payload": {
    "kind": "systemEvent",
    "text": "Token refresh check: Check token expiry, if < 30 days remaining, execute auto-refresh flow"
  },
  "sessionTarget": "main"
}
```

**2. Claude Session Keep-Alive (Every 3 Days)**
```json
{
  "name": "claude-session-keepalive",
  "schedule": {"kind": "every", "everyMs": 259200000},
  "payload": {
    "kind": "systemEvent",
    "text": "Claude login keepalive: Visit https://claude.ai in OpenClaw browser to maintain login session"
  },
  "sessionTarget": "main"
}
```

## Auto-Refresh Flow (For Agent)

When triggered, the agent should execute this flow:

### 1. Check Token Expiry
```bash
cat ~/.openclaw/agents/main/agent/auth-profiles.json | jq '.profiles["anthropic:setup-token"].expires'
```

Calculate days remaining:
```bash
EXPIRES=$(cat ~/.openclaw/agents/main/agent/auth-profiles.json | jq -r '.profiles["anthropic:setup-token"].expires')
NOW_MS=$(($(date +%s) * 1000))
DAYS_LEFT=$(( ($EXPIRES - $NOW_MS) / 1000 / 60 / 60 / 24 ))
echo "Days left: $DAYS_LEFT"
```

If `DAYS_LEFT > 30`, skip refresh.

### 2. Start Token Generation
```bash
# Run in PTY mode to capture output
claude setup-token
```

### 3. Extract Auth URL
From the CLI output, extract the authorization URL:
```
https://claude.ai/oauth/authorize?code=true&client_id=...
```

### 4. Browser Automation
```javascript
// Navigate to auth URL
browser.navigate(authUrl, {profile: "openclaw"})

// Wait for page load, take snapshot
browser.snapshot()

// Find and click "Authorize" button (usually ref like "e24")
browser.act({kind: "click", ref: "<authorize-button-ref>"})

// Take snapshot to get verification code
browser.snapshot()
// Code appears in format: "DYPxv9k9MyJ8lIW0G5AWYdbc..."
```

### 5. Input Verification Code
```bash
# Write code to the PTY session
process.write(sessionId, verificationCode + "\n")
```

### 6. Extract New Token
From CLI output, extract token:
```
sk-ant-oat01-XXXXX...
```

### 7. Update Config
```bash
NEW_TOKEN="sk-ant-oat01-..."
EXPIRES=$(($(date +%s) * 1000 + 365 * 24 * 60 * 60 * 1000))

cat ~/.openclaw/agents/main/agent/auth-profiles.json | \
  jq --arg token "$NEW_TOKEN" --argjson expires "$EXPIRES" \
  '.profiles["anthropic:setup-token"].token = $token | .profiles["anthropic:setup-token"].expires = $expires' \
  > /tmp/auth.tmp && mv /tmp/auth.tmp ~/.openclaw/agents/main/agent/auth-profiles.json
```

### 8. Verify
```bash
openclaw models status | grep "anthropic:setup-token"
```

## Session Keep-Alive

To prevent Claude login from expiring in the OpenClaw browser:

```javascript
// Every 3 days, visit Claude
browser.navigate("https://claude.ai", {profile: "openclaw"})
// Wait for page load
browser.snapshot()
// Verify logged in (should see user menu, not login page)
```

## Troubleshooting

### "Login page instead of authorize page"
The OpenClaw browser is not logged into Claude. Log in manually once.

### "Authorize button not found"
Page structure may have changed. Take a snapshot and find the correct button reference.

### "Token refresh failed"
1. Check Claude CLI: `claude --version`
2. Check browser: `openclaw browser status --profile openclaw`
3. Manual refresh: `claude setup-token`

### "Session expired in OpenClaw browser"
Run the keep-alive task more frequently, or log in again.

## Files

### Auth Profile Location
```
~/.openclaw/agents/main/agent/auth-profiles.json
```

### Browser Data Location
```
~/.openclaw/browser/openclaw/user-data/
```

## Security Notes

- Tokens are stored locally in `auth-profiles.json`
- OpenClaw browser profile is isolated from your main browser
- No credentials are sent to third parties
- The refresh process happens entirely on your local machine
