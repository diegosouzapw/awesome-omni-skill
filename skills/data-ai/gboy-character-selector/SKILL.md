---
name: gboy-character-selector
description: Select characters from the G*BOY universe for your OpenCLAW agent personality.
metadata:
  openclaw:
    emoji: "🎮"
    requires:
      bins: ["node"]
    os: ["darwin", "linux"]
user-invocable: true
---

# G*BOY Character Selector

Browse and install characters from the G*BOY universe as your OpenCLAW agent personality.

## Commands

- **select** — Interactively pick a character and install their personality
- **list** — Show all available G*BOY universe characters
- **restore** — Revert to a previous character from backup

## What It Does

Each character includes:
- **SOUL.md** — Internal values, tone, and behavioral principles
- **IDENTITY.md** — External presentation: name, emoji, vibe, avatar path
- **Avatar image** — Rendered in OpenCLAW's Control UI

When you select a character, this skill:
1. Backs up your current SOUL.md, IDENTITY.md, and avatar
2. Installs the selected character's personality files
3. Updates the avatar path for OpenCLAW's workspace

## Usage

```bash
node dist/index.js select    # Interactive character selection
node dist/index.js list      # List all characters
node dist/index.js restore   # Restore from backup
```
