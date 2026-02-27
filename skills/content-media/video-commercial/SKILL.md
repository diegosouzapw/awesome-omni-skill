---
name: video-commercial
description: Generate 30-second video commercials from a concept. Creates storyboard, generates scene images, adds narration via ElevenLabs, assembles final video. Use when asked to create commercials, promo videos, video ads, or short marketing videos.
---

# Video Commercial Generator

Create short video commercials (~30 seconds) from a concept through a multi-phase pipeline.

## Workflow

### Phase 1: Concept → Storyboard

Parse the concept into 5-6 scenes. Create `storyboard.json`:

```json
{
  "title": "Project Name",
  "concept": "Original concept text",
  "duration": 30,
  "scenes": [
    {
      "id": 1,
      "duration": 5,
      "visual": "Detailed image generation prompt",
      "motion": "zoom-in|zoom-out|pan-left|pan-right|static",
      "narration": "Voiceover text for this scene",
      "overlay": "Optional text overlay"
    }
  ]
}
```

### Phase 2: Generate Images

For each scene, generate an image using available image tools (nano-banana-pro or openai-image-gen).

Save to: `scenes/scene-{id}.png`
Save prompt metadata to: `scenes/scene-{id}.json`

### Phase 3: Generate Narration

Use ElevenLabs (sag skill) to generate voiceover from combined narration text.

Save to: `audio/narration.mp3`

### Phase 4: Create Video Clips

Use ffmpeg to create video clips from images with motion effects:

```bash
# Zoom in effect (5 seconds)
ffmpeg -loop 1 -i scene-01.png -vf "scale=8000:-1,zoompan=z='min(zoom+0.0015,1.5)':d=150:s=1920x1080" -t 5 -pix_fmt yuv420p clip-01.mp4

# Static with slight zoom
ffmpeg -loop 1 -i scene-01.png -vf "scale=2000:-1,zoompan=z='1.1':d=150:s=1920x1080" -t 5 -pix_fmt yuv420p clip-01.mp4
```

### Phase 5: Assemble Final Video

1. Concatenate clips
2. Add narration audio
3. Add background music (optional)
4. Add text overlays
5. Export final video

```bash
# Concat clips
ffmpeg -f concat -i clips.txt -c copy combined.mp4

# Add audio
ffmpeg -i combined.mp4 -i narration.mp3 -c:v copy -c:a aac -map 0:v -map 1:a final.mp4
```

## Project Structure

```
projects/video-commercial/{project-slug}/
├── concept.md           # Original concept
├── storyboard.json      # Scene breakdown
├── scenes/
│   ├── scene-01.png     # Generated images
│   ├── scene-01.json    # Prompt + metadata
│   └── ...
├── audio/
│   ├── narration.mp3    # ElevenLabs voiceover
│   └── music.mp3        # Background (optional)
├── clips/
│   ├── clip-01.mp4      # Individual scene videos
│   └── ...
├── output/
│   └── final.mp4        # Assembled video
└── review.md            # Review notes + iterations
```

## Review Process

After each phase, update `review.md` with:
- What was generated
- User feedback
- Scenes to regenerate
- Approved scenes (mark with ✓)

Allow regeneration of individual scenes without redoing entire project.

## Commands Reference

| Command | Action |
|---------|--------|
| `new commercial "{concept}"` | Create project + storyboard |
| `show storyboard` | Display current storyboard |
| `generate images` | Generate all scene images |
| `regen scene {n}` | Regenerate specific scene |
| `preview` | Show all scenes with status |
| `narrate` | Generate voiceover |
| `render` | Assemble final video |
| `status` | Show project progress |

## Notes

- Projects stored in workspace: `projects/video-commercial/`
- Image aspect ratio: 16:9 (1920x1080)
- Default scene duration: 5 seconds
- Narration voice: Configure in project or use default
