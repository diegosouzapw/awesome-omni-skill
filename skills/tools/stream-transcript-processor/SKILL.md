---
name: stream-transcript-processor
description: Process Twitch/YouTube stream transcripts to identify clip-worthy moments, generate short-form script notes, draft X posts, YouTube metadata, and extract content insights. Use when given a stream transcript, VOD URL, or when asked to find clips/highlights from a techfren stream.
---

# Stream Transcript Processor

Process @techfren stream transcripts into actionable content assets: clips, script notes, X posts, YouTube metadata, and analytics.

**This skill orchestrates existing skills and adds new capabilities:**
- Uses **clipper** for clip extraction
- Uses **x-post-extractor** patterns for X posts
- Uses **media-fetcher** for screenshots and media
- **NEW**: Fetch transcripts from URLs (auto-captions)
- **NEW**: Generate script notes for short-form recording
- **NEW**: YouTube metadata (title, description with timestamps, SEO tags)
- **NEW**: Content analytics and insights

---

## When to Use This Skill

Activate when user:
- Provides a Twitch VOD or YouTube stream URL
- Pastes a stream transcript
- Asks to "find clips" or "find highlights" from a stream
- Wants "script notes" for short-form content
- Asks for X post drafts from stream content
- Needs YouTube title, description, or tags
- Requests stream analytics or insights

---

## Quick Commands

| User Request | Action |
|--------------|--------|
| "Process this stream [URL]" | Full pipeline: fetch → analyze → clips → posts → YouTube metadata |
| "Find clips from [transcript]" | Delegate to clipper skill |
| "Generate script notes for [timestamp]" | Create recording notes |
| "Draft X posts from [stream]" | Delegate to x-post-extractor or use draft script |
| "Generate YouTube description" | Create title, description with timestamps, tags |
| "Analyze this stream" | Content analytics report |

---

## Execution Workflow

### Full Pipeline (URL Provided)

When user provides a stream URL:

**Phase 1: Fetch Transcript**
```bash
python .claude/skills/stream-transcript-processor/scripts/fetch_transcript.py "<URL>" transcript.txt
```

This creates:
- `transcript.txt` - Timestamped text file
- `transcript.json` - Structured JSON for processing

**Phase 2: Parse for Clipper Compatibility**
```bash
python .claude/skills/clipper/scripts/parse_transcription.py transcript.json > parsed.json
```

**Phase 3: Find Clips (Use Clipper Skill)**

Follow the clipper skill workflow:
1. Analyze `parsed.json` in windows
2. Identify segments using clipper categories + voice patterns
3. Create `segments.json`
4. Report clip candidates to user

**Phase 4: Generate Content Assets**

For each high-priority clip:

*Script Notes:*
```bash
python .claude/skills/stream-transcript-processor/scripts/generate_script_notes.py parsed.json <start_ts> <end_ts>
```

*X Post Drafts:*
```bash
python .claude/skills/stream-transcript-processor/scripts/draft_x_posts.py parsed.json <timestamp> --style discovery
```

**Phase 5: Fetch Media (Optional)**

Use media-fetcher for screenshots of tools mentioned:
```bash
python .claude/skills/media-fetcher/scripts/capture_screenshot.py "<url>" --output media/
```

---

### Transcript Already Exists

If `parsed.json` or `out.json` already exists:

1. Skip fetch_transcript step
2. Use existing files for analysis
3. Proceed with clipper workflow
4. Add script notes and X posts as needed

---

### Script Notes Only

When user asks for script notes for a specific segment:

```bash
python .claude/skills/stream-transcript-processor/scripts/generate_script_notes.py \
    <transcript_file> <start_timestamp> <end_timestamp> \
    --output script_notes.md
```

**Output format (sentence starters for recording):**
```markdown
## HOOK
- [Bold claim or result]...
- This is [Tool] and it...

## VALUE STACK
- Free, open source...
- Runs locally...

## DEMO
- Let's try it...
- [React naturally]

## CLOSE
- [Quick recap]...
- Try it out...
```

---

### X Posts Only

When user asks to draft X posts from stream:

**Option 1: Use x-post-extractor skill (recommended for full workflow)**

Follow x-post-extractor SKILL.md for comprehensive extraction.

**Option 2: Quick draft from timestamp**
```bash
python .claude/skills/stream-transcript-processor/scripts/draft_x_posts.py \
    <transcript_file> <timestamp> --style <style>
```

Styles: `discovery`, `result`, `comparison`, `tutorial`

---

### YouTube Metadata Generation

Generate SEO-optimized YouTube title, description with timestamps, and tags:

```bash
python .claude/skills/stream-transcript-processor/scripts/generate_youtube_metadata.py \
    <transcript_file> [--title "Custom Title"] [--output youtube_metadata.md]
```

**Output includes:**
- Engaging title with hook patterns
- Description with auto-generated timestamps
- Tool links for all mentioned products
- Social links section
- SEO-optimized tags (up to 30)

**Example output:**
```markdown
## Title
```
Claude Haiku Benchmark: New Record! | Live Testing
```

## Description
```
🎯 Claude Haiku Benchmark: New Record! | Live Testing

In this stream, I test and demo various AI tools and share my findings live.
Watch to see real-time benchmarks, comparisons, and honest reactions.

⏱️ TIMESTAMPS
0:00 - Intro
6:12 - Setting up Playwright MCP
11:05 - Claude Code $200 plan discussion
47:35 - Testing Playrider MCP
1:01:31 - Playrider speed results
1:04:29 - Haiku 43 second record!

🔗 TOOLS MENTIONED
• Claude: https://claude.ai
• Playwright: https://playwright.dev
• Augment: https://augmentcode.com

📱 CONNECT
• Twitter/X: https://x.com/techfren
• GitHub: https://github.com/aj47
• Website: https://techfren.net

#AI #Coding #Programming #Claude #Benchmark #TechStream
```

## Tags
```
claude, haiku, benchmark, ai coding, playwright mcp, live coding, ai tools, programming, developer tools, automation
```
```

See [youtube-description-template.md](templates/youtube-description-template.md) for full template and SEO guidelines.

---

## Analytics Report

When user asks to "analyze" a stream, generate insights:

### Report Template

```markdown
# Stream Analytics: [Title/Date]

## Overview
- Duration: X hours
- Total transcript entries: N
- High-energy moments detected: N

## Topics Covered
| Topic | Mentions | Peak Timestamp |
|-------|----------|----------------|
| Cursor | 15 | 01:23:45 |
| Claude | 12 | 02:15:30 |
| ...

## Energy Peaks
Timestamps with highest concentration of reaction words:
1. [01:23:45] "Dude, this is insane..." (Score: 12)
2. [02:15:30] "Whoa, that actually worked..." (Score: 10)
3. ...

## Clip Potential
- High priority clips: N
- Medium priority clips: N
- Estimated short-form content: N minutes

## Value Props Mentioned
- Free: X times
- Open source: X times
- Local/offline: X times

## Recommended Extractions
1. [Timestamp] - [Brief description] - **Clip + X Post**
2. [Timestamp] - [Brief description] - **Script Notes**
3. ...
```

---

## Integration with Existing Skills

### Using Clipper

For full clip extraction, delegate to clipper:

```
# Claude internally:
# 1. Ensure parsed.json exists
# 2. Follow clipper SKILL.md workflow
# 3. Create segments.json
# 4. Extract clips
```

The clipper skill handles:
- Window-based analysis
- Segment identification
- FFmpeg extraction
- Cleanup processing

### Using X-Post-Extractor

For comprehensive X post extraction:

```
# Claude internally:
# 1. Ensure parsed.json exists
# 2. Follow x-post-extractor SKILL.md workflow
# 3. Create x_posts_output/ folder
# 4. Include media capture
```

The x-post-extractor skill handles:
- Trigger phrase detection
- Scoring and prioritization
- Post/thread formatting
- Media fetching

### Using Media-Fetcher

For screenshots and supplementary media:

```bash
# Screenshots of tools/websites
python .claude/skills/media-fetcher/scripts/capture_screenshot.py "<url>"

# Fetch logos
python .claude/skills/media-fetcher/scripts/fetch_logos.py "<domain>"

# Extract video clips
ffmpeg -ss <start> -i "<video>" -t <duration> clip.mp4
```

---

## Voice Pattern Detection

When analyzing transcripts, look for these signals (see [voice-patterns.md](references/voice-patterns.md)):

### High-Value Signals (Score +4)
- Discovery phrases: "just discovered", "trending", "check this out"
- Comparison signals: "better than", "beats", "vs"
- Resolution markers: "finally", "figured it out", "turns out"

### Medium-Value Signals (Score +2-3)
- High-energy reactions: "dude", "whoa", "sick", "boom"
- Value props: "free", "open source", "local"
- Demo phrases: "let's see", "let's try", "here we go"
- Result moments: "it worked", "there we go", "boom"

### Tool Detection
```python
TOOLS = [
    'cursor', 'claude', 'gpt', 'copilot', 'windsurf', 'augment',
    'ollama', 'llama', 'mistral', 'groq', 'anthropic', 'openai'
]
```

---

## Clip Criteria Summary

From [clip-criteria.md](references/clip-criteria.md):

### What Makes a Good Clip
- **Discovery moments** with specific tools/results
- **Authentic reactions** that feel genuine
- **Value props** stacked together (free + open source + local)
- **Result moments** showing proof something works
- **Tool demos** with live testing

### Optimal Lengths
- TikTok/Reels/Shorts: 30-60 seconds
- Longer tutorials: 2-3 minutes
- X video: 30-45 seconds

### Skip These
- Slow starts ("so...", "hey what's up...")
- Incomplete thoughts
- Negative content without resolution
- Off-topic tangents

---

## Output Folder Structure

When running full pipeline:

```
./
├── transcript.txt          # Fetched auto-captions
├── transcript.json         # Structured transcript
├── parsed.json             # Clipper-compatible format
├── segments.json           # Identified clips
├── youtube_metadata.md     # Title, description, tags
├── clips/                  # Extracted video clips
│   ├── 001_Tool_Demo.mp4
│   ├── 002_Discovery.mp4
│   └── cleaned/
│       └── ...
├── script_notes/           # Recording notes
│   ├── 01_tool_demo.md
│   └── 02_discovery.md
├── x_posts_output/         # X post drafts
│   ├── 01_tool.txt
│   ├── 02_discovery.txt
│   └── README.md
└── media/                  # Screenshots, logos
    ├── cursor.com.png
    └── github_tool.png
```

---

## Requirements

### For Transcript Fetching
- `yt-dlp` - Download auto-captions (`brew install yt-dlp`)

### For Clip Extraction
- `ffmpeg` - Video processing (`brew install ffmpeg`)
- Python 3.8+

### For Screenshots (Optional)
- `playwright` - Browser automation (`pip install playwright && playwright install chromium`)

---

## Example Usage

### Full Pipeline
```
User: "Process this stream https://youtube.com/watch?v=abc123"

Claude:
1. Fetches transcript → transcript.txt
2. Parses for clipper → parsed.json
3. Analyzes and finds 15 clips
4. Generates script notes for top 5
5. Drafts 3 X posts
6. Captures screenshots for tools mentioned
7. Reports summary with links to all outputs
```

### Quick Script Notes
```
User: "Generate script notes for the segment at 1:23:45"

Claude:
1. Loads transcript
2. Extracts 60-second window around timestamp
3. Generates HOOK/VALUE/DEMO/CLOSE structure
4. Outputs to terminal or saves to file
```

### Analytics Only
```
User: "Analyze this stream for clip potential"

Claude:
1. Scans transcript for voice patterns
2. Counts topics, energy peaks, value props
3. Identifies top clip candidates
4. Generates analytics report
```

---

## Troubleshooting

### "No transcript found"
- Video may not have auto-generated captions
- Try different language variants (en-US, en-GB)
- Use full transcription instead (batch-processor skill with `tr` alias)

### "Low clip scores"
- Stream may be low-energy or monotone
- Lower thresholds or look for teaching moments
- Focus on specific tool mentions

### "Script notes feel generic"
- Need more specific timestamp with clear tool/demo
- Check if segment has enough context
- May need to expand timestamp window

---

## Supporting Documents

- [voice-patterns.md](references/voice-patterns.md) - Detection patterns for scoring
- [clip-criteria.md](references/clip-criteria.md) - What makes a good clip
- [script-notes-template.md](templates/script-notes-template.md) - Script notes format
- [youtube-description-template.md](templates/youtube-description-template.md) - YouTube SEO template
- [clipper/SKILL.md](../clipper/SKILL.md) - Full clip extraction workflow
- [x-post-extractor/SKILL.md](../x-post-extractor/SKILL.md) - X post extraction workflow
- [x-post-extractor/VOICE.md](../x-post-extractor/VOICE.md) - Writing voice guidelines
