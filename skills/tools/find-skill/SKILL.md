---
name: find-skill
description: Search for Agent Skills in local folders and on GitHub. Use when you need a capability you don't have—search before building.
license: Unlicense
metadata:
  author: community
  version: "0.1"
---

# Find Skill

Search for Agent Skills across local folders and GitHub.

## When to Use

You need a capability. Before building it, search. Someone may have already made it.

## Usage

```bash
# Search for a skill
python scripts/find.py "detect silence in audio"

# Local only (offline)
python scripts/find.py "resize images" --local-only

# JSON output for programmatic use
python scripts/find.py "send email" --json

# Fetch and display full SKILL.md content
python scripts/find.py "python" --fetch --limit 2
```

## Output

```
Found 2 skill(s) for "silence detection":

1. silence-detect
   Location: https://github.com/user/audio-tools
   Description: Detects silence gaps in audio files using ffmpeg...

2. audio-silence
   Location: ~/skills/audio-silence
   Description: Find silent regions in recordings...
```

## Configuration

Edit `scripts/config.json`:

```json
{
  "local_paths": ["~/skills/", "./skills/"],
  "github": {
    "enabled": true,
    "topic": "agentskills",
    "repos": [
      "your-username/your-skills-monorepo"
    ]
  }
}
```

### Config Options

| Field | Description |
|-------|-------------|
| `local_paths` | Local folders to search for skills |
| `github.enabled` | Enable/disable GitHub search |
| `github.topic` | Topic to search for (default: `agentskills`) |
| `github.repos` | Your personal skill repos (always searched first) |

### GitHub Token (Recommended)

A GitHub token provides higher rate limits (5000/hour vs 60/hour) and is required for searching private repos.

Create a `.env` file in the project root:

```
GITHUB_TOKEN=ghp_xxxxxxxxxxxx
```

Or set environment variables: `GITHUB_TOKEN` or `GH_TOKEN`

To create a token:
1. Go to GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token with `repo` scope (for private repos) or `public_repo` (for public only)

## What It Searches

1. **Local folders** — Scans configured paths for directories containing SKILL.md
2. **Your repos** — Searches repos listed in `github.repos` config (fast, reliable)
3. **Topic search** — Searches public repos with topic `agentskills`
4. **Code search** — Finds SKILL.md files containing your search terms

## After Finding

1. Read the description—is this what you need?
2. Clone or fetch the skill
3. Read the full SKILL.md before executing
4. Run in your environment

## Options

| Flag | Effect |
|------|--------|
| `--local-only` | Skip GitHub, search only local folders |
| `--json` | Output as JSON for parsing |
| `--fetch` | Fetch and display full SKILL.md content |
| `--limit N` | Maximum results (default: 10) |

## Notes

- This skill searches only; it does not execute found skills
- GitHub search requires network access
- With token: 5000 requests/hour. Without: 60 requests/hour
- Local search works offline
