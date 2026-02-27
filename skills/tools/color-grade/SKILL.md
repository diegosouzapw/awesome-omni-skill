---
name: color-grade
description: Generate .cube 3D LUTs for color correction in DaVinci Resolve and Adobe Premiere Pro. Fixes exposure, color casts, skin tones, and black levels. Use when the user needs color grading, LUT generation, or frame analysis for video footage.
argument-hint: [lut-type] [output-path]
allowed-tools: Read, Bash(ruby *), Bash(python3 *), Bash(ffmpeg *), Write, Glob, Grep
---

# Color Grade AI

Generate targeted .cube 3D LUTs for color correction. Works with DaVinci Resolve and Adobe Premiere Pro.

## Quick Start

```bash
# Generate a LUT (presets defined in presets.yml)
ruby generate_lut.rb <type> <output_path> [--strength=0.0-1.0]

# Analyze a frame for color stats
ruby analyze_frame.rb <image_path> <x1,y1,x2,y2> [label]

# Regenerate docs from presets.yml
ruby generate_docs.rb
```

## Config-Driven Presets

All LUT types are defined in `presets.yml`. Each preset is a pipeline of ordered processing steps (exposure, black crush, skin correction, etc.). Adding a new LUT type means adding a YAML entry — no Ruby code changes needed.

## Available LUT Types

| Type | What it fixes |
|------|--------------|
| `night_warm_fix` | **All-in-one** for underexposed scenes with warm/red practicals. ~1 stop lift + skin hue shift + black crush. No desaturation — preserves vivid reds. Use with just AMIRA. |
| `night_purple_fix` | **All-in-one** for underexposed scenes with purple/magenta stage lighting. RGB rebalancing + ~2 stop lift + purple desat + skin hue shift + black crush. Preserves atmospheric purple while fixing skin. |
| `yellow_fix` | Warm amber/yellow cast from stage lighting. H=10-60, 55% desat. |
| `red_skin_fix` | Sunburnt/flushed red skin from warm practicals. Hue shift to peach, skin-only targeting. |
| `overexposure_fix` | Scene-wide ~1 stop reduction with highlight rolloff. |
| `underexposure_fix` | Scene-wide ~1.2 stop lift with shadow recovery. |
| `black_crush` | Crushes milky/lifted blacks below 12% to true black. |
| `skin_highlight_fix` | Subtle skin-only highlight rolloff above 70% luminance. |

## Workflow

1. Extract a frame: `ffmpeg -i video.mov -ss 00:00:30 -frames:v 1 frame.png`
2. Analyze regions: `ruby analyze_frame.rb frame.png 1500,600,2000,900 skin`
3. Generate appropriate LUT: `ruby generate_lut.rb overexposure_fix output.cube`
4. Apply in your NLE on an adjustment layer / node after your main conversion LUT

## Applying LUTs

**DaVinci Resolve:** Add a serial node after your conversion LUT node. Right-click the node, select LUT, browse to the .cube file.

**Adobe Premiere Pro:** Add Lumetri Color effect on an adjustment layer. Go to Creative tab, click the Look dropdown, browse to the .cube file.

## Stacking LUTs

LUTs can be stacked on separate adjustment layers / nodes. Common stacks:

**Night scene with warm practicals (simplest):**
1. Camera conversion LUT (e.g. AMIRA LogC to Rec.709)
2. Night Warm Fix (single LUT handles everything)

**Night scene with purple stage lighting:**
1. Camera conversion LUT (e.g. AMIRA LogC to Rec.709)
2. Night Purple Fix (single LUT handles everything)

**Overexposed footage:**
1. Camera conversion LUT
2. Overexposure Fix
3. Black Crush

**Underexposed footage:**
1. Camera conversion LUT
2. Underexposure Fix

## Node Order (DaVinci Resolve)

Professional serial node chain:
1. White Balance
2. Exposure
3. Main Conversion LUT
4. General Color Adjustments
5. Color Saturation/Boost
6. Noise Reduction
7. Black Levels
8. Vignette
9. Sharpening

## Color Science Notes

- **Desaturating warm tones in HSL produces brown/sepia.** Use hue shifting or RGB rebalancing instead.
- **Skin tones occupy H=10-35, S=0.08-0.45, L=0.25-0.75.** Use all three windows (hue + saturation + luminance) to isolate skin from light sources and colored objects.
- **LUTs have no spatial awareness.** They can't do noise reduction, sharpening, or distinguish adjacent pixels. Each pixel is transformed independently.
- **3D LUTs allow cross-channel operations.** Unlike 1D LUTs, a 3D LUT can change the red output based on the green and blue input values.

## Requirements

- Ruby 2.7+
- Python 3 with Pillow (for analyze_frame.rb)
- ffmpeg (for frame extraction)
