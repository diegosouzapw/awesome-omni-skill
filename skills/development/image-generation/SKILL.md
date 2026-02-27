---
name: image-generation
description: "Generate, edit, and upscale AI images. Use when creating visual assets for apps, websites, or documentation. FREE Cloudflare tier for iterate generation (~96/day), Fal.ai for paid tiers. Four quality tiers (iterate/default/premium/max). Supports text specialists, multi-ref editing, SVG, background removal. Triggers: generate image, create image, edit image, upscale, logo, picture of, remove background."
metadata:
  author: ai-dev-atelier
  version: "3.0"
---

# Image Generation

Generate, edit, and upscale images with standardized quality tiers and embedded best practices.

## Quick Start

```
Need image?
├─ Text/Logo → bun scripts/gen.ts "..." --text [-t tier]
├─ Photo/Art → bun scripts/gen.ts "..." [-t tier]
├─ Edit existing → bun scripts/edit.ts <img> "..." [-t tier]
├─ Upscale → bun scripts/upscale.ts <img> [-t tier]
├─ Vectorize → bun scripts/svg.ts <img> ($0.01/img)
└─ Remove BG → bun scripts/rembg.ts <img> (FREE)

Tier selection:
├─ iterate  → FREE drafts (~96/day via Cloudflare)
├─ default  → Daily driver ($0.008/MP)
├─ premium  → Final assets ($0.03/MP)
└─ max      → Critical work, SOTA ($0.06-0.07/MP)
```

## Entry Points

| Script | Purpose |
|--------|---------|
| `bun scripts/gen.ts` | Text → Image |
| `bun scripts/edit.ts` | Image + Instruction → Image |
| `bun scripts/upscale.ts` | Image → Larger Image |
| `bun scripts/svg.ts` | Image → SVG ($0.01/img) |
| `bun scripts/rembg.ts` | Remove background (FREE) |

---

## Prompting Best Practices

**CRITICAL**: Good prompts are the difference between unusable output and production-ready assets.

### The Universal Prompt Structure

```
[Subject] + [Action/Pose] + [Environment] + [Style/Medium] + [Lighting] + [Camera/Composition]
```

**Example**:
> "A cybernetic owl perched on a neon sign in a rain-soaked alley. Cinematic lighting with teal and orange highlights. Shot on 35mm film, shallow depth of field, hyper-detailed textures."

### DO: Effective Prompting

| Technique | Example |
|-----------|---------|
| **Be specific** | "middle-aged man with salt-and-pepper hair wearing charcoal turtleneck" NOT "a man" |
| **Describe the result** | "person with clear eyes" NOT "remove glasses" |
| **Use camera terms** | "Shot on Hasselblad, 85mm lens, f/1.8" |
| **Specify lighting** | "golden hour rim lighting with deep shadows" |
| **Include textures** | "weathered sandstone", "anodized aluminum", "iridescent silk" |

### DON'T: Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Negative phrasing | "no glasses" often adds glasses | Describe what IS there |
| Vague subjects | AI interprets randomly | Be exhaustively specific |
| Keyword salad | "4k, trending, masterpiece" is noise | Use descriptive sentences |
| Short prompts | Under 20 words underperforms | Aim for 40-80 words |

### Style Keywords That Work

| Category | Keywords |
|----------|----------|
| **Lighting** | golden hour, volumetric lighting, Rembrandt lighting, neon rim light, bioluminescent |
| **Camera** | 35mm anamorphic, macro photography, tilt-shift, fisheye, drone shot |
| **Style** | cinematic, photorealistic, concept art, ukiyo-e, baroque, impressionist |
| **Quality** | hyper-detailed, sharp focus, 8k resolution, raytraced |

---

## Text & Logo Generation (--text flag)

Uses **Recraft V3** (iterate/default) or **Ideogram V3** (premium/max) - specialized for typography.

### Text Prompting Rules

**CRITICAL**: Put text in `"Double Quotes"` at the START of your prompt.

```bash
# Correct - text first, then describe
bun scripts/gen.ts '"QUANTUM" in bold futuristic font, metallic silver, dark space background' --text

# Wrong - text buried in description
bun scripts/gen.ts 'A logo with the word QUANTUM on it' --text
```

### Logo Design Patterns

| Style | Prompt Pattern |
|-------|----------------|
| **Minimalist** | `"BRAND" minimalist vector logo, clean lines, simple geometry, flat design` |
| **Vintage** | `"EST. 1920" vintage badge logo, circular emblem, ribbon banner, ornate border` |
| **Negative space** | `"PEAK" logo where the letter A forms a mountain, negative space design` |
| **3D/Modern** | `"TECHCORP" bold 3D chrome letters, gradient fill, dark background` |

### Font Specification

Use typography terms: `modern sans-serif`, `elegant script`, `bold blocky`, `blackletter`, `neon tubing`, `retro 70s serif`

### DO/DON'T for Text

| DO | DON'T |
|----|-------|
| "Three cats playing" (exact count) | "cats playing" (random count) |
| "wooden baseball bat" (specific) | "bat" (ambiguous) |
| Describe only what you want | "no cake" (will add cake) |

---

## Image Editing

```bash
bun scripts/edit.ts <image> <instruction> [-t TIER] [--mask <mask.png>] [--ref <img>...]
```

### Writing Edit Instructions

**Key**: Describe the TARGET STATE, not the change.

| Bad Instruction | Good Instruction |
|-----------------|------------------|
| "change car to blue" | "A sleek blue metallic sports car, reflections of neon lights on wet asphalt" |
| "add a hat" | "person wearing a vintage red fedora, matching the scene lighting" |
| "remove background" | Use `rembg.ts` instead (FREE and better) |

### Mask Best Practices

| Task | Mask Strategy |
|------|---------------|
| **Object removal** | Mask LARGER than object (10-20px margin) for seamless fill |
| **Object addition** | Mask exact shape or slightly smaller |
| **Outpainting** | Overlap 10-20px INTO original image |

**Feathering**: Apply 12-16px blur to masks. Sharp masks = visible seams.

### Multi-Reference Editing (--ref)

Using 2+ reference images auto-selects `max` tier (flux-2-flex).

```bash
# Style transfer: apply reference style to base image
bun scripts/edit.ts base.jpg "in the style of the reference" --ref style.jpg

# Multi-reference blending
bun scripts/edit.ts scene.jpg "forest sofa scene" --ref forest.jpg --ref sofa.jpg
```

**Tip**: When blending references, describe their relationship: "A velvet sofa placed in a misty pine forest"

---

## Upscaling

```bash
bun scripts/upscale.ts <image> [-t TIER] [--scale 2|4]
```

### When to Use 2x vs 4x

| Source Quality | Recommendation |
|----------------|----------------|
| High (RAW, clean PNG) | 4x safe - AI infers detail accurately |
| Medium (standard JPEG) | 2x preferred - denoise first if possible |
| Low (compressed, blurry) | 2x max - noise gets magnified |

### Use Case Guidelines

| Output | Scale | Notes |
|--------|-------|-------|
| Web/UI | 2x | Reduces file size, improves perceived sharpness |
| Print (300 DPI) | 4x | Target 300 DPI for print quality |
| Icons/Logos | 2x | Use `svg.ts` instead for infinite scaling |

### Common Artifacts & Fixes

| Artifact | Cause | Prevention |
|----------|-------|------------|
| Haloing (white edges) | Aggressive sharpening | Use iterate/default tier |
| Plasticky skin | Over-smoothing | Reduce to 2x, use premium tier |
| Grid patterns | Tile processing | Use higher tier models |

**Rule of Thumb**: If image looks "crunchy" at 100% zoom, don't exceed 2x.

---

## Tier Selection Guide

| Scenario | Tier | Why |
|----------|------|-----|
| Exploring 10+ variations | `iterate` | FREE, fast iteration |
| Daily work, 3-5 variations | `default` | Best cost/quality balance |
| Client deliverables | `premium` | Higher fidelity |
| Critical assets, multi-ref | `max` | SOTA quality, advanced features |
| Text/logos (any) | `default` | Recraft V3 already excellent |
| Text/logos (critical) | `premium` | Ideogram V3 for perfect typography |

### Cost Optimization

```
EXPENSIVE WORKFLOW (avoid):
  Generate at max tier → iterate on max → deliver

COST-EFFECTIVE WORKFLOW (recommended):
  Generate at iterate (FREE) → find best concept
  → Regenerate winner at default/premium → deliver
```

---

## Environment

```bash
# For FREE iterate generation (Cloudflare)
CLOUDFLARE_ACCOUNT_ID=xxx
CLOUDFLARE_API_TOKEN=xxx

# For paid tiers (Fal.ai)
FAL_API_KEY=xxx
```

**Quota**: Cloudflare FREE tier allows ~96 images/day at 1024x1024.

## Exit Codes

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | Image saved to `.ada/data/images/` |
| 1 | General error | Check error message |
| 2 | Config/auth error | Verify API keys in `.env` |
| 3 | Resource limit | Quota exceeded - wait 24h or use paid tier |

**CRITICAL**: Exit code 3 does NOT fall back to paid tier. This prevents accidental charges.

---

## Integration

| Skill | When to Use Together |
|-------|---------------------|
| `ui-animation` | Animate generated images for web/mobile |
| `docs-write` | Document image assets and parameters used |
| `search` | Find prompting resources and style references |
| `code-quality` | After modifying skill scripts |

## References

- `references/usage-guide.md` - Extended prompting guide, error codes, testing
- `README.md` - Architecture diagrams, model reference, CLI details
- [Fal.ai Docs](https://fal.ai/learn/devs) - Official API documentation

## Output

Images saved to `.ada/data/images/` with timestamped filenames:
```
20260118_gen_default_cyberpunk_city.jpg
20260118_svg_default_logo_vector.svg
```
