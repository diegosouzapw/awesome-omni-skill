---
name: modelscope
description: Use this skill to generate AI images using ModelScope's Tongyi-MAI/Z-Image-Turbo model. Simply describe the image you want and it will be generated. Supports Chinese and English prompts.
---

# ModelScope Image Generation Skill

Generate AI images using ModelScope's powerful text-to-image model.

## When to Use This Skill

Trigger when user:
- Asks to generate an image
- Wants to create a picture from description
- Says "生成图片" or "画一个..."
- Mentions ModelScope image generation
- Uses phrases like "create an image", "generate a picture"

## Quick Start

```bash
# Generate an image with a text prompt
python scripts/run.py generate.py --prompt "A golden cat sitting on a windowsill"

# Generate with Chinese prompt
python scripts/run.py generate.py --prompt "一只金色的猫坐在窗台上，阳光照进来"

# Generate and save to specific path
python scripts/run.py generate.py --prompt "A beautiful sunset over mountains" --save-path "sunset.png"

# Generate with custom model
python scripts/run.py generate.py --prompt "Cyberpunk city at night" --model "Tongyi-MAI/Z-Image"
```

## Configuration (One-Time Setup)

### Step 1: Set your API Key

Create a `.env` file in the skill directory:

```bash
MODELSCOPE_API_KEY=ms-your-api-key-here
```

Or set as environment variable:

```bash
# Windows PowerShell
$env:MODELSCOPE_API_KEY="ms-your-api-key-here"

# Windows CMD
set MODELSCOPE_API_KEY=ms-your-api-key-here

# Linux/Mac
export MODELSCOPE_API_KEY="ms-your-api-key-here"
```

### Step 2: Install dependencies (automatic on first run)

```bash
python scripts/run.py generate.py --help
```

This will automatically:
- Create `.venv` virtual environment
- Install required dependencies (requests, Pillow)

## Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `--prompt` | string | Yes | - | Text description of the image to generate |
| `--model` | string | No | Tongyi-MAI/Z-Image-Turbo | Model ID to use |
| `--save-path` | string | No | result_image.jpg | File path to save the image |
| `--timeout` | integer | No | 300 | Maximum wait time in seconds |
| `--async-mode` | boolean | No | true | Use async mode for generation |

## Examples

### Basic Usage

```bash
# Simple image generation
python scripts/run.py generate.py --prompt "A golden cat"
```

### Advanced Usage

```bash
# With custom save location
python scripts/run.py generate.py --prompt "A majestic dragon" --save-path "dragon.png"

# With longer timeout for complex images
python scripts/run.py generate.py --prompt "A detailed fantasy landscape with castles and dragons" --timeout 600

# Using the standard quality model
python scripts/run.py generate.py --prompt "A photorealistic portrait" --model "Tongyi-MAI/Z-Image"
```

## How It Works

1. **Submit Request**: Sends prompt to ModelScope API with async mode enabled
2. **Get Task ID**: Receives a task ID for tracking
3. **Poll for Result**: Checks task status every 5 seconds
4. **Download Image**: When complete, downloads the generated image
5. **Save to File**: Saves to specified path (default: result_image.jpg)

## Available Models

| Model ID | Description | Speed | Quality |
|----------|-------------|-------|---------|
| Tongyi-MAI/Z-Image-Turbo | Fast generation | Fast | Good |
| Tongyi-MAI/Z-Image | Standard generation | Medium | High |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `MODELSCOPE_API_KEY not set` | Set your API key in `.env` file or environment variable |
| `Image generation failed` | Check your API key is valid and prompt follows content policies |
| `Connection timeout` | Increase `--timeout` parameter (default: 300 seconds) |
| `ModuleNotFoundError` | Dependencies will auto-install on first run, or manually run `pip install -r requirements.txt` |

## Tips for Better Results

1. **Be Descriptive**: More detailed prompts produce better images
   ```
   "A golden cat" → "A fluffy golden cat with emerald eyes sitting on a velvet cushion"
   ```

2. **Specify Style**: Include art style for better control
   ```
   "A mountain landscape in the style of Studio Ghibli"
   ```

3. **Use Chinese or English**: The model supports both languages well
   ```
   "一只在月光下的猫，水彩画风格"
   ```

4. **Add Details**: Include lighting, mood, and composition
   ```
   "A serene sunset over calm ocean waters, warm orange and pink sky, peaceful atmosphere"
   ```

## Data Storage

- Generated images are saved to the current working directory or specified path
- No user data is stored externally
- API key is never shared or logged

## Limitations

- Requires valid ModelScope API key
- Generation time: 30-90 seconds typically
- Max timeout: 600 seconds (10 minutes)
- Image format: JPG by default (use .png extension for PNG)

## Skill Directory Structure

```
~/.claude/skills/modelscope-image-gen/
├── SKILL.md           # This file - skill documentation
├── scripts/
│   ├── run.py         # Wrapper script for environment management
│   └── generate.py    # Main image generation script
├── data/              # Local storage (auto-created)
├── .env               # API key configuration (create this)
├── requirements.txt   # Python dependencies
└── .venv/             # Virtual environment (auto-created)
```

## Resources

- [ModelScope Documentation](https://modelscope.cn/)
- [Tongyi-MAI Model Page](https://modelscope.cn/models/Tongyi-MAI/Z-Image-Turbo)
