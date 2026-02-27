---
name: prompt-expander
description: EXPAND vague prompts into precise, platform-optimized instructions. Detects target platform (Claude, GPT, Gemini, Midjourney, Sora, etc.) and applies appropriate prompting patterns. Use when user says "coggle", "expand", "improve prompt", "make better".
---

# Prompt Expander

Transform vague prompts into precise, platform-optimized instructions.

## Trigger Phrases

Users invoke this skill by saying:
- "coggle"
- "expand"
- "improve prompt"
- "make better"
- "optimize prompt"
- "enhance prompt"

## Platform Detection

Detect target platform from context clues:

| Clue | Platform |
|------|----------|
| "for Claude", "Claude Code", "system prompt" | Claude |
| "GPT", "ChatGPT", "OpenAI", "Codex" | OpenAI |
| "Gemini", "Google AI" | Gemini |
| "Perplexity", "search", "research", "citations" | Perplexity |
| "Copilot", "GitHub", "code completion" | GitHub Copilot |
| "Grok", "xAI", "X AI" | Grok |
| "Midjourney", "MJ", "--ar", "DALL-E", "image" | Image Generation |
| "Flux", "FLUX.1", "Black Forest" | Flux |
| "Sora", "Runway", "video", "animation" | Video Generation |
| "Nano Banana", "Higgsfield" | Nano Banana Pro |

**If unclear, ask user:** "What platform is this prompt for?"

## Expansion Workflow

### Step 1: Detect Platform

Identify target from prompt context or ask user.

### Step 2: Apply PRECISE Framework

Adapt elements based on platform:

| Element | Text AI | Image AI | Video AI |
|---------|---------|----------|----------|
| **P**ersona | Role definition | Style/artist reference | Director's vision |
| **R**equirements | Deliverables | Visual elements | Shots/scenes |
| **E**xamples | Reference outputs | Reference images | Reference clips |
| **C**ontext | Background info | Scene setting | Narrative context |
| **I**nstructions | Step-by-step | Composition notes | Storyboard |
| **S**pecifications | Output format | Parameters (--ar, --v) | Duration, resolution |
| **E**valuation | Success criteria | Visual quality checks | Motion coherence |

#### P - Persona
Define the AI's role or style:
- **Text AI**: "Act as a [specific expert role] with [relevant experience]..."
- **Image AI**: "In the style of [artist/movement]..."
- **Video AI**: "Directed as [cinematic style/vision]..."

#### R - Requirements
Specify deliverables:
- **Text AI**: Output format, length, style, technical requirements
- **Image AI**: Visual elements, composition, mood
- **Video AI**: Shot list, scene requirements, key moments

#### E - Examples
Provide concrete references:
- **Text AI**: Sample outputs, style guides
- **Image AI**: Reference images, similar works
- **Video AI**: Reference clips, similar scenes

#### C - Context
Add relevant background:
- **Text AI**: Industry, audience, purpose, constraints
- **Image AI**: Scene setting, environment, atmosphere
- **Video AI**: Narrative context, story arc, setting

#### I - Instructions
Break down the task:
- **Text AI**: Numbered steps, logical sequence
- **Image AI**: Composition notes, layer priorities
- **Video AI**: Storyboard sequence, shot progression

#### S - Specifications
Define format and structure:
- **Text AI**: markdown/JSON/list/table, length, sections
- **Image AI**: Aspect ratio (--ar 16:9), version (--v 6), quality (--q 2)
- **Video AI**: Duration, resolution, frame rate, transitions

#### E - Evaluation
Specify success criteria:
- **Text AI**: Quality metrics, what to avoid
- **Image AI**: Visual coherence, style consistency
- **Video AI**: Motion smoothness, narrative flow

### Step 3: Apply Platform Template

Reference platform-specific patterns from `./templates/[platform].md`:

| Platform | Best For | Key Pattern | Template File |
|----------|----------|-------------|---------------|
| Claude | Complex reasoning, coding | Role + Context + Task + Constraints | `claude.md` |
| Codex/GPT | Code generation, chat | Delimiters + Examples | `codex.md` |
| Gemini | Multimodal, reasoning | Few-shot examples | `gemini.md` |
| Perplexity | Research, citations | Search scope + Recency + Sources | `perplexity.md` |
| Copilot | Code completion | General goal + Specific requirements | `copilot.md` |
| Grok | Conversational, real-time | Role + Task + Format | `grok.md` |
| Image Gen | Midjourney, DALL-E | Descriptive phrases + Parameters | `image-gen.md` |
| Flux | Photorealistic images | Natural language descriptions | `flux.md` |
| Video Gen | Sora, Runway, Kling | Storyboard + Camera direction | `video-gen.md` |

When expanding a prompt, detect the target platform and apply the appropriate template structure.

### Step 4: Present Options

After expansion, offer:
- **Run** - Execute the prompt (if possible)
- **Save** - Save to `.prompts/[name].md`
- **Refine** - Iterate with user feedback

## Output Format

```markdown
## Coggle!

**Original:**
> [input]

**Platform:** [detected platform]

**Expanded:**

[Full expanded prompt in platform-appropriate format]

---

**Templates:** See `./templates/` for platform-specific patterns.

**Next Steps:**
- **Run** - Execute this prompt
- **Save** - Save to `.prompts/[name].md`
- **Refine** - Make adjustments
```

## Common Anti-Patterns to Avoid

| Anti-Pattern | Example | Fix |
|--------------|---------|-----|
| Vague verbs | "help", "something", "stuff" | Specific actions |
| Missing audience | No target reader defined | Add audience context |
| Implicit assumptions | Unstated requirements | Make explicit |
| Wall of text | No structure | Add sections/steps |
| Conflicting instructions | Contradictory asks | Resolve conflicts |
| Missing format | No output spec | Define structure |

## Success Indicators

This skill is successful when:
- [ ] Platform has been detected or confirmed
- [ ] PRECISE framework elements applied appropriately for platform
- [ ] Expanded prompt is significantly more actionable than original
- [ ] Output follows platform-specific conventions
- [ ] User can immediately use the expanded version

## Quick Examples

**Text AI (Claude):**
```
Before: "Write something about marketing"
After: "Act as a marketing strategist with 10+ years B2B SaaS experience.
Write a 500-word blog post about email marketing best practices for
startup founders. Include 3 actionable tips with examples. Format as:
Hook -> Problem -> Solution -> Examples -> CTA.
Tone: Professional but approachable."
```

**Image AI (Midjourney):**
```
Before: "sunset over mountains"
After: "Golden hour sunset over snow-capped mountain peaks, dramatic
god rays through clouds, alpine lake reflection in foreground,
cinematic composition, in the style of Ansel Adams, photorealistic,
--ar 16:9 --v 6 --q 2"
```

**Video AI (Sora):**
```
Before: "person walking in city"
After: "Cinematic tracking shot following a silhouetted figure walking
through rain-soaked Tokyo streets at night, neon reflections on wet
pavement, bokeh lights in background, film noir aesthetic, slow motion,
camera dollies forward maintaining subject in center frame,
4K resolution, 24fps, 10 seconds duration"
```

**Research AI (Perplexity):**
```
Before: "tell me about quantum computing"
After: "What are the latest breakthroughs in quantum error correction?
Focus on peer-reviewed research from 2024-2025.
Summarize the top 3 developments with inline citations.
Include practical implications for each breakthrough."
```

**Code AI (GitHub Copilot):**
```
Before: "make a retry function"
After: "Write a TypeScript function that retries async operations with
exponential backoff.
Requirements:
- Accept a function, max retries (default 3), initial delay (default 1000ms)
- Double delay after each retry
- Return successful result or throw after max retries
- Log each attempt with attempt number"
```

**Conversational AI (Grok):**
```
Before: "analyze this startup"
After: "Role: Skeptical tech analyst who's seen too many hype cycles
Task: Evaluate the claims in this startup pitch
Format: Bullet points with a reality-check score (1-10) for each claim
Be direct, flag red flags, and rate overall viability."
```

---

*Prompt Expander skill | Last Updated: 2025-12-26*
