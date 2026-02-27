---
name: md-to-image
description: Convert Markdown tables to PNG images for Telegram, WhatsApp, and other chat interfaces that don't support table formatting.
homepage: https://github.com/openclaw/openclaw
metadata:
  {
    "openclaw":
      {
        "emoji": "📊",
        "requires": { "bins": ["node"], "nodeModules": ["puppeteer-core"] },
        "install":
          [
            {
              "id": "npm",
              "kind": "npm",
              "path": "~/.openclaw/workspace/md-to-image",
              "label": "Install md-to-image dependencies",
            },
          ],
      },
  }
---

# md-to-image

Convert Markdown tables to PNG images for chat interfaces that don't support table formatting (like Telegram).

## When to use (trigger phrases)

Use this skill immediately when:

- Any response contains a markdown table destined for Telegram or WhatsApp
- User says "use table to image" or "convert this table"
- Output has `| column | column |` pattern and channel is Telegram

## Quick start

```bash
# Convert markdown file to PNG
node ~/.openclaw/workspace/md-to-image/md-to-image.js input.md output.png --scale=2

# Convert from stdin
echo "| A | B |\n|---|---|" | node md-to-image.js - output.png
```

## Auto-apply

This skill auto-applies at 95% confidence when:
- Channel is Telegram or WhatsApp
- Content contains `| table |` pattern
- 2+ rows, 2+ columns
- Data is informational (not ASCII art)

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `--scale` | 2 | Image scale (2x for retina) |
| `--theme` | calm | Color theme: calm, dark, colorful |
| `--width` | auto | Table width in pixels |

## Color themes

- **calm** (default): Black/gray headers, minimal
- **dark**: Dark mode friendly
- **colorful**: Blue/purple gradient headers

## How it works

1. Parses markdown table syntax
2. Generates HTML with CSS styling
3. Renders via Puppeteer (headless Chrome)
4. Captures screenshot of rendered table
5. Returns PNG file path

## Configuration

Add to agent config for auto-convert:

```json
{
  "skills": {
    "md-to-image": {
      "enabled": true,
      "autoConvert": ["telegram", "whatsapp"],
      "scale": 2,
      "theme": "calm"
    }
  }
}
```

## Requirements

- Node.js 16+
- npm
- Puppeteer-core (auto-installed)
- ~100MB disk space for Chromium

## Location

Converter script: `~/.openclaw/workspace/md-to-image/md-to-image.js`
