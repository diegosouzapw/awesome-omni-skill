---
name: omni-executor
description: Omni v2 message execution specialist — use for sending messages (text, TTS, media, reactions, stickers, polls), searching messages, batch operations, and testing automations via the omni CLI.
version: "1.0"
---
> ⚠️ DEPRECATED: Superseded by atomic skills in `plugins/omni/skills/`.
> omni-orchestrator → use omni-instances/SKILL.md + omni-config/SKILL.md
> omni-executor → use omni-send/SKILL.md
> omni-analytics → use omni-events/SKILL.md


# Omni Executor

## Prerequisites

- `omni` CLI installed and in PATH (`omni auth login --api-key sk_xxx --api-url <url>`)
- `jq` available for JSON processing

## Sending Messages

Use `omni send` with the appropriate flags for each message type:

```bash
# Text
omni send --to <recipient> --text "Hello!" --instance <id>

# TTS voice note
omni send --to <recipient> --tts "Voice message" --instance <id>

# Media (image, audio, video, document)
omni send --to <recipient> --media ./file.jpg --caption "Caption" --instance <id>

# Reaction
omni send --to <recipient> --reaction "👍" --message <msg-id> --instance <id>

# Sticker
omni send --to <recipient> --sticker https://example.com/sticker.webp --instance <id>

# Poll (Discord)
omni send --to <channel-id> --poll "Question?" --options "A,B,C" --instance <id>

# Embed (Discord)
omni send --to <channel-id> --embed --title "Title" --description "Body" --instance <id>
```

## Instance Resolution

The CLI matches instance IDs intelligently:

- Full UUID: `00000000-1111-2222-3333-444444444444`
- UUID prefix: `c3a4f`
- Exact name: `cezar-personal`
- Substring: `personal`

## Searching Messages

```bash
omni messages search "keyword" --since 7d --limit 50
omni messages search "" --type audio --since 30d --json
omni messages search "urgent" --chat <chat-id> --since 24h
```

### Filters

| Filter | Description |
|--------|-------------|
| `--content` | Full-text search |
| `--type` | text, audio, image, video |
| `--chat` | Specific chat ID |
| `--since` | Time range (7d, 24h, 30min) |
| `--limit` | Max results |

## JSON Output and Piping

Every command supports `--json` for structured output:

```bash
omni send --to <phone> --text "Hi" --json | jq -r '.data.messageId'
omni instances list --json | jq '.[] | select(.status=="connected")'
omni chats list --instance <id> --json | jq '.[] | select(.unreadCount > 0)'
```

## Testing Automations

```bash
omni automations test <id> --dry-run
omni automations logs <id>
```

## Rate Limiting

When sending in loops, add a delay between messages:

```bash
for recipient in "${recipients[@]}"; do
  omni send --to "$recipient" --text "Hello" --instance <id> --json
  sleep 1
done
```

## Error Handling

Check exit codes and capture stderr:

```bash
if ! output=$(omni send --to <phone> --text "Hi" --instance <id> --json 2>&1); then
  echo "Error: $output" >&2
  exit 1
fi
```
