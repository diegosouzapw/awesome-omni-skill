---
name: add-video-to-collection
description: Manually add individual YouTube URLs to a custom collection CSV. Use when adding one-off videos to folders like library-of-minds or random, not from a channel extraction.
---

# Add Video to Collection

**Why?** Curated collections contain videos from multiple sources—manually editing CSVs causes formatting bugs and duplicates. This skill uses the CLI to add videos safely.

## Quick Start

```bash
ytscriber add "<youtube-url>" --folder <collection>
```

> [!IMPORTANT]
> **Always quote URLs** to prevent shell interpretation of `?` and `&` characters.

| Collection | Description |
|------------|-------------|
| library-of-minds | Thought leaders, interviews, educational content |
| random | Miscellaneous, one-off interesting content |
| (custom) | Any folder name you specify |

---

## Workflow

### 1. Identify the Target Collection

Determine which collection the video belongs to:

| If the video is... | Use collection |
|--------------------|----------------|
| Thought leader, interview, educational | `library-of-minds` |
| Miscellaneous, one-off interesting content | `random` |
| User specifies a collection | Use that collection |

### 2. Run the CLI Command

```bash
ytscriber add "<youtube-url>" --folder <collection>
```

> [!CAUTION]
> **NEVER manually edit CSV files** to add videos. Always use `ytscriber add` to ensure proper formatting and duplicate detection.

**Examples:**

```bash
# Add to library-of-minds
ytscriber add "https://www.youtube.com/watch?v=dQw4w9WgXcQ" --folder library-of-minds

# Add to random collection
ytscriber add "https://www.youtube.com/watch?v=jNQXAC9IVRw" --folder random

# Verbose output for debugging
ytscriber add "https://www.youtube.com/watch?v=9bZkp7q19f0" --folder random -v
```

### 3. Verify the Output

| Output | Meaning |
|--------|---------|
| `Added video XXX to ...` | Success — video was added |
| `Video XXX already exists in ...` | Duplicate — no action needed |
| Error message | See Troubleshooting below |

### 4. Offer Transcript Download (Optional)

After adding, ask the user if they want to download the transcript:

```bash
ytscriber download --folder <collection>
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---------|-------|----------|
| `command not found: ytscriber` | CLI not installed | Run `pip install -e .` from project root |
| `Invalid YouTube URL` | Malformed URL | Use full URL: `https://www.youtube.com/watch?v=VIDEO_ID` |
| `Permission denied` | File permissions | Check write access to data directory |
| Video not appearing in CSV | Silent failure | Run with `-v` flag to see debug output |

---

## Common Mistakes

1. **Unquoted URLs** — Shell interprets `?` as glob. Always wrap URLs in double quotes.
2. **Editing CSV directly** — Causes newline corruption and duplicate entries. Always use CLI.
3. **Using short URLs** — `youtu.be/XXX` works, but full `youtube.com/watch?v=XXX` is preferred.
4. **Wrong collection** — Double-check the collection name matches an existing folder or will be created.
5. **Forgetting to download transcript** — Adding to CSV doesn't download content. Run `ytscriber download` separately.

---

## Reference

**What the CLI handles automatically:**
- Duplicate detection (skips existing URLs)
- URL normalization to canonical format
- Proper CSV escaping and formatting
- Header creation for new CSV files
- Folder creation if it doesn't exist

**Metadata populated during transcript download:**
- Video title
- Duration
- Upload date
- Channel name
