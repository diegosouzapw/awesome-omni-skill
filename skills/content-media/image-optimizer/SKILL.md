---
name: image-optimizer
description: Optimize and compress images for web use. Reduces file sizes of JPEG, PNG, GIF images using lossy/lossless compression. Can resize images to maximum dimensions, convert to WebP format, and process entire directories recursively. Use when images are too large for web, need compression, or need format conversion.
license: MIT
compatibility: Requires imagemagick, jpegoptim, pngquant, and webp packages
metadata:
  author: exedev
  version: "1.0"
---

# Image Optimizer

Compress and optimize images for web delivery, similar to Squoosh.

## Quick Start

```bash
# Ensure dependencies are installed
scripts/install-deps.sh

# Optimize images
scripts/imgopt.sh [OPTIONS] <files or directories>
```

## Common Use Cases

### Optimize a single image
```bash
scripts/imgopt.sh photo.jpg
```

### Compress all images in a folder
```bash
scripts/imgopt.sh images/
```

### Resize and compress for web (recommended for large images)
```bash
scripts/imgopt.sh -q 80 -w 1920 images/
```

### Create WebP versions (best compression for modern browsers)
```bash
scripts/imgopt.sh --webp images/
```

### Process recursively and keep originals
```bash
scripts/imgopt.sh -r --keep --webp ./website/images/
```

### Output to a different directory
```bash
scripts/imgopt.sh -o optimized/ *.jpg *.png
```

### Preview what would happen (dry run)
```bash
scripts/imgopt.sh -n -r images/
```

## Options Reference

| Option | Description | Default |
|--------|-------------|--------|
| `-q, --quality N` | Quality level 1-100 | 80 |
| `-w, --max-width N` | Maximum width in pixels | unlimited |
| `-h, --max-height N` | Maximum height in pixels | unlimited |
| `-o, --output DIR` | Output directory | in-place |
| `--webp` | Also create WebP versions | off |
| `--keep` | Keep originals (save as .opt.ext) | off |
| `-r, --recursive` | Process directories recursively | off |
| `-n, --dry-run` | Show what would be done | off |
| `-v, --verbose` | Show detailed progress | off |

## Supported Formats

- **Input**: JPEG, PNG, GIF, BMP, TIFF
- **Output**: Same format (optimized) + optional WebP

## Recommended Settings by Use Case

| Use Case | Command |
|----------|--------|
| Blog images | `-q 80 -w 1200 --webp` |
| Hero/banner images | `-q 85 -w 1920 --webp` |
| Thumbnails | `-q 75 -w 400` |
| E-commerce products | `-q 85 -w 1000 --webp` |
| Maximum compression | `-q 60 -w 1920 --webp` |

## Tips

1. **Always test on copies first** - Use `--keep` or `-o output/` until you're happy with results
2. **WebP is best for web** - Creates files typically 25-35% smaller than optimized JPEG
3. **Quality 80 is a good default** - Below 70 you may notice artifacts
4. **Resize matters most** - A 4000px image resized to 1920px will be much smaller regardless of compression

See [references/REFERENCE.md](references/REFERENCE.md) for detailed technical information.
