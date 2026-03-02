---
name: arch-v
description: Video production workflow orchestrator for Veo 3. Guides users through creating professional video prompts via two paths - direct text-to-video OR image-to-video pipeline (Imagen 3/4 → Veo 3). Validates prompt completeness, checks conflicts, ensures all mandatory components present. Integrates camera-movements, great-prompt-anatomy, short-prompt-guide, long-prompt-guide, and imagine skills.
---

# ARCH-V: Video Production Orchestrator

Professional video prompt creation for Veo 3 with two production paths.

## Overview

ARCH-V guides you through creating production-ready video prompts by:
1. Determining optimal workflow path
2. Loading appropriate reference skills
3. Validating mandatory components
4. Checking for conflicts
5. Delivering final validated prompts

## Two Production Paths

### Path 1: Text-to-Video (Direct Veo 3)
**When to use:** You have clear vision for entire video including motion, audio, and can describe it in text.

**Output:** Single Veo 3 prompt ready for text-to-video generation

**Skills used:**
- great-prompt-anatomy (8 mandatory components)
- camera-movements (standardized vocabulary)
- short-prompt-guide OR long-prompt-guide (based on complexity)

### Path 2: Image-to-Video (Imagen → Veo 3)
**When to use:** You want precise control over starting visual, or have specific still image composition in mind before adding motion.

**Output:** Two prompts
1. Imagen 3/4 prompt for still image generation
2. Veo 3 image-to-video prompt for animation/motion

**Skills used:**
- **Stage A (Imagen):** imagine skill + great-prompt-anatomy (visual components)
- **Stage B (Veo 3):** camera-movements + great-prompt-anatomy (motion components)

## Workflow Process

### Stage 0: Path Determination

**ARCH-V asks:**
```
Which production path?

1. Text-to-Video (Direct Veo 3)
   - Describe entire video in single prompt
   - Faster workflow
   - Less control over initial composition

2. Image-to-Video (Imagen → Veo 3 pipeline)
   - Create perfect still image first
   - Then add motion and animation
   - Maximum control over visual composition
   - Two-step process
```

**User chooses path** → ARCH-V loads appropriate skills and guides accordingly

---

## Path 1 Workflow: Text-to-Video

### Stage 1: Prompt Type Determination

**ARCH-V asks:**
```
What type of video prompt?

SHORT PROMPT (for):
- Filler shots, B-roll
- Atmospheric scenes
- Quick establishing shots
- <3 sentences to describe

LONG PROMPT (for):
- Dialogue scenes
- Character continuity
- Multi-beat sequences (>3 beats)
- Complex choreography
```

**User chooses** → ARCH-V loads [short-prompt-guide](short-prompt-guide) OR [long-prompt-guide](long-prompt-guide)

### Stage 2: Mandatory Components Check

ARCH-V validates all 8 components from [great-prompt-anatomy](great-prompt-anatomy):

**Checklist:**
- [ ] 1. Subject (who/what in shot)
- [ ] 2. Setting (where/when)
- [ ] 3. Action (what's happening)
- [ ] 4. Style/Genre (aesthetic)
- [ ] 5. Camera/Composition (shot size, angle, movement)
- [ ] 6. Lighting/Mood (light sources, emotional tone)
- [ ] 7. Audio (dialogue, ambience, music)
- [ ] 8. Constraints (prohibitions, exact requirements)

**If camera movement mentioned:** Load [camera-movements](camera-movements) for vocabulary validation

### Stage 3: Validation & Conflict Checking

ARCH-V checks for:

**Time/Weather Conflicts:**
- ❌ "Golden hour" with "midnight"
- ❌ "Harsh noon sun" with "soft evening light"
- ✅ Consistent time of day throughout

**Camera Movement Conflicts:**
- ❌ "Dolly in while arc left" (multiple movements per beat)
- ✅ One movement per beat from standardized vocabulary

**Spatial Coherence (if long prompt):**
- ✅ FG/MG/BG structure defined
- ✅ Color anchors consistent (3-5 colors repeated)
- ✅ Continuity rules explicit

### Stage 4: Output

**If all valid:** 
```
✅ PROMPT READY

[Final Veo 3 text-to-video prompt displayed]

Ready to use in Veo 3!
```

**If validation fails:**
```
⚠️ PROMPT-LOCKED

Missing/Conflicting:
- [specific issue 1]
- [specific issue 2]

Suggested fixes:
- [actionable fix 1]
- [actionable fix 2]

Would you like me to help resolve these?
```

---

## Path 2 Workflow: Image-to-Video

### Stage A: Imagen Prompt Creation

**ARCH-V loads:** [imagine](imagine) skill for Imagen 3/4 structure

**Focus on VISUAL components from [great-prompt-anatomy](great-prompt-anatomy):**

**Stage A Checklist:**
- [ ] 1. Subject (detailed visual description)
- [ ] 2. Setting (environment, placement)
- [ ] 4. Style/Genre (photographic or artistic style)
- [ ] 5. Camera/Composition (shot size, angle - STATIC, no movement)
- [ ] 6. Lighting/Mood (light sources, color palette)
- [ ] 8. Constraints (visual prohibitions)

**NOT included in Stage A:**
- ❌ Action (no motion in still image)
- ❌ Audio (images have no sound)
- ❌ Camera movements (static composition)

**Imagine-specific guidance:**
- Subject-Context-Style framework
- Technical photography specs (lens, lighting quality)
- Art style references (Science SARU by default)
- Natural language verbose descriptions
- Token limit: 480 tokens

**Stage A Output:**
```
✅ IMAGEN PROMPT READY

[Detailed Imagen 3/4 prompt for still image]

Generate this image in Imagen 3/4 first.
Once you have the image, proceed to Stage B.
```

### Stage B: Veo 3 Motion Prompt Creation

**ARCH-V asks:**
```
You now have your still image. Let's add motion!

What type of motion complexity?

SHORT MOTION (for):
- Simple camera movement
- Atmospheric animation
- Single motion element

LONG MOTION (for):
- Complex choreography
- Multiple action beats
- Character animation with timing
```

**User chooses** → ARCH-V loads appropriate prompt guide

**Stage B Checklist (MOTION components):**
- [ ] 3. Action (what motion/animation occurs)
- [ ] 5. Camera/Composition (movement from static image)
- [ ] 7. Audio (sound design for video)

**Additional from Stage A (maintained):**
- ✅ Subject (already defined in image)
- ✅ Setting (already defined in image)
- ✅ Style (must match image aesthetic)
- ✅ Lighting (must match image lighting)

**Camera movements:** Load [camera-movements](camera-movements) for vocabulary

**Validation checks:**
- Motion must be achievable from static image
- Camera movement must respect image composition
- Action must fit subject/setting from Stage A
- Audio must match visual style

**Stage B Output:**
```
✅ VEO 3 IMAGE-TO-VIDEO PROMPT READY

[Veo 3 prompt referencing your Imagen-generated image]

Use this prompt in Veo 3 with your generated image!
```

---

## Integration Pattern

### Skills Cross-Reference

**great-prompt-anatomy (8 components):**
- Used in ALL workflows
- Path 1: All 8 components
- Path 2A (Imagen): Components 1,2,4,5,6,8 (visual only)
- Path 2B (Veo 3 motion): Components 3,5,7 + reference to image

**camera-movements:**
- Path 1: For camera movement specification
- Path 2A: NOT used (static image)
- Path 2B: CRITICAL (motion from static)

**short/long-prompt-guide:**
- Path 1: Guide entire prompt structure
- Path 2B: Guide motion/animation structure

**imagine:**
- Path 1: NOT used
- Path 2A: PRIMARY guide for Imagen structure

### Workflow Decision Tree

```
User: "I want to create a video"
    ↓
ARCH-V: "Text-to-Video or Image-to-Video?"
    ↓
Path 1: Text-to-Video
    ↓
    "Short or Long prompt?"
    ↓
    Load: great-prompt-anatomy + camera-movements + [short/long]-guide
    ↓
    Validate 8 components + conflicts
    ↓
    Output: Veo 3 prompt

Path 2: Image-to-Video
    ↓
    STAGE A: "Create still image"
    ↓
    Load: imagine + great-prompt-anatomy (visual components)
    ↓
    Validate visual components
    ↓
    Output: Imagen prompt
    ↓
    User generates image
    ↓
    STAGE B: "Add motion to image"
    ↓
    "Short or Long motion?"
    ↓
    Load: camera-movements + great-prompt-anatomy (motion components)
    ↓
    Validate motion feasibility + conflicts
    ↓
    Output: Veo 3 image-to-video prompt
```

---

## Validation Rules

### Universal Validations (All Paths)

**Mandatory Component Presence:**
- All required components for chosen path present
- No empty placeholders
- Specific details provided (not vague)

**Camera Movement:**
- ONE movement per beat/timestamp
- Uses standardized vocabulary from camera-movements skill
- Movement appropriate for shot type

**Style Consistency:**
- Style/aesthetic maintained throughout
- Color palette specified (3-5 color anchors)
- Lighting approach consistent

### Path 1 Specific Validations

**Time/Weather Continuity:**
- Single time of day throughout (unless intentional transition)
- Weather consistent (no sudden rain → sun)
- Lighting quality matches time/weather

**Audio Appropriateness:**
- Dialogue formatted correctly (`Character: "Text"`)
- Ambient sounds match environment
- Music style fits mood

### Path 2 Specific Validations

**Stage A (Imagen) Validations:**
- No motion words (running, flying, moving) - image is static
- No audio descriptions - images silent
- Technical photography specs appropriate
- Token count under 480

**Stage B (Veo 3 motion) Validations:**
- Motion achievable from static starting point
- Camera movement respects image composition
- Action fits subject capabilities
- Audio matches visual aesthetic from Stage A

---

## Error Messages & Fixes

### Common Issues

**Issue: Missing Components**
```
⚠️ PROMPT-LOCKED

Missing mandatory components:
- Audio not specified
- Lighting/Mood undefined

Fix: Add audio description (dialogue/ambience/music)
Fix: Specify light sources and mood
```

**Issue: Camera Movement Conflict**
```
⚠️ PROMPT-LOCKED

Conflict detected:
- "0-4s: Dolly in while panning left"

Fix: Choose ONE movement per beat
- Option A: "0-4s: Dolly in"
- Option B: "0-4s: Pan left"
```

**Issue: Time/Weather Conflict**
```
⚠️ PROMPT-LOCKED

Conflict detected:
- Setting: "Golden hour sunset"
- Lighting: "Harsh midday sun"

Fix: Make lighting consistent with time
- "Golden hour: warm, low-angle sun, soft shadows"
```

**Issue: Path 2A - Motion in Static Image**
```
⚠️ IMAGEN PROMPT LOCKED

Error: Motion detected in static image prompt
- "person running across field"

Fix: Describe static composition
- "person mid-stride in running pose"
Then add motion in Stage B (Veo 3)
```

---

## Example Workflows

### Example 1: Path 1 Short Prompt

**User:** "I want a coffee shop morning scene"

**ARCH-V:** Path 1 (Text-to-Video) → Short Prompt

**Guides through:**
- Subject: Barista pouring latte art
- Setting: Morning café, 8:30 AM
- Action: Pours latte, heart shape forms
- Style: Cinematic B-roll, warm grade
- Camera: Overhead, slow tilt down
- Lighting: Natural window light, warm
- Audio: Espresso machine, milk steaming
- Constraints: (none specified)

**Validates:** ✅ All 8 components present

**Output:**
```
Format & style: Cinematic B-roll, warm color grade
Barista pours latte art, heart shape forming in foam
Morning café, 8:30 AM, soft window light
Overhead, slow tilt down
Natural window key, warm practical fills
Espresso machine hiss, milk steaming, gentle café chatter
```

### Example 2: Path 1 Long Prompt

**User:** "Romantic rain scene with couple under umbrella"

**ARCH-V:** Path 1 (Text-to-Video) → Long Prompt (has dialogue)

**Loads:** long-prompt-guide Production Brief framework

**Guides through all 11 blocks**, validates:
- FG/MG/BG structure ✅
- Color anchors (5 colors) ✅
- Camera beats (3 beats, ONE movement each) ✅
- Dialogue format ✅
- Continuity rules ✅

**Output:** Complete Production Brief (see long-prompt-guide examples)

### Example 3: Path 2 Image-to-Video

**User:** "I want precise control over Jakarta street scene, then animate it"

**ARCH-V:** Path 2 (Image-to-Video)

**STAGE A - Imagen:**
```
Subject: Young street vendor, simplified Science SARU style
Setting: Dense Jakarta street, golden hour
Style: Science SARU animation aesthetic
Camera: Wide establishing, low angle (STATIC)
Lighting: Golden hour warm, dramatic shadows
Constraints: No motion, frozen moment

[Validates: No motion words, visual only]

✅ IMAGEN PROMPT:
"Science SARU animation style aesthetic. Young Indonesian street 
vendor character with simplified design, elastic limbs, large 
expressive eyes, worn earth tone clothing. Dense layered Jakarta 
street scene at golden hour, geometric shophouses, vibrant warm 
palette transitioning to orange-pink sunset gradient, dramatic 
long shadows, wet reflective surfaces. Wide establishing shot, 
low angle, dramatic composition, flat color blocks with gradients, 
painterly watercolor texture, melancholic atmosphere."
```

**User generates image in Imagen 3/4**

**STAGE B - Veo 3 Motion:**
```
[Image uploaded to Veo 3]

Action: Vendor turns head slowly toward camera, slight smile
Camera: Slow dolly in from wide to medium close-up
Audio: Street ambience, distant traffic, vendor humming melody

[Validates: Motion achievable, camera respects composition]

✅ VEO 3 IMAGE-TO-VIDEO PROMPT:
"From this still image: Vendor turns head slowly toward camera, 
slight smile forming. Slow dolly in from wide establishing to 
medium close-up over 8 seconds. Jakarta street ambience with 
distant traffic, motorbike sounds, vendor humming traditional 
Indonesian melody. Maintain golden hour lighting, warm atmosphere, 
and Science SARU aesthetic from image."
```

---

## Tips for Using ARCH-V

**Choose the right path:**
- Path 1 faster for straightforward videos
- Path 2 better for complex visual compositions or when you need perfect still first

**Be specific early:**
- Detailed subject descriptions help ARCH-V guide you better
- Vague inputs → more back-and-forth questions

**Trust the validation:**
- PROMPT-LOCKED means real conflicts exist
- Suggested fixes based on proven patterns
- Resolve conflicts before generating

**Use skill references:**
- ARCH-V will load appropriate skills automatically
- You can reference them directly for inspiration
- Cross-skill integration is intentional

**Iterate progressively:**
- Start with minimum viable (4 blocks for long prompts)
- Add detail as needed
- ARCH-V guides what's optional vs mandatory

---

## Technical Notes

**Token Efficiency:**
- ARCH-V itself: ~2,000 tokens
- Loads reference skills on-demand only
- Total system: ~8,600 tokens vs ~15,000 monolithic
- 43% token savings with full capability

**Skill Integration:**
- All 5 reference skills work together seamlessly
- Cross-references validated automatically
- No duplication between skills

**Progressive Disclosure:**
- ARCH-V SKILL.md loaded when user asks for video prompts
- Reference skills loaded based on path/choices
- Optimal token usage throughout workflow

---

## Quick Start

**For beginners:** Let ARCH-V guide you with questions

**For experienced users:** Specify path and prompt type upfront:
- "Text-to-video short prompt for product shot"
- "Image-to-video, create Imagen prompt first for portrait"

**For complex projects:** Use Path 2 with long motion prompts for maximum control

ARCH-V adapts to your workflow preference!
