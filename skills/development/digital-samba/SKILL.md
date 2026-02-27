---
name: digital-samba
description: Build video conferencing integrations using Digital Samba's API and SDK. Use when creating meeting rooms, embedding video calls, generating participant tokens, managing recordings, or integrating real-time collaboration features. Triggers include "Digital Samba", "video conferencing API", "embed video calls", "meeting room integration", "WebRTC iframe", "participant tokens".
---

# Digital Samba Integration

Build video conferencing into your applications using Digital Samba's prebuilt infrastructure. No WebRTC/Janus/TURN setup required.

## Two Integration Approaches

1. **REST API** - Server-side room/session/participant management
2. **Embedded SDK** - Client-side iframe control and event handling

## Quick Start

### 1. Create a Room (Server-side)
```bash
curl -X POST https://api.digitalsamba.com/api/v1/rooms \
  -H "Authorization: Bearer {DEVELOPER_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"friendly_url": "my-meeting", "privacy": "public"}'
```

### 2. Generate Access Token (Server-side)
```javascript
const jwt = require('jsonwebtoken');

const token = jwt.sign({
  td: "team-uuid",      // Your team ID
  rd: "room-uuid",      // Room ID from step 1
  u: "John Doe",        // User display name
  role: "moderator"     // Optional: user role
}, DEVELOPER_KEY, { algorithm: 'HS256' });
```

### 3. Embed the Room (Client-side)

**Option A: Plain iframe** — simplest, no SDK needed:
```html
<iframe
  id="video-frame"
  allow="camera; microphone; display-capture; autoplay;"
  src="https://yourteam.digitalsamba.com/my-meeting?token={jwt}"
  style="width: 100%; height: 100vh; border: none;"
  allowfullscreen="true">
</iframe>
```

**Option B: SDK creates the iframe** — lets you add event listeners before loading:
```javascript
import DigitalSambaEmbedded from '@digitalsamba/embedded-sdk';

// SDK injects an iframe into this container element
const sambaFrame = DigitalSambaEmbedded.createControl({
  url: 'https://yourteam.digitalsamba.com/my-meeting?token={jwt}',
  root: document.getElementById('video-container') // Container div, not an iframe
});

// Set up events before the iframe loads
sambaFrame.on('userJoined', (e) => console.log(`${e.data.name} joined`));
sambaFrame.on('connectionFailure', (e) => console.error('Failed:', e.data));

sambaFrame.load(); // Now create and load the iframe
```

**Option C: SDK wraps an existing iframe** — control an iframe you already placed in HTML:
```javascript
import DigitalSambaEmbedded from '@digitalsamba/embedded-sdk';

// Wrap the iframe from Option A to add SDK control
const sambaFrame = DigitalSambaEmbedded.createControl({
  frame: document.getElementById('video-frame') // Existing iframe element
});

sambaFrame.on('userJoined', (e) => console.log(`${e.data.name} joined`));
```

> **Important**: The SDK iframe container must have explicit CSS dimensions (width + height). The iframe does **not** auto-size. See [Iframe Sizing](patterns.md#iframe-sizing-important) in patterns.md.

## When to Use What

| Need | Use |
|------|-----|
| Create/delete rooms | REST API |
| User authentication | JWT tokens |
| Embed video UI | iframe + SDK |
| Start/stop recording | REST API or SDK |
| React to events | SDK events |
| Manage participants | REST API |
| Customize UI | Room settings API |

## Pre-Built Integration Patterns

The skill includes ready-to-use code patterns for common use cases. Ask your AI assistant for a pattern by describing your use case (e.g., "build a virtual classroom", "add video to my booking system", "set up a webinar page").

| Pattern | Best For | Key Features |
|---------|----------|--------------|
| **Simple Public Room** | Quick demos, open meetings | Minimal setup, public access, no auth required |
| **Authenticated Users** | SaaS integrations, known users | JWT tokens, role-based access, error handling |
| **SDK-Controlled Room** | Custom UIs, programmatic control | Event handling, mute/unmute, custom buttons |
| **Scheduled Meetings** | Calendar integrations, booking systems | Time constraints, invite tokens, email invites |
| **Webinar Mode** | One-to-many broadcasts | Presenter/attendee roles, Q&A, raise hand |
| **Recording & Playback** | Content archiving, compliance | Start/stop recording, download, playback |
| **Online Learning Platform** | LMS, virtual classrooms, tutoring | Instructor/student roles, attendance tracking, lesson recordings, per-course rooms |
| **Playwright E2E Testing** | Automated testing, CI/CD | Iframe testing, SDK events, demo recordings |

Each pattern includes complete server-side and client-side code. See **[patterns.md](patterns.md)** for full implementations.

## Reference Documentation

For detailed information, see these reference files:

- **[api-reference.md](api-reference.md)** - Complete REST API endpoints
- **[sdk-reference.md](sdk-reference.md)** - SDK methods, events, properties
- **[patterns.md](patterns.md)** - Pre-built integration patterns with full code examples
- **[jwt-tokens.md](jwt-tokens.md)** - Authentication deep-dive

## Key Concepts

### Room Types
- **Public**: Anyone with URL can join (enters name on join screen)
- **Private**: Requires JWT token to join

### Roles & Permissions
Assign roles via JWT `role` field. Common roles:
- `moderator` - Full control (mute others, recording, etc.)
- `speaker` - Can present and speak
- `attendee` - View/listen only (configurable)

### Authentication Flow
1. **Developer key** → Server-side API calls only (find it in **Dashboard → Team Settings → Developer**)
2. **JWT tokens** → Client-side room access (signed with the developer key using HS256)
3. **Never expose developer key to browsers** — use it only on your server

## Common Errors

### API Errors

| Code | Meaning | Solution |
|------|---------|----------|
| 401 | Invalid/missing key | Check Authorization header |
| 403 | Insufficient permissions | Verify role/permissions |
| 404 | Room not found | Check room UUID/URL |
| 422 | Validation error | Check request body; see `errors` field for per-field details |
| 429 | Rate limited | Back off and retry with exponential delay |

### SDK / Client Errors

| Issue | Cause | Solution |
|-------|-------|----------|
| SDK won't load | Not a secure context | Serve over HTTPS (localhost exempt) — check `window.isSecureContext` |
| `connectionFailure` event | Invalid room URL, network error, or room deleted | Verify room exists and URL is correct |
| `appError` event | Runtime error (e.g., media permission denied) | Check `e.data.code` and `e.data.message` for details |
| iframe blank / no video | Missing `allow` attribute | Add `allow="camera; microphone; display-capture; autoplay"` to iframe |

For detailed troubleshooting steps, diagnostic code examples, and API error breakdowns, see the **[Troubleshooting & Diagnostics](patterns.md#troubleshooting--diagnostics)** section in patterns.md.

## Check for Updates

To check if your installed skill is up to date:

1. **Local version**: `cat .claude/skills/digital-samba/VERSION`
2. **Latest version**: `curl -s https://api.github.com/repos/digitalsamba/digital-samba-skill/releases/latest | grep '"tag_name"'`

**To update (submodule install):**
```bash
git submodule update --remote .claude/skills/digital-samba
```

**To update (manual install):** Re-clone and copy skill files from https://github.com/digitalsamba/digital-samba-skill

## Resources

- API Reference: https://developer.digitalsamba.com/rest-api/
- SDK NPM: https://www.npmjs.com/package/@digitalsamba/embedded-sdk
- Dashboard: https://dashboard.digitalsamba.com
- Skill Releases: https://github.com/digitalsamba/digital-samba-skill/releases
