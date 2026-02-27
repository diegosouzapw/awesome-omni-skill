---
name: vpn-localhost-fix
description: Fix VPN proxy conflicts with local development tools. Use when apps like OpenCode Desktop, VS Code, or other Electron/Tauri-based applications fail to start or connect to their local servers when a VPN proxy is enabled. Symptoms include "Failed to spawn server" errors, connection refused to 127.0.0.1 ports, or apps hanging on startup. Supports Clash Verge and other system proxy VPNs on macOS.
---

# VPN Localhost Bypass Fix

## Overview

This skill fixes issues where VPN system proxies interfere with local development tools and applications. When a VPN enables system-wide proxying, local connections (localhost/127.0.0.1) may be intercepted, preventing apps from connecting to their own backend servers.

**Common symptoms:**
- Electron/Tauri apps (OpenCode Desktop, VS Code) fail to start
- Error: `Failed to spawn server` or `Connection refused` to 127.0.0.1
- Apps work fine when VPN is disabled, fail when enabled

**Root cause:** VPN system proxy lacks bypass rules for localhost traffic.

---

## Quick Diagnosis

Confirm the issue with these checks:

```bash
# Check if system proxy is enabled
scutil --proxy | grep "HTTPSEnable : 1"

# Check if localhost is in bypass list
scutil --proxy | grep "ExceptionsList"
```

If HTTPSEnable is 1 but ExceptionsList doesn't include localhost/127.0.0.1, use this skill.

---

## Supported VPNs

### Clash Verge (macOS)

**Config file:** `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/verge.yaml`

**Add/modify these settings:**

```yaml
use_default_bypass: true
system_proxy_bypass: localhost, 127.0.0.1, *.local, 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12
```

**Steps:**
1. Backup config: `cp verge.yaml verge.yaml.backup`
2. Edit `verge.yaml` with the bypass settings above
3. Restart Clash Verge
4. Verify: `scutil --proxy | grep ExceptionsList`

### Other VPNs

Look for settings named:
- "Bypass List" / "例外列表" / "绕过规则"
- "Proxy Exclusions" / "Direct Connection"
- "Local addresses bypass"

Add: `localhost, 127.0.0.1, *.local`

---

## Verification

After applying the fix:

```bash
# Verify bypass rules are active
scutil --proxy | grep -A 10 "ExceptionsList"
```

Expected output should include:
```
ExceptionsList : <array> {
  0 : 127.0.0.1
  1 : localhost
  2 : *.local
  ...
}
```

---

## Standard Bypass Rules

Recommended bypass list for most users:

```
localhost, 127.0.0.1, *.local, 192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12
```

| Entry | Purpose |
|-------|---------|
| `localhost` | Local hostname |
| `127.0.0.1` | Loopback interface |
| `*.local` | Bonjour/mDNS local domains |
| `192.168.0.0/16` | Private network class C |
| `10.0.0.0/8` | Private network class A |
| `172.16.0.0/12` | Private network class B |

---

## Troubleshooting

**Problem:** Still failing after adding bypass rules

**Solutions:**
1. Restart the VPN application completely
2. Restart the affected application (OpenCode Desktop, etc.)
3. Check for multiple VPNs running simultaneously - disable extras
4. Verify config file syntax (YAML for Clash)

**Problem:** Multiple VPNs with conflicting ports

**Solution:** Ensure each VPN uses different proxy ports (e.g., Clash: 7897)
