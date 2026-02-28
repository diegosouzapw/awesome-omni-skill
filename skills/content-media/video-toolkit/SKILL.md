---
name: video-toolkit
description: Intelligent video processor for downloading media and extracting transcripts from YouTube and 1000+ supported sites. Automatically handles format selection, subtitle extraction, and post-processing.
allowed-tools: Read Write Glob Grep Bash(yt-dlp:*) Bash(ffmpeg:*) Bash(which:*) Bash(python3:*)
---

# Video Toolkit

This skill provides intelligent capabilities to download video/audio and extract transcripts using `yt-dlp`.

## Usage Guidelines

**Goal-Oriented Workflow**: DO NOT ask for confirmation unless the user's intent is ambiguous.
- If user says "Download": Download the best available quality video.
- If user says "Transcribe" or "Summarize": Extract subtitles (auto-generated if needed) and convert to clean text.
- If user says "Audio only": Extract mp3/m4a.

## Smart Defaults
- **Language**: Match the video's recognized language. Fallback to English (`en`).
- **Cookies**: Use specific browser cookies (e.g., `--cookies-from-browser chrome`) **ONLY** if the download fails with "Sign in" or "403 Forbidden" errors.
- **Format**: `mp4` for video, `srt` -> `txt` for transcripts.

## Workflows

### 1. Transcript & Text Extraction
Best for extracting content for reading, summarizing, or RAG.

1. **Extract Subtitles**:
   Downloads subtitles (manual or auto-generated) without downloading the video.
   ```bash
   yt-dlp --write-sub --write-auto-sub --sub-lang "en,zh-Hans,.*" --convert-subs srt --skip-download -o "%(title)s.%(ext)s" "[URL]"
   ```
   *(Note: Adjust `--sub-lang` priority based on the video's likely language if known)*

2. **Convert to Text**:
   Run the cleaning script to produce a readable `.txt` file:
   ```bash
   python3 scripts/clean_transcript.py "[FILE].srt"
   ```

### 2. Media Download
Best for saving files locally.

- **Standard Video**:
  ```bash
  yt-dlp -o "%(title)s.%(ext)s" "[URL]"
  ```

- **Audio Only**:
  ```bash
   yt-dlp -x --audio-format mp3 -o "%(title)s.%(ext)s" "[URL]"
  ```

### 3. Advanced & Troubleshooting
- **Permission Errors**: If you encounter 403 or Sign-in errors, **retry** with cookies:
  `yt-dlp --cookies-from-browser chrome ...` (or firefox/safari/edge depending on user OS).
- **Specific Formats**: If user requests specific resolution (e.g., 1080p) or format, consult the [yt-dlp Full Manual](references/yt-dlp-readme.md).
- **Unsupported Sites**: Check the [Supported Sites List](references/supports-site.md) if the URL seems obscure.

## References
- [yt-dlp Full Manual](references/yt-dlp-readme.md)
- [Supported Sites List](references/supports-site.md)
