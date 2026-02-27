---
name: "favicon-generator"
description: "Use when users need favicon/app icon generation or full PWA icon-pack export. Do not use for general non-icon image editing tasks."
---

# Favicon Generator Skill

Use this skill when the user asks to generate a favicon, app icon, or full PWA icon set.

## Scope

This skill is icon-focused. For broad image editing/generation tasks that are not icon-related, use a general image generation workflow.

## Required outputs

A complete run should produce/update:

- `public/icon-512x512.png`
- `public/icon-192x192.png`
- `public/icon-maskable-512x512.png`
- `public/icon-maskable-192x192.png`
- `public/apple-touch-icon.png`
- `public/badge-72x72.png`
- `app/favicon.ico`

## Hard rules

- Do **not** read or print `.env.local` / `.env*` contents to terminal.
- Do **not** echo key fragments (prefix/suffix) for `OPENAI_API_KEY`.
- Do **not** search for `scripts/image_gen.py` in the project root.
- Do **not** modify `scripts/image_gen.py` unless explicitly requested.
- Stop after one failed live API attempt caused by network/DNS issues and report blocker + exact next command.
- Shell tool calls are isolated. Environment variables from one command are **not** guaranteed to exist in later commands.
- The CLI auto-loads `.env.local` then `.env` from current working directory before key checks.

## Critical path

The bundled CLI lives inside this skill folder. Do **not** look for `scripts/image_gen.py` in the project root.

Resolve the CLI path first:

```bash
if [ -f ".agents/skills/favicon-generator/scripts/image_gen.py" ]; then
  ICON_CLI=".agents/skills/favicon-generator/scripts/image_gen.py"
elif [ -f "$HOME/.agents/skills/favicon-generator/scripts/image_gen.py" ]; then
  ICON_CLI="$HOME/.agents/skills/favicon-generator/scripts/image_gen.py"
else
  echo "favicon-generator CLI not found" >&2
  exit 1
fi
```

## Safe preflight (run before generation)

```bash
python3 "$ICON_CLI" generate --prompt "test icon" --dry-run
```

The CLI output will report whether `OPENAI_API_KEY` is set after auto-loading env files.

Check DNS before live API calls:

```bash
python3 - <<'PY'
import socket
socket.gethostbyname("api.openai.com")
print("DNS ok: api.openai.com")
PY
```

If DNS lookup fails, do not attempt a live generate call in that environment.

## Dependency check

```bash
python3 - <<'PY'
import importlib.util
print('openai', bool(importlib.util.find_spec('openai')))
print('PIL', bool(importlib.util.find_spec('PIL')))
PY
```

If missing, run exactly one deterministic install path:

```bash
python3 -m pip install --user openai pillow
```

If Python is externally managed (PEP-668), use a local venv and continue there. Do not thrash between multiple install strategies.

## Prompt crafting guidelines

The API model (`gpt-image-1.5`) has specific strengths and limitations that affect icon quality:

- **Text rendering is unreliable** — always include "no text, no lettering" in constraints.
- **Composition control is limited** — keep the concept to a single centered element. Do not ask for multiple objects or complex layouts.
- **The model follows color instructions well** — use exact hex values from the project's `manifest.ts` (`theme_color`, `background_color`).
- **Fine detail disappears at small sizes** — the generated 1024x1024 master gets scaled down to 16x16 for `favicon.ico` and 72x72 for `badge`. Avoid thin strokes, fine lines, or intricate patterns.

### Prompt structure (for `--prompt`)

Use short labeled lines, not a long paragraph:

```
Primary request: <what the icon depicts — keep it to one simple concept>
Style/medium: flat vector icon mark; clean geometric shapes; minimal
Composition/framing: single centered symbol with generous padding; all critical elements within center 80% safe zone
Color palette: <symbol color hex> on <background color hex>; high contrast
Subject: <concrete visual description — e.g. "interlocking angular shapes" not "speed and modularity">
```

### Default constraints (for `--constraints`)

```
single centered app icon mark, no text, no lettering, no watermark, no gradients, opaque solid background, high contrast, clean shape language, strong silhouette readable at 16px, all critical elements within center 80% safe zone for maskable icon cropping, no fine lines or thin strokes that disappear at small sizes
```

### What to avoid in prompts

- Vibe-only buzzwords: "epic", "cinematic", "trending", "8K", "award-winning"
- Abstract concepts the model can't visualize: "speed", "progressive", "modularity"
- Multiple subjects or complex scenes
- Gradients (break at small sizes and complicate maskable icon backgrounds)
- Any text or lettering (will render poorly and become illegible at favicon sizes)

## Workflow A: generate new icon from prompt

1. Generate a 1024x1024 master icon in **one command** (env + call together):

```bash
mkdir -p tmp/favicon-generator
python3 "$ICON_CLI" generate --prompt "<USER_PROMPT>" --size 1024x1024 --quality high --output-format png --out tmp/favicon-generator/app-icon-master.png --use-case logo-brand --constraints "single centered app icon mark, no text, no lettering, no watermark, no gradients, opaque solid background, high contrast, clean shape language, strong silhouette readable at 16px, all critical elements within center 80% safe zone for maskable icon cropping, no fine lines or thin strokes that disappear at small sizes" --force
```

2. Export required PWA sizes:

```bash
mkdir -p public tmp/favicon-generator
cp tmp/favicon-generator/app-icon-master.png public/icon-512x512.png
sips -s format png public/icon-512x512.png --resampleHeightWidth 192 192 --out public/icon-192x192.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 180 180 --out public/apple-touch-icon.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 72 72 --out public/badge-72x72.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 512 512 --out public/icon-maskable-512x512.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 192 192 --out public/icon-maskable-192x192.png >/dev/null
python3 - <<'PY'
from PIL import Image
img = Image.open("public/icon-512x512.png").convert("RGBA")
img.save("app/favicon.ico", format="ICO", sizes=[(16, 16), (32, 32), (48, 48), (64, 64)])
print("Wrote app/favicon.ico")
PY
```

3. Verify outputs:

```bash
ls -la public/icon-512x512.png public/icon-192x192.png public/icon-maskable-512x512.png public/icon-maskable-192x192.png public/apple-touch-icon.png public/badge-72x72.png app/favicon.ico
```

## Workflow B: start from existing source image

If the user already has a source icon, skip generation and export from it:

```bash
SOURCE="path/to/source-icon.png"
mkdir -p public
cp "$SOURCE" public/icon-512x512.png
sips -s format png public/icon-512x512.png --resampleHeightWidth 192 192 --out public/icon-192x192.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 180 180 --out public/apple-touch-icon.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 72 72 --out public/badge-72x72.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 512 512 --out public/icon-maskable-512x512.png >/dev/null
sips -s format png public/icon-512x512.png --resampleHeightWidth 192 192 --out public/icon-maskable-192x192.png >/dev/null
python3 - <<'PY'
from PIL import Image
img = Image.open("public/icon-512x512.png").convert("RGBA")
img.save("app/favicon.ico", format="ICO", sizes=[(16, 16), (32, 32), (48, 48), (64, 64)])
print("Wrote app/favicon.ico")
PY
```

## Troubleshooting

- Missing API key:
  - Ensure `.env.local` (or `.env`) in repo root contains `OPENAI_API_KEY=...`.
- Missing Python deps:
  - `python3 -m pip install --user openai pillow`
  - If system Python is externally managed (PEP-668), use a local venv.
- `sips` ICO conversion failure (`Error 13`):
  - Use Pillow for `app/favicon.ico` generation (already shown in workflows).
- Network/DNS failure during API call:
  - Prefer DNS precheck before API call to avoid wasting attempts.
  - Report failure clearly and stop after that attempt.
  - Provide the exact same generation command for user to run in a network-enabled shell.
  - Offer Workflow B (export from existing source image) to unblock immediately.

## Execution notes

- Keep temp files under `tmp/favicon-generator/`.
- Keep final artifacts in standard project icon locations listed above.
- For status/notes updates, prefer direct file edits over fragile quoted `printf` shell snippets.
