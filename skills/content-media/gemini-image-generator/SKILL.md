---
name: gemini-image-generator
description: "Generate and edit images using Google Gemini. Use when the user asks to generate, create, edit, or modify images."
---

# Gemini Image Generator

Generate images using Google's Gemini image generation model.

## For Claude

When the user asks you to generate an image:

1. **Find the skill directory** - Look for this skill in `~/.claude/skills/gemini-image-generator/` (global) or `./.claude/skills/gemini-image-generator/` (project-local)

2. **Run the generation script:**
   ```bash
   npx tsx <skill-dir>/generate-image.ts "<descriptive prompt>" -n <filename> -a <aspect-ratio> -o <output-dir>
   ```

3. **Handle API key errors:** If you see "GOOGLE_API_KEY not found", offer to help the user set it up:
   - Recommend creating a `.env` file in the skill directory with `GOOGLE_API_KEY=their-key`
   - Alternative: set `GOOGLE_API_KEY` environment variable

**Options:**
- `--name, -n <filename>`: File name. Default: `generated-image.png`
- `--aspect, -a <ratio>`: Aspect ratio (`1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`). Default: `1:1`
- `--output, -o <dir>`: Output directory. Default: current directory
- `--ref, -r <image>`: Reference image for editing or style transfer (can be used multiple times, up to 14 images)

## Reference Images

Use `-r` to edit an existing image or use images as style/content references:

**Edit an image:**
```bash
npx tsx <skill-dir>/generate-image.ts "Add a party hat to this cat" -r ./cat.png -n cat-party.png
```

**Style transfer:**
```bash
npx tsx <skill-dir>/generate-image.ts "A portrait in this art style" -r ./reference-style.jpg -n portrait.png
```

**Multiple references:**
```bash
npx tsx <skill-dir>/generate-image.ts "Group photo of these people at a beach" -r person1.jpg -r person2.jpg -r person3.jpg
```

## Prompting Guide

**Core principle:** Describe the scene, don't just list keywords. A narrative, descriptive paragraph will almost always produce a better, more coherent image than a list of disconnected words.

### Photorealistic Scenes
Use photography terms. Mention camera angles, lens types, lighting, and fine details.

> "A photorealistic close-up portrait of an elderly Japanese ceramicist with deep, sun-etched wrinkles and a warm, knowing smile. He is carefully inspecting a freshly glazed tea bowl. The setting is his rustic, sun-drenched workshop with pottery wheels and shelves of clay pots in the background. The scene is illuminated by soft, golden hour light streaming through a window, highlighting the fine texture of the clay and the fabric of his apron. Captured with an 85mm portrait lens, resulting in a soft, blurred background (bokeh). The overall mood is serene and masterful."

### Stylized Illustrations & Stickers
Be explicit about the style and request a specific background color if needed.

> "A kawaii-style sticker of a happy red panda wearing a tiny bamboo hat. It's munching on a green bamboo leaf. The design features bold, clean outlines, simple cel-shading, and a vibrant color palette. The background must be white."

### Text in Images
Gemini excels at rendering text. Be clear about the text content, font style (descriptively), and overall design.

> "Create a modern, minimalist logo for a coffee shop called 'The Daily Grind'. The text should be in a clean, bold, sans-serif font. The color scheme is black and white. Put the logo in a circle. Use a coffee bean in a clever way."

### Product Mockups & Commercial Photography
For clean, professional product shots, describe studio lighting setup and camera positioning.

> "A high-resolution, studio-lit product photograph of a minimalist ceramic coffee mug in matte black, presented on a polished concrete surface. The lighting is a three-point softbox setup designed to create soft, diffused highlights and eliminate harsh shadows. The camera angle is a slightly elevated 45-degree shot to showcase its clean lines. Ultra-realistic, with sharp focus on the steam rising from the coffee."

### Minimalist & Negative Space Design
Great for backgrounds where text will be overlaid. Explicitly mention negative space.

> "A minimalist composition featuring a single, delicate red maple leaf positioned in the bottom-right of the frame. The background is a vast, empty off-white canvas, creating significant negative space for text. Soft, diffused lighting from the top left."

### Sequential Art (Comics/Storyboards)
Describe the art style, number of panels, and scene for visual storytelling.

> "Make a 3 panel comic in a gritty, noir art style with high-contrast black and white inks. Put the character in a humorous scene."

## Best Practices

- **Be Hyper-Specific:** The more detail you provide, the more control you have. Instead of "fantasy armor," describe it: "ornate elven plate armor, etched with silver leaf patterns, with a high collar and pauldrons shaped like falcon wings."

- **Provide Context and Intent:** Explain the purpose of the image. For example, "Create a logo for a high-end, minimalist skincare brand" will yield better results than just "Create a logo."

- **Iterate and Refine:** Don't expect a perfect image on the first try. Use follow-up prompts like, "That's great, but can you make the lighting a bit warmer?" or "Keep everything the same, but change the character's expression to be more serious."

- **Use Step-by-Step Instructions:** For complex scenes with many elements, break your prompt into steps. "First, create a background of a serene, misty forest at dawn. Then, in the foreground, add a moss-covered ancient stone altar. Finally, place a single, glowing sword on top of the altar."

- **Use Semantic Negative Prompts:** Instead of saying "no cars," describe the desired scene positively: "an empty, deserted street with no signs of traffic."

- **Control the Camera:** Use photographic and cinematic language to control the composition. Terms like wide-angle shot, macro shot, low-angle perspective.

## Styling

The section above is about **HOW** to generate the right image but this section is about **What** to generate. Try to have good taste in the image. For header images, you want to choose something that represents the philosophy discussed in the blog in an artistic, heady way. So if the blog is about AI's effect on the economy for example, you wouldnt want to just generate an image of the economy, but you would want to instead think of a kind of art piece that would send a message about the topic of AI's effect on the economy. Be high brow and intelligent about your choices of image.

## Requirements

- Node.js and npm
- Dependencies installed via `npm install` in the skill directory
- `GOOGLE_API_KEY` set in `.env` file or as environment variable
