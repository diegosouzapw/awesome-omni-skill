---
name: nano-banana-skill
description: Generates, edits, and restores images using Google Gemini image models (Nano Banana). Use when the user wants to create images from text prompts, edit existing images with natural language, restore or enhance photos, or generate icons, patterns, diagrams, or visual content. Requires a GEMINI_API_KEY environment variable.
---

# Nano Banana Skill

Image generation, editing, and restoration powered by Google Gemini.

## Setup (One-Time)

Run the setup script to install dependencies automatically:

```bash
bash <skill-dir>/scripts/setup.sh
```

Then set an API key (get one at https://aistudio.google.com/apikey):

```bash
export GEMINI_API_KEY="your-api-key"
```

The API key can be set as any of these environment variables (checked in order):
`NANOBANANA_GEMINI_API_KEY`, `NANOBANANA_GOOGLE_API_KEY`, `GEMINI_API_KEY`, `GOOGLE_API_KEY`

## Quick Reference

| Mode | Purpose | Required Flags |
|------|---------|----------------|
| `generate` | Create image from text | `--prompt` |
| `edit` | Modify an existing image | `--input`, `--prompt` |
| `restore` | Enhance/fix an image | `--input` |

| Flag | Description | Default |
|------|-------------|---------|
| `--prompt` | Text description of desired image | — |
| `--input` | Input image path (edit/restore) | — |
| `--reference` | Style reference image(s), repeatable | — |
| `--output` | Output file path | Auto-named |
| `--size` | `1K`, `2K`, or `4K` | `1K` |
| `--format` | `png` or `jpeg` | `png` |
| `--preview` | Open image after generation | off |
| `--json` | Output structured JSON | off |

## Generate Images

Create images from text prompts:

```bash
# Basic generation
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "A banana floating in space"

# With specific size and format
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "Cyberpunk cityscape at night" --size 2K --format jpeg

# With explicit output path
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "Minimalist logo" --output ./my-logo.png
```

When `--output` is omitted, files are saved to `./nanobanana-output/` with auto-generated names based on the prompt.

## Edit Images

Modify existing images using natural language instructions:

```bash
# Change style
python3 <skill-dir>/scripts/nanobanana.py --mode edit --input photo.jpg --prompt "Convert to watercolor painting style"

# Modify content
python3 <skill-dir>/scripts/nanobanana.py --mode edit --input scene.png --prompt "Add a rainbow in the sky"

# Change colors
python3 <skill-dir>/scripts/nanobanana.py --mode edit --input logo.png --prompt "Change the color scheme to blue and gold"
```

## Restore Images

Enhance, repair, or upscale images:

```bash
# Auto-restore (uses default restoration prompt)
python3 <skill-dir>/scripts/nanobanana.py --mode restore --input old_photo.jpg

# Targeted restoration
python3 <skill-dir>/scripts/nanobanana.py --mode restore --input damaged.png --prompt "Remove scratches and improve sharpness"

# Enhance quality
python3 <skill-dir>/scripts/nanobanana.py --mode restore --input blurry.jpg --prompt "Enhance clarity and increase detail" --size 4K
```

## Reference Images

Use reference images to guide the style of generated images:

```bash
# Single reference
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "A mountain landscape" --reference style_guide.png

# Multiple references
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "Product photo" --reference brand_style.png --reference color_palette.png
```

The script searches for input and reference files in these locations:
- Current directory
- `./images/`
- `./input/`
- `./nanobanana-output/`
- `~/Downloads/`
- `~/Desktop/`

## Output

### Smart Naming

When `--output` is omitted, files are automatically named based on the prompt text:
- Prompt is converted to lowercase with special characters removed
- Spaces become underscores, limited to 32 characters
- Duplicate names get a numeric suffix (`_1`, `_2`, etc.)
- All auto-named files go into `./nanobanana-output/`

### Sizes

| Size | Description |
|------|-------------|
| `1K` | Standard resolution (default, fastest) |
| `2K` | High resolution |
| `4K` | Maximum resolution (slowest) |

## Model Selection

Set the `NANOBANANA_MODEL` environment variable to choose a model:

```bash
# Default model (fast, good quality)
export NANOBANANA_MODEL="gemini-2.5-flash-image"

# Pro model (highest quality)
export NANOBANANA_MODEL="gemini-3-pro-image-preview"
```

If unset, defaults to `gemini-2.5-flash-image`.

## Specialized Patterns

No special flags are needed for these -- just use descriptive prompts:

### Icons

```bash
python3 <skill-dir>/scripts/nanobanana.py --mode generate \
  --prompt "Flat design app icon for a weather app, rounded corners, minimal style, solid background" \
  --size 1K
```

### Patterns

```bash
python3 <skill-dir>/scripts/nanobanana.py --mode generate \
  --prompt "Seamless tileable geometric pattern, blue and white, high density, suitable for fabric print"
```

### Diagrams

```bash
python3 <skill-dir>/scripts/nanobanana.py --mode generate \
  --prompt "Architecture diagram showing microservices: API gateway, auth service, user service, database. Clean lines, labeled boxes"
```

### Sequential / Story Images

Generate each frame as a separate call, referencing previous outputs to maintain visual consistency:

```bash
# Frame 1
python3 <skill-dir>/scripts/nanobanana.py --mode generate \
  --prompt "Scene 1 of 3: A hero stands at the edge of a forest, sunrise, cinematic style" \
  --output ./nanobanana-output/story_1.png

# Frame 2 (reference previous for consistency)
python3 <skill-dir>/scripts/nanobanana.py --mode generate \
  --prompt "Scene 2 of 3: The same hero enters the forest, dappled light, cinematic style" \
  --reference ./nanobanana-output/story_1.png \
  --output ./nanobanana-output/story_2.png
```

## Preview

Add `--preview` to automatically open the generated image:

```bash
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "Sunset over ocean" --preview
```

Uses the system default image viewer (macOS: `open`, Linux: `xdg-open`, Windows: `start`).

## JSON Output

Add `--json` for structured output suitable for agent parsing:

```bash
python3 <skill-dir>/scripts/nanobanana.py --mode generate --prompt "A banana" --json
```

Output format:
```json
{
  "success": true,
  "files": ["nanobanana-output/a_banana.png"],
  "message": "Image generated successfully."
}
```

On failure:
```json
{
  "success": false,
  "files": [],
  "message": "Error description here"
}
```

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| No API key found | Missing environment variable | Set `GEMINI_API_KEY` |
| 403 / Permission denied | Invalid API key | Check key at https://aistudio.google.com/apikey |
| 429 / Rate limit | Too many requests | Wait and retry |
| Safety filter | Prompt flagged | Rephrase the prompt |
| No image generated | Model returned text only | Try a more specific prompt |
| File not found | Input/reference image missing | Check path or place in a searched directory |

See `references/ADVANCED.md` for advanced prompt engineering patterns, model comparison, and troubleshooting.
