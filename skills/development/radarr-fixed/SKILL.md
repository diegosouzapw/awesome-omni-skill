---
name: radarr-fixed
version: 1.0.4
changelog: "Fixed config path from .openclagent to .openclaw and updated script paths"
description: Search and add movies to Radarr. Supports collections, search-on-add option. FORK of jordyvandomselaar/radarr with fixed metadata.
metadata:
  openclaw:
    emoji: "🎬"
    requires:
      bins:
        - curl
        - jq
      config:
        - path: ~/.openclaw/credentials/radarr/config.json
          description: "Radarr API configuration with url, apiKey, and defaultQualityProfile"
      env:
        - name: RADARR_URL
          optional: true
          description: "Radarr instance URL (overrides config file)"
        - name: RADARR_API_KEY
          optional: true
          description: "Radarr API key (overrides config file)"
    fork:
      original: jordyvandomselaar/radarr
      original_url: https://clawhub.com/jordyvandomselaar/radarr
      reason: "Fixed metadata to properly declare required credentials and config paths"
---

# Radarr (Fixed)

**⚠️ FORK NOTICE:** This is a fork of [jordyvandomselaar/radarr](https://clawhub.com/jordyvandomselaar/radarr) with corrected metadata declarations.

Add movies to your Radarr library with collection support.

## Setup

Create `~/.openclaw/credentials/radarr/config.json`:
```json
{
  "url": "http://localhost:7878",
  "apiKey": "your-api-key",
  "defaultQualityProfile": 1
}
```
- `defaultQualityProfile`: Quality profile ID (run `config` to see options)

### Alternative: Environment Variables
Instead of config file, you can use:
- `RADARR_URL` - Radarr instance URL
- `RADARR_API_KEY` - Radarr API key

## Workflow

1. **Search**: `search "Movie Name"` - returns numbered list
2. **Present results with TMDB links** - always show clickable links
3. **Check**: User picks a number
4. **Collection prompt**: If movie is part of collection, ask user
5. **Add**: Add movie or full collection

## Important
- **Always include TMDB links** when presenting search results to user
- Format: `[Title (Year)](https://themoviedb.org/movie/ID)`
- Uses `defaultQualityProfile` from config; can override per-add

## Commands

### Search for movies
```bash
bash scripts/radarr.sh search "Inception"
```

### Check if movie exists in library
```bash
bash scripts/radarr.sh exists <tmdbId>
```

### Add a movie (searches immediately by default)
```bash
bash scripts/radarr.sh add <tmdbId>           # searches right away
bash scripts/radarr.sh add <tmdbId> --no-search  # don't search
```

### Add full collection (searches immediately by default)
```bash
bash scripts/radarr.sh add-collection <collectionTmdbId>
bash scripts/radarr.sh add-collection <collectionTmdbId> --no-search
```

### Remove a movie
```bash
bash scripts/radarr.sh remove <tmdbId>              # keep files
bash scripts/radarr.sh remove <tmdbId> --delete-files  # delete files too
```
**Always ask user if they want to delete files when removing!**

### Get root folders & quality profiles (for config)
```bash
bash scripts/radarr.sh config
```

---
*Original skill by [jordyvandomselaar](https://clawhub.com/jordyvandomselaar). Fork maintained with permission under open source principles.*
