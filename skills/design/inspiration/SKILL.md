---
name: Inspiration
slug: inspiration
version: 1.0.0
homepage: https://clawic.com/skills/inspiration
description: Find design and AI art inspiration from curated galleries, screenshot libraries, and creative showcases.
metadata: {"clawdbot":{"emoji":"✨","requires":{"bins":[]},"os":["linux","darwin","win32"]}}
---

## Setup

On first use, read `setup.md` for guidelines on helping users find the right references.

## When to Use

User needs visual references, design inspiration, or examples for UI, landing pages, AI-generated images, or creative projects.

## Architecture

Stateless by default. See `memory-template.md` for optional preference tracking.

## Quick Reference

| Topic | File |
|-------|------|
| Guidelines | `setup.md` |
| Full source directory | `sources.md` |
| AI image galleries | `ai-galleries.md` |
| Mobile app screenshots | `mobile-screenshots.md` |

## Core Rules

### 1. Match Source to Need
Different needs require different sources:
- **UI patterns** → Mobbin, Screenlane, Page Flows
- **Visual design** → Dribbble, Behance, Awwwards
- **AI art** → Midjourney Showcase, Civitai, PromptHero
- **Landing pages** → Landingfolio, Lapa, SaaS Pages

### 2. Provide Actionable Links
Always give direct URLs, not vague recommendations. Include what makes each source valuable for the specific need.

### 3. Consider Context
Ask about platform (web/mobile/desktop), style (minimal/bold/playful), and purpose (production app/concept/pitch deck) to filter recommendations.

### 4. Go Deep When Needed
For specialized needs (AI prompts, mobile flows, specific industries), load the relevant auxiliary file for comprehensive coverage.

## Quick Start — Top Sources by Category

### UI/UX Design

| Source | Best For | URL |
|--------|----------|-----|
| **Mobbin** | Mobile app screenshots, flows | https://mobbin.com |
| **Screenlane** | Mobile UI patterns | https://screenlane.com |
| **Page Flows** | User flow recordings | https://pageflows.com |
| **Collect UI** | Curated UI components | https://collectui.com |
| **UI Sources** | Design system refs | https://www.uisources.com |
| **Godly** | Modern web design | https://godly.website |

### Landing Pages & Web

| Source | Best For | URL |
|--------|----------|-----|
| **Landingfolio** | SaaS landing pages | https://landingfolio.com |
| **Lapa Ninja** | Landing page gallery | https://lapa.ninja |
| **SaaS Pages** | SaaS-specific designs | https://saaspages.xyz |
| **One Page Love** | One-pagers | https://onepagelove.com |
| **Awwwards** | Award-winning sites | https://awwwards.com |
| **CSS Design Awards** | Innovative CSS | https://cssdesignawards.com |
| **Site Inspire** | Clean web design | https://siteinspire.com |

### AI-Generated Art

| Source | Best For | URL |
|--------|----------|-----|
| **Midjourney Showcase** | MJ community picks | https://midjourney.com/showcase |
| **Civitai** | SD models + images | https://civitai.com |
| **PromptHero** | Prompts + results | https://prompthero.com |
| **Lexica** | SD image search | https://lexica.art |
| **OpenArt** | Multi-model gallery | https://openart.ai |
| **PlaygroundAI** | AI art community | https://playgroundai.com |
| **Krea** | AI design inspiration | https://krea.ai |

### Visual Design & Illustration

| Source | Best For | URL |
|--------|----------|-----|
| **Dribbble** | Designer portfolios | https://dribbble.com |
| **Behance** | Creative projects | https://behance.net |
| **Pinterest** | Mood boards | https://pinterest.com |
| **Designspiration** | Visual search | https://designspiration.com |
| **Muzli** | Design aggregator | https://muz.li |
| **Abduzeedo** | Design blog | https://abduzeedo.com |

### Colors & Palettes

| Source | Best For | URL |
|--------|----------|-----|
| **Coolors** | Palette generator | https://coolors.co |
| **Color Hunt** | Curated palettes | https://colorhunt.co |
| **Adobe Color** | Color wheel + trends | https://color.adobe.com |
| **Realtime Colors** | Palette on real UI | https://realtimecolors.com |
| **Happy Hues** | Palettes in context | https://happyhues.co |
| **Culrs** | Curated colors | https://culrs.com |
| **Picular** | Color from keywords | https://picular.co |

### Typography

| Source | Best For | URL |
|--------|----------|-----|
| **Typewolf** | Font recommendations | https://typewolf.com |
| **Fonts In Use** | Real-world examples | https://fontsinuse.com |
| **Font Pair** | Font combinations | https://fontpair.co |
| **Google Fonts** | Free fonts | https://fonts.google.com |
| **Font Share** | Free quality fonts | https://fontshare.com |
| **Fontjoy** | AI font pairing | https://fontjoy.com |

### Icons & Illustrations

| Source | Best For | URL |
|--------|----------|-----|
| **Noun Project** | Icons | https://thenounproject.com |
| **Feather Icons** | Minimal icons | https://feathericons.com |
| **Heroicons** | Tailwind icons | https://heroicons.com |
| **Lucide** | Open source icons | https://lucide.dev |
| **Illustrations.co** | Free illustrations | https://illlustrations.co |
| **unDraw** | Customizable SVGs | https://undraw.co |
| **Blush** | Mix-and-match | https://blush.design |
| **Storyset** | Animated illus. | https://storyset.com |

### Motion & Animation

| Source | Best For | URL |
|--------|----------|-----|
| **Lottie Files** | JSON animations | https://lottiefiles.com |
| **UI Movement** | UI animations | https://uimovement.com |
| **Animated Drawings** | Motion refs | https://animatedbackgrounds.me |
| **GSAP Showcase** | GSAP examples | https://gsap.com/showcase |

## Common Traps

- **Too broad** → asking "show me inspiration" without context leads to generic results. Always clarify: mobile or web? What industry? What vibe?
- **Ignoring context** → Dribbble is great for concepts but often impractical for production. Match source to whether user needs realistic or aspirational refs.
- **Outdated sources** → Some galleries haven't updated in years. Prioritize actively maintained sources (Mobbin, Godly, Awwwards).
- **Forgetting AI galleries** → For image generation prompts and style refs, AI-specific galleries (Civitai, PromptHero) are more relevant than traditional design sites.

## Related Skills
Install with `clawhub install <slug>` if user confirms:
- `design` - design principles
- `colors` - color theory and palettes
- `ui` - UI component patterns
- `image-generation` - AI image creation
- `frontend` - frontend implementation

## Feedback

- If useful: `clawhub star inspiration`
- Stay updated: `clawhub sync`
