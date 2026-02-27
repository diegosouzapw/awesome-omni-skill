---
name: video-to-claude
description: Analyze videos with Gemini and get build instructions for Claude. Supports YouTube URLs, GIFs, screen recordings, and local video files. Use when user shares a video/GIF and wants to understand how to build it, clone a UI, reverse engineer a product, or extract tutorial steps. Triggers on "watch this video", "clone this", "reverse engineer", "how do I build this", "analyze this demo".
---

# video-to-claude

Use Gemini's video understanding to extract build instructions from any video.

## Quick Start

```bash
python ~/.claude/skills/video-to-claude/scripts/video_to_claude.py <source> [flags]
```

## Flags

- `--micro` - Detailed micro-interaction specs (exact colors, timing, easing)
- `--pro` - Use Gemini Pro instead of Flash (4x cost, better quality)

## Examples

```bash
# YouTube tutorial
python ~/.claude/skills/video-to-claude/scripts/video_to_claude.py "https://youtube.com/watch?v=..."

# UI micro-interaction (detailed specs)
python ~/.claude/skills/video-to-claude/scripts/video_to_claude.py --micro button.gif

# Complex product demo (best quality)
python ~/.claude/skills/video-to-claude/scripts/video_to_claude.py --pro "https://youtube.com/watch?v=..."
```

## Supported Sources

- YouTube: `https://youtube.com/watch?v=...` (native, no download)
- Local files: `.mp4`, `.mov`, `.webm`, `.gif`
- Direct URLs: `video.twimg.com`, `cloudfront.net`, etc.
- Tweets: `https://x.com/user/status/...` (requires yt-dlp)

## Output

Returns JSON with:
- `product_name` - What was shown
- `description` - What it does
- `features` - List of capabilities
- `workflow` - Step-by-step flow
- `technical_notes` - Stack, APIs, libraries
- `skill_instructions` - How to build it

For `--micro` mode, also includes:
- Exact dimensions, colors (hex), typography
- Animation timing (ms), easing curves (cubic-bezier)
- State transitions and keyframes

## Environment

Requires `GEMINI_API_KEY` in environment.

## Cost

- Flash (default): ~$0.05/min of video
- Pro (`--pro`): ~$0.20/min of video
