---
name: youtube-ai-digest
description: Browses AI-related YouTube videos from subscribed channels, fetches transcripts, generates summaries, and creates Markdown reports. Use when the user mentions YouTube AI videos, video summaries, channel subscriptions, or asks about recent AI content from YouTube creators.
---

# YouTube AI Digest

Browse subscribed YouTube channels for AI-related videos, extract transcripts, and generate structured Markdown reports.

## Prerequisites

- Python 3.9+
- yt-dlp (`pip install yt-dlp`)

## Quick Start

### 1. Fetch Recent Videos

```bash
python scripts/fetch_videos.py --days 7 --keyword AI
```

Output: `data/videos.json` with filtered video list.

### 2. Get Transcript

```bash
python scripts/get_transcript.py --video-id VIDEO_ID
```

Output: `data/transcript_{VIDEO_ID}.txt` and `.json`.

### 3. Generate Report

```bash
python scripts/generate_report.py --video-id VIDEO_ID --summary "Your summary here"
```

Output: `data/output/{VIDEO_ID}/report.md` with thumbnail.

## Configuration

Edit `data/channels.json` to manage subscribed channels:

```json
{
  "channels": [
    {"name": "Two Minute Papers", "id": "UCbfYPyITQ-7l4upoX8nvctg"},
    {"name": "AI Explained", "id": "UCNJ1Ymd5yFuUPtn21xtRbbw"}
  ]
}
```

Find channel IDs from YouTube channel URLs: `youtube.com/channel/{CHANNEL_ID}`.

## Workflow

Copy this checklist to track progress:

```
Task Progress:
- [ ] Step 1: Fetch recent videos from channels
- [ ] Step 2: Review video list and select target
- [ ] Step 3: Get transcript for selected video
- [ ] Step 4: Analyze transcript and create summary
- [ ] Step 5: Generate Markdown report
```

**Step 1: Fetch recent videos**

Run `python scripts/fetch_videos.py --days 7` to get videos from the past week.

**Step 2: Review and select**

Check `data/videos.json` for available videos. Select one for analysis.

**Step 3: Get transcript**

Run `python scripts/get_transcript.py --video-id {ID}` to download subtitles.

**Step 4: Analyze and summarize**

Read the transcript file and create a concise summary covering:
- Main topics discussed
- Key insights and takeaways
- Notable timestamps

**Step 5: Generate report**

Run `python scripts/generate_report.py --video-id {ID} --summary "..."` to create the final Markdown report.

## Output Format

```markdown
# [Video Title]

![Thumbnail](thumbnail.webp)

## Video Info
- Channel: [Name]
- Published: [Date]
- Duration: [Length]
- Link: [URL]

## Summary
[AI-generated summary of content]

## Transcript
[Timestamped transcript excerpt]
```

## Scripts Reference

| Script | Purpose | Output |
|--------|---------|--------|
| `fetch_videos.py` | Fetch channel videos | `data/videos.json` |
| `get_transcript.py` | Download subtitles | `data/transcript_*.txt/json` |
| `generate_report.py` | Create Markdown report | `data/output/*/report.md` |

## Error Handling

**No transcript available**: Some videos lack subtitles. Check if auto-generated captions exist.

**Rate limiting**: Add delays between requests if fetching many channels.

**Network issues**: Retry with `--days 1` for fewer results.
