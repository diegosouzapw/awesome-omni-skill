---
name: ipinfo
description: Guides IP lookups with context reuse and proper parameters. Triggers on IP lookups, VPN detection, abuse contacts.
metadata:
  version: 1.0.0
  category: operations
  tags: [ipinfo, ip-lookup, vpn-detection, mcp]
  author:
    name: NimbleBrain
    url: https://www.nimblebrain.ai
---

# IPInfo

## Critical: Context Reuse

**Follow-up questions ("this IP", "that one", "is it a VPN?") → reuse IP from prior result.**

```
User: "lookup 195.179.201.16" → get_ip_info(ip="195.179.201.16")
User: "is this a VPN?"       → get_plus_ip_info(ip="195.179.201.16")  ← SAME IP
```

Never call with empty `{}`. Always pass `ip` explicitly.

## Tool Selection

| Intent | Tool |
|--------|------|
| General lookup | `get_ip_info(ip)` |
| VPN/proxy/Tor | `get_plus_ip_info(ip)` |
| Abuse contact | `get_abuse_contact(ip)` |
| Company info | `get_company_info(ip)` |
| Hosted domains | `get_hosted_domains(ip)` |
| WHOIS | `whois_lookup_by_ip(ip)` |
| Multiple IPs | `batch_lookup(ips)` |

## VPN Detection

Use `get_plus_ip_info` (not `get_ip_info`) for privacy flags:
- `anonymous.vpn`, `anonymous.proxy`, `anonymous.tor`
- `is_hosting` = datacenter/cloud provider
