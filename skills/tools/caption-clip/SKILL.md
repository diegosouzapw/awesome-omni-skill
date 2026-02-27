---
name: caption-clip
description: Download and caption YouTube clips. Use when the user asks to "caption a YouTube video", "download and add subtitles", "create captioned clips", "transcribe and caption a video", or mentions yt-dlp captioning workflows.
---

# Caption YouTube Clips

Download YouTube video clips with timestamps, transcribe with Deepgram, and burn styled captions.

## Prerequisites

- yt-dlp installed
- ffmpeg installed
- DEEPGRAM_API_KEY in .env file (or environment variable)
- Avenir Next font (falls back to Arial if unavailable)

## Workflow

### Step 1: Download Clip

Download a specific section of a YouTube video using yt-dlp:

```bash
yt-dlp --download-sections "*START-END" -o "clip_name.%(ext)s" "YOUTUBE_URL"
```

Example with timestamps `02:53-04:15`:
```bash
yt-dlp --download-sections "*02:53-04:15" -o "clip_intro.%(ext)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

### Step 2: Convert to MP4

Ensure consistent encoding with ffmpeg defaults:

```bash
ffmpeg -i clip_name.webm -y clip_name.mp4
```

### Step 3: Transcribe with Deepgram

Load API key from .env and call Deepgram API:

```bash
DEEPGRAM_API_KEY=$(grep '^DEEPGRAM_API_KEY=' .env | cut -d'=' -f2 | tr -d '"' | tr -d "'")

curl -s --request POST \
  --url 'https://api.deepgram.com/v1/listen?model=nova-2&smart_format=true&punctuate=true&utterances=true' \
  --header "Authorization: Token $DEEPGRAM_API_KEY" \
  --header 'Content-Type: audio/mp4' \
  --data-binary @clip_name.mp4 > clip_transcript.json
```

### Step 4: Generate SRT

Convert Deepgram JSON to SRT format using bundled script:

```bash
python3 ~/.claude/skills/caption-clip/scripts/json-to-srt.py clip_transcript.json clip.srt
```

### Step 5: Clean Up Transcription

Read the generated SRT file and create a cleaned version (`clip_clean.srt`):

1. **Fix transcription errors** - Correct misheard words
2. **Remove filler words** - Delete: "like", "you know", "I'd say", "yeah", "um", "uh", "sort of", "kind of"
3. **Consolidate fragments** - Merge short utterances into complete sentences
4. **Improve readability** - Adjust phrasing to flow better when read vs. heard
5. **Fix timestamps** - Ensure no invalid times (e.g., `00:00:60` should be `00:01:00`)

Write the cleaned content to `clip_clean.srt`.

### Step 6: Burn Captions

Apply styled subtitles with semi-transparent background:

```bash
ffmpeg -i clip_name.mp4 \
  -vf "subtitles=clip_clean.srt:force_style='FontSize=24,FontName=Avenir Next Medium,PrimaryColour=&H00FFFFFF,BackColour=&H80000000,BorderStyle=4,Outline=0,Shadow=0,MarginV=30,MarginL=40,MarginR=40'" \
  -c:a copy -y clip_captioned.mp4
```

## Caption Styling Reference

| Parameter | Value | Description |
|-----------|-------|-------------|
| FontSize | 24 | Point size |
| FontName | Avenir Next Medium | Clean sans-serif (use Arial as fallback) |
| PrimaryColour | &H00FFFFFF | White text (AABBGGRR format) |
| BackColour | &H80000000 | 50% transparent black background |
| BorderStyle | 4 | Opaque box using BackColour |
| MarginV | 30 | Vertical margin from bottom |
| MarginL/R | 40 | Horizontal margins |

## Alternative Fonts

If Avenir Next is unavailable, these work well for captions:
- SF Pro Display
- Helvetica Neue
- Futura
- Arial (universal fallback)

Check available fonts:
```bash
fc-list : family | sort -u | grep -iE '(avenir|sf pro|helvetica|futura)'
```

## Output Files

After completing the workflow:

| File | Description |
|------|-------------|
| `clip_name.mp4` | Original downloaded clip |
| `clip_transcript.json` | Raw Deepgram transcription (preserved for re-runs) |
| `clip.srt` | Auto-generated subtitles |
| `clip_clean.srt` | Cleaned/edited subtitles |
| `clip_captioned.mp4` | Final video with burned-in captions |
