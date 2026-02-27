---
name: elevenlabs
description: "Generates audio using Eleven Labs API: TTS narration, sound effects, voice cloning, music, transcription. Use for audio generation, voice synthesis, or sound effects."
---

# Eleven Labs Audio Generation

Generate high-quality audio using the Eleven Labs API.

## Setup

Requires `ELEVEN_API_KEY` environment variable and `uv` package manager.

## Scripts

All scripts are UV-based and self-contained. Run directly:

### 1. Text-to-Speech Narration
Convert text/markdown to speech with female voices.

```bash
scripts/narrate.py "Hello world" -o hello.mp3
scripts/narrate.py -f article.md -o article.mp3
scripts/narrate.py "Welcome!" -v rachel -o welcome.mp3
```

**Voices:** sarah (default), rachel, dorothy

### 2. Sound Effects Generation
Create one-shot audio effects from descriptions.

```bash
scripts/sound-effect.py "short notification ding" -o ding.mp3 -d 1.5
scripts/sound-effect.py "whoosh swoosh sound" -o whoosh.mp3
scripts/sound-effect.py "rain on window" -d 10 --loop -o rain.mp3
```

### 3. Voice Cloning
Clone a voice from audio samples (min 3 recommended).

```bash
scripts/clone-voice.py "My Voice" sample1.mp3 sample2.mp3 sample3.mp3
```

### 4. Speech-to-Speech (Voice Conversion)
Transform audio to use a different voice.

```bash
scripts/voice-convert.py recording.mp3 -o sarah_version.mp3
scripts/voice-convert.py speech.wav -v rachel -o rachel_version.mp3
scripts/voice-convert.py noisy.mp3 -o clean.mp3 --remove-noise
```

### 5. Music Generation
Compose original music from prompts.

```bash
scripts/compose-music.py "upbeat electronic background music" -o bg.mp3
scripts/compose-music.py "calm piano melody" -d 60 -o piano.mp3
```

### 6. Audio Isolation
Remove background noise from recordings.

```bash
scripts/isolate-audio.py noisy_recording.mp3 -o clean.mp3
```

### 7. Speech-to-Text Transcription
Transcribe audio with timestamps.

```bash
scripts/transcribe.py recording.mp3
scripts/transcribe.py podcast.mp3 -o transcript.json
scripts/transcribe.py video.mp4 --format srt -o subtitles.srt
```

### 8. Voice Design
Generate new synthetic voices from descriptions.

```bash
scripts/design-voice.py "A warm, friendly female voice with British accent"
scripts/design-voice.py "Deep male narrator" -t "Welcome to the show" -o preview.mp3
```

## Voice IDs

**Female voices:**
- `EXAVITQu4vr4xnSDxMaL` — Sarah (soft, warm)
- `21m00Tcm4TlvDq8ikWAM` — Rachel (calm, clear)
- `ThT5KcBeYPX3keUQqHPh` — Dorothy (pleasant)

## Models

- `eleven_flash_v2_5` — Fastest, 32 languages, lowest cost
- `eleven_multilingual_v2` — Best quality, 29 languages
- `eleven_turbo_v2_5` — Balanced speed/quality

## Output Formats

- `mp3_44100_128` — Good quality (default)
- `mp3_22050_32` — Smallest file size
- `pcm_16000` — Raw PCM for processing
- `ulaw_8000` — Twilio compatible
