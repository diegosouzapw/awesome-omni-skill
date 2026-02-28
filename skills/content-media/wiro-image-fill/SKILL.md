---
name: wiro-image-fill
description: Generate missing or placeholder images in a project by calling the Wiro image generation API, saving assets under public/assets generated folders, and producing a JSON mapping. Use when you see empty img src, placeholder.png, or other image gaps that need real assets.
---

# Wiro Image Fill

## Overview

Generate real images via Wiro and write a mapping from placeholder keys to generated asset paths for later replacement.

## How-to (quick start)

1. Get API credentials from Wiro dashboard for the target project.
2. Provide credentials using one of the options below.
3. Run the generator script with a prompt.
4. Use the mapping output to update image references.

Option A: export environment variables in the shell session.

```bash
export WIRO_API_KEY="your_project_api_key"
export WIRO_API_SECRET="your_project_api_secret"
```

Option B: create a `.env` file in the project root (user creates this file).

```bash
WIRO_API_KEY=your_project_api_key
WIRO_API_SECRET=your_project_api_secret
```

If `.env` does not exist, the script creates it with placeholder `WIRO_API_KEY` and `WIRO_API_SECRET` lines.
The script reads `.env` by default if it exists. Use `--env-file path/to/.env` to point to a different file.

## Workflow

1. Locate missing images.
2. Draft a prompt from nearby UI context.
3. Run the generator script.
4. Use the mapping to update image references.

## 1) Locate missing images

Use ripgrep to find placeholders:

```bash
rg -n "src=\"\"|src=''|placeholder\.png" <project-root>
```

Capture the placeholder key you will map (the literal placeholder string or path).

## 2) Draft a prompt

Use nearby UI copy, alt text, component names, and page intent. Keep the prompt short and specific to the UI need.

## 3) Run the generator script

Set credentials:

```bash
export WIRO_API_KEY="..."
export WIRO_API_SECRET="..."
```

Run per placeholder:

```bash
python3 scripts/wiro_generate_image.py \
  --model "google/nano-banana-pro" \
  --prompt "Modern dashboard hero showing analytics cards on a clean desk." \
  --map-key "placeholder.png"
```

Notes:

- Default output dir: `public/generated` if `public/` exists, else `assets/generated`, else `generated/`.
- Mapping file default: `<output-dir>/image-map.json`.
- Use `--output-dir` or `--mapping-file` to override.
- `--input-image` can be repeated or comma-separated.
- `--body-mode multipart` (default) sends multipart form fields; use `--body-mode json` if needed.
- IPv4 is preferred by default to avoid IPv6 delays. Use `--ip-mode auto` to disable IPv4 forcing.
- Timeouts: `--connect-timeout` (default 5s), `--read-timeout` (default 30s), `--task-timeout` (default 180s).
- `--verbose` prints run/poll progress to stderr; helpful for diagnosing stalls.
- Network retries use `--request-retries` (default 2) and `--retry-wait` (default 1s backoff).

## 4) Apply the mapping

Use the JSON mapping file to replace placeholders in code or templates. The script only writes the mapping entry when `--map-key` is provided.

## References

Read `references/wiro_api.md` for endpoint and status details.

## Scripts

- `scripts/wiro_generate_image.py`: Run Wiro generation, poll task status, download output, and update mapping.
