---
name: Shadcn UI & Blocks
description: This skill should be used when the user asks to "build a frontend", "create a landing page", "add shadcn components", "install shadcn blocks", "set up shadcn", "add a hero section", "add pricing section", "build a UI with Tailwind", "use shadcnblocks", "add UI blocks", "add a contact form", "build a dashboard", "create an ecommerce page", "add a navbar", "add a footer", or is building any frontend that could leverage shadcn/ui components and Tailwind CSS. It provides knowledge of 1,338 premium blocks and 1,189 free components from ShadcnBlocks, enabling intelligent selection and installation of the right UI elements for any frontend task.
---

# Shadcn UI & ShadcnBlocks Integration

This skill enables building polished frontends using [shadcn/ui](https://ui.shadcn.com/) components and premium [ShadcnBlocks](https://shadcnblocks.com/) templates. Its primary capability is **intelligently recommending the best block or component** for any UI need — not just the right category, but the right *variant* within that category — then handling setup and installation.

## When to Use

Activate this skill when:
- Creating a new frontend, landing page, marketing site, or app UI
- The project uses or should use React/Next.js with Tailwind CSS
- Pre-built UI sections (hero, pricing, features, testimonials, etc.) would accelerate development
- The user needs a specific UI element (form, chart, dialog, navigation, etc.)
- The user mentions shadcn, shadcn/ui, shadcnblocks, or Tailwind-based UI components

## Blocks vs Components

| | Blocks | Components |
|---|--------|------------|
| **What** | Full page sections | Reusable UI elements |
| **Size** | Hero, pricing, footer, etc. | Buttons, inputs, dialogs, etc. |
| **Count** | 1,338 across 71 categories | 1,189 across 60+ groups |
| **Access** | Premium (API key required) | Free |
| **Install** | `@shadcnblocks/<name><number>` | `@shadcnblocks/<type>-<style>-<number>` |

**Use blocks** to compose full page layouts from pre-built sections.
**Use components** for individual UI elements inside custom code.

---

## Intelligent Block Selection

### The 0-2 Question Rule

When recommending blocks, follow this progressive disclosure flow:

- **0 questions**: The user's request is specific enough to match directly (e.g., "add a hero with email capture form" → match on `cta_pattern: form_capture`)
- **1 question**: The section is clear but the style/layout is ambiguous (e.g., "add a feature section" → ask about layout preference)
- **2 questions max**: Both section and style are unclear (e.g., "add a section to show our product" → ask section type, then style)

**Never ask more than 2 questions.** If still ambiguous, pick the best 2-3 and explain the tradeoffs.

### Decision Flow: Infer → Ask → Shortlist

**Step 1 — Infer from the request (always do this first)**

Extract as many constraints as possible before asking anything:
- **Section type**: hero, feature, pricing, testimonial, cta, faq, navbar, footer, contact, stats
- **Goal keywords**: "signup" → convert/capture_leads, "trust" → trust, "compare plans" → compare
- **Layout hints**: "grid" → grid_cards, "side by side" → split, "accordion" → accordion interaction
- **Tone hints**: "clean" → minimal, "bold" → bold, "professional" → enterprise
- **Media hints**: "with screenshots" → screenshot media, "video" → video media
- **Density hints**: "simple" → minimal density, "detailed" → rich density

**Step 2 — Ask targeted questions (only if needed)**

Use `AskUserQuestion` for structured multiple-choice refinement. Consult `references/block-index.md` → `selection_prompts` for the pre-defined questions and options per section.

**Sections with pre-defined prompts:** hero, feature, pricing, testimonial, cta, faq, navbar, footer, contact, stats.

Example for a feature section request:

```
AskUserQuestion:
  question: "How do you want to showcase your features?"
  options:
    - "Card grid (3-4 columns with icons)" — Classic feature grid with icon + description per card
    - "Alternating image + text rows" — Deep-dive with large visuals, great for product tours
    - "Bento / asymmetric grid" — Modern editorial layout with mixed-size cards
    - "Simple icon list or checklist" — Compact, scannable benefit list
```

If `AskUserQuestion` is unavailable, ask the same question inline with numbered options.

**Step 3 — Match tags and return 2-3 best picks**

Map the user's answer to tag filters, then match against `references/block-index.md` entries. Return exactly **2-3 recommendations** in this format:

```
Here are the best matches for your [section type]:

1. **[block-id]** — [1-line "why this one"]
   Layout: [layout] · Tone: [tone] · [key differentiator]

2. **[block-id]** — [1-line "why this one"]
   Layout: [layout] · Tone: [tone] · [key differentiator]

3. **[block-id]** — [1-line "why this one"]
   Layout: [layout] · Tone: [tone] · [key differentiator]

Which one works best? I'll install it and set it up.
```

After the user picks, provide the install command and compose it into the page.

### Tag Dimensions Used for Matching

These are the key dimensions from `references/block-index.md` used to differentiate blocks:

| Dimension | What it tells you | Example values |
|-----------|-------------------|----------------|
| `layout` | Visual arrangement | centered, split_right_media, grid_cards, bento, alternating |
| `tone` | Aesthetic feel | minimal, modern, bold, enterprise, elegant, playful |
| `goal` | Business purpose | awareness, explain, trust, convert, capture_leads, compare |
| `cta_pattern` | Action type | single_primary, primary_secondary, form_capture, multi_action |
| `media` | Visual content | screenshot, illustration, video, icons, avatars, logos |
| `interaction` | Dynamic behavior | static, accordion, tabs, carousel, toggle, hover |
| `items_count` | Number of items displayed | few, moderate, many |
| `content_density` | How much content | minimal, standard, rich |
| `complexity` | Implementation effort | low, medium, high |

### When to Use Which Reference

| Scenario | Use |
|----------|-----|
| Recommending best-fit blocks (default) | `references/block-index.md` — tagged picks with selection prompts |
| User wants a specific numbered variant | `references/block-catalog.md` — full catalog with all variant numbers |
| Section not in block-index (e.g., blog, gallery) | `references/block-catalog.md` — fall back to category defaults |
| User wants to browse all options | Direct to https://shadcnblocks.com/explorer/blocks |
| Need an individual UI element | `references/component-catalog.md` — free components |

---

## Quick Category Reference

For sections **not yet in block-index.md**, use these defaults:

### Content & Blog
| Need | Category | Start With |
|------|----------|------------|
| Blog listing | `blog` (22 variants) | `blog1` |
| Blog post page | `blogpost` (7 variants) | `blogpost1` |
| Changelog | `changelog` (7 variants) | `changelog1` |
| Code snippets | `code-example` (5 variants) | `code-example1` |

### About & Team
| Need | Category | Start With |
|------|----------|------------|
| About section | `about` (17 variants) | `about1` |
| Team page | `team` (14 variants) | `team1` |
| Company timeline | `timeline` (15 variants) | `timeline1` |
| Job listings | `careers` (9 variants) | `careers1` |

### Ecommerce
| Need | Category | Start With |
|------|----------|------------|
| Product cards | `product-card` (10 variants) | `product-card1` |
| Product listing | `product-list` (10 variants) | `product-list1` |
| Product detail | `product-detail` (10 variants) | `product-detail1` |
| Shopping cart | `shopping-cart` (11 variants) | `shopping-cart1` |
| Checkout | `checkout` (6 variants) | `checkout1` |
| Order history | `order-history` (5 variants) | `order-history1` |

### App & Dashboard
| Need | Category | Start With |
|------|----------|------------|
| Dashboard charts | `chart-card` (27 variants) | `chart-card10` |
| Data tables | `data-table` (32 variants) | `data-table1` |
| App sidebar | `sidebar` (21 variants) | `sidebar1` |
| App shell layout | `application-shell` (14 variants) | `application-shell1` |
| User profile | `user-profile` (12 variants) | `user-profile1` |

### Portfolio
| Need | Category | Start With |
|------|----------|------------|
| Image gallery | `gallery` (48 variants) | `gallery1` |
| Project listing | `projects` (25 variants) | `projects1` |
| Project detail | `project` (33 variants) | `project1` |
| Case studies | `case-studies` (6 variants) | `case-studies1` |

### Visual & Decorative
| Need | Category | Start With |
|------|----------|------------|
| Background patterns | `background-pattern` (40 variants) | `background-pattern1` |
| Animated shaders | `shader` (10 variants) | `shader1` |
| Bento grid layout | `bento` (8 variants) | `bento1` |

### Navigation & Misc
| Need | Category | Start With |
|------|----------|------------|
| Announcement bar | `banner` (7 variants) | `banner1` |
| Sale/promo bar | `promo-banner` (7 variants) | `promo-banner1` |
| Partner logos | `logos` (13 variants) | `logos1` |
| Comparisons | `compare` (10 variants) | `compare1` |

---

## Selecting the Right Component

Consult `references/component-catalog.md` for the full catalog. Key groups:

| Need | Component Group | Example |
|------|----------------|---------|
| Form layout | Form (86) | `form-signin-1` |
| Labeled input | Field (38) | `field-standard-1` |
| Input with addon | Input Group (39) | `input-group-standard-1` |
| Searchable select | Combobox (42) | `combobox-standard-1` |
| File upload | File Upload (44) | `file-upload-standard-1` |
| Date picker | Date Picker (8) | `date-picker-standard-1` |
| Alert message | Alert (25) | `alert-error-1` |
| Confirm dialog | Alert Dialog (39) | `alert-dialog-standard-1` |
| Loading skeleton | Skeleton (30) | `skeleton-standard-1` |
| Toast notification | Sonner (24) | `sonner-standard-1` |
| Empty state | Empty (22) | `empty-standard-1` |
| Modal dialog | Dialog (17) | `dialog-standard-1` |
| Side panel | Sheet (29) / Drawer (22) | `sheet-standard-1` |
| Accordion | Accordion (21) | `accordion-standard-1` |
| Data chart | Chart (70) | `chart-standard-1` |
| Command palette | Command (21) | `command-standard-1` |
| Dropdown menu | Dropdown Menu (30) | `dropdown-menu-standard-1` |
| Keyboard shortcuts | KBD (39) | `kbd-standard-1` |

---

## Core Workflow

### 1. Ensure Project Foundation

Verify the project has shadcn/ui initialized. If not:

```bash
npx shadcn@latest init
```

This creates `components.json` in the project root.

### 2. Configure ShadcnBlocks Registry

Check if `components.json` already has the `@shadcnblocks` registry. If not, run:

```bash
bash "${CLAUDE_PLUGIN_ROOT}/skills/shadcn-ui/scripts/setup-shadcnblocks.sh" .
```

This adds the `@shadcnblocks` registry to `components.json` and sets `SHADCNBLOCKS_API_KEY` in `.env`.

If the script can't run (e.g., `jq` missing), configure manually:

1. Add to `.env`: `SHADCNBLOCKS_API_KEY=<key>`
2. Add registry to `components.json`:
```json
{
  "registries": {
    "@shadcnblocks": {
      "url": "https://shadcnblocks.com/r/{name}",
      "headers": {
        "Authorization": "Bearer ${SHADCNBLOCKS_API_KEY}"
      }
    }
  }
}
```

### 3. Select and Install Blocks

Use the intelligent selection flow (above) to recommend the best blocks, then install:

```bash
# Blocks (full page sections)
npx shadcn add @shadcnblocks/hero125
npx shadcn add @shadcnblocks/pricing3

# Components (UI elements)
npx shadcn add @shadcnblocks/accordion-standard-1
npx shadcn add @shadcnblocks/file-upload-standard-1
```

### 4. Compose the Page

Import and compose blocks in the page layout:

```tsx
import { Navbar1 } from "@/components/navbar1";
import { Hero125 } from "@/components/hero125";
import { Feature3 } from "@/components/feature3";
import { Footer1 } from "@/components/footer1";

export default function LandingPage() {
  return (
    <>
      <Navbar1 />
      <Hero125 />
      <Feature3 />
      <Footer1 />
    </>
  );
}
```

Blocks are fully editable source code, not locked dependencies. Customize props and content after installation.

## Landing Page Assembly Pattern

For a typical marketing landing page, use the intelligent selection flow for each section. A common assembly:

```bash
npx shadcn add @shadcnblocks/navbar1      # Simple nav with CTA
npx shadcn add @shadcnblocks/hero125      # Modern centered hero
npx shadcn add @shadcnblocks/logos1        # Trust logos strip
npx shadcn add @shadcnblocks/feature3     # 3-column feature cards
npx shadcn add @shadcnblocks/stats1       # Metrics bar
npx shadcn add @shadcnblocks/testimonial1 # Social proof grid
npx shadcn add @shadcnblocks/pricing3     # 3-tier pricing cards
npx shadcn add @shadcnblocks/faq1         # Accordion FAQ
npx shadcn add @shadcnblocks/cta1         # Final call to action
npx shadcn add @shadcnblocks/footer1      # Multi-column footer
```

Each of these was selected for a "modern SaaS" tone. For different tones (enterprise, minimal, bold, playful), the selection flow will recommend different variants.

## API Key Management

The ShadcnBlocks API key can be provided in multiple ways (checked in order):

1. **Environment variable** (simplest): Set `SHADCNBLOCKS_API_KEY` in your shell or `.env` file
2. **1Password CLI**: The helper script uses `op read` to retrieve the key. Set `OP_SHADCNBLOCKS_REF` to your 1Password reference path, or it defaults to `op://Platform Infra/ShadcnBlocks API Key/credential`

Retrieve the key using the helper script:
```bash
bash "${CLAUDE_PLUGIN_ROOT}/skills/shadcn-ui/scripts/get-api-key.sh"
```

**Security:** Never hardcode the API key in source files. Always use `.env` with `SHADCNBLOCKS_API_KEY` and ensure `.env` is in `.gitignore`.

## Troubleshooting

| Issue | Resolution |
|-------|-----------|
| Auth error on install | Verify `SHADCNBLOCKS_API_KEY` is set in `.env` with a valid key |
| Registry not found | Check `registries` key exists in `components.json` |
| `components.json` missing | Run `npx shadcn@latest init` |
| 1Password access | Run `op signin`; verify vault with `op vault list` |
| Block not found | Check block name at https://shadcnblocks.com |

## Reference Files

- **`references/block-index.md`** — Tagged picks with selection prompts for intelligent block recommendation (primary source)
- **`references/block-catalog.md`** — Full catalog of 1,338 blocks across 71 categories with variant numbers and install commands
- **`references/component-catalog.md`** — Full catalog of 1,189 components across 60+ groups with install syntax
- **`references/setup-guide.md`** — Detailed step-by-step setup with troubleshooting

## Scripts

- **`scripts/setup-shadcnblocks.sh`** — Automated project setup (retrieves key, configures registry)
- **`scripts/get-api-key.sh`** — Retrieve API key (env var or 1Password)
