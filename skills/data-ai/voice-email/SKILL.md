---
name: voice-email
version: 1.0.2
description: Send emails via natural voice commands - designed for accessibility
security:
  requires:
    - network
    - file_access
  credentials:
    - deepgram_api_key
    - telegram_bot_token
    - gmail_oauth  # via gogcli, stored in system keyring
  optional_credentials:
    - elevenlabs_api_key  # optional, for voice responses
  permissions:
    - modify_config
  env_vars:
    - DEEPGRAM_API_KEY
---

# Voice Email Skill

Send emails using natural voice commands. Perfect for accessibility use cases.

## What It Does

When you receive a voice message, parse and send an email:

**Input format:**
```
new email to [recipient], subject [subject], body [body], send
```

**Examples:**
- "new email to john@example.com, subject Hello, body How are you doing, send"
- "send email to mom@gmail.com, subject Dinner, body See you at 7pm, send"

## What This Skill CANNOT Do

- ❌ Execute arbitrary code
- ❌ Access files outside of logging/debugging
- ❌ Modify system files
- ❌ Access other accounts without explicit OAuth
- ❌ Send emails to unknown recipients without user confirmation

## Prerequisites

This skill requires:
1. **gogcli** - Google CLI for Gmail (must be installed separately)
2. **Deepgram** - For voice transcription (API key required)
3. **Telegram bot** - For receiving voice messages (already configured in OpenClaw)
4. **ElevenLabs** - Optional, for voice responses (not required)

### Install gogcli (once, manually)

**Option A - via npm (recommended):**
```bash
npm install -g gogcli
```

**Option B - via binary (verify source):**
Download from https://gogcli.ai and verify the binary

Then authenticate:
```bash
gog auth add your-email@gmail.com
```

### Configure Deepgram (REQUIRED)

Add to openclaw.json:
```json
{
  "tools": {
    "media": {
      "audio": {
        "enabled": true,
        "models": [{"provider": "deepgram", "model": "nova-3"}]
      }
    }
  },
  "env": {
    "DEEPGRAM_API_KEY": "your-deepgram-key"
  }
}
```

### Configure ElevenLabs (OPTIONAL)

For voice responses, add to openclaw.json:
```json
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "elevenlabs",
      "elevenlabs": {
        "apiKey": "YOUR_ELEVENLABS_KEY",
        "voiceId": "YOUR_VOICE_ID"
      }
    }
  }
}
```

Without ElevenLabs, text responses still work.

## Usage

Simply send a voice message with the command. The agent will:
1. Transcribe it (via Deepgram)
2. Parse the fields
3. Send the email (via gogcli)
4. Confirm via text (or voice if ElevenLabs configured)

## Command Parser

The agent extracts:
- `to`: Email address (after "to", "email to", "send to")
- `subject`: Text after "subject"
- `body`: Text after "body" (before "send")

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| DEEPGRAM_API_KEY | Yes | For voice transcription |
| ELEVENLABS_API_KEY | No | For voice responses |
| ELEVENLABS_VOICE_ID | No | Voice to use |

## Security Notes

- **Network**: Requires access to Telegram API, Deepgram API, Gmail API
- **Credentials**: 
  - gogcli stores OAuth tokens in system keyring
  - Deepgram key in openclaw.json (or environment)
  - ElevenLabs key in openclaw.json (optional)
- **Data**: Voice recordings processed by Deepgram, emails sent via user's Gmail
- **Privilege**: Modifies openclaw.json to enable media/audio
- **Does NOT**: Execute arbitrary code, access unrelated files, or modify system

## Best Practices for Production

1. **Use test accounts**: Create dedicated Gmail account for testing
2. **Limit Gmail OAuth**: Use app-specific passwords if needed
3. **Scope Deepgram**: Use minimal quota for testing
4. **Review logs**: Check `/tmp/openclaw-*.log` for unexpected activity
5. **Backup config**: `cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak`

## Uninstall

```bash
clawhub uninstall voice-email
```

Then remove API keys from openclaw.json if desired.

## Validation / Testing

To verify the skill is working:

1. Test Deepgram directly:
```bash
curl -X POST "https://api.deepgram.com/v1/listen" \
  -H "Authorization: Token $DEEPGRAM_API_KEY" \
  -H "Content-Type: audio/ogg" \
  --data-binary @sample.ogg
```

2. Test gogcli:
```bash
gog auth status
gog gmail send --to "your-email@gmail.com" --subject "Test" --body "Working!"
```

3. Send a voice message on Telegram:
   "new email to your-email@gmail.com, subject test, body hello, send"
