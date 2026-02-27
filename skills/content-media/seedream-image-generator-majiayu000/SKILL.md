---
name: seedream-image-generator
description: Generate images using the Doubao SeeDream API based on text prompts. Use this skill when users request AI-generated images, artwork, illustrations, or visual content creation. The skill handles API calls, downloads generated images to the project's /pic folder, and supports batch generation of up to 4 sequential images.
---

# SeeDream Image Generator

## Overview

This skill enables image generation using the Doubao SeeDream 4.0 API (model: doubao-seedream-4-0-250828). It converts text prompts into high-quality images and automatically downloads them to the project's `/pic` folder. The skill supports single and batch image generation (up to 4 sequential images), customizable image sizes, and watermark control.

## When to Use This Skill

Use this skill when users request:
- "Generate an image of [description]"
- "Create artwork showing [scene/concept]"
- "Make an illustration of [subject]"
- "Generate 4 seasonal variations of [scene]"
- Any request involving AI image generation, visual content creation, or artwork generation

## Quick Start

### Prerequisites

Before generating images, obtain the user's Volcano Engine (火山引擎) ARK API key:

**IMPORTANT**: Always ask the user for their ARK API key before proceeding with image generation, as the skill does not include a pre-configured key.

Example prompt to user:
> "To generate images, I need your Volcano Engine ARK API key. You can find it at: https://console.volcengine.com/ark/region:ark+cn-beijing/apikey
>
> Please provide your ARK_API_KEY."

### Basic Workflow

1. **Receive user request** for image generation
2. **Request API key** from the user if not already provided
3. **Clarify requirements**:
   - Prompt/description
   - Number of images (1-4)
   - Image size preference
   - Watermark preference
4. **Execute generation** using the `generate_image.py` script
5. **Report results** with file paths and preview the images if possible

## Image Generation Tasks

### Single Image Generation

Generate a single image based on a text prompt.

**Example user request:**
> "Generate an image of a futuristic city at sunset with flying cars"

**Script usage:**
```bash
python scripts/generate_image.py "futuristic city at sunset with flying cars" \
    --api-key "YOUR_ARK_API_KEY" \
    --size "2K"
```

**Parameters:**
- `prompt` (required): Text description of the desired image
- `--api-key`: ARK API key (can also use ARK_API_KEY environment variable)
- `--size`: Image size, options include:
  - `"2K"` (default)
  - `"1024x1024"`
  - `"2048x2048"`
  - `"3104x1312"` (widescreen)
  - Custom dimensions in format `"{width}x{height}"`
- `--no-watermark`: Disable watermark (watermark enabled by default)
- `--output-dir`: Custom output directory (defaults to `project_root/pic`)

### Batch Image Generation

Generate multiple sequential/related images (2-4 images) with a single prompt.

**Example user request:**
> "Generate 4 images showing the same garden courtyard through all four seasons"

**Script usage:**
```bash
python scripts/generate_image.py \
    "同一庭院一角的四季变迁，统一风格展现四季独特色彩、元素与氛围" \
    --api-key "YOUR_ARK_API_KEY" \
    --size "2048x2048" \
    --max-images 4
```

**Additional parameter:**
- `--max-images`: Number of images to generate (1-4, default: 1)

**Note**: When `--max-images` is greater than 1, the API automatically uses sequential image generation to create related/coherent images.

### Advanced Prompt Engineering

For high-quality results, guide users to provide detailed prompts that include:

1. **Subject**: Main focus of the image
2. **Style**: Art style, rendering technique (e.g., "photorealistic", "oil painting", "anime style")
3. **Lighting**: Lighting conditions and atmosphere
4. **Color**: Color palette or dominant colors
5. **Composition**: Perspective, framing, depth of field
6. **Details**: Specific elements to include

**Example of a well-crafted prompt:**
```
"Interstellar scene with a massive black hole, a vintage train emerging from it half-destroyed,
strong visual impact, cinematic, apocalyptic atmosphere, dynamic motion, contrasting colors,
octane render, ray tracing, motion blur, depth of field, surrealism, deep blue tones,
detailed color layers shaping the subject, realistic textures, dark background with dramatic
lighting creating atmosphere, artistic fantasy feel, exaggerated wide-angle perspective,
lens flare, reflections, extreme lighting and shadows, strong gravity, consuming effect"
```

## Script Details

### Location
`scripts/generate_image.py`

### Key Features
- Automatic project root detection (looks for `.git`, `.claude`, `package.json`, etc.)
- Creates `/pic` folder if it doesn't exist
- Timestamps filenames to prevent overwrites (format: `seedream_YYYYMMDD_HHMMSS.png`)
- Downloads images directly from API response URLs
- Prints usage statistics (tokens, generated images count)
- Error handling for API calls and downloads

### Output Format
- Images are saved to: `{project_root}/pic/seedream_{timestamp}.png`
- For batch generation: `{project_root}/pic/seedream_{timestamp}_1.png`, `_2.png`, etc.
- Default format: PNG

### Requirements
The script requires the following Python packages:
- `openai` (for API client)
- `requests` (for image downloading)

Install with:
```bash
pip install openai requests
```

## Workflow Decision Tree

```
User requests image generation
    ↓
Do we have ARK API key?
    ├─ No → Request API key from user → Store for session
    └─ Yes → Continue
    ↓
Single image or multiple images?
    ├─ Single (default)
    │   └─ Run: generate_image.py "prompt" --api-key KEY --size SIZE
    └─ Multiple (2-4 images)
        └─ Run: generate_image.py "prompt" --api-key KEY --max-images N
    ↓
Script executes:
    1. Calls SeeDream API
    2. Receives image URL(s)
    3. Downloads to /pic folder
    4. Reports file paths and statistics
    ↓
Inform user of results
    ├─ Success → Show file paths, offer to view images
    └─ Failure → Report error, suggest troubleshooting
```

## Troubleshooting

### Common Issues

**"ARK API key is required"**
- Ensure the user has provided their API key
- Verify the key is correctly passed via `--api-key` or ARK_API_KEY environment variable

**"Error calling API"**
- Check API key validity
- Verify network connectivity to `ark.cn-beijing.volces.com`
- Ensure the prompt doesn't violate content policies
- Check API quota/limits

**"Error downloading image"**
- Check network connectivity
- Verify the image URL is accessible
- Ensure sufficient disk space in output directory

**Module not found errors**
- Install required dependencies: `pip install openai requests`

## Best Practices

1. **Always request API key first** - Don't assume the user has configured it
2. **Clarify image requirements** - Ask about size, quantity, and style preferences
3. **Optimize prompts** - Help users craft detailed, descriptive prompts for better results
4. **Batch generation for variations** - Suggest `--max-images` when users want variations or sequences
5. **Inform about output location** - Always tell users where images are saved
6. **Preview results** - After generation, offer to display or describe the generated images
7. **Respect content policies** - Ensure prompts comply with API content guidelines

## Example Interactions

**Example 1: Simple request**
```
User: "Generate a sunset over mountains"
Claude: "I'll generate that image for you. First, I need your Volcano Engine ARK API key..."
[User provides key]
Claude: [Executes generate_image.py]
Claude: "✅ Image generated successfully! Saved to: /project/pic/seedream_20250112_143022.png"
```

**Example 2: Batch generation**
```
User: "Create 4 images of a coffee shop in different seasons"
Claude: "I'll generate 4 seasonal variations of a coffee shop. Using your API key..."
[Executes with --max-images 4]
Claude: "✅ Generated 4 images:
- /project/pic/seedream_20250112_143530_1.png (Spring)
- /project/pic/seedream_20250112_143530_2.png (Summer)
- /project/pic/seedream_20250112_143530_3.png (Autumn)
- /project/pic/seedream_20250112_143530_4.png (Winter)"
```

**Example 3: Custom specifications**
```
User: "Generate a 2048x2048 image of a cyberpunk street without watermark"
Claude: [Executes with --size "2048x2048" --no-watermark]
Claude: "✅ Image generated (2048x2048, no watermark)
Saved to: /project/pic/seedream_20250112_144015.png"
```
