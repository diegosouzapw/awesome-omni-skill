---
name: leak
description: Compatibility stub for migrating from the legacy mixed leak skill to split hardened skills.
compatibility: Requires access to the internet
version: 2026.2.17
metadata:
  openclaw:
    emoji: 💦
    os: ["darwin", "linux"]
    requires:
      env:
      bins: ["leak"]
  author: eucalyptus-viminalis
---

# leak (migration stub)

This legacy skill is now a compatibility stub.

Use the split hardened skills instead:
- `leak-buy` for buying and downloading content.
- `leak-publish` for publishing and selling content.

## Migration

1. Install `leak-buy` when the user asks to buy/download.
2. Install `leak-publish` when the user asks to publish/sell.
3. Do not execute buy/publish workflows from this stub.

## Rationale

The split model reduces risk surface, removes mixed high-risk behavior from one skill, and hardens scanner posture for Clawhub.
