---
name: teams-hack
version: 1.0.0
description: "Your agent reads Teams chats, posts to channels, searches everything. One stolen token. 90 days per browser tap."
metadata:
  openclaw:
    emoji: "💬"
    os: ["linux", "darwin"]
    requires:
      capabilities: ["browser"]
    notes:
      security: "Shares the Outlook MSAL refresh token. One extraction covers both skills. Token stored at ~/.openclaw/credentials/outlook-msal.json (0600). Auto-rotates on use, lasts 90+ days."
---

# Teams Hack

**One token. Two skills. 90 days of access.**

This skill shares the same MSAL refresh token as [**outlook-hack**](https://clawhub.com/globalcaos/outlook-hack). Extract once from Teams localStorage, get both email and chat access.

## What It Does

- 💬 Read and send chat messages (1:1 and group)
- 📢 Read and post to team channels
- 🔍 Search messages across all of Teams
- 👥 Browse org directory, check presence status
- 📅 View calendar with Teams meeting join links
- 🏢 List joined teams and channels

## Quick Start

### 1. Token Extraction (one-time, ~30 seconds)

Open **Microsoft Teams** (`teams.cloud.microsoft`) in Chrome. Attach the tab via OpenClaw browser relay. The agent runs this in the page:

```javascript
(() => {
  const keys = Object.keys(localStorage).filter(k =>
    k.includes('refreshtoken') || k.includes('RefreshToken')
  );
  const results = keys.map(k => {
    const parsed = JSON.parse(localStorage.getItem(k));
    return { key: k, secret: parsed.secret, client_id: parsed.client_id };
  });
  // Also get tenant ID
  const accountKeys = Object.keys(localStorage).filter(k => {
    try { return JSON.parse(localStorage.getItem(k)).tenantId; } catch { return false; }
  });
  let tenantId = null;
  for (const k of accountKeys) {
    try { tenantId = JSON.parse(localStorage.getItem(k)).tenantId; break; } catch {}
  }
  return { tokens: results, tenantId };
})();
```

Then store the token:

```bash
teams token store --refresh-token <secret> --tenant-id <tenantId>
```

### 2. Verify

```bash
teams token test
```

### 3. Use

```bash
teams chats                          # Recent conversations
teams chat <id> --top 10             # Read messages
teams chat-send <id> --message "hi"  # Send message
teams teams                          # List teams
teams channels <teamId>              # List channels
teams search "project update"        # Search everything
teams users --search "Oscar"         # Find people
teams presence                       # Your status
teams calendar --days 3              # Upcoming meetings
```

## How It Works

Same mechanism as the Outlook hack:

1. Teams stores an MSAL refresh token in `localStorage`
2. This token is exchanged for a Graph API access token using Teams' first-party client ID
3. The client ID (`5e3ce6c0-2b1f-4285-8d4b-75ee78787346`) has pre-authorized Graph scopes
4. Token auto-rotates on each use — perpetual access as long as it's used within 90 days

## Shared Token Architecture

Both skills read from the same file:

```
~/.openclaw/credentials/outlook-msal.json
```

Extract the token once → both `outlook` and `teams` CLIs work. If either skill refreshes the token, the other benefits.

## CLI Reference

| Command | Description |
|---------|-------------|
| `teams chats` | List recent chats with last message preview |
| `teams chat <id>` | Read messages (newest first) |
| `teams chat-send <id> --message <text>` | Send to a chat |
| `teams teams` | List all joined teams |
| `teams channels <teamId>` | List channels in a team |
| `teams channel <teamId> <channelId>` | Read channel messages |
| `teams channel-send <teamId> <channelId> --message <text>` | Post to channel |
| `teams search "<query>"` | Full-text search across messages |
| `teams users --search <name>` | Search org directory |
| `teams presence` | Your availability status |
| `teams calendar --days 7` | Calendar with meeting links |
| `teams me` | Your profile |

## Sibling Skill: Outlook Hack

This skill shares the same MSAL refresh token with [**outlook-hack**](https://clawhub.com/globalcaos/outlook-hack). **One extraction covers both.** Extract the token once → get full chat access (this skill) AND email access (Outlook Hack).

Both skills read and write to the same credentials file:
```
~/.openclaw/credentials/outlook-msal.json
```

If either skill refreshes the token, the other benefits automatically.

| Skill | What it does | Send-blocked? |
|-------|-------------|---------------|
| **[outlook-hack](https://clawhub.com/globalcaos/outlook-hack)** | Email: read, search, draft, folders, attachments, calendar, contacts | ✅ Cannot send |
| **teams-hack** (this) | Chat: read, send, channels, search, presence, org directory | No (chat sending enabled) |

## Architecture

- **Zero external deps** — pure Node.js (v22+)
- **Shared credentials** — same token file as Outlook
- **Graph API v1.0** — standard Microsoft endpoints
- **Beta fallback** — some features use `/beta` when v1.0 lacks support

## The Full Stack

Pair with [**outlook-hack**](https://clawhub.com/globalcaos/outlook-hack) for email, [**whatsapp-ultimate**](https://clawhub.com/globalcaos/whatsapp-ultimate) for messaging, and [**jarvis-voice**](https://clawhub.com/globalcaos/jarvis-voice) for voice.

👉 **[Clone it. Fork it. Break it. Make it yours.](https://github.com/globalcaos/clawdbot-moltbot-openclaw)**
