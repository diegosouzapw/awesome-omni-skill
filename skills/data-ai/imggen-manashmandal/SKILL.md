---
name: imggen
description: Use this skill when users want to generate images using OpenAI's image generation API (DALL-E or gpt-image-1), or extract text from images using OCR. Invoke when users request AI-generated images, artwork, logos, illustrations, visual content from text prompts, or need to extract text/data from images.
version: 1.1.0
allowed-tools: Bash(imggen:*), Read, Write
model: inherit
---

# imggen - OpenAI Image Generation and OCR CLI

Generate images from text prompts and extract text from images using OpenAI's APIs.

## Overview

`imggen` is a command-line tool that interfaces with OpenAI's image generation API. It supports multiple models (gpt-image-1, dall-e-3, dall-e-2) and provides options for image size, quality, format, and style.

## Prerequisites

- `imggen` binary installed and available in PATH
- `OPENAI_API_KEY` environment variable set with a valid OpenAI API key
- Sufficient OpenAI API credits for image generation

## Usage

```bash
imggen [flags] "prompt"
```

## Available Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--model` | `-m` | `gpt-image-1` | Model: gpt-image-1, dall-e-3, dall-e-2 |
| `--size` | `-s` | `1024x1024` | Image dimensions |
| `--quality` | `-q` | `auto` | Quality level |
| `--count` | `-n` | `1` | Number of images (1-10 for gpt-image-1, 1 for dall-e-3) |
| `--output` | `-o` | auto-generated | Output filename or directory |
| `--format` | `-f` | `png` | Output format: png, jpeg, webp |
| `--style` | | `vivid` | Style for dall-e-3: vivid, natural |
| `--transparent` | `-t` | `false` | Transparent background (gpt-image-1 + png/webp only) |
| `--prompt` | `-P` | | Prompt (can be specified multiple times) |
| `--parallel` | `-p` | `1` | Number of parallel workers for multiple prompts |
| `--api-key` | | `$OPENAI_API_KEY` | Override API key |

## Model-Specific Parameters

### gpt-image-1 (Default, Recommended)
- **Sizes**: 1024x1024, 1536x1024 (landscape), 1024x1536 (portrait), auto
- **Quality**: auto, low, medium, high
- **Max images**: 10 per request
- **Supports**: Transparent backgrounds, multiple output formats

### dall-e-3
- **Sizes**: 1024x1024, 1024x1792, 1792x1024
- **Quality**: standard, hd
- **Max images**: 1 per request
- **Supports**: Style parameter (vivid/natural)

### dall-e-2
- **Sizes**: 256x256, 512x512, 1024x1024
- **Max images**: 10 per request

## Instructions

1. Verify `OPENAI_API_KEY` is set in the environment
2. Construct the imggen command with appropriate flags based on user requirements
3. Execute the command using Bash tool
4. Report the generated filename and any revised prompt returned by the API
5. If the user wants to view the image, use Read tool on the generated file

## Output Format

The tool outputs:
- Progress message: "Generating N image(s) with MODEL..."
- Saved filename: "Saved: filename.png"
- Cost information: "Cost: $X.XXXX (N image(s) @ $X.XXXX each)"
- Revised prompt (if returned by API): "Revised prompt: ..."
- Completion message: "Done!"

Generated files are saved to the current working directory with timestamp-based names (e.g., `image-20251216-120000.png`) unless `--output` is specified.

## Cost Tracking

All image generation costs are automatically logged to `~/.imggen/sessions.db`. View costs using the `cost` subcommand:

```bash
# View total costs
imggen cost

# View today's costs
imggen cost today

# View this week's costs (last 7 days)
imggen cost week

# View this month's costs (last 30 days)
imggen cost month

# View costs by provider
imggen cost provider
```

### Interactive Mode Cost Commands

In interactive mode (`imggen -i`), use the `cost` or `$` command:
- `cost today` - Today's costs
- `cost week` - This week's costs
- `cost month` - This month's costs
- `cost total` - All-time total
- `cost provider` - Breakdown by provider
- `cost session` - Current session's costs

## Database Management

Manage the SQLite database storing sessions and cost data:

```bash
# Reset database (delete all data)
imggen db reset

# Reset with backup of old data
imggen db reset --backup

# Show database location and stats
imggen db info
```

## Examples

### Basic image generation
```bash
imggen "a sunset over mountains"
```

### High-quality landscape with DALL-E 3
```bash
imggen -m dall-e-3 -s 1792x1024 -q hd "panoramic view of a futuristic city"
```

### Multiple images with gpt-image-1
```bash
imggen -n 4 -q high "abstract geometric pattern"
```

### Logo with transparent background
```bash
imggen -t -f png "minimalist tech company logo, flat design"
```

### Custom output filename
```bash
imggen -o hero-image.png "website hero banner with gradient"
```

### Natural style portrait
```bash
imggen -m dall-e-3 --style natural "professional headshot, studio lighting"
```

### Multiple prompts via command line
```bash
# Generate multiple images with --prompt flag
imggen --prompt "a sunset" --prompt "a cat" --prompt "a dog" -o ./output

# Short form with parallel processing (3 workers)
imggen -P "sunset" -P "mountains" -P "ocean" -o ./images -p 3
```

### Batch generation from file
```bash
# From a text file (one prompt per line)
imggen batch prompts.txt -o ./output

# From a JSON file with per-prompt options
imggen batch prompts.json -o ./output

# With parallel processing
imggen batch prompts.txt -o ./output -p 3
```

## Multiple Prompts

Generate multiple images from command-line prompts using the `--prompt/-P` flag:

```bash
imggen --prompt "a sunset over mountains" --prompt "a cat playing piano" -o ./output
```

This processes all prompts and saves images to the output directory with indexed filenames:
- `001-a-sunset-over-mountains.png`
- `002-a-cat-playing-piano.png`

Use `--parallel/-p` to control concurrent processing (default: 1 = sequential).

## Batch Generation

Generate multiple images from a file of prompts using the `batch` subcommand:

```bash
imggen batch <input-file> [flags]
```

### Input File Formats

**Text file (.txt)** - One prompt per line (lines starting with `#` are ignored):
```
a sunset over mountains
a cat playing piano
abstract geometric art
```

**JSON file (.json)** - Array of objects with optional per-prompt settings:
```json
[
  {"prompt": "a sunset over mountains"},
  {"prompt": "a cat playing piano", "model": "dall-e-3", "quality": "hd"},
  {"prompt": "abstract art", "size": "1792x1024"}
]
```

### Batch Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--output` | `-o` | current dir | Output directory |
| `--model` | `-m` | `gpt-image-1` | Default model |
| `--size` | `-s` | model default | Default image size |
| `--quality` | `-q` | model default | Default quality level |
| `--format` | `-f` | `png` | Output format |
| `--parallel` | `-p` | `1` | Number of parallel workers |
| `--stop-on-error` | | `false` | Stop on first error |
| `--delay` | | `0` | Delay between requests (ms) |

## Error Handling

Common errors and solutions:
- **"API key required"**: Set `OPENAI_API_KEY` environment variable
- **"invalid size"**: Use a size supported by the selected model
- **"supports maximum N images"**: Reduce `--count` value
- **"does not support --style"**: Only dall-e-3 supports style flag
- **"does not support --transparent"**: Only gpt-image-1 supports transparency

## Pricing Reference

Costs per image (USD):

### gpt-image-1
| Size | Low | Medium | High |
|------|-----|--------|------|
| 1024x1024 | $0.011 | $0.042 | $0.167 |
| 1536x1024 | $0.016 | $0.063 | $0.250 |
| 1024x1536 | $0.016 | $0.063 | $0.250 |

### dall-e-3
| Size | Standard | HD |
|------|----------|-----|
| 1024x1024 | $0.040 | $0.080 |
| 1024x1792 | $0.080 | $0.120 |
| 1792x1024 | $0.080 | $0.120 |

### dall-e-2
| Size | Cost |
|------|------|
| 256x256 | $0.016 |
| 512x512 | $0.018 |
| 1024x1024 | $0.020 |

## OCR (Optical Character Recognition)

Extract text from images using OpenAI's vision API with optional structured output support.

### OCR Usage

```bash
imggen ocr <image-path> [flags]
```

### OCR Flags

| Flag | Short | Default | Description |
|------|-------|---------|-------------|
| `--model` | `-m` | `gpt-5-mini` | Model: gpt-5.2, gpt-5-mini, gpt-5-nano |
| `--schema` | `-s` | | JSON schema file for structured output |
| `--schema-name` | | `extracted_data` | Name for the JSON schema |
| `--suggest-schema` | | `false` | Suggest a JSON schema based on image content |
| `--prompt` | `-p` | auto | Custom extraction prompt |
| `--output` | `-o` | stdout | Output file |
| `--url` | | | Image URL instead of file path |
| `--api-key` | | `$OPENAI_API_KEY` | Override API key |
| `--verbose` | `-v` | `false` | Log HTTP requests and responses |

### OCR Models

| Model | Cost (Input) | Cost (Output) | Best For |
|-------|-------------|---------------|----------|
| gpt-5-nano | $0.05/1M tokens | $0.40/1M tokens | Ultra budget, simple text |
| gpt-5-mini | $0.25/1M tokens | $2.00/1M tokens | Cost-effective, most OCR tasks |
| gpt-5.2 | $1.75/1M tokens | $14.00/1M tokens | Complex documents, highest accuracy |

### OCR Examples

#### Basic text extraction
```bash
imggen ocr document.png
```

#### Extract from URL
```bash
imggen ocr --url https://example.com/image.png
```

#### Save output to file
```bash
imggen ocr receipt.jpg -o extracted.txt
```

#### Structured output with JSON schema
```bash
# Create a schema file (invoice_schema.json):
# {
#   "type": "object",
#   "properties": {
#     "vendor": {"type": "string"},
#     "date": {"type": "string"},
#     "total": {"type": "number"},
#     "items": {
#       "type": "array",
#       "items": {
#         "type": "object",
#         "properties": {
#           "name": {"type": "string"},
#           "price": {"type": "number"}
#         },
#         "required": ["name", "price"],
#         "additionalProperties": false
#       }
#     }
#   },
#   "required": ["vendor", "date", "total"],
#   "additionalProperties": false
# }

imggen ocr receipt.jpg --schema invoice_schema.json -o invoice.json
```

#### Auto-suggest a JSON schema
```bash
# Analyze image and suggest appropriate schema
imggen ocr document.png --suggest-schema

# Save suggested schema to file
imggen ocr document.png --suggest-schema -o suggested_schema.json
```

#### Use higher accuracy model
```bash
imggen ocr complex-document.pdf -m gpt-5.2
```

#### Custom extraction prompt
```bash
imggen ocr business-card.jpg -p "Extract the name, title, email, and phone number"
```

### OCR Structured Output

When using the `--schema` flag, the output will be structured JSON matching your schema. This is useful for:
- Extracting data from receipts, invoices, forms
- Parsing business cards, ID documents
- Converting tables and structured content to JSON
- Data entry automation

The schema must follow JSON Schema draft-07 format with `additionalProperties: false` for strict validation.

### OCR Tips

1. **Use gpt-5-nano** for simple text extraction (plain documents, basic receipts)
2. **Use gpt-5-mini** (default) for most OCR tasks (receipts, business cards, forms)
3. **Use gpt-5.2** for complex documents (dense tables, handwriting, multi-language)
4. **Suggest schema first** if unsure about document structure
5. **Custom prompts** help when you need specific fields or formatting
6. **Supported formats**: PNG, JPEG, GIF, WEBP, PDF (first page)
