---
name: newsroom
description: "AI Newsroom: Generate professional podcasts, debates, and news briefings from any topic using ElevenLabs."
metadata:
  clawdbot:
    emoji: "mic"
    requires:
      bins: ["ffmpeg", "python3"]
      env: ["ELEVENLABS_API_KEY"]
    optional_env: ["OPENAI_API_KEY", "BRAVE_API_KEY"]
---

# Newsroom

Your personal AI newsroom. Turns any topic into a professional audio broadcast using ElevenLabs v3 audio tags.

## How It Works (Clawdbot Flow)

You (Claude) generate the script, then newsroom renders the audio. This is the primary workflow:

1. **Research** the topic using your own web search
2. **Write a script** in the format below with ElevenLabs v3 audio tags
3. **Save the script** to a temp file
4. **Run** `newsroom generate "Topic" --script /path/to/script.txt --format <fmt>`

### Script Format

One line per speaker turn. Tags are interpreted directly by ElevenLabs v3.

**IMPORTANT: No TV-isms.** This is audio only. Never write:
- "Thanks for watching" / "you're watching X" / "tune in next time"
- "Good evening, I'm X" or any self-introductions with show names
- Sign-offs, visual references, or any television conventions

Jump straight into the content.

```
ANCHOR: [serious] Tonight's top story: quantum computing just hit a major milestone.
ANCHOR: Researchers at MIT have achieved [excited] a breakthrough in error correction.
```

```
HOST: [excited] Alright, AI agents. Let's get into it.
CO-HOST: [laughing] Oh no, not again! [sigh] Just kidding, this is actually fascinating.
HOST: [thoughtful] So the big question is... can they actually replace us?
CO-HOST: [whisper] I hope not.
```

### Speaker Labels by Format

| Format | Speakers |
|--------|----------|
| `news` | ANCHOR |
| `podcast` | HOST, CO-HOST |
| `debate` | MODERATOR, SIDE-A, SIDE-B |
| `narrative` | NARRATOR |

### Available v3 Audio Tags

- **Emotions:** [excited], [sad], [angry], [surprised], [thoughtful], [happy], [annoyed], [appalled]
- **Delivery:** [whisper], [sarcastic], [dramatic], [serious]
- **Non-verbal:** [laughing], [sigh], [clears throat], [short pause], [long pause], [chuckles]
- Tags can appear anywhere: start of line, mid-sentence, between sentences

## CLI Usage

```bash
# Clawdbot flow: render a pre-written script
newsroom generate "Topic" --script script.txt --format podcast

# Standalone flow: auto-research + auto-script via OpenAI (requires OPENAI_API_KEY)
newsroom generate "Topic" --format podcast
newsroom generate "Topic" --format debate --length long
newsroom generate "Topic" --freshness pd                 # Only past day results
newsroom generate "Topic" --freshness pw                 # Only past week results
newsroom generate "Topic" --freshness 2026-01-30to2026-02-02  # Custom date range
newsroom generate "Topic" --dry-run                      # Script only, no audio
newsroom generate "Topic" --skip-research                # Skip Brave Search

# Utilities
newsroom voices                                          # Show voice assignments
newsroom config                                          # Show/init config file
```

## Natural Language Examples

- "Create a news briefing about SpaceX Starship"
- "Make a podcast about AI agents vs humans"
- "Generate a short debate on remote work"
- "Write a long narrative about the history of Rome"
- "Last 48 hours in AI news" (use `--freshness pd` or a custom date range)
- "This week's tech news" (use `--freshness pw`)

### Freshness Filter (Brave Search)

When the user asks for recent news within a specific timeframe, use `--freshness`:

| Value | Meaning |
|-------|---------|
| `pd` | Past day (24 hours) |
| `pw` | Past week |
| `pm` | Past month |
| `py` | Past year |
| `YYYY-MM-DDtoYYYY-MM-DD` | Custom date range |

Map user intent to the right filter: "last 48 hours" -> `pd`, "this week" -> `pw`, etc.

## Formats

| Format | Style | Default Voices |
|--------|-------|----------------|
| **news** | Professional single anchor | Eric |
| **podcast** | Two hosts with banter | Eric & Liam |
| **debate** | Moderator + 2 opposing sides | Eric + Liam & Bella |
| **narrative** | Cinematic storytelling | Domi |

## Length Options (standalone mode only)

| Length | Duration | Words |
|--------|----------|-------|
| `short` | ~2 min | ~300 |
| `medium` | ~5 min | ~750 |
| `long` | ~10 min | ~1500 |

## Configuration

Config file: `~/.config/newsroom/config.yaml`

Run `newsroom config` to generate defaults:

```yaml
voices:
  anchor: cjVigY5qzO86Huf0OWal     # Eric
  host: cjVigY5qzO86Huf0OWal       # Eric
  cohost: TX3LPaxmHKxFdv7VOQHJ     # Liam
  moderator: cjVigY5qzO86Huf0OWal  # Eric
  sidea: TX3LPaxmHKxFdv7VOQHJ      # Liam
  sideb: EXAVITQu4vr4xnSAxGW1      # Bella
  narrator: nPczCjz82KWdKScP46A1    # Domi
model: eleven_v3
output_format: mp3_44100_128
openai_model: gpt-5-mini
```

## Under the Hood

1. **Script** - Either provided by clawdbot or auto-generated via OpenAI (standalone)
2. **Parse** - Extract speaker turns, preserve v3 audio tags inline
3. **Audio** - Each segment sent to ElevenLabs v3 with tags intact, plus request stitching for continuity
4. **Mix** - ffmpeg concatenates interleaved segments into final MP3

### Request Stitching

Segments are generated in script order. Each voice tracks its last 3 request IDs, which are passed to subsequent generations for prosody continuity across the conversation.

## Data Layout

```
~/.clawd/skills/newsroom/{topic-slug}/{format}/
  script.txt
  audio/
    000_anchor.mp3
    001_anchor.mp3
    ...
  final.mp3
```
