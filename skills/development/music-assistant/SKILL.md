---
name: music-assistant
description: Control Home Assistant Music Assistant - browse library, search, play, manage preferences and moods.
version: 1.0.0
metadata:
  clawdbot:
    emoji: 🎵
    requires:
      env:
        - HA_URL
        - HA_TOKEN
---

# Music Assistant Skill

Control Home Assistant with Music Assistant integration - browse your music library, search for tracks, manage playback, set preferences, and create mood-based playlists.

## Requirements

- Home Assistant instance with [Music Assistant](https://music-assistant.io/) integration
- `HA_URL` - Your Home Assistant URL (e.g., `https://ha.example.com`)
- `HA_TOKEN` - Long-lived access token from Home Assistant

## Commands

### Speakers & Discovery

```bash
# List available media player speakers
ha-ma speakers

# Show Music Assistant configuration
ha-ma ma-config [--cached] [--refresh]
```

### Browse Library

```bash
# Browse by type: artists, albums, tracks, playlists, radio
ha-ma browse <type> [--parent <id>] [--limit <n>] [--offset <n>]

# Examples
ha-ma browse artists --limit 20
ha-ma browse albums --parent "artist:123"
ha-ma browse tracks --parent "album:456"
```

### Search

```bash
# Search for music
ha-ma search <query> [--limit <n>] [--type <media_type>]

# Examples
ha-ma search "Beatles" --limit 10
ha-ma search "jazz" --type artists
```

### Playback Control

```bash
# Play media on a speaker
ha-ma play --speaker <entity_id> --uri <uri> [--enqueue <mode>]

# Pause/Stop/Navigation
ha-ma pause --speaker <entity_id>
ha-ma stop --speaker <entity_id>
ha-ma next --speaker <entity_id>
ha-ma prev --speaker <entity_id>

# Volume control (0-100)
ha-ma volume --speaker <entity_id> --level <0-100>

# Add to queue
ha-ma queue --speaker <entity_id> --uri <uri>
```

Enqueue modes: `play`, `replace`, `next`, `add`

### User Preferences

```bash
# Set preference for user or household
ha-ma prefs set --user <slug> --<entity-type> <value> --score <score>
ha-ma prefs set --household <slug> --<entity-type> <value> --score <score>

# Get preferences
ha-ma prefs get --user <slug>
ha-ma prefs get --household <slug>

# Clear specific preference
ha-ma prefs clear --user <slug> --<entity-type> <value>
ha-ma prefs clear --household <slug> --<entity-type> <value>
```

Entity types: `--artist`, `--album`, `--genre`, `--track`

### Mood Mapping

```bash
# Create or update a mood
ha-ma mood set --user <slug> --name <name> --genres <g1,g2> [--decades <d1,d2>] [--energy <low|medium|high>]
ha-ma mood set --household <slug> --name <name> --genres <g1,g2> [--energy <level>]

# List moods
ha-ma mood list --user <slug>
ha-ma mood list --household <slug>

# Get specific mood
ha-ma mood get --user <slug> --name <name>

# Delete mood
ha-ma mood delete --user <slug> --name <name>
```

### Play History

```bash
# Recent play history
ha-ma history recent --user <slug> [--limit 20]

# List listening sessions
ha-ma history sessions --user <slug> [--limit 10]

# Avoid list management
ha-ma history avoid --user <slug> --uri <uri> [--reason <text>]
ha-ma history unavoid --user <slug> --uri <uri>
ha-ma history avoidlist --user <slug>
```

### Memory (Play Event Logging)

```bash
# Log a play event
ha-ma memory log --user <slug> --speaker-entity <entity_id> --uri <uri> [--title ...] [--artist ...] [--album ...]

# Recent play events
ha-ma memory recent --user <slug> [--limit 10]
```

## Configuration

Set environment variables in your OpenClaw config:

```json
{
  "skills": {
    "entries": {
      "music-assistant": {
        "env": {
          "HA_URL": "https://your-ha-instance.com",
          "HA_TOKEN": "your-long-lived-access-token"
        }
      }
    }
  }
}
```

## Database

The skill uses SQLite with Prisma for storing:
- User and household preferences
- Mood definitions
- Play history and sessions
- Avoid lists

Database is automatically initialized on first use.
