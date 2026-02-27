---
name: video
description: Generate videos using fal.ai (Wan, Kling) or Sora. Text-to-video and image-to-video.
user-invocable: false
disable-model-invocation: true
disabled: false
icon: film.fill
aliases: [sora, generate video, vid, fal]
parameters:
  - name: prompt
    placeholder: Describe the video...
    required: true
---

# Video Generation Skill

Two providers: **fal.ai** (Wan, Kling — API-based, $5/day budget) and **Sora** (Playwright automation, currently disabled).

**Default provider: fal.ai** — use Sora only when explicitly requested.

## IMPORTANT RULES

- **ALWAYS confirm with the user before generating** — describe what you plan to generate and wait for approval. Never auto-generate.
- **$5/day budget** for fal.ai — check budget before generating. Run `--budget` to see status.
- **Default: Wan 480p** (cheapest at ~$0.04/sec). Use Kling only for premium quality.
- **Default aspect ratio is 16:9**. Use 9:16 for portrait/mobile content.

## fal.ai Commands

```bash
# Text-to-video (Wan 2.2, 480p, 16:9 — cheapest)
python3 .claude/skills/video/src/fal_create.py "a goldfish swimming in clear water"

# Portrait (9:16)
python3 .claude/skills/video/src/fal_create.py "neon dice rolling" -a 9:16

# Higher quality (720p — 2x cost)
python3 .claude/skills/video/src/fal_create.py "epic landscape" -r 720p

# Kling Standard (better quality, $0.28/5s)
python3 .claude/skills/video/src/fal_create.py "robot playing poker" -m kling-std

# Kling Pro (best quality, $0.49/5s)
python3 .claude/skills/video/src/fal_create.py "cinematic casino scene" -m kling-pro

# 10-second Kling ($0.56)
python3 .claude/skills/video/src/fal_create.py "dice duel showdown" -m kling-std -d 10

# Image-to-video (auto-selects i2v model)
python3 .claude/skills/video/src/fal_create.py "camera slowly zooms out" --image /path/to/photo.png

# Batch mode
python3 .claude/skills/video/src/fal_create.py --batch /path/to/jobs.json

# Check budget
python3 .claude/skills/video/src/fal_create.py --budget

# Override daily limit
python3 .claude/skills/video/src/fal_create.py "prompt" --daily-limit 3.00
```

## fal.ai Options

| Flag | Options | Default | Notes |
|------|---------|---------|-------|
| `-m` | wan-t2v, wan-i2v, kling-std, kling-pro, kling-std-i2v | wan-t2v | Auto-switches to i2v with --image |
| `-r` | 480p, 580p, 720p | 480p | Wan only |
| `-a` | 16:9, 9:16, 1:1 | 16:9 | All models |
| `-d` | 5, 10 | 5 | Kling only (seconds) |
| `-f` | 17-161 | 81 | Wan only (frames, 81≈5s at 16fps) |
| `-i` | path | — | Reference image |
| `-b` | path | — | Batch JSON |

## fal.ai Pricing

| Model | Resolution/Duration | Cost | Videos per $5 |
|-------|-------------------|------|---------------|
| **wan-t2v** | 480p | ~$0.04/sec | ~125 |
| **wan-t2v** | 720p | ~$0.08/sec | ~62 |
| **wan-i2v** | 480p | $0.20 | 25 |
| **wan-i2v** | 720p | $0.40 | 12 |
| **kling-std** | 5s | $0.28 | 17 |
| **kling-std** | 10s | $0.56 | 8 |
| **kling-pro** | 5s | $0.49 | 10 |
| **kling-pro** | 10s | $0.98 | 5 |

Budget tracked in `~/.config/fal/budget.json`, resets daily.

## fal.ai Batch Format

```json
[
  {"prompt": "neon dice rolling on felt", "model": "wan-t2v", "aspect_ratio": "9:16"},
  {"prompt": "robot casino", "model": "kling-std", "duration": "5"},
  {"prompt": "animate this", "model": "wan-i2v", "image": "/path/to/img.png"}
]
```

Each job supports: `prompt` (required), `model`, `resolution`, `aspect_ratio`, `duration`, `num_frames`, `image`.

---

## Sora Commands (currently disabled — needs re-auth)

## When to Use This Skill

- User asks for a video, animation, or motion content
- A concept would be better shown as video than a still image
- User explicitly mentions Sora
- Image-to-video: user provides a reference image and wants it animated

Do NOT use when:
- A still image is sufficient (use the `image` skill instead)
- User needs code or text, not visuals

## Commands

```bash
# Text-to-video (portrait, 5 seconds — portrait is default)
python3 .claude/skills/video/src/create.py "a goldfish swimming in clear water" -o portrait

# Landscape orientation (only when explicitly requested)
python3 .claude/skills/video/src/create.py "a person walking through rain" -o landscape

# Square
python3 .claude/skills/video/src/create.py "abstract shapes morphing" -o square

# Longer duration (10s = 300 frames, 15s = 450, 20s = 600)
python3 .claude/skills/video/src/create.py "timelapse of clouds" -f 300

# Image-to-video (animate a reference image)
python3 .claude/skills/video/src/create.py "camera slowly zooms out" --image /path/to/photo.png

# Larger resolution (512x896 portrait, 1024x576 landscape)
python3 .claude/skills/video/src/create.py "epic landscape" -s large

# Batch mode (submit multiple jobs in one browser session)
python3 .claude/skills/video/src/create.py --batch /path/to/jobs.json

# Download recent videos WITHOUT generating (recover from timeouts)
python3 .claude/skills/video/src/download.py

# Download more history (default: 20 recent drafts)
python3 .claude/skills/video/src/download.py --limit 50
```

## Options

- First positional arg: prompt (required, unless using --batch)
- `-o`, `--orientation`: `landscape` (default), `portrait` (square is NOT supported by Sora)
- `-s`, `--size`: `small` (default), `large`
- `-f`, `--frames`: `150` (5s, default), `300` (10s), `450` (15s), `600` (20s)
- `-i`, `--image`: path to reference image for image-to-video generation
- `-b`, `--batch`: path to JSON file with array of jobs

## Batch Mode

Submit multiple videos in a single browser session. Write a JSON file with an array of jobs:

```json
[
  {"prompt": "first video prompt", "orientation": "portrait"},
  {"prompt": "second video", "image": "/path/to/ref.png", "frames": 300},
  {"prompt": "third video", "orientation": "square", "size": "large"}
]
```

Each job supports: `prompt` (required), `orientation`, `size`, `frames`, `image` (all optional).

All jobs are submitted at once, polled together, and downloaded when ready. Much faster than running one at a time.

## Multi-Scene Story Workflow (Mood Board Chaining)

For multi-scene videos with consistent visual style, use this Gemini→Gemini→Sora pipeline:

### Step 1: Mood Board
Generate 2-3 mood boards with the `image` skill to establish the visual identity. Include color palette, key elements, atmosphere, character design. Present to user for approval.

### Step 2: Chained Frame Generation
Generate each scene frame with Gemini, chaining the previous frame as reference:
- **Frame 1**: mood board (--edit) + scene 1 prompt → Gemini
- **Frame 2**: frame 1 output (--edit) + scene 2 prompt + "match the visual style of the reference" → Gemini
- **Frame 3**: frame 2 output (--edit) + scene 3 prompt + same style instruction → Gemini
- ...each frame uses the PREVIOUS frame as --edit reference

The mood board is the anchor preventing style drift. The previous frame gives scene continuity. Always include "Match the visual style of the reference image exactly. Same color palette, same pixel art fidelity, same lighting approach." in every prompt.

### Step 3: Composite Review
Build a single composite image with all frames for efficient review:
```python
from PIL import Image
frames = [Image.open(f'frame-{i}.jpg') for i in range(1, N+1)]
target_h = 400
frames = [f.resize((int(f.width * target_h/f.height), target_h), Image.LANCZOS) for f in frames]
padding = 8
comp = Image.new('RGB', (sum(f.width for f in frames) + padding*(len(frames)+1), target_h + padding*2), (20,20,30))
x = padding
for f in frames:
    comp.paste(f, (x, padding))
    x += f.width + padding
comp.save('storyboard.jpg', 'JPEG', quality=85)
```

### Step 4: Sora Animation
Feed each approved frame to Sora as image-to-video in a batch. Prompt only describes MOTION, not the scene content (Sora sees the image).

### Brand Alignment Tricks
When aligning with a specific brand:
- Include brand logo/colors as reference images in the mood board generation
- Build a manual collage of reference images (brand assets, color swatches, style examples) to feed alongside the mood board
- Keep the mood board as a constant anchor across all generations

## Director's Playbook (Text-to-Video)

Tested vocabulary for controlling Sora's output. Use these terms with confidence — they've been verified.

### Camera Angles
All work reliably for text-to-video. Sora interprets loosely but distinctly.

| Prompt term | What you get | Best for |
|-------------|-------------|----------|
| "Aerial drone shot" | High-angle follow (~60°), NOT true bird's eye | Establishing shots, spectacle |
| "Extreme close-up" | Tight face/detail, beautiful bokeh, minimal motion | Emotional beats, texture |
| "Low angle looking up" | Camera on ground, subject towering, dramatic | Power shots, imposing feel |
| "Wide establishing shot" | Full environment, subject small in frame | Scene-setting, mood |
| "Over-the-shoulder" | Behind subject, shallow DOF, intimate framing | Following, voyeuristic |

**Note:** "Straight down / bird's eye / top-down" doesn't work — you'll get ~60° instead. True overhead needs stronger language.

### Camera Movement
Text-to-video supports some movement. Image-to-video does NOT (confirmed in earlier experiments).

| Prompt term | Actual movement? | Quality |
|-------------|-----------------|---------|
| "Slow dolly forward" | Barely perceptible | Beautiful scene but almost static |
| "Slow pan left to right" | YES — clear horizontal reveal | Reliable, new elements appear |
| "Camera slowly orbits" | NO — micro-shift at best | Becomes a static beauty shot |
| "Handheld shaky camera" | Subtle natural motion, NOT shaky | Sora smooths everything |
| "Steadicam tracking shot following [subject]" | YES — best actual movement | Spatial progression, most cinematic |

**Winner: "steadicam tracking shot following"** — the words "tracking" + "following" trigger real spatial movement.
**Avoid: "dolly" and "orbit"** — these produce nearly static results.
**Note: "pan" works but may override your style** — in testing it produced Pixar/3D instead of requested photorealism.

### Visual Styles — Anti-AI Tier
Styles that completely avoid the "AI-generated" look, ranked by convincingness:

| Style keyword | What you get | AI-looking? |
|---------------|-------------|------------|
| "Japanese woodblock print, ukiyo-e, flat perspective, bold outlines" | Authentic Hiroshige-quality print | NO — could frame it |
| "Loose watercolor painting, visible brushstrokes, paint bleeding" | Real paper texture, paint bleed | NO — hand-painted feel |
| "Stop motion, handcrafted felt and wool textures, miniature set" | Felt characters, tiny props, Wes Anderson palette | NO — genuinely handcrafted |
| "Claymation, visible fingerprints on clay, handmade miniature set" | Wallace & Gromit quality clay figures | NO — physical materials |
| "Charcoal sketch on cream paper, rough expressive strokes, smudged" | Actual charcoal texture, rough marks | NO — traditional medium |
| "16-bit pixel art, retro game aesthetic, dithering" | SNES RPG quality, dithered sky | NO — retro constraint hides AI |
| "VHS camcorder footage, tracking lines, dated color" | Authentic 90s home video degradation | NO — artifacts mask tells |
| "Kodak Portra 400 film, warm grain, overexposed highlights" | Warm film grain, soft highlights | NO — passes as real footage |
| "Anime, Studio Ghibli aesthetic, hand-drawn cel animation" | Clean linework, warm flat colors | Slightly — rain too 3D |
| "Anamorphic widescreen, blue and orange grade, lens flare" | Bokeh orbs, cinematic grade | Slightly — too clean/rendered |

**Top 3 for marketing:** Kodak Portra (authentic), stop motion/felt (charming + unique), VHS (nostalgic).

### Continuity Across Clips
Tested: 3 shots of same scene (wide, close-up, OTS) with identical style keywords.

**What Sora CAN match across generations:**
- Color palette and temperature (warm amber stayed consistent)
- Mood and lighting direction
- General setting (nighttime workspace, city bokeh)
- Film grain / style treatment

**What Sora CANNOT match:**
- Specific objects (laptop model, mug design changed per shot)
- Character appearance (clothing, hands, hair differ)
- Exact room layout / furniture position

**Continuity recipe:** Use identical style keywords + setting description + color words across all prompts. Cut between shots that FEEL the same but avoid showing the same specific object prominently in multiple clips. For character consistency, use Gemini mood board chaining (see Multi-Scene workflow above).

### Scene Comfort Zones
Sora produces its best work with:
- **Neon Tokyo night + rain** — maximum reflections, flicker, atmosphere
- **Warm cafe / workspace** — golden hour, steam, bokeh, cozy
- **Nature + soft light** — parks, windows, morning/sunset
- **Underwater** — turquoise water, light refraction, swimmer silhouettes (batch 9)
- **Server rooms / tech corridors** — blue LEDs, fog, symmetrical composition (batch 13)
- **Candle / single light source in darkness** — meditative, minimal, perfect ambient motion (batch 9)
- **Couch coding at night** — laptop + phone + cat + warm lamp = cozy developer scene (batch 11)
- **Rooftop at night + city bokeh** — silhouette with laptop, contemplative wide shots (batch 12)
- **Rain-streaked windows** — city bokeh through glass, laptop glow reflected (batch 12)
- **Abstract/macro** — ink in water, particles, light painting = pure motion, no scene understanding needed (batch 15)

Avoid: interiors with harsh fluorescent lighting, anything requiring readable text.

### Expanded Style Discoveries (Batches 9-15)

**New anti-AI styles confirmed:**
| Style keyword | What you get | AI-looking? |
|---------------|-------------|------------|
| "Paper craft, folded paper, cardboard, origami" | Physical-looking paper city with hanging clouds | NO — real model feel |
| "Knitted yarn landscape, button fruits, wool clouds" | Yoshi's Woolly World quality, joyful | NO — craft materials |
| "Pottery wheel, wet clay, kiln glow, 16mm film grain" | Tactile hands shaping clay, meditative | NO — earthy, real |
| "Stained glass window, light streaming, dust particles" | Cathedral light beams, rainbow on stone floor | NO — breathtaking |
| "Chalk drawing on blackboard, being drawn in real time" | Hand actually drawing with chalk, satisfying | NO — classroom feel |
| "Light painting photography, long exposure trails" | Neon trails forming shapes in pure darkness | NO — photographic |
| "Double exposure: [subject] overlaid with [background]" | Clean composite, neon accents, artistic | Slightly — but intentionally surreal |

**Composition techniques that work:**
- **Split screen** — CAN work for static compositions (four seasons tree, batch 9). Not "unsupported" as previously thought.
- **Double exposure** — Sora understands overlaying two scenes, especially face + cityscape
- **Extreme macro** — ink in water, candle flame, coffee pour. Sora excels at detail-level scenes.
- **Chalk/drawing animation** — "being drawn in real time" triggers actual drawing motion

**Text rendering update:**
- Sora rendered readable "DEPLOY" in neon signs (batch 11 felt character). Simple, short words in neon/sign context CAN work. Still unreliable for body text.

**Crowded scenes update:**
- Previously marked as "avoid." Marrakech market (batch 9) worked beautifully when combined with "16mm film grain" + "handheld." Film grain + handheld camera feel masks AI artifacts in busy scenes.

## Sora Behavior Model (Experimental Findings)

### The Core Insight
Sora is a **storyteller, not an animator**. Given an image + motion prompt, it doesn't move existing elements — it reinterprets the scene through the lens of the prompt. The more motion/narrative you request, the more it invents.

### Motion Intensity Scale (image-to-video)

| Level | Prompt Type | Scene Preservation | Quality |
|-------|-----------|-------------------|---------|
| 1 - Still | "subtle breathing, faint particles" | Near-perfect | Best |
| 2 - Ambient | "rain falls, lights flicker, ripples" | Very good, minor color shift | Great |
| 2.5 - Detail | "cape flutters, eyes blink, cloud rises" | Very good | Great |
| 2.5 - Micro cam | "barely perceptible zoom in" | Very good | Great |
| 3 - Camera | "zoom out", "pan", "dolly" | **Destroyed** — scene reimagined | Broken |
| 4 - Orbit | "camera orbits around" | **Destroyed** — new scene invented | Broken |
| 5 - Action | "character flies, intense action" | **Destroyed** — new narrative created | Broken but creative |

**Safe zone: Levels 1–2.5.** The boundary is not "ambient vs everything" — it's **"effect-like motion" vs "motion requiring 3D scene understanding."** Cape flutter, rain, micro zoom, object displacement (cloud rising) = effects that work. Camera orbit, walking, flying = requires 3D understanding = breaks.

Safe at the boundary:
- Heavy ambient (intense rain, rapid flicker, strong wind particles)
- Small character details (cape flutter, eye blink)
- Micro camera ("barely perceptible zoom in")
- Simple object displacement ("cloud slowly rises upward")
- Character actions are ignored but don't destroy (head turn → nothing happens)

### Speech Hallucination
**~95% speech hallucination rate** across 30+ videos. Every prompt-based approach to prevent it fails:
- "Silent, no speech, no dialogue" — ignored
- "Ambient sounds only, no voices, no speaking characters" — ignored
- Speech content is random noise: "Thank you", "See ya!", "Really, good work", "vídeo!", etc.
- **Exception: scenes without humanoid characters** (e.g. coffee shop, objects only) sometimes produce silent audio (1/30+ was silent — a text-to-video coffee shop scene)
- **Only fix: strip audio in post** (`ffmpeg -i input.mp4 -an -c:v copy output.mp4`)
- Or replace audio entirely with music/Kokoro TTS narration

### Stochasticity
Same image + same prompt produces **visually consistent** results across 5 runs. Composition, character position, and scene layout are highly deterministic. Only minor variations in neon sign text and color temperature. Speech content varies wildly (random noise).

### Duration Scaling
- **Min duration: 5s** (150 frames). Requesting 75 frames rounds up to 150.
- **Max duration: 15s** (450 frames). Requesting 600 frames caps at 450.
- **Duration does NOT increase drift** for ambient prompts. 5s, 10s, and 15s all hold perfectly.
- Gen time scales: 5s ≈ 40s, 10s ≈ 90s, 15s ≈ 3-5min.
- **10s (300 frames) = sweet spot** for marketing-length clips. Reliable, no quality loss.
- **15s (450 frames) = works but slower** — confirmed working (batch 10 aerial mountain lake, 14.5MB). Generation takes 3-5min. Poll script may timeout before it completes — use download.py to recover.

### Resolution & Orientation
API metadata reports half the actual resolution. Real values:

| Setting | Actual Resolution | File Size (5s) |
|---------|------------------|---------------|
| Portrait small | **704x1280** | ~3.6 MB |
| Portrait large | **1024x1792** | ~4.5 MB |
| Landscape small | **1280x704** | ~2.9 MB |
| Square | **Not supported** (400 error) | N/A |

- **Use portrait large for quality** — noticeably sharper, more detail in rain/neon
- Landscape crops/recomposes a portrait reference image — adds horizontal context
- All videos: h264 + aac audio track (even when "silent")

### Watermark
- `download_urls.no_watermark` always returns None
- Watermark ("Sora @solai") appears starting from mid-frames, not on first frame
- First frame ≈ reference image (no watermark)

### Prompt Wording Sensitivity
**Wording barely matters for ambient motion.** Tested 5 variations (plain, constrained, "looping ambient", negative framing, "cinemagraph") — all produced nearly identical results. The simplest prompt works just as well as complex ones. Don't overthink it.

### Text-to-Video vs Image-to-Video
Two fundamentally different modes:

| Aspect | Image-to-Video | Text-to-Video |
|--------|---------------|--------------|
| Style control | Exact (from reference) | Unreliable ("pixel art" may become 3D) |
| Camera movement | Destroys scene | Works well |
| Motion quality | Ambient only | Full cinematic |
| Speech hallucination | ~95% rate | Lower (0% on character-free scenes) |
| Best for | Animating existing art | Cinematic/atmospheric content |

**Strategy: Use image-to-video for style-exact ambient scenes. Use text-to-video for cinematic motion and character-free atmospheric content.**

### Art Style Performance (image-to-video)
All styles preserve scene with ambient prompts. But motion quality varies:

| Style | Motion Amount | Best For |
|-------|-------------|----------|
| **Pixel art / neon** | Most dramatic | Many light sources → lots to flicker |
| **Flat illustration** | Charming subtle | Eyes blink, warm light shifts |
| **Art nouveau / graphic** | Minimal | Fewer moving elements = less animation |

**Rule: styles with many light sources = better ambient animation.** Sora needs "things that can flicker."

### Prompt Template (Best Known)
```
[ambient motion description]. No new objects, preserve exact art style.
```
Note: speech constraints are useless — always strip audio. "No new objects, preserve exact art style" helps visual fidelity. Prompt complexity doesn't matter — keep it simple.

## Prompt Tips

**What works well (image-to-video):**
- **Ambient/atmospheric motion**: rain, particles, flicker, glow, breathing, ripples
- **Small character details**: cape flutter, eye blink
- **Micro camera**: "barely perceptible zoom in"
- **Simple object displacement**: "cloud slowly rises upward"
- Keep prompts short and simple — wording doesn't matter much
- Add "no new objects, preserve exact art style" for visual fidelity

**What breaks (image-to-video):**
- Full camera movement (pan, dolly, orbit) = scene reimagined
- Character locomotion (walk, fly, fight) = new narrative invented
- Complex multi-character interactions
- Rapid scene changes in a single generation
- Text or readable words (Sora struggles with text)
- Note: small character actions (head turn) are ignored but don't destroy

**For text-to-video:**
- Describe camera movement: "slow dolly forward", "aerial tracking shot"
- Specify lighting and mood: "golden hour", "neon-lit", "foggy morning"
- Be cinematic: "shallow depth of field", "anamorphic lens flare"
- Simple subjects with clear motion work best

## Before Running ANYTHING

**Pre-flight checklist — do ALL of these before every batch submission:**

### Step 1: Check for running processes
```bash
ps aux | grep -E "src/create.py|src/download.py" | grep -v grep
```
If anything is running, **STOP**. Wait for it to finish. Do not submit a new batch.

### Step 2: Download any missed videos
```bash
python3 .claude/skills/video/src/download.py
```
The CLI often times out but the script keeps running in the background. Videos generate server-side even after a timeout. Always recover missed downloads before starting new work.

### Step 3: Check recent output
```bash
ls -lt .claude/skills/video/data/raw/*.mp4 | head -10
```
Verify what you already have so you don't re-generate.

### Step 4: Write the batch JSON first
Create the experiment JSON file BEFORE running the script. This way the experiment is documented even if the CLI times out mid-run.

### Step 5: Submit
Only now run `create.py --batch`. The Sora jobs are submitted server-side the moment the script runs — even if the CLI connection drops, the videos will generate. Use `download.py` to recover them later if needed.

## Known Limitations

- **One browser at a time**: Playwright uses a persistent Chrome profile, so only one Sora process can run. Use batch mode for parallelism (Sora generates up to 5 simultaneously server-side).
- **Double-run danger**: If a batch command times out in the CLI, the Sora jobs were ALREADY submitted server-side and will generate. Always check `ls -lt output/raw/*.mp4 | head` for new files before re-running — you'll burn duplicate credits otherwise.
- **Audio (sy_8 model)**: `audio_transcript: {"text": "..."}` and `audio_caption: "string"` are accepted by the API but videos with audio never produce download URLs on the `sy_8` model. Audio appears broken — use Kokoro TTS locally instead for narration overlay.
- **Sora can't maintain consistency across clips**: Don't expect character/style consistency between separate Sora generations. Use Gemini for consistency (mood board chaining) and Sora only for animation.
- **100 credits/day**: ChatGPT Pro gives ~100 generations per day. Each batch job costs 1 credit per video. Credits reset on a rolling ~4.5hr timer, not midnight. **Last reset ETA: 2026-02-11 ~14:23** (update this after each rate limit hit).
- **Poll crash on HTML response**: The poll_until_done function sometimes gets an HTML page instead of JSON from the Sora API. Jobs are still submitted server-side — use download.py to recover. This is a known intermittent issue, not a show-stopper.
- **Browser lock file**: If Chrome crashes or times out, a `SingletonLock` file persists in `browser/`. Delete it before retrying: `rm -f .claude/skills/video/browser/SingletonLock`. Also kill zombie Chrome processes: `ps aux | grep "Chrome.*browser" | grep -v grep | awk '{print $2}' | xargs kill -9`.

## Setup (One-Time)

If the session expires, re-authenticate:
```bash
python3 .claude/skills/video/src/session.py login
# Opens Chrome, sign in to sora.chatgpt.com, type 'done'
```

## After Generating

1. The script prints the output path when done (MP4 file)
2. Present the full path to the user (renders as clickable file pill in iOS)
3. Generation takes 1-3 minutes depending on duration and size
4. Videos remaining count is printed after creation

## Output

Default output: `.claude/skills/video/data/raw/`

Files are named `sora_{timestamp}.mp4`.

## Data Layout

```
data/
├── raw/              # Individual Sora generations
├── compilations/     # Finished stitched videos
├── shorts/           # Short-form edits (daily-short v1/v2/v3)
├── experiments/      # JSON batch configs
├── characters/       # Pixel character designs + rosters
├── moodboards/       # Style reference collages
├── storyboards/      # Composite overview images
└── frames/           # Gemini reference frames for Sora input
```

**Where to save things:**
- Sora generates → `data/raw/` (automatic via scripts)
- Stitched/edited videos → `data/compilations/`
- Short-form content → `data/shorts/`
- Batch JSON configs → `data/experiments/`
- Gemini reference frames (input to Sora) → `data/frames/`
- Mood board images → `data/moodboards/`
- Storyboard composites → `data/storyboards/`
- Character designs → `data/characters/`
- Reviews are ephemeral — delete after evaluating. Do not accumulate review screenshots.

## Experiment Catalog

All batch JSONs live in `data/experiments/`.

| Batch | File | Theme | Videos | Top Picks |
|-------|------|-------|--------|-----------|
| 1 | batch1-camera-angles.json | Camera angles (aerial, close-up, low, wide, OTS) | 5/5 | All solid reference |
| 2 | batch2-camera-movement.json | Camera movement (pan, orbit, dolly, tracking, handheld) | 5/5 | Steadicam tracking = winner |
| 3 | batch3-visual-styles.json | Visual styles (Kodak Portra, woodblock, watercolor) | 3/3 | Woodblock print |
| 4 | batch4-non-ai-styles.json | Non-AI aesthetics (stop motion, claymation, pixel art, VHS) | 4/4 | Stop motion felt |
| 5 | batch5-continuity.json | Continuity across shots (same scene, different angles) | 4/4 | Mood matches, details don't |
| 6 | batch6-lighting.json | Lighting variations and atmospheric effects | 5/5 | All good |
| 7 | batch7-marketing-scenes.json | Marketing/commercial scenes | 5/5 | Cozy workspace |
| 8 | batch8-mixed-styles.json | Mixed creative combos (felt+neon, claymation, aerial ocean) | 5/5 | Felt neon city walk (#1) |
| 9 | batch9-untested.json | Risky/untested (market, timelapse, split screen, underwater, candle) | 5/5 | Split screen 4 seasons, underwater |
| 10 | batch10-longer-duration.json | 10s and 15s marketing clips (forest, felt city, aerial lake) | 3/3 | Aerial mountain lake 15s |
| 11 | batch11-cloude-marketing.json | Cloude-specific (couch+cat, city walk, desk, felt DEPLOY, bed phone) | 5/5 | Couch coder (#1), felt DEPLOY (#4) |
| 12 | batch12-night-mood.json | Night cinematics (rain window, subway, rooftop, noir, aurora) | 4/5 | Rooftop 3am (#3), rain window (#1) |
| 13 | batch13-tech-aesthetic.json | Tech/coding (code screen, server room, hologram, keyboard, glasses) | 5/5 | Server room (#2), code glasses (#5) |
| 14 | batch14-textures.json | Material textures (paper, pottery, knitted, stained glass, chalk) | 5/5 | Stained glass (#4), knitted world (#3) |
| 15 | batch15-abstract.json | Abstract (ink water, geometric, light painting, double exposure, particles) | 5/5 | Double exposure face+city (#4), particle brain (#5) |
| 16 | batch16-universe-creatures.json | Cloude Universe — pixel creatures in cloud city (aerial, home, market, sunset, tavern) | 5/5 | Sunset panorama, tavern meeting |
| 17 | batch17-universe-clouds.json | Cloude Universe — cloud squad (village, home, forest, stargazing, festival) | 5/5 | Magic forest, cloud festival |
| 18 | batch18-gaming-creatures.json | Cloude Gaming — creatures in retro games (pac-man, fighter, platformer, invaders, kart) | 4/5 | Kart racing, fighter game |
| 19 | batch19-gaming-clouds.json | Cloude Gaming — clouds in retro games (pac-man, tetris, pokemon, cooking, arcade) | 4/5 | Pokemon battle, cooking mama |

### Compilations
Stitched videos from batches 16-19:
- `compilations/01-universe-creatures.mp4` — 5 clips, 25s
- `compilations/02-universe-clouds.mp4` — 5 clips, 25s
- `compilations/03-gaming-creatures.mp4` — 4 clips, 20s ⭐ BEST
- `compilations/04-gaming-clouds.mp4` — 4 clips, 20s

### Character Roster
10 pixel art variants from ref-creature.png saved in `data/characters/`:
DJ, Hacker, Astronaut, Samurai, Scientist, Boxer, Skater, King, Rockstar, Pirate

### Mood Board → Frame → Sora Pipeline
Best pipeline discovered for character-consistent animated pixel art:
1. **Characters** — Gemini edit ref-creature.png into themed variants, BiRefNet bg removal
2. **Roster composite** — combine all characters into one image
3. **Mood board** — send roster to Gemini with "Director's mood board" prompt (9:16)
4. **First frames** — send mood board as --edit to Gemini with "Single scene, full frame, no panels:" prefix (16:9)
5. **Sora** — image-to-video with ambient-only motion prompts
6. **Stitch** — ffmpeg concat, no re-encode

Full recipe documented in `plans/20_active/gaming-creatures-video.md`.

### Best Marketing Clips (Cloude-specific)
1. **Couch coder** — batch 11 #1: laptop + phone + sleeping orange cat + warm lamp
2. **Felt DEPLOY** — batch 11 #4: knitted character at tiny desk, neon DEPLOY sign
3. **Felt city walker** — batch 10 #2: felt character with laptop in backpack, neon miniature city
4. **Rooftop 3am** — batch 12 #3: silhouette with laptop, city bokeh, blue hour
5. **Code in glasses** — batch 13 #5: green terminal reflected in round glasses, hoodie
6. **Rain window Berlin** — batch 12 #1: Fernsehturm through rain, laptop glow in glass
7. **Particle brain** — batch 15 #5: bioluminescent brain forming from particles in darkness
