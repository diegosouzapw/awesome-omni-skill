---
name: Image Generation
description: Generate images using OpenAI's GPT Image model. Use for creating images from text prompts. Keywords: image, generate, openai, gpt-image, dalle, picture, art, create.
---

# Image Generation

AI image generation powered by OpenAI's GPT Image model (chatgpt-image-latest).

## Variables

- **IMAGE_GEN_CLI_PATH**: `.claude/skills/image-gen/image_gen_cli/`

## Instructions

Run from IMAGE_GEN_CLI_PATH:
```bash
cd .claude/skills/image-gen/image_gen_cli/
uv run img --help                  # Discover all commands
uv run img generate --help         # Image generation options
uv run img emojify --help          # Discord emoji conversion
```

## Naming Convention

Images and emojis follow a strict naming convention:

| Type | Format | Example |
|------|--------|---------|
| Source images | `word_word.png` (underscores) | `town_guard.png` |
| Discord emojis | `word-word.png` (hyphens) | `town-guard.png` |

**Rules:**
- Names must have **at least 2 words**
- Source images use **underscores** (`_`)
- Emoji versions use **hyphens** (`-`)
- The `emojify` command auto-converts naming

## Rules

- **Requires OPENAI_API_KEY** - must be set in environment or .env
- **Cost aware** - high quality + large size = more tokens = higher cost
- **Discord emoji limits** - 128x128px recommended, 256KB max file size

## Troubleshooting

- **"OPENAI_API_KEY not found"**: Add `OPENAI_API_KEY=sk-...` to .env
- **"Rate limit exceeded"**: Wait and retry, or upgrade API tier
- **Emoji too large**: Use `--max-size` to target smaller file size
