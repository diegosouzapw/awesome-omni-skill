---
name: ai-interaction-patterns
description: "AI-specific interaction design patterns covering wayfinding, prompt UX, human-in-the-loop controls, trust & transparency, AI identity, and context management. Based on Shape of AI (shapeof.ai). Use when asking about 'AI UX', 'AI interaction', 'prompt UX', 'AI trust', 'AI disclosure', 'AI avatar', 'AI personality', 'AI memory UX', 'action plan UX', 'stream of thought', 'AI citations', 'AI controls', 'AI wayfinding', 'AI suggestions', 'gallery pattern', 'follow-up pattern', 'draft mode', 'AI variations', 'AI consent', 'AI caveat', 'human-in-the-loop', 'AI transparency', 'AI state', 'prompt design', 'AI onboarding', or 'generative UI'."
---

# AI Interaction Patterns

AI-specific UX patterns for designing interfaces where users interact with AI models. Covers the full interaction lifecycle: from first prompt to output verification, memory persistence, and trust building.

**Source**: Based on [Shape of AI](https://www.shapeof.ai) pattern library (CC-BY-NC-SA) by Emily Campbell.

## Overview

This skill provides:

- 60+ AI interaction patterns across 6 categories
- Pattern selection decision framework
- Trust-level assessment for AI interfaces
- Wayfinding strategies for AI onboarding
- Human-in-the-loop governance patterns
- AI identity and personality design guidance
- Memory and context persistence patterns
- Anti-patterns to avoid in AI UX

## How This Differs from Other UI/UX Skills

| Skill                                 | Focus                                                                             |
| ------------------------------------- | --------------------------------------------------------------------------------- |
| **23-uiux-design-principles**         | Layout, hierarchy, responsive design (framework-agnostic)                         |
| **21-enterprise-ai-ux**               | Enterprise context: challenge taxonomy, professional palettes, RBAC               |
| **22-conversation-ux**                | Thread management, branching data model, context switching                        |
| **20-interactive-widgets**            | Widget protocols, rendering pipeline, state management                            |
| **25-ai-interaction-patterns** (this) | AI-SPECIFIC interaction logic: how users prompt, control, trust, and relate to AI |

## Reference Documentation

### Complete Pattern Catalog

- **[ai-interaction-patterns](ai-interaction-patterns.md)** - Full reference for all 60+ patterns
  - Wayfinders (8 patterns)
  - Prompt Actions (14 patterns)
  - Tuners (10 patterns)
  - Governors (13 patterns)
  - Trust Builders (7 patterns)
  - Identifiers (5 patterns)

## Quick Pattern Selection

### By User Problem

| User Says/Feels                     | Apply Pattern                           |
| ----------------------------------- | --------------------------------------- |
| "I don't know what to ask"          | Gallery, Suggestions, Templates         |
| "AI didn't understand me"           | Follow-ups, Nudges, Prompt Enhancer     |
| "I want alternatives"               | Variations, Branches, Randomize         |
| "Is this accurate?"                 | Citations, References, Caveat           |
| "This is taking too long"           | Draft Mode, Controls, Cost Estimates    |
| "I need AI to do something complex" | Action Plan, Stream of Thought          |
| "Is this AI or human?"              | Disclosure, Avatar, Name                |
| "Don't store my data"               | Incognito Mode, Consent, Data Ownership |
| "AI forgot what I said"             | Memory (scoped/global/ephemeral)        |

### By AI Product Type

| Product Type             | Essential Patterns                                                     | Nice-to-Have                    |
| ------------------------ | ---------------------------------------------------------------------- | ------------------------------- |
| **Chat assistant**       | Open Input, Suggestions, Follow-ups, Memory, Disclosure                | Gallery, Voice & Tone, Branches |
| **Code copilot**         | Inline Action, Stream of Thought, Controls, Citations                  | Action Plan, Draft Mode         |
| **Image generator**      | Gallery, Parameters, Variations, Inpainting, Preset Styles             | Draft Mode, Randomize           |
| **Document AI**          | Attachments, Citations, Caveat, Disclosure, Summary                    | Transform, Expand, Follow-ups   |
| **AI agent (agentic)**   | Action Plan, Controls, Verification, Stream of Thought, Cost Estimates | Memory, Consent                 |
| **Voice assistant**      | Voice Avatar, Personality, Controls, Disclosure                        | Memory, Consent                 |
| **Enterprise analytics** | Citations, Connectors, Filters, Modes, Disclosure                      | Action Plan, Memory             |

### Trust Level Decision

```
Is this a high-stakes domain (healthcare, finance, legal)?
  YES → CRITICAL trust: Citations + Verification + Disclosure + Caveat + Audit
  NO  →
    Is AI output mixed with human content?
      YES → HIGH trust: Disclosure + Citations + Caveat
      NO  →
        Could AI output cause harm if wrong?
          YES → MEDIUM trust: Caveat + Citations (optional)
          NO  → LOW trust: Minimal caveat, focus on UX quality
```

## The Six Pattern Categories

### 1. Wayfinders

_Help users construct their first prompt and get started._

| Pattern        | Purpose                       | When to Use                                   |
| -------------- | ----------------------------- | --------------------------------------------- |
| Gallery        | Showcase what's possible      | Onboarding, inspiration, capability discovery |
| Suggestions    | Context-aware prompt starters | Cold start, idle moments, mode changes        |
| Templates      | Structured prompt scaffolds   | Recurring tasks, complex prompts              |
| Follow-ups     | Conversation continuations    | After every AI response                       |
| Initial CTA    | First-interaction entry point | Landing pages, empty states                   |
| Nudges         | Proactive guidance            | When user prompt could be improved            |
| Prompt Details | Expose generation parameters  | Educational, reverse-engineering              |
| Randomize      | Serendipity exploration       | Creative tools, discovery                     |

### 2. Prompt Actions

_Different actions users can direct AI to complete._

| Pattern        | Purpose                          | When to Use                         |
| -------------- | -------------------------------- | ----------------------------------- |
| Open Input     | Natural language dialogue        | Universal starting point            |
| Inline Action  | Edit within existing content     | Document editors, code tools        |
| Chained Action | Multi-step sequential tasks      | Complex workflows                   |
| Regenerate     | Re-run with modifications        | When output needs improvement       |
| Transform      | Change content format/structure  | Data processing, content adaptation |
| Restyle        | Change visual/tonal style        | Creative tools, voice adjustment    |
| Expand         | Elaborate on content             | Drafting, content generation        |
| Summary        | Condense content                 | Reading, research, analysis         |
| Synthesis      | Combine multiple sources         | Research, report generation         |
| Describe       | Generate from visual/audio input | Multi-modal tools                   |
| Auto-fill      | AI-populated form fields         | Data entry, profile completion      |
| Restructure    | Reorganize content structure     | Documents, presentations            |
| Madlibs        | Guided prompt construction       | Onboarding, structured tasks        |
| Inpainting     | Region-based selective editing   | Image/video editing                 |

### 3. Tuners

_Adjust contextual data and settings to refine the prompt._

| Pattern          | Purpose                        | When to Use                       |
| ---------------- | ------------------------------ | --------------------------------- |
| Attachments      | Upload files as context        | Document analysis, RAG            |
| Connectors       | Link external data sources     | Enterprise integration            |
| Parameters       | Fine-tune generation settings  | Advanced users, precision tasks   |
| Model Management | Select/switch AI models        | Multi-model products              |
| Modes            | Switch AI behavior profiles    | Multi-purpose tools               |
| Filters          | Narrow input/output scope      | Search, data analysis             |
| Prompt Enhancer  | Auto-improve user prompts      | Novice users, quality improvement |
| Preset Styles    | Pre-configured parameter sets  | Quick style selection             |
| Saved Styles     | User-created parameter presets | Returning users, consistency      |
| Voice and Tone   | Configure AI personality       | Customization, brand alignment    |

### 4. Governors

_Human-in-the-loop features for oversight and agency._

| Pattern           | Purpose                                  | When to Use              |
| ----------------- | ---------------------------------------- | ------------------------ |
| Action Plan       | Preview steps before execution           | Complex/expensive tasks  |
| Stream of Thought | Show AI reasoning in real-time           | Transparency, debugging  |
| Controls          | Stop, pause, resume, queue               | During generation        |
| Draft Mode        | Low-fidelity preview first               | Expensive generation     |
| Branches          | Divergent exploration paths              | Creative exploration     |
| Variations        | Multiple outputs for comparison          | Selection, quality       |
| Citations         | Source attribution                       | Factual claims, research |
| References        | Link to supporting materials             | Evidence, verification   |
| Verification      | Confirmation before irreversible actions | Destructive operations   |
| Memory            | Cross-session context persistence        | Personalization          |
| Cost Estimates    | Resource usage transparency              | Paid/metered services    |
| Sample Response   | Quick preview before full gen            | Expensive operations     |
| Shared Vision     | Align user-AI understanding              | Complex instructions     |

### 5. Trust Builders

_Build confidence in AI ethics, accuracy, and trustworthiness._

| Pattern        | Purpose                          | When to Use                   |
| -------------- | -------------------------------- | ----------------------------- |
| Disclosure     | Label AI content as AI-generated | Blended content, agents       |
| Caveat         | Warn about AI limitations        | All AI outputs                |
| Consent        | Obtain permission for data use   | Recording, training, sharing  |
| Data Ownership | Clarify data storage/usage       | Enterprise, privacy-sensitive |
| Watermark      | Mark AI-generated media          | Images, audio, video          |
| Footprints     | Attribute aggregated sources     | Synthesized content           |
| Incognito Mode | Non-persistent sessions          | Sensitive topics, privacy     |

### 6. Identifiers

_Distinct qualities of AI that can be modified at brand/model level._

| Pattern     | Purpose                     | When to Use               |
| ----------- | --------------------------- | ------------------------- |
| Avatar      | Visual AI representation    | Chat, voice, multi-agent  |
| Personality | Tone, warmth, authority     | All AI interactions       |
| Name        | AI naming strategy          | Branding, trust-setting   |
| Color       | Brand and state signaling   | UI theming, state changes |
| Iconography | AI-specific visual language | Throughout UI             |

## CRITICAL Gotchas

| Rule                                                                         | Why                                                                      |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| NEVER use photorealistic avatars unless the AI truly matches that capability | Sets unrealistic expectations, erodes trust when capabilities fall short |
| ALWAYS show Stream of Thought for tasks > 5 seconds                          | Users abandon or re-submit when they can't see progress                  |
| NEVER let Memory be a black box                                              | Users must see, edit, and delete what AI remembers                       |
| ALWAYS offer Controls (at minimum: stop) during generation                   | Users need escape hatches for wrong-direction outputs                    |
| NEVER rely solely on Caveats for safety                                      | Caveat blindness is real; design the system to be safe, then add caveats |
| ALWAYS distinguish AI content from human content in blended UIs              | Users may unknowingly present AI work as their own                       |
| NEVER overwrite user work without Verification                               | Accidental overwrites destroy trust instantly                            |
| ALWAYS pair Suggestions with the ability to edit before sending              | Users need to refine, not just accept blindly                            |

## When to Use This Skill

Use this skill when:

- Designing any AI-powered user interface
- Choosing interaction patterns for AI products
- Evaluating AI UX for trust and transparency
- Building prompt input experiences
- Implementing human-in-the-loop workflows
- Designing AI identity (avatar, name, personality)
- Planning AI memory and context management UX
- Auditing AI interfaces for anti-patterns

## Related Skills

- **[21-enterprise-ai-ux](../21-enterprise-ai-ux/SKILL.md)** - Enterprise-specific AI design (RBAC, compliance, professional palettes)
- **[22-conversation-ux](../22-conversation-ux/SKILL.md)** - Thread management and conversation data models
- **[23-uiux-design-principles](../23-uiux-design-principles/SKILL.md)** - General design principles (layout, hierarchy)
- **[20-interactive-widgets](../20-interactive-widgets/SKILL.md)** - Widget rendering and state management

## Support

For AI interaction design questions, invoke:

- `ai-ux-designer` - AI-specific interaction pattern selection and design
- `uiux-designer` - General layout, hierarchy, and visual design
- `kaizen-specialist` - AI agent capabilities informing UX decisions
- `frontend-developer` - Implementation of AI interaction patterns
