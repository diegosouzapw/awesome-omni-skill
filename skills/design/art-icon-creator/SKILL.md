---
name: art-icon-creator
description: This skill should be used when creating artistic icon variations from images. It generates 10 different greyscale icon styles from a single image source, automatically compressing to under 20KB with high contrast appearance. Supports both URL and local file inputs.
---

# Art Icon Creator

## Purpose

Convert images into artistic icon variations suitable for branding, UI design, or artistic purposes. Generate 10 distinct artistic variations of an image as compressed, greyscale, high-contrast icons—from simple black & white to poster effects to comic book styles.

## When to Use This Skill

Use this skill when:
- Creating icon variations from a single source image
- Needing multiple artistic interpretations of the same image
- Preparing images for icon sets or UI design systems
- Converting any image to greyscale, posterized, high-contrast artwork
- The source is a URL or local file path

## Quick Start

Execute the script with an image source (URL or local file):

```bash
python scripts/create_art_icons.py "https://example.com/image.jpg"
```

Or with a local file:

```bash
python scripts/create_art_icons.py "/path/to/image.png"
```

Output files are created in the same directory as the source (or current directory for URLs) with naming pattern:
- `<original-name>_art_icon_01.png` through `<original-name>_art_icon_10.png`

### Output Display

The script displays:
1. **JSON results** with all file information (names, sizes, styles)
2. **Output directory** - absolute path where files were saved
3. **Absolute paths** - complete file paths for each generated icon

Example output:
```
✅ Created 10 variations

📁 Output Directory: /Users/benno/projects/ai/bassi/chats/xyz/DATA_FROM_USER

  ✓ /Users/benno/projects/ai/bassi/chats/xyz/DATA_FROM_USER/image_art_icon_01.png
  ✓ /Users/benno/projects/ai/bassi/chats/xyz/DATA_FROM_USER/image_art_icon_02.png
  ...
```

## The 10 Artistic Variations

Each variation applies different artistic techniques to create distinct visual styles:

### Black & White High Contrast (Variations 01-03)
- **01: High Contrast Soft** - Simple B&W with softer threshold
- **02: High Contrast Strict** - Pure B&W with strict 100-level threshold
- **03: High Contrast Extreme** - Extreme contrast black & white

Best for: Logo variations, UI icon sets, clean graphic design

### Artistic Poster Effects (Variations 04-06)
- **04: Poster 8-Color** - Artistic posterized with 8 color levels
- **05: Poster 16-Color** - Artistic posterized with 16 color levels
- **06: Poster Smooth** - Smooth posterized effect with edge smoothing

Best for: Design system assets, artistic interpretations, print media

### Comic Book Styles (Variations 07-09)
- **07: Comic Bold** - Comic book style with bold lines and edges
- **08: Comic Outline** - Comic book style emphasizing outlines
- **09: Comic Smooth** - Comic book style with smooth edge blending

Best for: Artistic graphics, illustrative designs, playful aesthetics

### Artistic Blend (Variation 10)
- **10: Artistic Blend** - Combination of posterization and edge emphasis

Best for: When you want something between poster and comic styles

## Output Specifications

All generated icons have these guaranteed properties:
- **Format**: Greyscale PNG
- **Size**: < 20 KB (PNG optimized)
- **Dimensions**: 256 × 256 pixels (square)
- **Aspect Ratio**: Original preserved with white padding
- **Colors**: Greyscale with high contrast
- **Background**: White (transparent areas and detected backgrounds replaced)

## Image Processing Pipeline

1. **Load**: Supports URLs (http/https) and local file paths
2. **Convert**: Handles any color mode or transparency
3. **Background**: Detects and removes background, replaces with white
4. **Resize**: 256×256 square with aspect ratio preservation and white padding
5. **Convert to Greyscale**: All variations are monochrome
6. **Apply Artistic Effect**: Different techniques per variation
7. **Compress**: PNG optimization to <20 KB

## Advanced Usage

### Specify Output Directory

```bash
python scripts/create_art_icons.py "source" "/path/to/output"
```

### Python Import

```python
from create_art_icons import create_variations

results = create_variations(
    source="https://example.com/image.jpg",
    output_dir="/output/path"
)

# results contains:
# - success: Boolean
# - files: List of created files with paths and sizes
# - output_directory: Where files were saved
```

## Tips for Best Results

1. **Source Image Quality**: Works best with clear, well-defined subjects
2. **Complex Images**: May lose fine detail due to posterization (expected for icon style)
3. **Choosing Variations**:
   - For logos: Try 01-03 (high contrast) or 10 (artistic blend)
   - For UI icons: Try 04-06 (poster effects)
   - For artistic graphics: Try 07-09 (comic styles)
4. **File Size**: All outputs automatically optimize; actual sizes vary by image complexity

## Examples

### Processing a URL Image

```bash
python scripts/create_art_icons.py "https://github.com/user/image.jpg"
```

Creates `image_art_icon_01.png` through `image_art_icon_10.png` in current directory.

### Processing a Local File with Custom Output

```bash
python scripts/create_art_icons.py "/Users/me/pictures/photo.png" "/Users/me/icons"
```

Creates icons in `/Users/me/icons`.

## Technical Implementation

The skill includes `scripts/create_art_icons.py` which implements:
- URL and local file loading
- 10 different artistic variation algorithms
- Automatic background detection and removal
- PNG compression and optimization
- JSON result reporting
- CLI interface for automation

**Key Functions**:
- `create_variations(source, output_dir)` - Main function (returns dict with results)
- `variation_01_high_contrast_soft()` through `variation_10_artistic_blend()` - Individual artistic effects
