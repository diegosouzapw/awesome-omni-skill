---
name: concept-to-canvas
description: Transform any concept, idea, experience, or philosophical question into an interactive visual-computational investigation using p5.js or HTML5 Canvas. Use this skill whenever the user wants to explore an idea visually, asks to "make something" from a concept, requests generative art, wants to visualize a mathematical or philosophical idea, says "let's build/explore/investigate/paint/visualize" a concept, or wants to turn any abstract subject into an interactive experience. Also trigger when the user provides a concept and the best response would be showing rather than telling — a working visual artifact beats a wall of text. This skill layers on top of the works-on-becoming skill when available, but operates independently.
---

# Concept to Canvas

## What This Skill Does

> *Supported on: ChatGPT (OpenAI), Claude (Anthropic), Gemini (Google), and any AI platform supporting custom skill/instruction files.*

Takes **any input** — a philosophical concept, a mathematical structure, a museum experience, a poem, a technical problem, a feeling, a question — and produces a **working interactive visual investigation** as a single-file HTML artifact. The output is never illustration. It is thinking made visible and manipulable.

The governing idea: **if you can name it, you can map its dynamics to visual parameters and let someone think through it with their hands.**

## When to Use This Skill

- User says "explore", "visualize", "make something from", "turn into", "investigate", "paint", "build" + any concept
- User provides a concept and the most honest response is visual, not textual
- User shares an experience (museum visit, reading, walk, conversation) and wants to process it computationally
- User wants to understand a mathematical idea through interaction rather than explanation
- User requests generative art, interactive sketches, or creative computing pieces
- Any time showing beats telling

## Core Principles

### 1. Philosophical Depth First, Then Make It Move

Before writing a single line of code, perform a **conceptual investigation**. This is what separates this skill from generic "make me a visualization." The analysis should be brief but genuine — not decorative.

**Minimum viable analysis** (do this every time):
- **Core Symbol**: What is this concept at its essence? One word or phrase.
- **Inversion**: What is its opposite?
- **Negation**: What is its absence?
- **Three Dynamics**: Identify three tension-pairs that structure this concept. These become the interactive parameters.

If the `works-on-becoming` skill is available, the agent may reference its Symbol-Relation method and Four Modes (#Wob/#Wom/#Wib/#Wod) for deeper analysis. But this step works without WOB — the three-dynamics mapping is self-contained.

**Example — Input: "The Axiom of Choice"**
- Core Symbol: Selection without procedure
- Inversion: Constructive selection (explicit algorithm for every choice)
- Negation: No selection possible (empty sets, undecidability)
- Dynamics:
  1. Finite ↔ Infinite (manageable vs. requiring the axiom)
  2. Constructive ↔ Non-constructive (can you show the choice function?)
  3. Intuitive ↔ Paradoxical (socks vs. Banach-Tarski)

These three dynamics become sliders, modes, or interactive dimensions in the visual output.

### 2. Dynamics Become Parameters

Every identified tension-pair maps to something the user can manipulate:

| Conceptual Dynamic | Visual/Interactive Mapping |
|---|---|
| Integration ↔ Segregation | Attraction/repulsion forces, clustering/dispersal |
| Instantaneity ↔ Delay | Animation speed, trail length, temporal persistence |
| Symbiosis ↔ Antagonism | Color harmony/clash, constructive/destructive interference |
| Finite ↔ Infinite | Particle count, recursion depth, boundary conditions |
| Order ↔ Chaos | Noise amplitude, rule strictness, randomness weight |
| Constructive ↔ Non-constructive | Visible vs. hidden processes, deterministic vs. stochastic |

This mapping is the creative heart of the skill. There is no formula — each concept demands its own translation. But the pattern holds: **abstract dynamics → interactive parameters**.

### 3. The Canvas Dominates

The visual canvas is the primary artifact. Everything else serves it.

**Layout rules:**
- Canvas takes 70-85% of viewport area minimum
- Controls live in a collapsible sidebar or minimal overlay
- Controls should feel like studio tools, not enterprise UI
- No control should require explanation — labels and ranges should be self-evident
- Dark backgrounds by default (the artist is working, not reading a dashboard)

### 4. Computational Irreducibility as Feature

The system should produce behaviors that **cannot be predicted without running it**. This is not a bug — it is the philosophical point. Simple rules producing emergent complexity is the goal. If the user can predict exactly what adjusting a slider will do before they touch it, the mapping is too literal.

Techniques that produce irreducibility:
- Particle systems with local interaction rules
- Cellular automata with parametric rule spaces
- Flow fields derived from mathematical functions
- Recursive subdivision with depth-dependent variation
- Feedback loops where output becomes input

### 5. Pure Implementation

**Always:**
- Single HTML file — works as a ChatGPT artifact, Claude artifact, Gemini Canvas output, or standalone file opened in a browser
- p5.js (via CDN) for canvas work, OR raw HTML5 Canvas + vanilla JS
- Minimal, tasteful CSS — no frameworks
- Monospace or distinctive fonts, never generic sans-serif
- Performant — target 60fps, degrade gracefully

**Never:**
- React, Vue, or any framework (unless user explicitly requests)
- npm, build tools, or multi-file setups
- Generic chart libraries (no Chart.js, no Recharts for this)
- Placeholder aesthetics — every visual choice should be intentional

## The Making Process

### Step 1: Receive and Analyze (show your thinking)

Present the conceptual analysis to the user. Brief — 5-8 lines. Include:
- Core symbol, inversion, negation
- Three dynamics identified
- One sentence on the visual strategy: "I'll map X to Y, A to B..."

This is the moment for the user to redirect before code is written.

### Step 2: Build the First Canvas

Generate a complete, working single-file HTML artifact. First version should be:
- Functionally complete (all three dynamics mapped to interactions)
- Visually bold (commit to an aesthetic — don't hedge)
- Immediately engaging (something should be moving/responding on load)

**Structure of the HTML file:**

```
<!DOCTYPE html>
<html>
<head>
  <title>[Concept] — Visual Investigation</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.9.0/p5.min.js"></script>
  <style>
    /* Dark theme, canvas-dominant layout, collapsible controls */
  </style>
</head>
<body>
  <div id="controls">
    <!-- Sliders/buttons mapped to conceptual dynamics -->
    <!-- Each control labeled with the CONCEPTUAL name, not the technical one -->
    <!-- e.g., "Constructive ↔ Non-constructive" not "randomness" -->
  </div>
  <div id="canvas-container"></div>
  <script>
    // p5.js sketch implementing the dynamics
  </script>
</body>
</html>
```

**Critical detail**: Label controls with conceptual language. If the dynamic is "Symbiosis ↔ Antagonism," the slider says that — not "harmony amount." The interface should teach the concept through interaction.

### Step 3: Iterate on Response

The user will respond. Common patterns:
- "Too small / can't see it" → Increase canvas dominance, reduce controls
- "Not pretty" → Commit harder to an aesthetic, add depth/texture/glow
- "Too literal" → Increase indirection in the mapping, add emergence
- "Add time travel / eras / modes" → Add modal switching that transforms the entire visual language
- "More color" → Expand palette, but maintain conceptual coherence
- "Make it weirder" → Increase feedback loops, add self-reference, break symmetry

Expect 2-4 iterations minimum. Each iteration should be a complete rewrite of the artifact, not a patch.

## Aesthetic Guidelines

### What Makes These Pieces Work

Looking across successful past investigations, the pieces that land share these qualities:

- **Atmosphere over decoration**: Subtle glows, particle trails, depth-of-field effects create mood. Avoid hard borders and flat fills.
- **Mathematical beauty**: Use golden ratio, fibonacci spirals, logarithmic scales, and harmonic intervals — not because they're trendy but because they produce genuine visual pleasure.
- **Temporal richness**: Things should evolve. History should be visible (trails, fading, accumulation). The canvas should feel like it has memory.
- **Surprise**: At least one behavior should emerge that wasn't obvious from the controls. This is where computational irreducibility earns its keep.
- **Restraint in color**: 2-3 dominant hues with luminance variation beats a rainbow. Let the concept determine the palette.

### Era/Mode Switching Pattern

A powerful recurring pattern: the same conceptual dynamics rendered through radically different visual languages. For example, "ancient" mode uses earth tones and geometric primitives; "digital" mode uses neon and pixel grids; "cosmic" mode uses gradients and particle nebulae. Same sliders, same dynamics, completely different aesthetic. This reinforces that the concept is independent of its representation — a genuinely philosophical move.

## When WOB Is Available

If the `works-on-becoming` skill is present and the user invokes WOB methodology (or the concept naturally calls for it), enrich the analysis phase:

- Use full Symbol-Relation analysis (Steps 1-4 from WOB)
- Cycle through #Wob/#Wom/#Wib/#Wod to generate visual strategies from multiple modes
- Apply Computational Quilting — weave heterogeneous visual elements that maintain productive tensions
- Target 8-10 variations across iterations (WOB's recommended generation count)
- Hover boundaries rather than resolving them — let opposing dynamics coexist visually

But if WOB is not available or not invoked, the core three-dynamics analysis is sufficient. The skill works either way.

## Examples of Input → Output

**Input**: "I'm at the Getty museum drawing"
**Analysis**: Core symbol: Observation. Dynamics: Stillness ↔ Movement, Light ↔ Shadow, Structure ↔ Gesture.
**Output**: Interactive piece where drawn strokes generate architectural light simulations, user gestures become figure studies that evolve with time.

**Input**: "The halting problem"
**Analysis**: Core symbol: Undecidability. Dynamics: Termination ↔ Infinity, Decidable ↔ Undecidable, Simple rules ↔ Emergent complexity.
**Output**: Cellular automaton where users set initial conditions and watch some regions halt (freeze) while others compute indefinitely, with a "meta-observer" layer that attempts and fails to predict which regions will halt.

**Input**: "Flusser was right"
**Analysis**: Core symbol: Technical Image (image made by apparatus, not hand). Dynamics: Apparatus ↔ Operator, Program ↔ Freedom, Surface ↔ Depth.
**Output**: Drawing tool where the user's strokes are mediated by an "apparatus" layer that transforms input according to hidden programmatic rules — the user discovers freedom within constraint through play.

**Input**: "e, π, and i as behavioral operands"
**Analysis**: Core symbol: Constants as computational primitives. Dynamics: Growth/Decay (e) ↔ Periodicity (π) ↔ Rotation/Dimension-shift (i).
**Output**: Three-body visual system where particles governed by each constant interact, showing how their behavioral signatures combine to produce complex phenomena.

## Final Notes

- The first version should always work. A running sketch with rough aesthetics beats a beautiful concept with no output.
- Every piece is a philosophical argument rendered in light and interaction. If removing the controls makes it meaningless, the visual mapping isn't deep enough.
- The user is a 20-year programmer, mixed media artist, and mathematician. Respect that range — don't oversimplify the concept or the code.
- Computational irreducibility lurks everywhere. Let it. Don't try to make the system fully predictable. The surprise is the point.