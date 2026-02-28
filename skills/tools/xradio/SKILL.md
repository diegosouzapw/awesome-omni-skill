---
name: xradio
description: CLI tool for pulling, caching, and reading Twitter/X posts offline
---

# xradio

A command-line tool for managing a local cache of Twitter/X posts. Pull tweets from your feed, read them offline, track read/unread status, and play videos.

## Prerequisites

- Browser server must be running for `pull` command: `browser server start`
- Must be logged into Twitter/X in the browser session
- `yt-dlp` and `mpv` for video playback

## Commands

### Pull Tweets from Feed

```bash
xradio pull --limit <n>
```

Fetches tweets from your Twitter home feed and saves them to local storage.

- `--limit <n>`: Number of tweets to pull (default: 10)
- Automatically scrolls to load more if needed
- **Never overwrites existing cached tweets** - skips duplicates
- Requires browser server to be running and logged into Twitter

**Example:**
```bash
xradio pull --limit 20
```

**Output:**
```
📡 Pulling up to 20 new tweets from feed...
🌐 Browser connected: https://x.com/home
🔄 Refreshing feed...
   scroll 1: +8 new, 2 cached (8/20 total new)
   scroll 2: +6 new, 4 cached (14/20 total new)
   scroll 3: +6 new, 3 cached (20/20 total new)

   ✅ [a1b2c3d] User Name: Tweet text preview...
   ✅ [e4f5g6h] Another User: More tweet content here...
📦 Done! Saved 20 new tweets.
```

### List Cached Tweets

```bash
xradio list           # Show unread tweets only
xradio list -A        # Show all tweets (including read)
xradio list --all     # Same as -A
```

Lists tweets in the offline cache.

- By default, only shows **unread** tweets
- Use `-A` or `--all` to include read tweets
- Shows media indicators: 🎬 (video), 🖼️ (image)
- Shows ✓ for read tweets (when using --all)

**Output:**
```
📚 4 unread tweets in cache:

  a1b2c3d🎬🖼️  Man, Crimson Desert looks like the kind of game that would make me disappear...
  e4f5g6h🖼️   Breaking: New development in the ongoing situation as officials announce...

Use 'xradio read <id>' to view a tweet, 'xradio mark <id>' to mark as read.
```

### Read a Tweet

```bash
xradio read <id>
```

Display the full content of a cached tweet.

- Supports **short IDs** (like git): `xradio read a1b` instead of full hash
- Shows formatted tweet with author, text, media indicators
- If ambiguous (multiple matches), asks for longer prefix

**Example:**
```bash
xradio read a1b
```

**Output:**
```
────────────────────────────────────────────────────────────
# User Name
**@handle** · 2h

This is the full tweet text content here...

---
🎬 Video | 🖼️ Image

[View on Twitter](https://x.com/handle/status/123456789)
────────────────────────────────────────────────────────────

ID: a1b2c3d (a1b2c3d4e5f6g7h8i9j0...)
Pulled: 2026-01-08T16:00:00.000Z

🎬 This tweet has a video. Use 'xradio play a1b' to play it.
```

### Mark as Read

```bash
xradio mark <id>
```

Mark a tweet as read. It will no longer appear in `xradio list` (only in `xradio list -A`).

**Example:**
```bash
xradio mark a1b
# Output: ✅ Marked a1b2c3d as read
```

### Unmark (Mark as Unread)

```bash
xradio unmark <id>
```

Remove the read status from a tweet, making it appear in the unread list again.

**Example:**
```bash
xradio unmark a1b
# Output: ↩️  Unmarked a1b2c3d (now unread)
```

### Describe Images

```bash
xradio describe <id>
xradio describe <id> "custom prompt here"
```

Describe images in a tweet using AI vision. Downloads images and generates a text description suitable for radio narration.

- Works only on tweets with images (🖼️ indicator)
- Uses AI vision model to describe the image content
- Optional custom prompt for specific questions about the image

**Example:**
```bash
xradio describe 836
```

**Output:**
```
🖼️  Describing 1 image(s) from: Visegrád 24
This is a line graph from Netblocks.org showing normalized network 
connectivity in Iran from January 7 to 8, 2026...
```

### Play Video

```bash
xradio play <id>
```

Play a video from a tweet that has video content.

- Extracts video URL via yt-dlp
- Plays with mpv (video + audio)
- Interactive controls: `q` to quit, `space` to pause

**Example:**
```bash
xradio play a1b
```

**Output:**
```
🎬 Extracting video URL for: User Name
   https://x.com/handle/status/123456789
📺 Playing video...
   (Press 'q' to quit, 'space' to pause)
```

## Storage Format

Tweets are stored in `storage/<id>.md` where `id` is SHA1 hash of the permalink.

Each file has YAML frontmatter + Markdown content:

```markdown
---
id: a1b2c3d4e5f6...
permalink: https://x.com/handle/status/123456789
displayName: User Name
handle: "@handle"
timestamp: "2h"
hasVideo: true
hasImage: false
pulledAt: 2026-01-08T16:00:00.000Z
offline:
  read: true
  readAt: 2026-01-08T17:00:00.000Z
---

# User Name
**@handle** · 2h

Tweet text content...

---
🎬 Video

[View on Twitter](https://x.com/handle/status/123456789)
```

## Typical Workflow

```bash
# 1. Start browser (if not running)
browser server start

# 2. Pull fresh tweets
xradio pull --limit 10

# 3. See what's new
xradio list

# 4. Read a tweet
xradio read a1b

# 5. If it has video, play it
xradio play a1b

# 6. Mark as read when done
xradio mark a1b

# 7. Continue with next unread tweet
xradio list
```

## Notes

- Short IDs work like git refs - use just enough characters to be unique
- The `offline.*` namespace in frontmatter is for our metadata (read status, etc.)
- Tweet data is never overwritten once cached - pull is additive only
- Video playback requires working yt-dlp Twitter extraction (may have intermittent issues with Twitter API)
