---
name: fix-api-403
description: Diagnose and fix Anthropic API 403 "Request not allowed" errors in OpenClaw gateway. Use when the webchat or embedded agent returns HTTP 403, "forbidden", or "Request not allowed". Common cause is geographic network restrictions (e.g. China) where api.anthropic.com is blocked without a proxy.
triggers:
  - pattern: "403|forbidden|Request not allowed"
    description: "Detect API 403 forbidden errors"
  - pattern: "代理|proxy|网络|无法访问|连接失败"
    description: "Detect proxy/network related issues"
auto_invoke: true
examples:
  - "webchat returns 403 error"
  - "HTTP 403 forbidden: Request not allowed"
  - "gateway agent returns 403"
  - "fix the 403 error on webchat"
---

# Fix Anthropic API 403 "Request not allowed"

## Problem

The OpenClaw gateway embedded agent fails with:
```
HTTP 403 forbidden: Request not allowed
```

This typically means the Anthropic API (`api.anthropic.com`) is blocking direct requests due to geographic restrictions (common in China).

## Diagnosis Steps

### Step 1: Confirm the error

Check gateway error logs:
```bash
tail -30 ~/.openclaw/logs/gateway.err.log | grep -E "403|forbidden|Request not allowed"
```

Expected match:
```
[agent/embedded] embedded run agent end: ... isError=true error=HTTP 403 forbidden: Request not allowed
```

### Step 2: Verify it's a network/geo issue

Test the API directly without proxy:
```bash
curl -s --noproxy '*' -w "\n%{http_code}" "https://api.anthropic.com/v1/messages" \
  -H "Authorization: Bearer $ANTHROPIC_OAUTH_TOKEN" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: claude-code-20250219,oauth-2025-04-20" \
  -H "user-agent: claude-cli/1.0.0 (external, cli)" \
  -H "x-app: cli" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}],"system":[{"type":"text","text":"You are Claude Code."}]}'
```

If this returns **403**, it's a geo-restriction issue. Test WITH proxy to confirm:
```bash
curl -s -w "\n%{http_code}" "https://api.anthropic.com/v1/messages" \
  -H "Authorization: Bearer $ANTHROPIC_OAUTH_TOKEN" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: claude-code-20250219,oauth-2025-04-20" \
  -H "user-agent: claude-cli/1.0.0 (external, cli)" \
  -H "x-app: cli" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}],"system":[{"type":"text","text":"You are Claude Code."}]}'
```

If this returns **200**, proxy is the fix.

### Step 3: Check current proxy settings

```bash
# Check shell proxy
echo "http_proxy=$http_proxy"
echo "https_proxy=$https_proxy"

# Check system proxy (macOS)
networksetup -getwebproxy Wi-Fi
networksetup -getsecurewebproxy Wi-Fi
```

## Fix Procedure

The fix requires THREE changes because:
1. The gateway LaunchAgent doesn't inherit shell environment variables
2. Node.js's built-in `fetch()` doesn't respect `HTTP_PROXY`/`HTTPS_PROXY` env vars
3. A preload script is needed to override `globalThis.fetch` with a proxy-aware version

### Fix 1: Deploy the proxy preload script

Run the bundled script to generate a proxy-preload.js tailored to your environment:

```bash
bash /path/to/fix-api-403/scripts/deploy-proxy-fix.sh
```

Or manually create `~/.openclaw/scripts/proxy-preload.js` — see [scripts/proxy-preload.js.template](scripts/proxy-preload.js.template) for the template. You MUST replace `OPENCLAW_MODULES_PATH` with the actual path to your OpenClaw node_modules.

### Fix 2: Update the LaunchAgent plist

Find the plist:
```bash
ls ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

Add these keys inside the `<dict>` under `<key>EnvironmentVariables</key>`:
```xml
<key>http_proxy</key>
<string>http://127.0.0.1:7897</string>
<key>https_proxy</key>
<string>http://127.0.0.1:7897</string>
<key>HTTP_PROXY</key>
<string>http://127.0.0.1:7897</string>
<key>HTTPS_PROXY</key>
<string>http://127.0.0.1:7897</string>
<key>ALL_PROXY</key>
<string>socks5://127.0.0.1:7897</string>
<key>NODE_OPTIONS</key>
<string>--require /Users/YOUR_USERNAME/.openclaw/scripts/proxy-preload.js</string>
```

Replace `127.0.0.1:7897` with your actual proxy address (Clash, V2Ray, etc.).

### Fix 3: Reload the gateway

```bash
launchctl bootout gui/$(id -u)/ai.openclaw.gateway
sleep 2
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

### Fix 4: Verify

```bash
# Test via gateway (not --local)
openclaw agent --message "hello" --agent main --json 2>&1 | head -10
```

If you see a successful text response (not "HTTP 403"), the fix is working.

## Automated Fix

For a one-command fix, run the deploy script:

```bash
bash ~/.openclaw/skills/fix-api-403/scripts/deploy-proxy-fix.sh [proxy-url]
```

Default proxy: `http://127.0.0.1:7897`. Pass a custom URL as the first argument.

## Why This Happens

| Layer | Problem | Solution |
|-------|---------|----------|
| Network | `api.anthropic.com` blocked in China | Use HTTP proxy |
| LaunchAgent | Doesn't inherit shell env vars (`http_proxy`) | Add proxy vars to plist |
| Node.js | Built-in `fetch()` ignores `HTTP_PROXY` env vars | Preload script overrides `globalThis.fetch` |
| Anthropic SDK | Uses `globalThis.fetch`, no built-in proxy support | Replaced with undici's proxy-aware fetch |

## Key Technical Details

- **OAuth tokens** (`sk-ant-oat01-*`) require specific headers: `anthropic-beta: claude-code-20250219,oauth-2025-04-20,...` and `Authorization: Bearer` (not `x-api-key`)
- The preload script uses OpenClaw's bundled `undici` package to create a `ProxyAgent`, then replaces `globalThis.fetch` with `undiciFetch` that routes through the proxy
- The preload also patches `http.globalAgent` and `https.globalAgent` via `proxy-agent` for legacy HTTP requests
- `openclaw agent --local` works because it inherits shell proxy env vars; the gateway LaunchAgent does not

## Troubleshooting

| Symptom | Check |
|---------|-------|
| Preload error: "Cannot find module 'undici'" | Verify `OPENCLAW_MODULES_PATH` in proxy-preload.js points to the correct node_modules |
| Still 403 after fix | Ensure proxy (Clash/V2Ray) is actually running on the configured port |
| Gateway won't start | Check `launchctl print gui/$(id -u)/ai.openclaw.gateway` and error log |
| `openclaw agent --local` works but gateway doesn't | The plist is missing proxy env vars or NODE_OPTIONS |
