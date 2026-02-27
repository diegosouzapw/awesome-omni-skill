---
name: ultraplan
description: Record multi-modal context (audio, screenshots, clipboard, keystrokes) to generate detailed prompts. Use when user wants to capture context from meetings, research sessions, debugging workflows, or any activity they want to later analyze with Claude.
---

# ultraplan

A CLI tool for recording multi-modal context that can be loaded into Claude Code sessions.

## What It Captures

- **Audio transcription** - Real-time Whisper transcription of speech/system audio
- **Screenshots** - Triggered by "jj" hotkey (press j twice quickly)
- **Clipboard** - Content copied during session
- **Keystrokes** - What was typed (optional, can disable with `--no-keys`)

## Commands

### Start Recording

```bash
uv run ultraplan record
```

Options:
- `-o, --output <dir>` - Output directory (default: `~/.ultraplan/sessions`)
- `-m, --model <size>` - Whisper model: tiny, base, small, medium, large-v3 (default: base)
- `--device <name>` - Audio device (use `--list-devices` to see options)
- `--no-keys` - Disable keystroke logging
- `--no-clipboard` - Disable clipboard monitoring
- `--no-audio` - Don't save raw audio.wav
- `--list-devices` - Show available audio devices

Press **Ctrl+C** to stop recording.

### Check Setup (macOS)

```bash
uv run ultraplan setup
```

Verifies BlackHole audio driver and accessibility permissions.

## Output Structure

After recording, creates:
```
~/.ultraplan/sessions/session_YYYYMMDD_HHMMSS/
├── recording.md      # Human-readable timeline
├── recording.json    # Machine-parseable events
├── audio.wav         # Raw audio file
└── img_*.png         # Screenshots (timestamp in filename)
```

## Loading Recordings into Claude

### Slash Commands

- `/latest` - Load most recent recording
- `/context <session_name>` - Load specific session
- `/record` - Start a new recording

### SessionStart Hook

Auto-loads latest recording when starting Claude. See `examples/hooks/` for setup.

## macOS Audio Setup

To capture system audio (not just microphone):

1. Install BlackHole: `brew install blackhole-2ch`
2. Open Audio MIDI Setup (Spotlight → "Audio MIDI Setup")
3. Click + → Create Multi-Output Device
4. Check both your speakers AND BlackHole 2ch
5. Set Multi-Output Device as system output
6. Run: `uv run ultraplan record --device "BlackHole 2ch"`

## When to Use ultraplan

Suggest recording when the user:
- Is about to join a meeting they want notes from
- Is debugging something complex they want to document
- Is doing research they want to synthesize later
- Wants to capture their workflow for documentation
- Says "let me show you" or "I'll walk through this"

## Reading Recordings

When analyzing a recording:
1. Read the `recording.md` for the timeline
2. Check for `img_*.png` screenshots and read them
3. The JSON file has structured event data if needed

Screenshots are referenced in markdown as `![Screenshot](img_NNNNNN.png)` where NNNNNN is milliseconds since session start.
