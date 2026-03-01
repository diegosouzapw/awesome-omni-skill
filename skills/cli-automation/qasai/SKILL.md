---
name: qasai
description: Image compression CLI with lossless/lossy options, multiple engines, batch processing, and format conversion. Use when compressing, optimizing, or converting images.
license: Apache-2.0
metadata:
  author: ahmadawais
  version: "0.1.0"
  repository: https://github.com/ahmadawais/qasai
allowed-tools:
  - shell
---

# QASAI - Image Compression CLI

## Installation

```bash
npx qasai
# or install globally
npm install -g qasai
```

## When to Use

- Compressing images for web optimization
- Batch processing image directories
- Converting between image formats (JPG, PNG, WebP, AVIF)
- Resizing images while maintaining aspect ratio
- Lossless optimization without quality loss

## Supported Formats

JPEG, PNG, WebP, AVIF, GIF, TIFF, SVG

## Usage

### Interactive Mode

```bash
npx qasai
```

Launches guided prompts for compression options.

### Direct Compression

```bash
npx qasai [input] [options]
```

If you pass an input path or any options, it runs compression directly.

## Common Workflows

### 1. Basic Compression (Web Optimized)

```bash
npx qasai ./images -q 80 -w 1920 -r
```

### 2. Maximum Compression

```bash
npx qasai ./images -q 60 -j mozjpeg -p pngquant
```

### 3. Lossless Optimization

```bash
npx qasai ./images -j jpegtran -p optipng -l
```

### 4. Convert to Modern Formats

```bash
# WebP (good browser support)
npx qasai ./images -f webp -q 80

# AVIF (best compression)
npx qasai ./images -f avif -q 70
```

### 5. Resize Images

```bash
# Max width 1920px (maintains aspect ratio)
npx qasai ./images -w 1920

# Specific dimensions
npx qasai ./images -s 800x600

# Scale to 50%
npx qasai ./images -s 50%
```

### 6. In-Place Compression (Overwrite Originals)

```bash
# With backup
npx qasai ./images -i -B -q 80

# Without backup (destructive)
npx qasai ./images -i -q 80
```

### 7. Custom Output Directory

```bash
npx qasai ./images -o ./compressed
```

### 8. Scripting & Automation

```bash
# Quiet mode (only errors)
npx qasai ./images -Q -q 80

# JSON output for parsing
npx qasai ./images --json

# Skip existing files
npx qasai ./images -S

# Dry run (preview)
npx qasai ./images -D
```

### 9. Performance

```bash
# Parallel processing (8 workers)
npx qasai ./images -P 8 -r
```

## CLI Options Reference

| Short | Long | Description | Default |
|-------|------|-------------|---------|
| `-o` | `--output <dir>` | Output directory | `qasai/` folder |
| `-i` | `--in-place` | Overwrite original files | `false` |
| `-q` | `--quality <n>` | Quality 1-100 | `80` |
| `-l` | `--lossless` | Lossless compression | `false` |
| `-s` | `--resize <dim>` | Resize (e.g., `800x600`, `50%`) | - |
| `-w` | `--max-width <px>` | Max width (keeps aspect ratio) | - |
| `-H` | `--max-height <px>` | Max height (keeps aspect ratio) | - |
| `-f` | `--format <fmt>` | Convert to format (jpg, png, webp, avif) | - |
| `-r` | `--recursive` | Process subdirectories | `false` |
| `-m` | `--keep-metadata` | Preserve EXIF data | `false` |
| `-e` | `--effort <n>` | Compression effort 1-10 | `6` |
| `-j` | `--jpeg-engine <e>` | mozjpeg, jpegtran, sharp | `mozjpeg` |
| `-p` | `--png-engine <e>` | pngquant, optipng, sharp | `pngquant` |
| `-g` | `--gif-engine <e>` | gifsicle, sharp | `gifsicle` |
| `-c` | `--colors <n>` | Max colors for PNG/GIF | `256` |
| `-D` | `--dry-run` | Preview without compressing | `false` |
| `-Q` | `--quiet` | Silent mode (only errors) | `false` |
| | `--json` | Output results as JSON | `false` |
| `-S` | `--skip-existing` | Skip existing output files | `false` |
| `-P` | `--parallel <n>` | Parallel processing workers | `4` |
| `-B` | `--backup` | Backup before in-place compression | `false` |

## Compression Engines

### JPEG Engines

- **mozjpeg** (default): Best compression, 5-10% smaller than libjpeg
- **jpegtran**: Pure lossless, only reorganizes data
- **sharp**: Fastest option

### PNG Engines

- **pngquant** (default): 60-80% size reduction, reduces to 256 colors
- **optipng**: Pixel-perfect lossless
- **sharp**: Fast with good compression

### GIF Engine

- **gifsicle** (default): Best for animated GIFs, preserves animation

## Output

Results show per-file:
- Original size
- Compressed size
- Bytes saved
- Percentage saved
- Time elapsed

## Examples for Agents

When user asks to optimize images:

```bash
# Default web optimization
npx qasai ./path/to/images -r -q 80

# For production deployment
npx qasai ./public/images -r -w 1920 -f webp -o ./public/optimized
```

When user wants lossless:

```bash
npx qasai ./images -j jpegtran -p optipng
```

When user wants smallest file size:

```bash
npx qasai ./images -q 60 -f avif
```

When scripting/automation:

```bash
# JSON output for parsing
npx qasai ./images --json -Q

# Skip already processed
npx qasai ./images -S -o ./out
```

## References

- [Full Specification](./spec.md) - Detailed CLI specification and all options
