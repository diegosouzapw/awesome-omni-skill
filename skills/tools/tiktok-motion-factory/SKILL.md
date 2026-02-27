---
name: tiktok-motion-factory
description: Run the deterministic, non-AI Remotion TikTok motion graphics pipeline on a local audio file and produce a creative-direction spec.
---

# TikTok Motion Factory Skill (Repo Root)

This repository is the skill directory. Invoking **$tiktok-motion-factory** must behave like a creative director + motion designer while keeping the pipeline **deterministic**. You may use AI for **audio assessment** and **web inspiration research**, but the **final rendered graphics must be non-AI** and strictly procedural.

## Hard Constraints (Must Repeat and Enforce)

- **AI usage is limited.** You may use AI for **audio assessment** and **web inspiration research** only. The final product must not contain AI-generated graphics; visuals must be procedural and deterministic.
- **Audio must be local.** If a URL is provided, refuse and ask for a local file path.
- **Only 1–2 optional preference questions** (vibe: `editorial` vs `club`; text `on/off`). If the user doesn’t answer, choose defaults.
- **Tailored concept every time.** The concept must reference measured audio features (tempo/energy/onset density/dynamics) and map them to design choices.

## Agent Protocol (Step-by-Step)

1) **Collect input (minimal questions):**
   - Ask for a local audio file path. If missing, state you will use the newest file in `input/`.
   - Ask at most two optional preference questions:
     - Vibe: `editorial` or `club` (default: `editorial`).
     - Text: `on` or `off` (default: `on`).
   - If the user provides a URL, **refuse** and ask for a local path.

2) **Prepare audio (deterministic):**
   - Run: `tools/prep_audio.sh` on the chosen audio file to create a normalized WAV for analysis and rendering.

3) **Analyze cues (deterministic):**
   - Run: `tools/analyze_cues.py` to generate `jobs/<job>/cues.json`.
   - Compute additional stats from cues (see “Audio Analysis Requirements”).

4) **Synthesize a creative concept:**
   - Use the “Creative Intelligence Protocol” and “Concept Synthesis” rules below to produce a tailored concept tied to audio metrics.

5) **Write the creative spec:**
   - Save a machine-readable creative spec to `jobs/<job>/creative.json` using the schema below.

6) **Generate props:**
   - Create `jobs/<job>/props.json` that includes the creative spec **verbatim** under `creative`.

7) **Execute render:**
   - Run `./run.sh` with the audio and any user parameters (start/dur/title/slug) **plus** the creative spec inclusion.

8) **Report output:**
   - Print the final render path and job folder path.
   - Suggest next steps (upload, add caption, add hashtags).

## Creative Intelligence Protocol (Mandatory)

You must not skip concept work. Complete **all** checklist items before rendering.

**Checklist (all required):**
- [ ] Compute audio metrics (density, avg strength, peaks, quiet sections, estimated BPM, section boundaries).
- [ ] Define scene structure (intro/build/drop/outro) aligned to energy and cues.
- [ ] Map cue types/tiers to motion responses.
- [ ] Define art direction (palette, contrast, background treatment, geometric primitives, motion language).
- [ ] Define typography system (font choice strategy, hierarchy, grid, kinetic behavior).
- [ ] Define transitions and loop strategy.
- [ ] Write `jobs/<job>/creative.json` (schema below).
- [ ] Ensure `jobs/<job>/props.json` embeds `creative.json`.
- [ ] Ensure `jobs/<job>/notes.md` includes commands, rationale, and audio metrics.

## Audio Analysis Requirements (Deterministic)

You must run:
- `tools/prep_audio.sh`
- `tools/analyze_cues.py`

Then compute the following stats from `cues.json`:
- **cueDensity**: cues per second.
- **avgStrength**: mean of cue strengths.
- **peakStrength**: max cue strength.
- **quietSections**: contiguous ranges where no cues appear for ≥ 0.6s (or ≥ 18 frames @30fps).
- **estimatedBpm**: rough BPM from median inter-onset interval (IOI), mapped to BPM in [60, 180].
- **sectionBoundaries**: infer segments by energy trend using a rolling window (e.g., 1.5s) and label as `intro`, `build`, `drop`, `outro` by relative energy and cue density.

**Deterministic design rules (examples, must use):**
- If `cueDensity > 2.2` cues/sec → use **micro-shake** on impacts + **higher contrast** palette.
- If `avgStrength < 0.35` → reduce impact scale to `1.05` max and favor smoother easing.
- If `peakStrength > 0.8` → allow a single **flash** accent on strongest hit in each section.
- If `estimatedBpm >= 130` → increase motion speed (`motionTempo = fast`) and tighter grid spacing.
- If `quietSections` exist → add **breathing** background drift during quiet ranges.

## Concept Synthesis (Required Outputs)

You must create the following and include it in `creative.json`:

1) **Narrative Concept (1 paragraph)**
   - Tie the concept directly to measured audio features (tempo, density, dynamics).

2) **Motion Grammar (bullets)**
   - **Primitives**: circles, lines, ribbons, soft rects, dot field, etc.
   - **Behaviors**: scale pop on cues, rotation tick, parallax drift, micro-shake, flash.

3) **Palette (3–6 hex colors)**
   - Justify palette based on audio features (e.g., high BPM → higher contrast; low density → muted neutrals).

4) **Typography System**
   - Use local/system fonts (e.g., `Inter`, `Helvetica`, `Arial`, `SF Pro`, `Roboto`).
   - Define hierarchy (title/subtitle), grid rules, and kinetic behavior tied to cues.

5) **Scene Plan (exact frame ranges)**
   - Provide `startFrame`/`endFrame` for each scene (intro/build/drop/outro).
   - Align boundaries to section boundaries derived from cues.

6) **Loop Plan**
   - Define how the last frames return to the first (e.g., gradient phase, shape positions, opacity fade).

## Files to Generate

### `jobs/<job>/creative.json`
**Required schema (keys must exist):**
```json
{
  "audio": {
    "source": "<path>",
    "durationSec": 10,
    "fps": 30,
    "metrics": {
      "cueDensity": 1.8,
      "avgStrength": 0.42,
      "peakStrength": 0.91,
      "estimatedBpm": 124,
      "quietSections": [{"startSec": 3.2, "endSec": 4.0}],
      "sectionBoundaries": [
        {"name": "intro", "startSec": 0, "endSec": 2.5},
        {"name": "build", "startSec": 2.5, "endSec": 5.0},
        {"name": "drop", "startSec": 5.0, "endSec": 8.0},
        {"name": "outro", "startSec": 8.0, "endSec": 10.0}
      ]
    }
  },
  "concept": {
    "narrative": "<1 paragraph>",
    "motionGrammar": {
      "primitives": ["circles", "ribbons", "soft-rects"],
      "behaviors": ["scale-pop", "rotation-tick", "micro-shake", "flash"]
    },
    "palette": ["#0B0F1A", "#1E2A44", "#F0E9D2", "#FF5C7C"],
    "typography": {
      "fonts": ["Inter", "Helvetica", "Arial"],
      "hierarchy": {"titleSize": 110, "subtitleSize": 48},
      "grid": {"columns": 6, "margin": 80},
      "kinetic": {"onCue": "scale-pop", "idle": "slow-drift"}
    },
    "scenes": [
      {"name": "intro", "startFrame": 0, "endFrame": 75, "energy": "low"},
      {"name": "build", "startFrame": 75, "endFrame": 150, "energy": "mid"},
      {"name": "drop", "startFrame": 150, "endFrame": 240, "energy": "high"},
      {"name": "outro", "startFrame": 240, "endFrame": 300, "energy": "low"}
    ],
    "transitions": ["smear", "crossfade"],
    "loop": {"strategy": "phase-align", "lastFrames": 15}
  },
  "cueMap": [
    {"frame": 12, "strength": 0.71, "type": "kick"},
    {"frame": 24, "strength": 0.36, "type": "hat"}
  ]
}
```

### `jobs/<job>/props.json`
Must embed the creative spec exactly:
```json
{
  "audioSrc": "../assets/audio/<job>.wav",
  "fps": 30,
  "width": 1080,
  "height": 1920,
  "durationInFrames": 300,
  "cues": ["..."],
  "text": {"title": "...", "subtitle": "..."},
  "style": {"vibe": "editorial", "accent": "#FF5C7C"},
  "creative": {"...full creative.json content..."}
}
```

### `jobs/<job>/notes.md`
Must include:
- Exact commands executed.
- Key creative rationale (1–3 bullets).
- Derived audio metrics (explicit numbers).

## Execution (Run the Pipeline)

1) Use the computed creative spec to generate `creative.json` and embed in `props.json`.
2) Run `./run.sh` with user inputs (audio/start/dur/title/slug).
3) Print:
   - `DONE: <path>` (render output)
   - `JOB: <path>` (job folder)

## Remotion Template Support Requirement

If the current Remotion template does not yet support `creative.json` fields, you **must** implement support in `remotion/src`:
- Parse `creative` from props.
- Apply palette, background style, primitives, and cue-driven animations deterministically according to the creative spec.
- Do **not** add any AI dependencies or AI-driven logic.
