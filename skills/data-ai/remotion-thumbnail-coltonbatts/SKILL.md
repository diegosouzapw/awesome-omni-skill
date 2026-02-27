---
name: remotion-thumbnail
description: Generate professional YouTube thumbnails with AI-powered expression cutouts and Remotion rendering. Perfect for content creators who want consistent, high-quality thumbnails at scale.
license: MIT
compatibility: Requires Node.js 18+, npm/npx. Uses Replicate API for background removal.
metadata:
  author: alternative-design
  version: "1.0.0"
  category: video-production
  tags: youtube thumbnails remotion ai content-creation
---

# Remotion Thumbnail Generator

Generate professional YouTube thumbnails using your own expression cutouts and Remotion's powerful rendering engine.

## What This Skill Does

This skill helps you create consistent, high-quality YouTube thumbnails by:

1. **Capturing your expressions** - Guide you through 25 preset expressions
2. **Creating cutouts** - Automatically remove backgrounds using AI
3. **Generating thumbnails** - Render professional thumbnails with custom text, backgrounds, and styles

## Quick Start

### Prerequisites

- Node.js 18 or higher
- Replicate API key (for background removal) - Get one at https://replicate.com
- A webcam or camera for capturing expressions

### Installation

The skill files should be in your agent's skills directory. The renderer is a self-contained Remotion project.

```bash
# Install renderer dependencies
cd <skill-dir>/renderer
npm install
```

### Set Up Your Environment

```bash
# Set your Replicate API key
export REPLICATE_API_TOKEN="your-api-key-here"
```

## Workflow

### Phase 1: Capture Expressions

Run the interactive capture script to photograph yourself in 25 different expressions:

```bash
node <skill-dir>/scripts/capture.js
```

**What it does:**
- Shows you each expression to capture (e.g., "Happy/Smiling", "Shocked", "Pointing Up")
- Tells you where to save each photo
- Tracks which expressions you've captured
- Saves metadata to `storage/expressions_db.json`

**Tips:**
- Use good lighting (natural light from a window works great)
- Keep the same framing/distance for all shots
- Wear solid colors (avoid busy patterns)
- Plain background makes cutout processing easier

### Phase 2: Generate Cutouts

Process all your raw photos to create transparent-background cutouts:

```bash
node <skill-dir>/scripts/process_cutouts.js
```

**What it does:**
- Reads all captured expressions from the database
- Sends each photo to Replicate's background removal AI
- Downloads and saves the cutout PNGs
- Updates the database with cutout URLs

**Time:** ~30-60 seconds per expression

### Phase 3: Generate Thumbnails

Once you have cutouts, generate thumbnails on demand:

```bash
npx remotion still Thumbnail <skill-dir>/renderer/out/thumbnail.png \
  --props='{
    "headline": "This Changed Everything",
    "emotionId": 5,
    "stylePreset": "bold",
    "cutoutUrl": "<skill-dir>/storage/cutouts/5_cutout.png",
    "bgUrl": "https://example.com/background.jpg"
  }'
```

**Parameters:**
- `headline` (string): The main text to display
- `emotionId` (number): Which expression to use (1-25)
- `stylePreset` (string): Visual style - "bold", "dramatic", or "clean"
- `cutoutUrl` (string): Path to your expression cutout
- `bgUrl` (string): URL or path to background image

**Output:** High-res PNG thumbnail ready for YouTube

## Expression Reference

See [EXPRESSIONS.md](references/EXPRESSIONS.md) for the full list of 25 expressions and usage tips.

## Style Presets

### "bold"
- Vibrant colors
- Strong contrast
- Dynamic composition
- Best for: Entertainment, gaming, reaction content

### "dramatic"
- Moody lighting
- Cinematic feel
- Subdued colors
- Best for: Documentaries, serious topics, storytelling

### "clean"
- Minimal design
- Soft colors
- Professional look
- Best for: Educational, business, tutorials

## Advanced Usage

### Custom Backgrounds

You can use:
- **Image URLs:** Direct link to any image
- **Local files:** File path to an image on disk
- **AI-generated:** Use DALL-E, Midjourney, etc. and pass the URL
- **Solid colors:** Pass a color code (implementation detail in renderer)

### Batch Generation

Generate multiple thumbnails programmatically:

```bash
# See references/BATCH_GENERATION.md for examples
```

### Customizing the Renderer

The Remotion renderer is a standard React/Remotion project. You can:
- Modify styles in `renderer/src/Thumbnail.tsx`
- Add new style presets
- Adjust layout/composition
- Add animations (for video thumbnails)

See [Remotion docs](https://remotion.dev) for details.

## Troubleshooting

### "Expression not found"
- Make sure you've run Phase 1 (capture) and Phase 2 (cutouts)
- Check `storage/expressions_db.json` to see which expressions are ready

### "Replicate API error"
- Verify your `REPLICATE_API_TOKEN` is set correctly
- Check your Replicate account has credits
- See [TROUBLESHOOTING.md](references/TROUBLESHOOTING.md) for more

### "Renderer not found"
- Ensure you've run `npm install` in the renderer directory
- Check that Node.js 18+ is installed

For more issues, see [references/TROUBLESHOOTING.md](references/TROUBLESHOOTING.md)

## File Structure

```
remotion-thumbnail/
├── SKILL.md                    # This file
├── scripts/
│   ├── capture.js              # Interactive expression capture
│   └── process_cutouts.js      # Background removal pipeline
├── references/
│   ├── SETUP.md                # Detailed setup guide
│   ├── EXPRESSIONS.md          # Expression reference
│   ├── BATCH_GENERATION.md     # Batch processing examples
│   └── TROUBLESHOOTING.md      # Common issues & solutions
├── assets/
│   └── expressions.json        # Expression definitions
├── renderer/                   # Remotion project
│   ├── src/
│   │   └── Thumbnail.tsx       # Main thumbnail component
│   └── package.json
└── storage/                    # Generated at runtime
    ├── raw/                    # Your captured photos
    ├── cutouts/                # AI-processed cutouts
    └── expressions_db.json     # Expression metadata
```

## Performance Tips

- **First run:** Cutout generation takes time (~30-60s per expression)
- **Subsequent thumbnails:** Instant once cutouts are ready
- **Caching:** Cutouts are reusable forever - generate once, use unlimited times

## License

MIT - See LICENSE file for details.

## Credits

Created by [Alternative Design](https://github.com/alternative-design)  
Powered by [Remotion](https://remotion.dev) and [Replicate](https://replicate.com)

## Support

- **Issues:** https://github.com/alternative-design/remotion-thumbnail/issues
- **Discussions:** https://github.com/alternative-design/remotion-thumbnail/discussions
- **Twitter:** [@alternativedesign](https://twitter.com/alternativedesign)

---

**Pro tip:** Once you have your expression library set up, you can generate unlimited thumbnail variations in seconds. Perfect for A/B testing or maintaining consistent branding across your channel!
