---
name: youtube-transcripter
description: Extract a YouTube video transcript from a URL and summarize it into important content, learnings, and suggestions. Use when the user provides a YouTube link and wants a clean transcript and structured notes without timestamps, with fallback extraction methods if the primary tool is unavailable.
---

# Youtube Transcripter

## Overview

Extract a transcript from a YouTube URL and produce a concise, structured summary focused on key content, learnings, and actionable suggestions.

## Workflow

### 1) Confirm input

- Require a YouTube URL or video ID. Multiple inputs are allowed.
- Assume no timestamps unless the user explicitly asks.
- Default language is auto-detected; ask only if a specific language is required.

### 2) First-time setup (one-time)

Make sure at least one transcript extractor is installed. Prefer `yt-dlp`:

```bash
choco install yt-dlp -y
```

If Chocolatey is not available or not elevated, use pip:

```bash
python -m pip install yt-dlp
```

Optional (API-based):

```bash
python -m pip install youtube-transcript-api
```

### 3) Extract transcript (best-effort with fallback)

Preferred: run the bundled script:

```bash
python scripts/get_transcript.py "https://www.youtube.com/watch?v=VIDEO_ID"
```

Multiple inputs (URLs or IDs) into a folder:

```bash
python scripts/get_transcript.py VIDEO_ID_1 "https://youtu.be/VIDEO_ID_2" --out-dir transcripts
```

Notes:
- `--out` is only for single input.
- `--out-dir` writes `transcript_<id>.txt` for each input. Default is `transcripts/`.
- `--wrap 125` reflows lines to 125 chars; use `--wrap 0` to keep raw line breaks.

To force a language:

```bash
python scripts/get_transcript.py "https://www.youtube.com/watch?v=VIDEO_ID" --lang en --out transcript_VIDEO_ID.txt
```

Fallback order if the script fails:

1) Install `yt-dlp` (prefer Chocolatey if available)

Check:
```bash
choco -v
```

If Chocolatey is available, install (requires admin shell):
```bash
choco install yt-dlp -y
```

If Chocolatey is not available or not elevated, use pip:
```bash
python -m pip install yt-dlp
```

2) `yt-dlp` with auto captions (binary or Python module)
```bash
yt-dlp --skip-download --write-auto-sub --sub-lang en --sub-format vtt "https://www.youtube.com/watch?v=VIDEO_ID"
```
Then strip timestamps and cue markers from the `.vtt` file.

3) If tools are unavailable, ask the user for the transcript text or a downloaded caption file.

### 4) Capture metadata (title + channel)

If `yt-dlp` is available, fetch title and channel:

```bash
yt-dlp --print "%(title)s|%(channel)s|%(uploader)s" "https://www.youtube.com/watch?v=VIDEO_ID"
```

PowerShell-safe quoting:

```powershell
yt-dlp --print '%(title)s|%(channel)s|%(uploader)s' "https://www.youtube.com/watch?v=VIDEO_ID"
```

If tools are unavailable, ask the user for the video title and channel name.
If `yt-dlp` warns about missing JavaScript runtime, note that it may omit some fields; use `channel` if `uploader` is missing.

### 5) Summarize and structure notes

Always save the structured summary to a file for agent-friendly reading, e.g. `transcript_summary.md` in the working directory. Keep the raw transcript as `transcript.txt`.
Always keep raw transcripts in `transcripts/` and save summaries in the working directory.
For multiple videos, include the YouTube ID in filenames: `transcript_<id>.txt` and `transcript_summary_<id>.md`.
Write files as UTF-8 to preserve umlauts and other non-ASCII characters.
Do not paste the full summary into chat; confirm file paths only.

Use this output template. Keep it tight and information-dense.

```markdown
# Title
Channel: <channel name>
URL: <youtube url>
Video ID: <id>

## Summary
- 3-6 bullets capturing the core message

## Key Points
- ...

## Learnings
- ...

## Suggestions / Recommendations
- ...

## Action Items (optional)
- ...

## Open Questions (optional)
- ...
```

## Quality checks

- Remove filler, intros, sponsor reads, and repeated segments.
- Preserve important claims, stats, or steps.
- If advice is speculative, label it as such.
- Keep language plain and direct.
- If you hit HTTP 429 (rate limit), wait and retry later or switch to a different network/IP; changing language rarely resolves rate limits.

## Resources

### scripts/

- `scripts/get_transcript.py` extracts transcripts for one or more URLs/IDs via `youtube-transcript-api` with `yt-dlp` binary or Python module fallback.

