---
name: imaginepro-api
description: Generate AI images via ImaginePro API (Midjourney, Flux, Nano Banana, Lumi Girl, video)
version: 1.0.0
metadata:
  openclaw:
    requires:
      env:
        - IMAGINEPRO_API_KEY
      bins:
        - python3
        - curl
    primaryEnv: IMAGINEPRO_API_KEY
    homepage: https://platform.imaginepro.ai/ai-agent-skill
    emoji: "\U0001F3A8"
---

# ImaginePro AI Image Generation API

Generate stunning AI images, videos, and edits using the ImaginePro API. This skill wraps the full ImaginePro backend and supports 5 generation models, image upscaling, background removal, prompt enhancement, and video generation.

## Quick Start

```bash
# Set your API key (get one at https://platform.imaginepro.ai/dashboard/setup)
export IMAGINEPRO_API_KEY="your-api-key-here"

# Generate an image with Flux (fastest)
python3 imaginepro_api.py wait --prompt "a cyberpunk cityscape at sunset" --model flux

# Generate with Midjourney
python3 imaginepro_api.py wait --prompt "portrait of a warrior queen, cinematic lighting --ar 2:3"

# List available models and costs
python3 imaginepro_api.py models
```

## Authentication

All requests require a Bearer token obtained from the ImaginePro Platform dashboard:

1. Sign up at https://platform.imaginepro.ai
2. Purchase credits from https://platform.imaginepro.ai/pricing
3. Get your API key at https://platform.imaginepro.ai/dashboard/setup
4. Set the environment variable: `export IMAGINEPRO_API_KEY="your-key"`

**Base URL:** `https://api.imaginepro.ai/api/v1`

**Header:** `Authorization: Bearer <IMAGINEPRO_API_KEY>`

## Available Models

| Model | Endpoint | Credits | Best For |
|-------|----------|---------|----------|
| Midjourney (alpha v6) | `/midjourney/imagine` | 10 (fast) / 5 (relax) | Artistic, photorealistic images |
| Flux | `/flux/imagine` | 6 | Fast general-purpose generation |
| Nano Banana | `/universal/imagine` | 6 | Reference image + text (try-on, mockup, staging) |
| Lumi Girl | `/universal/zimage` | 6 | Character portraits, anime, stylized |
| MJ Video | `/video/mj/generate` | 10 | Video generation from start/end frames |

## API Reference

### Image Generation

#### POST `/midjourney/imagine` — Midjourney Generation

The flagship model. Supports Midjourney parameters in the prompt string (e.g., `--ar 16:9`, `--style raw`, `--relax`).

**Request:**
```json
{
  "prompt": "a majestic eagle soaring over mountains --ar 16:9",
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- Append `--relax` to the prompt for relax mode (5 credits instead of 10, slower).
- Supports all standard Midjourney prompt parameters: `--ar`, `--style`, `--chaos`, `--no`, `--seed`, `--q`, etc.

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-generation-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 10 (fast mode) / 5 (relax mode)

---

#### POST `/flux/imagine` — Flux Generation

Fast, high-quality generation. Supports batch generation.

**Request:**
```json
{
  "prompt": "a cozy cabin in the woods, watercolor style",
  "n": 1,
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- `n` (optional, default 1): Number of images to generate (credits multiply by n).

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-generation-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 6 per image

---

#### POST `/universal/imagine` — Nano Banana (Reference Image Generation)

Multi-modal generation using text + reference images. Ideal for virtual try-on, product mockups, interior staging, and style transfer.

**Request:**
```json
{
  "contents": [
    { "type": "text", "text": "Image creation: woman wearing this dress in a garden" },
    { "type": "image", "url": "https://example.com/dress.jpg" }
  ],
  "model": "nano-banana-2",
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- `contents`: Array of content items. First item should be `type: "text"` with the prompt prefixed by `"Image creation: "`. Subsequent items can be `type: "image"` with a `url` field for reference images.
- `model`: Must be `"nano-banana-2"`.
- Supports multiple reference images (e.g., a person photo + a garment photo for virtual try-on).

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-generation-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 6

---

#### POST `/universal/zimage` — Lumi Girl

Specialized model for character portraits and stylized images. Supports aspect ratio via `--ar` in the prompt.

**Request:**
```json
{
  "prompt": "anime girl with silver hair in a moonlit forest --ar 3:4",
  "steps": 4,
  "width": 768,
  "height": 1024,
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- `steps`: Always 4 (fixed).
- `width` / `height`: Max 1024 per dimension, must be divisible by 8. If `--ar W:H` is in the prompt, dimensions are auto-calculated from the ratio (max dimension = 1024).
- Default: 1024x1024 if no aspect ratio specified.

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-generation-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 6

---

#### POST `/video/mj/generate` — MJ Video

Generate a video from start and end frame images.

**Request:**
```json
{
  "prompt": "smooth camera pan with cinematic motion",
  "startFrameUrl": "https://example.com/start.jpg",
  "endFrameUrl": "https://example.com/end.jpg",
  "timeout": 900,
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- `startFrameUrl` (required): URL of the starting frame image.
- `endFrameUrl` (required): URL of the ending frame image.
- `prompt` (optional, default "smooth motion transition"): Motion description.
- `timeout` (optional, default 900): Max processing time in seconds.

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-generation-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 10

---

### Post-Processing

#### POST `/midjourney/button` — Midjourney Upscale / Variant

Upscale or create variants of Midjourney-generated images.

**Request:**
```json
{
  "messageId": "original-task-message-id",
  "button": "U1",
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- `messageId`: The `messageId` (task ID) from the original Midjourney generation.
- `button`: Action to perform. `U1`-`U4` for upscaling quadrants, `V1`-`V4` for variants.

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-upscale-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 5

---

#### POST `/flux/upscale` — Flux Upscale

Upscale any image (not limited to Flux-generated).

**Request:**
```json
{
  "image": "https://example.com/image.jpg",
  "scale": 2,
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

- `image` (required): URL of the image to upscale.
- `scale` (required): Upscale factor, must be between 2 and 4 (inclusive).

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-upscale-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 2

---

#### POST `/tools/remove-bg` — Background Removal

Remove the background from an image.

**Request:**
```json
{
  "image": "https://example.com/photo.jpg",
  "ref": "optional-tracking-id",
  "webhookOverride": "https://your-server.com/webhook"
}
```

**Response:**
```json
{
  "success": true,
  "messageId": "uuid-of-the-removebg-task",
  "createdAt": "2026-01-15T12:00:00+00:00"
}
```

**Credits:** 5

---

#### POST `/tools/prompt-extend` — Prompt Enhancement (Free)

Expand a short prompt into a detailed, high-quality prompt.

**Request:**
```json
{
  "prompt": "a sunset"
}
```

**Response:**
```json
{
  "prompt": "a breathtaking sunset over the Pacific Ocean, golden hour light casting warm amber and coral tones across scattered cumulus clouds, silhouetted palm trees in the foreground..."
}
```

**Credits:** Free

---

### Status & History

#### GET `/midjourney/message/{messageId}` — Check Generation Status

Poll this endpoint to check the progress of any generation task (works for all models, not just Midjourney).

**Response (in progress):**
```json
{
  "prompt": "a simple red circle on white background",
  "status": "PROCESSING",
  "progress": 0,
  "messageId": "2346e0bc-c3c3-48ea-adec-3a21609fd288",
  "createdAt": "2026-02-22T07:43:37+00:00",
  "updatedAt": "2026-02-22T07:43:37+00:00"
}
```

**Response (completed):**
```json
{
  "prompt": "a simple red circle on white background",
  "status": "DONE",
  "images": ["https://cdn-new.imaginepro.ai/storage/v1/object/public/cdn/2346e0bc-c3c3-48ea-adec-3a21609fd288.png"],
  "uri": "https://cdn-new.imaginepro.ai/storage/v1/object/public/cdn/2346e0bc-c3c3-48ea-adec-3a21609fd288.png",
  "progress": 100,
  "messageId": "2346e0bc-c3c3-48ea-adec-3a21609fd288",
  "createdAt": "2026-02-22T07:43:37+00:00",
  "updatedAt": "2026-02-22T07:43:49+00:00"
}
```

**Response (failed):**
```json
{
  "status": "FAIL",
  "error": "Description of what went wrong"
}
```

**Statuses:** `SUBMITTED` → `PROCESSING` → `DONE` | `FAIL`

**Credits:** Free (polling is free)

---

## Async Workflow

All generation endpoints are **asynchronous**. The workflow is:

1. **Submit** a generation request → receive a `messageId`
2. **Poll** `GET /midjourney/message/{messageId}` every 3-5 seconds
3. Wait for `status` to be `DONE` (images ready) or `FAIL` (error)
4. **Download** the result from the `uri` or `images` array

The helper script's `wait` command automates this entire flow.

## Credit Cost Summary

| Operation | Credits |
|-----------|---------|
| Midjourney Imagine (fast) | 10 |
| Midjourney Imagine (relax) | 5 |
| Midjourney Upscale/Variant | 5 |
| Flux Imagine | 6 per image |
| Flux Upscale | 2 |
| Nano Banana Imagine | 6 |
| Lumi Girl Imagine | 6 |
| MJ Video | 10 |
| Background Removal | 5 |
| Prompt Enhancement | Free |
| Status Polling | Free |

## Python Helper Script

The included `imaginepro_api.py` is a zero-dependency Python script (stdlib only) that wraps all API calls.

### Commands

```bash
# Generate an image (async — returns messageId immediately)
python3 imaginepro_api.py imagine --prompt "a sunset over mountains" --model flux

# Generate and wait for result (blocking — polls until done)
python3 imaginepro_api.py wait --prompt "a sunset over mountains" --model flux

# Check status of a generation
python3 imaginepro_api.py status --id <messageId>

# Upscale (Midjourney)
python3 imaginepro_api.py upscale --id <messageId> --button U1

# Upscale (Flux)
python3 imaginepro_api.py upscale --image "https://example.com/img.jpg" --scale 2

# Remove background
python3 imaginepro_api.py removebg --image "https://example.com/photo.jpg"

# Enhance a prompt
python3 imaginepro_api.py enhance --prompt "a cat"

# List available models
python3 imaginepro_api.py models
```

### Flags

- `--json` — Output raw JSON (default is human-readable)
- `--timeout <seconds>` — Max wait time for `wait` command (default: 300)
- `--interval <seconds>` — Polling interval for `wait` command (default: 5)

## curl Examples

```bash
# Generate with Flux
curl -X POST https://api.imaginepro.ai/api/v1/flux/imagine \
  -H "Authorization: Bearer $IMAGINEPRO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "a cyberpunk cityscape at sunset"}'

# Check status
curl https://api.imaginepro.ai/api/v1/midjourney/message/<messageId> \
  -H "Authorization: Bearer $IMAGINEPRO_API_KEY"

# Enhance a prompt (free)
curl -X POST https://api.imaginepro.ai/api/v1/tools/prompt-extend \
  -H "Authorization: Bearer $IMAGINEPRO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "a cat"}'
```

## Error Handling

API errors return JSON with `message`, `error`, and `statusCode`:

```json
{
  "message": "Prompt is required.",
  "error": "Bad Request",
  "statusCode": 400
}
```

Common error codes:
- `400` — Missing required parameter or invalid value
- `401` — Invalid or missing API key / insufficient credits
- `404` — Message not found
- `500` — Internal server error (retry after a short delay)

## Tips for AI Agents

1. **Always use the `wait` command** for simple generation tasks — it handles the submit + poll loop automatically.
2. **Check credits on the dashboard** at https://platform.imaginepro.ai/dashboard before large batches to avoid failures mid-way through. There is no API endpoint for checking credits.
3. **Use `enhance` on short prompts** before generating — it's free and dramatically improves quality.
4. **Midjourney supports prompt parameters** like `--ar 16:9 --style raw --chaos 20` directly in the prompt string.
5. **For Nano Banana (virtual try-on, mockups)**, always provide reference image URLs in the `contents` array.
6. **Poll interval:** 5 seconds is recommended. Don't poll faster than 3 seconds.
7. **Timeouts:** Midjourney can take 30-120s. Video generation can take up to 15 minutes. Set timeouts accordingly.
