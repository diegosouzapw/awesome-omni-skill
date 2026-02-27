---
name: podcastfy-clawdbot
description: Generate an AI podcast (MP3) from one or more URLs using the open-source Podcastfy project. Use when the user says “make a podcast from this URL/article/video/PDF”, “turn this webpage into a podcast”, or wants an MP3 conversation-style summary from links. Uses Gemini for transcript generation via GEMINI_API_KEY and Edge TTS for free voice.
---

# Podcastfy (Clawdbot)

Generate a podcast-style audio conversation (MP3) from a URL (or multiple URLs) using `podcastfy`.

This skill provides a wrapper script that:
- creates/uses a local venv (`{baseDir}/.venv`)
- installs/updates `podcastfy`
- runs Podcastfy with **Gemini** for transcript generation and **Edge** for TTS

## One-time setup

1) Ensure `ffmpeg` is installed on the host.

Ubuntu/Debian:
```bash
sudo apt-get update && sudo apt-get install -y ffmpeg
```

2) Provide a Gemini API key:

Set env var `GEMINI_API_KEY` (recommended), or create a project `.env` and export it before running.

Example keys file: `{baseDir}/references/env.example`

## Quick start

Generate an MP3 from a single URL:
```bash
cd {baseDir}
export GEMINI_API_KEY="..."
./scripts/podcastfy_generate.py --url "https://example.com/article"
```

Multiple URLs:
```bash
cd {baseDir}
./scripts/podcastfy_generate.py --url "https://a" --url "https://b"
```

Long-form:
```bash
cd {baseDir}
./scripts/podcastfy_generate.py --url "https://example.com/long" --longform
```

## Output

The script writes outputs under:
- `{baseDir}/output/audio/` (MP3)
- `{baseDir}/output/transcripts/` (transcript)

Podcastfy prints the final MP3 path on success.

## Optional tuning

- `PODCASTFY_LLM_MODEL` (default: `gemini-1.5-flash`)
- `PODCASTFY_EDGE_VOICE_Q` (default: `en-US-JennyNeural`)
- `PODCASTFY_EDGE_VOICE_A` (default: `en-US-EricNeural`)

## Automation / reliability tips (important)

### Prefer RSS feeds over browser automation for cron jobs

In isolated cron jobs, avoid relying on a system browser (Chrome/Chromium) or extra Python deps (e.g. `bs4`).
For TechCrunch categories, prefer the RSS feed:
- https://techcrunch.com/category/artificial-intelligence/feed/

This reduces breakage from:
- “No supported browser found …”
- missing site parsing dependencies in the cron runtime

### Validate MP3 output (avoid 0-second audio)

We observed `podcastfy` can occasionally produce an MP3 that is effectively empty/truncated (e.g., a ~261-byte file), which shows up as **0s** in Telegram.

This wrapper now validates MP3 output and will automatically fall back to **edge-tts** synthesis from the latest transcript when the MP3 is invalid.

## Safety / workflow

- Prefer “draft-first”: tell the user what will be generated (language/length) before running.
- Never paste API keys into chat logs.
