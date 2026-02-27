---
name: youtube-summarize
description: Summarize YouTube videos by extracting and analyzing their transcripts. Use when a user shares a YouTube URL and asks to summarize, explain, analyze, or extract information from the video. Triggers on YouTube links (youtube.com, youtu.be) with requests like "summarize this video", "what's this video about", "give me the key points", or "explain this video".
---

# YouTube Video Summarizer

Summarize YouTube videos by extracting transcripts and providing intelligent summaries. Saves summaries to an Obsidian vault and opens a styled preview in the browser.

## Prerequisites

The venv is already set up at `~/.claude/skills/youtube-summarize/venv` with `youtube-transcript-api` installed.

## Workflow

1. **Extract video ID** from the URL (supports youtube.com/watch, youtu.be, shorts)
2. **Fetch transcript** using `scripts/get_transcript.py`
3. **Summarize** the transcript based on user request
4. **Save & Preview** using `scripts/save_summary.py` - saves to Obsidian vault and opens styled HTML in browser

## Scripts

### Get Transcript

```bash
# Activate venv and get plain text transcript
source ~/.claude/skills/youtube-summarize/venv/bin/activate && python ~/.claude/skills/youtube-summarize/scripts/get_transcript.py "https://www.youtube.com/watch?v=VIDEO_ID"

# Get timestamped transcript
source ~/.claude/skills/youtube-summarize/venv/bin/activate && python ~/.claude/skills/youtube-summarize/scripts/get_transcript.py "URL" --format timestamped

# Specify language
source ~/.claude/skills/youtube-summarize/venv/bin/activate && python ~/.claude/skills/youtube-summarize/scripts/get_transcript.py "URL" --lang es
```

### Save Summary & Open Preview

After generating the summary, save it and open the preview:

```bash
source ~/.claude/skills/youtube-summarize/venv/bin/activate && python ~/.claude/skills/youtube-summarize/scripts/save_summary.py \
  --title "Video Title Here" \
  --url "https://youtube.com/watch?v=VIDEO_ID" \
  --summary "Your markdown summary here"
```

Options:
- `--no-preview`: Skip opening browser preview
- `--no-save`: Skip saving to Obsidian vault
- `--stdin`: Read summary from stdin instead of --summary flag

## Output Locations

- **Obsidian Vault**: `~/Documents/YouTube-Summaries/` (customize in `save_summary.py`)
- **Browser Preview**: Opens automatically in default browser (temp HTML file)

## Summarization Guidelines

After extracting the transcript, provide summaries based on user needs:

| Request Type | Output |
|-------------|--------|
| "Summarize" | 3-5 paragraph overview of main points |
| "Key points" | Bullet list of 5-10 main takeaways |
| "TLDR" | 2-3 sentence ultra-brief summary |
| "Explain" | Detailed explanation with context |
| "Timestamps" | Key moments with timestamps |

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `TranscriptsDisabled` | Video has no captions | Inform user; cannot extract |
| `NoTranscriptFound` | No transcript in requested language | Try without `--lang` flag |
| `VideoUnavailable` | Private/deleted video | Inform user |

## Notes

- Works with auto-generated and manual captions
- Supports 100+ languages (use ISO 639-1 codes)
- Shorts and regular videos both supported
- Summaries are saved with frontmatter for Obsidian compatibility
