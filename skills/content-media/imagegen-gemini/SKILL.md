---
name: imagegen-gemini
description: Generate/edit images via Gemini API (Nano Banana). Triggers: generate image, create picture, AI art, edit image, make illustration.
---

# Gemini Image Generation

Requires `GEMINI_API_KEY` env var. Get key: https://aistudio.google.com/apikey

## Usage

```bash
# Generate
python scripts/generate_image.py "prompt" -o output.png

# Edit existing image
python scripts/generate_image.py "prompt" -i input.jpg -o output.png

# Models: flash (default), flash-image (fast), pro-image (quality)
python scripts/generate_image.py "prompt" -m pro-image -o output.png
```

## Options

| Flag | Description |
|------|-------------|
| -o | Output path |
| -i | Input image (for editing) |
| -m | Model: flash, flash-image, pro-image |
| --json | JSON output |
