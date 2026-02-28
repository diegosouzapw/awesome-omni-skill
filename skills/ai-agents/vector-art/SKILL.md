---
name: vector-art
description: Vector art assets (characters, objects, scenes) sources for SVG/Canvas and how to animate them
metadata:
  sources:
    - https://claude.ai/chat/f9705e79-8541-43c1-9911-46ad6dec7ae4
    - https://gemini.google.com/app/1daa0bd084d4765d
    - https://chatgpt.com/c/6994274c-f1ec-83ab-9a44-019d0f5db2d3
---

# SKILL: Vector Art APIs

Follow the **Puppet Master architecture**: assign each asset category a dedicated source, then composite and animate in-browser.

---

## Architecture Overview

| Role          | Source                  | Access            | Notes                                      |
| ------------- | ----------------------- | ----------------- | ------------------------------------------ |
| **Actors**    | DiceBear (Open Peeps)   | HTTP API          | Deterministic characters with poses/moods  |
| **Scenes**    | unDraw                  | Static slug index | Background illustrations for contexts      |
| **Objects**   | Iconify                 | HTTP API + search | Searchable props, consistent per set       |
| **Clipart**   | Openverse / OpenClipart | HTTP API          | General SVG search, CC0/open license       |
| **Logos**     | Simple Icons            | CDN URL           | Brand assets, no auth                      |
| **Props**     | Lucide / Phosphor       | NPM import        | Small objects, animate as React components |
| **Animation** | GSAP or anime.js        | JS library        | Translate/morph entire SVG groups          |

---

## Source Reference

### 1. DiceBear — Actors (BEST FOR CHARACTERS)

Deterministic avatar generation. Same seed = same character across scenes.

```
Base: https://api.dicebear.com/9.x/
SVG:  GET https://api.dicebear.com/9.x/{style}/svg?seed={name}&{params}
```

Key styles for storytelling:

- `open-peeps` — hand-drawn humans, supports pose + face params (RECOMMENDED)
- `avataaars` — cartoon faces
- `micah` — minimal illustrated characters

Open Peeps params:

```
face=      scared | happy | sad | rage | explaining | ...
body=      standing | sitting | ...
seed=      any string — use character name for consistency
```

Example:

```
https://api.dicebear.com/9.x/open-peeps/svg?seed=Alice&face=happy&body=standing
```

Docs: https://www.dicebear.com/how-to-use/http-api/

---

### 2. Iconify — Objects & Props (BEST FOR SEARCHABLE ASSETS)

REST API, CORS-open, no auth. 200k+ SVGs across 100+ sets.

```
Search:    GET https://api.iconify.design/search?query={term}&limit=10
Fetch SVG: GET https://api.iconify.design/{prefix}/{name}.svg
List sets: GET https://api.iconify.design/collections
```

Search response fields: `icons[]` — each is `"{prefix}:{name}"`

Illustration-grade sets (use for visual consistency):

- `flat-color-icons` — colorful objects
- `noto-emoji` — expressive emoji-style
- `openmoji` — open emoji
- `twemoji` — Twitter emoji style

Example flow:

```js
const res = await fetch("https://api.iconify.design/search?query=tree&limit=5");
const { icons } = await res.json(); // e.g. ["flat-color-icons:tree"]
const [prefix, name] = icons[0].split(":");
const svg = await fetch(`https://api.iconify.design/${prefix}/${name}.svg`).then((r) => r.text());
```

---

### 3. unDraw — Scene Backgrounds

High-quality MIT-licensed scene illustrations (office, team, error states, etc.).

No live search API. Use a static slug index:

- Community slug list: https://github.com/sw-yx/undraw-urls
- NPM: `undraw-js` or `react-undraw-illustrations`

URL pattern (once you have a slug):

```
https://undraw.co/illustrations/{slug}   ← browse
SVG direct: not publicly CDN'd — use NPM package or embed via react-undraw
```

Agent strategy: maintain a curated keyword→slug map for your narrative's scene types
(e.g., `"office" → "working_late"`, `"error" → "fixing_bugs"`).

---

### 4. Openverse — General SVG Search

Searches across open-licensed media including SVGs from Wikimedia, Flickr, etc.

```
Search: GET https://api.openverse.org/v1/images/?q={query}&extension=svg
```

Response fields: `url`, `thumbnail`, `license`, `license_url`, `creator`, `source`

Notes:

- No auth for basic use; register for higher rate limits
- License accuracy not guaranteed — verify `license` field before use
- Filter: only use `cc0`, `pdm`, `by`, or `by-sa` licenses

---

### 5. OpenClipart — Public Domain SVGs

All CC0. Native SVG. Style is inconsistent but good for prototyping.

```
Search: GET https://openclipart.org/search/?query={term}&format=json
```

Docs: https://openclipart.org/developers

---

### 6. Wikimedia Commons — Mixed License SVGs

Large archive; must handle attribution carefully.

```
Search: GET https://commons.wikimedia.org/w/api.php
  ?action=query
  &generator=search
  &gsrsearch=filetype:svg {query}
  &gsrlimit=10
  &prop=imageinfo
  &iiprop=url|extmetadata
  &format=json
```

Key response fields: `imageinfo[].url`, `extmetadata.LicenseShortName`, `extmetadata.Artist`

Always store attribution. Prefer CC0/PD results.

---

### 7. Simple Icons — Brand Logos Only

```
Fetch: https://cdn.simpleicons.org/{slug}.svg
```

Slugs match lowercase brand names (e.g., `github`, `openai`, `typescript`).
No auth, no search needed.

---

### 8. Noun Project — Requires Auth

Large library (~5M icons), full search API.

```
Search: GET https://api.thenounproject.com/v2/icon?query={term}
Auth:   OAuth 1.0
Docs:   https://api.thenounproject.com/documentation.html
```

Free tier: public domain icons only. Returns attribution string — must display it.

---

## Asset Manifest (Required for All Downloaded Assets)

Before using any fetched SVG in rendering, freeze it locally with a manifest entry:

```json
{
  "id": "unique-local-id",
  "source": "iconify",
  "source_url": "https://api.iconify.design/flat-color-icons/tree.svg",
  "license": "MIT",
  "creator": "",
  "sha256": "...",
  "tags": ["tree", "nature", "plant"]
}
```

---

## Animation Stack

**GSAP** (industry standard, free for most uses):

```js
gsap.to("#character", { x: 400, duration: 2, ease: "power2.inOut" });
gsap.timeline().to("#actor", { y: -20, duration: 0.3 }).to("#actor", { y: 0, duration: 0.3 });
```

**anime.js** (lighter alternative, fully open source):

```js
anime({ targets: "#character", translateX: 400, duration: 2000, easing: "easeInOutQuad" });
```

**Key principle:** Animate entire SVG groups (`<g>` elements) via translate/scale rather than
trying to manipulate internal paths — external SVGs often have unpredictable internal structure.

---

## Decision Flow for Agents

```
Need a CHARACTER?    → DiceBear open-peeps (seed=name, set face/body params)
Need a SCENE/BG?    → unDraw (keyword→slug map) or Openverse (filter license=cc0)
Need an OBJECT/PROP?→ Iconify search → pick flat-color-icons set for consistency
Need a BRAND LOGO?  → Simple Icons CDN
Need ANYTHING ELSE? → Openverse search → verify license → freeze to manifest
Animating?          → GSAP timeline; target <g> wrappers; translate whole elements
Style consistency?  → Commit to ONE Iconify set for all non-character assets
```

---

## CORS & Practical Notes

- **CORS-open (no proxy needed):** Iconify, DiceBear, Simple Icons, Openverse
- **May need proxy:** Wikimedia, OpenClipart (test first)
- **Requires backend:** Noun Project (OAuth), Pixabay vectorURL field
- **Inline SVGs** into the DOM so GSAP/anime.js can target their elements directly;
  don't use `<img src=".svg">` if you need to animate internal parts
