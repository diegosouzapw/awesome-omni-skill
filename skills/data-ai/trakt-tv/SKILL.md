---
name: trakt-tv
description: Trakt.tv integration for tracking movies and TV shows. Use when syncing watch history, checking what user is watching, updating watchlists, or integrating Trakt data into memory. Triggers on "trakt", "watch history", "what am I watching", "TV shows", "movies watched".
metadata:
  clawdbot:
    emoji: "📺"
    env:
      - TRAKT_CLIENT_ID
      - TRAKT_USERNAME
---

# Trakt.tv Skill

Sync and track movies/TV shows via Trakt.tv API.

## Setup

Required environment variables:
- `TRAKT_CLIENT_ID`: Get from https://trakt.tv/oauth/applications (create app, use Client ID)
- `TRAKT_USERNAME`: Your Trakt username (profile must be public, or use OAuth for private)

## API Endpoints

Base URL: `https://api.trakt.tv`

Headers required:
```
Content-Type: application/json
trakt-api-version: 2
trakt-api-key: {TRAKT_CLIENT_ID}
```

## Common Operations

### Get Watch History
```bash
curl -s "https://api.trakt.tv/users/{username}/history?limit=10" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

### Get Currently Watching
```bash
curl -s "https://api.trakt.tv/users/{username}/watching" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

### Get Watchlist
```bash
curl -s "https://api.trakt.tv/users/{username}/watchlist" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

### Get Watched Shows (with progress)
```bash
curl -s "https://api.trakt.tv/users/{username}/watched/shows" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

### Get Watched Movies
```bash
curl -s "https://api.trakt.tv/users/{username}/watched/movies" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

### Get Ratings
```bash
curl -s "https://api.trakt.tv/users/{username}/ratings" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

### Get Stats
```bash
curl -s "https://api.trakt.tv/users/{username}/stats" \
  -H "Content-Type: application/json" \
  -H "trakt-api-version: 2" \
  -H "trakt-api-key: $TRAKT_CLIENT_ID"
```

## Memory Integration

When syncing to memory, create/update a file like `entertainment.md`:

```markdown
# Entertainment

## Currently Watching
- **Show Name** - Season X, Episode Y (last watched: DATE)

## Recently Watched
- Movie Name (DATE) - ⭐ RATING/10

## Watchlist
- Show/Movie to watch

## Stats
- Total shows: X
- Total movies: Y
- Total hours: Z
```

## Polling Strategy

For regular sync, check:
1. `/users/{username}/history?limit=5` - Recent activity
2. Compare with last known state
3. Update memory only if changes detected
