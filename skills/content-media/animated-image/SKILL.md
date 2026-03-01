---
name: animated-image
description: Generate a minimalist static image from a prompt, then animate it into a short video. Use when users want a concept image with a simple, obvious motion.
---

# Animated Image

## Instructions
- If missing, ask the user for a concise image prompt.
- Generate a static image from the prompt and present it.
- Ask for a brief animation prompt; if omitted, pick the most obvious motion in the scene automatically.
- Generate a short video conditioned on the image and animation prompt.
- Return both files and a one‑sentence caption describing the result.

## Examples
- Image prompt: "A wooden ball on a ramp, clean primary colors, minimalist, clean lines."
- Animation prompt: "Animate the scene to show the wooden ball rolling down the ramp."


## Overview

This Skill creates a static image using Google's image model and then animates it into a short video using Google's video model. Use it when you need a quick concept render (clean, minimalist shapes and colors) and an accompanying short animation.

## When to Use

- You want to generate a single-frame image concept and animate a clear, primary motion.
- You have a short animation idea (e.g., a ball rolling) that can be expressed as a brief prompt.

## Inputs

- image_prompt (required): short description of the desired image
- animation_prompt (optional): short description of the desired animation

## Outputs

- image file (.png)
- video file (.mp4)

## Examples

- image_prompt: "A wooden ball on a ramp, clean primary colors, minimalist, clean lines."
- animation_prompt: "Animate the scene to show the wooden ball rolling down the ramp."

## Execution

The Skill runs local Node.js scripts bundled in this Skill directory. Ensure `GOOGLE_API_KEY` is set in your environment before running.

### Setup (one-time)

1. Install dependencies:
   ```bash
   npm i
   ```
2. Configure environment variables:
   - Copy `.env.example` to `.env` and set `GOOGLE_API_KEY`
   - Optionally set `OUTPUT_DIR` (defaults to `~/..claude/outputs` for personal Skill)

Scripts:

- `scripts/generateImage.mjs "<image_prompt>"`
- `scripts/generateVideo.mjs <image_path> "<animation_prompt>"`

If no animation_prompt is provided, the script infers an obvious motion from the image context (e.g., ball rolls, child runs, flag waves). The scripts print `file://` links for quick opening.

## Reference

- How to create custom Skills: https://support.claude.com/en/articles/12512198-how-to-create-custom-skills


