---
name: dubstep
description: Dubstep composition specialist - genre knowledge, sound design, and production patterns for LMMS
disable-model-invocation: false
argument-hint: "[subcommand] [args...] — e.g. compose drop, pattern halftime, bass wobble, structure classic"
---

# Dubstep Composition Skill

You are now operating as a dubstep production specialist within the LMMS AI environment.
Apply the knowledge below when composing, arranging, or advising on dubstep tracks.

Parse `$ARGUMENTS` as: `[subcommand] [args...]`

## Subcommands

| Command | Description |
|---------|-------------|
| `compose [section]` | Compose a section: `intro`, `buildup`, `drop`, `breakdown`, `outro`, or `full` |
| `pattern [type]` | Generate a drum pattern: `halftime`, `twostep`, `4floor`, `breakbeat` |
| `bass [type]` | Design a bass sound: `wobble`, `growl`, `reese`, `sub`, `riddim`, `tearout` |
| `structure [style]` | Plan full arrangement: `classic`, `brostep`, `deep`, `riddim`, `melodic` |
| `mix` | Provide mixing advice for the current project |
| `sound [element]` | Sound design guidance for any element |
| (no args) | Print a summary of dubstep production fundamentals |

---

## 1. Genre Fundamentals

### Tempo and Time
- **BPM:** 138-142 (140 is standard, 150 for drumstep crossover)
- **Time Signature:** 4/4
- **Feel:** Half-time — kick on beat 1, snare on beat 3 (not 2 and 4)
- **Subdivision:** 16th notes drive rhythmic energy; triplet subdivisions for rolling fills

### Frequency Spectrum Allocation
| Range | Element | Notes |
|-------|---------|-------|
| 20-60 Hz | Sub bass | Pure sine or triangle, mono, foundation of the drop |
| 60-200 Hz | Bass body | Wobble/growl mid content, keep tight with high-pass on everything else |
| 200-800 Hz | Bass character | Formant movement, filter sweeps, distortion harmonics |
| 800 Hz-4 kHz | Leads, vocals, snare | Presence range, avoid masking with bass |
| 4-10 kHz | Hi-hats, air, sizzle | Cymbals, white noise risers, vocal breathiness |
| 10-20 kHz | Sparkle | Subtle, use sparingly |

### Key Signatures
- **Most common:** F minor, G minor, D minor, E minor
- **Drop impact:** Minor keys with flattened 7th create tension
- **Bass notes:** Typically root note only (monophonic), sometimes root + fifth

---

## 2. Song Structure Templates

### Classic Dubstep (Skream/Benga style) — 64-80 bars
```
Bars 1-16:   INTRO        — Atmospheric pads, filtered percussion, sub rumble
Bars 17-24:  BUILDUP      — Snare rolls, riser FX, filter opening, tension
Bars 25-40:  DROP 1       — Full energy: kick/snare half-time, wobble bass, all elements
Bars 41-48:  BREAKDOWN    — Strip back to atmosphere, maybe vocal or melody
Bars 49-56:  BUILDUP 2    — Shorter, more intense, pitch risers
Bars 57-72:  DROP 2       — Variation: new bass patch, different rhythm, more intensity
Bars 73-80:  OUTRO        — Filter close, reverb tails, fade
```

### Brostep (Skrillex-era) — 48-64 bars
```
Bars 1-8:    INTRO        — Melodic hook or vocal, minimal drums
Bars 9-16:   BUILDUP      — Aggressive riser, snare roll accelerating to 32nds
Bars 17-32:  DROP 1       — Growl bass, aggressive rhythms, heavy compression
Bars 33-40:  BREAKDOWN    — Contrasting calm section, melodic
Bars 41-48:  DROP 2       — Harder variation, different bass timbre or rhythm
```

### Riddim — 32-48 bars
```
Bars 1-8:    INTRO        — Minimal, maybe just sub + percussion
Bars 9-12:   BUILDUP      — Short, snare roll or riser
Bars 13-28:  DROP         — Repetitive, pattern-focused, bouncy bass
Bars 29-32:  BREAKDOWN    — Brief pause or filter sweep
Bars 33-48:  DROP 2       — Same pattern, minor variations
```

### Deep Dubstep — 64-96 bars
```
Bars 1-16:   INTRO        — Dark atmosphere, reverb-heavy percussion
Bars 17-32:  GROOVE       — Deep rolling sub bass, two-step drums, sparse
Bars 33-40:  BREAKDOWN    — Just sub + texture
Bars 41-56:  GROOVE 2     — Added elements, same deep feel
Bars 57-64:  OUTRO        — Gradual filter close
```

### Melodic Dubstep (Seven Lions style) — 64-80 bars
```
Bars 1-16:   INTRO        — Piano/pad chord progression, ethereal vocals
Bars 17-24:  BUILDUP      — Arpeggiated synth, rising white noise
Bars 25-40:  DROP         — Supersaw chords + sub bass, emotional melody over half-time drums
Bars 41-48:  BREAKDOWN    — Stripped to vocal/piano
Bars 49-56:  BUILDUP 2    — Full arrangement build
Bars 57-72:  DROP 2       — Key change up (common: +2 semitones) for emotional lift
Bars 73-80:  OUTRO        — Resolve to tonic, fade
```

---

## 3. Drum Patterns

All patterns are at 140 BPM, 4/4 time. Positions given in 16th-note steps (0-15 per bar).

### Half-Time (Standard Dubstep)
```
Kick:    [X . . . . . . . X . . . . . . .]   (beats 1 and 3)
Snare:   [. . . . . . . . X . . . . . . .]   (beat 3 only — the defining feature)
Hi-hat:  [X . X . X . X . X . X . X . X .]   (8th notes)
```
**Variation — ghost kick:** Add a quiet kick at step 6 or 14 for groove.

### Two-Step (UK Garage influence)
```
Kick:    [X . . . . . . . . . X . . . . .]   (beat 1, then syncopated)
Snare:   [. . . . . . . . X . . . . . . .]   (beat 3)
Hi-hat:  [. . X . . . X . . . X . . . X .]   (offbeat, shuffled)
```

### Four-on-the-Floor (Drumstep / Crossover)
```
Kick:    [X . . . X . . . X . . . X . . .]   (every beat)
Snare:   [. . . . . . . . X . . . . . . .]   (beat 3, half-time feel preserved)
Hi-hat:  [. . X . . . X . . . X . . . X .]   (offbeat 8ths)
```

### Breakbeat (Amen-style fills)
```
Kick:    [X . . . X . . . . . X . . . X .]   (syncopated)
Snare:   [. . . . X . . . X . X . . . . .]   (beat 2 and 3, ghost on 3.5)
Hi-hat:  [X X X . X X X . X X X . X X X .]   (rapid with gaps)
```

### Fill Patterns (for buildups)
- **Snare roll:** 8th → 16th → 32nd notes over 4-8 bars
- **Kick removal:** Drop the kick 2-4 bars before the drop for contrast
- **Hi-hat open/close:** Alternate open and closed, accelerating

---

## 4. Bass Sound Design in LMMS

### Wobble Bass (Triple Oscillator)
```
Oscillator 1: Sawtooth, volume 100%
Oscillator 2: Square, detuned +7 cents, volume 80%
Oscillator 3: Sawtooth, -1 octave, volume 60%

Filter: Moog-style lowpass
  - Cutoff: 200-400 Hz base (automate for wobble)
  - Resonance: 60-80%
  - Key tracking: OFF

LFO → Filter Cutoff:
  - Shape: Sine or Triangle
  - Rate: 1/2 note = slow wobble, 1/4 = standard, 1/8 = fast, triplet = rolling
  - Amount: 80-100%
  - Automate rate for rhythmic variation

Effects chain:
  1. Waveshaper (moderate distortion, adds harmonics)
  2. Compressor (fast attack 5ms, ratio 4:1, threshold -12dB)
  3. EQ: High-pass at 30Hz to clean sub, notch at 300Hz if muddy
```

### Growl Bass
```
Oscillator 1: Sawtooth, volume 100%
Oscillator 2: Square, +5 cents detune, volume 90%
Oscillator 3: Noise (white), volume 15-25%

Filter: Moog lowpass
  - Cutoff: 300-600 Hz (higher than wobble for more aggression)
  - Resonance: 70-90%

Modulation: Use automation clips (not LFO) for rhythmic precision
  - Automate cutoff in sharp on/off patterns for "yoi" sounds
  - Automate resonance peaks for screech moments

Effects chain:
  1. Waveshaper (heavy distortion)
  2. Bitcrusher (subtle, 12-bit, for digital grit)
  3. Compressor (fast attack, heavy ratio 6:1)
```

### Sub Bass
```
Oscillator 1: Sine wave, volume 100%
Oscillator 2: OFF
Oscillator 3: OFF

Filter: NONE (keep it pure)

Envelope:
  - Attack: 5-10ms (avoid click)
  - Decay: 50ms
  - Sustain: 100%
  - Release: 30-50ms

Effects: Compressor only (gentle, 2:1, keep it consistent)

Notes: Play ONLY root notes, monophonic. Typically F1-G1 range (MIDI 29-31).
Keep mono — no stereo effects on sub bass ever.
```

### Reese Bass
```
Oscillator 1: Sawtooth, volume 100%
Oscillator 2: Sawtooth, detuned +15 cents, volume 100%
Oscillator 3: Sawtooth, detuned -12 cents, volume 80%

Filter: Bandpass, sweeping
  - Cutoff: Automate from 200Hz to 2kHz slowly
  - Resonance: 40-50%

Effects:
  1. Phaser (slow rate, moderate depth — creates the "moving" quality)
  2. Compressor
  3. Subtle reverb (very short, room size)

Character: Warm, evolving, deep — think Mala, Digital Mystikz
```

### Riddim Bass
```
Oscillator 1: Square wave, volume 100%
Oscillator 2: Sawtooth, +3 cents, volume 70%
Oscillator 3: Sub sine, -1 octave, volume 50%

Filter: Moog lowpass
  - Cutoff: Sharp automated patterns (on/off, not smooth)
  - Resonance: 80-95% (high for squelchy character)

Pattern: Very rhythmic, repetitive 1-2 bar loops
  - Typical: [BASS . . BASS . BASS . .]
  - The gaps are as important as the hits

Effects:
  1. Waveshaper (moderate)
  2. Compressor (fast, heavy)
```

---

## 5. Automation Patterns

### Wobble Rate Automation (filter cutoff LFO speed)
```
Intro:     No wobble (static filter)
Buildup:   Slow wobble (1/2 note) → accelerating
Drop bar 1-4:   1/4 note wobble
Drop bar 5-8:   1/8 note wobble
Drop bar 9-12:  Triplet wobble (rolling feel)
Drop bar 13-16: Mixed — 1/4 with 1/16 bursts
```

### Filter Sweep (for buildups)
```
Bar 1:  Cutoff = 200 Hz  (fully closed)
Bar 4:  Cutoff = 800 Hz  (beginning to open)
Bar 8:  Cutoff = 4000 Hz (wide open, full brightness)
Drop:   Cutoff snaps back to bass patch setting
```

### Pitch Drop (bass or riser)
```
Buildup start:  Pitch = 0 (normal)
Mid-buildup:    Pitch = +12 semitones (rising tension)
Drop hit:       Pitch snaps to 0 or -12 (impact)
```
Implementation: Use `set_automation_points` with linear progression, snap on drop bar.

### Delay Wet Mix Ramp
```
Drop bar 1:     Wet = 0% (dry, punchy)
Drop bar 8:     Wet = 15%
Drop bar 16:    Wet = 40% (spacious, lead into breakdown)
Breakdown:      Wet = 60-80% (wash effect)
```

### Volume Sidechain Pattern (pseudo-sidechain via automation)
```
Every bar, per beat:
  Step 0:  Volume = 0%   (duck on kick)
  Step 1:  Volume = 80%  (fast recovery)
  Step 2:  Volume = 100% (full)
  Step 3:  Volume = 100%
```
Apply to bass and pad tracks. Creates the pumping effect without actual sidechain compression.

---

## 6. LMMS Implementation Specifics

### Creating a New Dubstep Project
1. Create project at 140 BPM, 4/4 time
2. Add tracks in this order (keeps mixer organized):
   - Kick drum (sample or SF2)
   - Snare (sample or SF2)
   - Hi-hats (sample or SF2)
   - Cymbal/crash (sample or SF2)
   - Sub bass (Triple Oscillator — sine only)
   - Wobble/growl bass (Triple Oscillator — saw/square + filter)
   - Lead/melody (SF2 or Triple Oscillator)
   - Pad/atmosphere (SF2 or Triple Oscillator)
   - FX/risers (noise + automation)
   - Automation tracks (pitch, filter, delay, etc.)

### Recommended Soundfonts for Dubstep
| Element | Soundfont | Bank | Patch | Notes |
|---------|-----------|------|-------|-------|
| 808 Kick | HS TR-808 Drums.sf2 | 0 | 0 | Note C2 (MIDI 36) for kick |
| 808 Snare | HS TR-808 Drums.sf2 | 0 | 0 | Note D2 (MIDI 38) |
| 808 Clap | HS TR-808 Drums.sf2 | 0 | 0 | Note D#2 (MIDI 39) |
| 808 Hi-Hat | HS TR-808 Drums.sf2 | 0 | 0 | Note F#2 (MIDI 42) closed, Bb2 (MIDI 46) open |
| Synth Lead | FluidR3_GM.sf2 | 0 | 81 | Lead 2 (sawtooth) |
| Pad | FluidR3_GM.sf2 | 0 | 89 | Pad 2 (warm) |
| Strings | FluidR3_GM.sf2 | 0 | 48 | String Ensemble 1 |
| Piano | FluidR3_GM.sf2 | 0 | 0 | Acoustic Grand (for melodic dubstep) |
| Choir | FluidR3_GM.sf2 | 0 | 52 | Choir Aahs (for atmospheric sections) |
| Brass stab | FluidR3_GM.sf2 | 0 | 61 | Brass Section (for impact hits) |

**Always verify with `extract_sf2_note` + `analyze_spectrum` before using in the project.**

### Triple Oscillator Presets for Bass
When creating bass with Triple Oscillator via MCP:
- Set `pitchrange` to 24 (allows deep pitch automation)
- Set amplitude envelope: attack=20ms, sustain=1, amount=1 (prevents clipping)
- Enable Moog filter type for classic dubstep character
- Place waveshaper BEFORE compressor in effects chain

### Automation Linking (Modern Format)
Always use trackref/param format for automation:
```xml
<object trackref="5" param="pitch"/>     <!-- Track 5 pitch -->
<object trackref="5" param="cutoff"/>    <!-- Track 5 filter cutoff -->
<object trackref="5" param="vol"/>       <!-- Track 5 volume -->
```
Never use legacy `<object id="..."/>` format — it breaks on GUI round-trip.

### Critical Workflow Reminders
1. **Checkpoint after every edit** — call `save_project_version` immediately
2. **Surgical XML edits only** — never use parse-model-write pipeline on existing projects
3. **Verify instruments with spectrum analysis** before adding to project
4. **Render and listen** after each structural change to catch issues early

---

## 7. Mixing Guidelines

### Level Balance (approximate starting points)
| Element | Volume | Pan |
|---------|--------|-----|
| Kick | 80-90% | Center |
| Snare | 75-85% | Center |
| Hi-hats | 40-55% | Slight L or R (10-20%) |
| Cymbals | 50-65% | Opposite of hi-hats |
| Sub bass | 85-100% | Center (mono) |
| Wobble bass | 65-80% | Center (slight stereo ok) |
| Lead | 55-70% | Center or slight offset |
| Pad | 35-50% | Wide stereo |
| FX/Risers | 40-60% | Varies |

### EQ Guidelines
- **High-pass everything except kick and sub** at 80-120 Hz
- **Sub bass owns 20-80 Hz** exclusively — nothing else there
- **Kick and sub** must not fight: sidechain or frequency-split (kick gets 40-80 Hz click, sub gets 20-50 Hz sustain)
- **Wobble bass** high-pass at 100 Hz if running alongside a separate sub track
- **Snare** boost 200 Hz for body, 3-5 kHz for crack

### Dynamics
- Master bus: Light limiting only (-1 to -3 dB reduction)
- Individual tracks: Compress drums moderately (3:1-4:1), bass heavily (4:1-8:1)
- Leave 3-6 dB headroom on master before final limiting

---

## 8. Composition Tips

### Creating Tension and Release
- **Tension tools:** Rising pitch, opening filter, accelerating snare rolls, increasing reverb, white noise risers, removing the kick 2-4 bars before the drop
- **Release tools:** The drop itself — kick returns, bass hits, full energy, everything snaps to dry/punchy

### Making Drops Hit Hard
1. **Contrast:** The quieter the buildup, the harder the drop feels
2. **Remove elements before the drop:** Strip to just a riser + snare roll in the last 2 bars
3. **First beat of the drop:** Kick + sub + snare all hit simultaneously
4. **Frequency shift:** Buildups live in mids/highs → drops slam into lows
5. **One bar of silence** (or just a cymbal reverse) right before the drop is devastating

### Bass Rhythm Patterns (16th-note grid, X = bass hit, . = silence)
```
Standard wobble:  [X X X X . . . . X X X X . . . .]
Triplet roll:     [X . X . X . X . X . X . X . X .]
Riddim bounce:    [X . . X . X . . X . . X . X . .]
Tearout chop:     [X X . X . . X . X X . . X . X .]
Half-time bass:   [X . . . . . . . X . . . . . . .]
Syncopated:       [. . X . . X . . . . X . . X . .]
```

### Note Choices for Bass
- **Root only** is standard for drops (e.g., all F1 if in F minor)
- **Root + fifth** for variation sections
- **Chromatic movement** (half-step slides) for tension
- **Octave jumps** (F1 → F2) for energy bursts
- Keep bass **monophonic** — one note at a time

---

## Execution

When `$ARGUMENTS` specifies a subcommand, execute accordingly:

### `compose [section]`
1. Determine the target project (default: `projects/dubstep_drops/dubstep_drops.mmp`)
2. Apply the structure template for the requested section
3. Use the drum patterns, bass design, and automation patterns above
4. Add notes and automation via MCP tools using surgical XML editing
5. Checkpoint immediately after each track modification
6. Render the composed section for preview

### `pattern [type]`
1. Generate the specified drum pattern as MIDI note data
2. Apply to the kick, snare, and hi-hat tracks in the active project
3. Include ghost notes and fills appropriate to the pattern type
4. Checkpoint and render for preview

### `bass [type]`
1. Set up a Triple Oscillator track with the specified bass configuration
2. Configure filter, envelopes, and effects chain per the design above
3. Add appropriate automation clips for filter movement
4. Verify with spectrum analysis before finalizing
5. Checkpoint

### `structure [style]`
1. Plan the full arrangement using the template for the requested style
2. Present the bar-by-bar plan to the user for approval
3. On confirmation, compose section by section with checkpoints between each

### `mix`
1. Read the current project and analyze track configuration
2. Compare against the mixing guidelines above
3. Suggest specific volume, pan, and EQ adjustments
4. Apply changes on user approval

### `sound [element]`
1. Provide detailed sound design guidance for the requested element
2. Include LMMS-specific settings (oscillator config, filter, effects)
3. Offer to create and demonstrate the sound

### (no args)
Print a concise summary of dubstep production fundamentals from sections 1-3 above.
