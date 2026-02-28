---
name: service-website-generator
description: Orchestrates automated service-based website generation with local SEO optimization. Creates 200+ service+location pages using parallel agents, Unsplash images via Jina AI, NextJS with dynamic routing, and PostgreSQL database. Use when building service business websites (plumbers, electricians, pressure washing, HVAC, etc.) targeting multiple locations.
---

# Service Website Generator

Build complete service-based websites with hundreds of local SEO pages in one automated workflow.

**Stack**: NextJS 14 + TypeScript + Tailwind CSS + PostgreSQL + Prisma ORM + Jina AI + Unsplash

## Quick Start

### Step 0: Collect User Inputs (REQUIRED)

Before proceeding, collect these inputs:
1. **Service Niche**: What service business? (e.g., "Plumber", "Pressure Washing", "HVAC")
2. **Service Area**: Main city/region (e.g., "Port Orange, Florida", "Galway, Ireland")
3. **Business Name** (optional): For personalization (e.g., "Murphy's Plumbing Services")
4. **Jina API Key**: Required for research and Unsplash image gathering
5. **HTML/CSS/JS Design** (optional): User provides OR system generates

**DO NOT proceed until you have**: Service niche, service area, and Jina API key confirmed.

### Step 1: Design Generation

**IMPORTANT**: Use the `frontend-aesthetics` skill guidelines when generating designs!

**If user did NOT provide design:**
```
Invoke: design-generator agent
Input:
  - Service niche
  - Target audience (local customers)
  - MUST follow frontend-aesthetics skill guidelines:
    - Typography: Use distinctive fonts (Plus Jakarta Sans, Space Grotesk, Cabinet Grotesk)
    - Colors: Industry-specific palettes, NOT purple gradients
    - Motion: CSS animations for delight
    - Backgrounds: Layered gradients, subtle patterns
    - Tailwind v4: Use gap-* not space-x-*, no custom CSS resets
Output: /design/index.html, /design/styles.css, /design/script.js
```

**If user DID provide design:**
Save their design to `/design/` folder.

### Step 2: Location Discovery

```
Invoke: location-generator agent
Input: Service area, service niche, Jina API key
Output: /locations.json (20-50+ locations with metadata)
```

Discovers all towns, suburbs, and neighborhoods within service radius (typically 15-50 miles depending on geography).

### Step 3: Service Schema Creation

```
Invoke: service-schema-creator agent
Input: Service niche, Jina API key, sample locations
Output: /service-schema-template.json (5-15 services with comprehensive schema)
```

### Step 4: Database Setup

```
Invoke: database-agent agent
Input: Project directory, service niche, service area
Output: PostgreSQL + Prisma ORM configured
```

Creates tables: ContactForm, QuoteRequest, CallbackRequest, PageView, EmailSubscriber, ServiceAreaLead

### Step 5: Calculate Page Generation Strategy

```
Total pages = Services × Locations
Agents needed = Total pages ÷ 12 (average pages per agent)

Example: 15 services × 48 locations = 720 pages → 60 agents
```

### Step 6: Spawn Page Generators (PARALLEL)

**CRITICAL: Spawn ALL agents simultaneously, not sequentially!**

```
Invoke: service-page-generator agents (N agents in parallel)
Input per agent:
  - Service schema template path
  - Assigned service+location combinations (10-15 pages)
  - Jina API key (for Unsplash image scraping)
  - Service niche context
Output: /pages/*.json (10-15 files per agent)
```

Each agent:
- Searches Unsplash via Jina for relevant images
- Generates service descriptions, benefits, process, FAQ
- Creates complete JSON files with all data + images

### Step 7: NextJS Site Build

```
Invoke: nextjs-builder agent
Input: Design files, all JSON pages, service schema, locations
Output: /website/ folder with complete NextJS app
```

Creates:
- Homepage with service overview
- Main services page
- Main locations page
- Individual service+location pages (e.g., /emergency-plumber-athenry)
- Service category pages
- Location pages
- Click-to-call functionality
- Trust signals and CTAs

### Step 8: Playwright Testing

```bash
# Start dev server in background
cd [project]/website && npm run dev &

# Invoke playwright-tester agent
# Tests all pages, validates SEO, checks images

# Kill dev server after tests
kill [PID]
```

### Step 9: GitHub Deployment

```bash
cd [project-directory]
git init
git add -A
git commit -m "Initial commit: [Service Niche] in [Service Area] website"
gh repo create [repo-name] --public --source=. --push
```

If `gh` CLI unavailable, provide manual instructions.

## Available Agents

| Agent | Purpose | Invocation |
|-------|---------|------------|
| `business-researcher` | Research specific business (if name provided) | Once, Step 1 |
| `design-generator` | Generate HTML/CSS/JS design | Once, Step 1 |
| `location-generator` | Discover locations in service area | Once, Step 2 |
| `service-schema-creator` | Create service list and schema | Once, Step 3 |
| `database-agent` | Set up PostgreSQL + Prisma | Once, Step 4 |
| `service-page-generator` | Generate 10-15 pages with images | N agents, parallel |
| `nextjs-builder` | Build complete NextJS website | Once, Step 7 |
| `playwright-tester` | Test all pages and functionality | Once, Step 8 |

## Design & Style Guidelines

**CRITICAL: Follow the `frontend-aesthetics` skill to avoid "AI slop"!**

### Typography (from frontend-aesthetics)
- **Never use**: Inter, Roboto, Open Sans, Lato, Arial, system fonts
- **Good choices**: Plus Jakarta Sans, Space Grotesk, Cabinet Grotesk, Outfit, Bricolage Grotesque
- **Use extremes**: 100/200 weight vs 800/900, size jumps of 3x+

### Colors by Service Type
- **Pressure Washing/Cleaning**: Strong blue (#1e40af) + Orange (#ea580c) + Green (#059669)
- **Plumbing**: Water blue (#0284c7) + Emergency red (#dc2626) + Eco green (#16a34a)
- **HVAC**: Cool blue (#0369a1) + Warm orange (#ea580c) + Green (#059669)
- **Electrical**: Electric yellow (#eab308) + Professional blue (#1e40af) + Red (#dc2626)
- **Landscaping**: Natural green (#15803d) + Earth brown (#92400e) + Blue (#0284c7)
- **Marketing/B2B**: Navy (#0A66C2) + Purple accent (#8B5CF6) + Success green (#10B981)

### Tailwind v4 Compatibility (CRITICAL)
- **Never use**: `space-x-*`, `space-y-*` (removed in v4)
- **Always use**: `gap-*` with flex/grid containers
- **Don't add**: Custom `* { margin: 0 }` resets (breaks mx-auto)
- **Add**: `@config "../tailwind.config.ts"` to CSS if using config

### Service Website Requirements
- Strong CTAs (Call Now buttons, Get Quote forms)
- Click-to-call functionality on mobile
- Trust signals (certifications, guarantees, reviews)
- Local phone numbers prominently displayed
- Service area coverage clearly shown
- Before/after image galleries
- Motion/animations for page loads and interactions

## File Structure

```
project-root/
├── design/
│   ├── index.html
│   ├── styles.css
│   └── script.js
├── pages/
│   └── [service]-[location].json (200+ files)
├── website/
│   ├── app/
│   │   ├── [slug]/page.tsx
│   │   ├── services/page.tsx
│   │   ├── locations/page.tsx
│   │   └── ...
│   ├── components/
│   └── package.json
├── prisma/
│   └── schema.prisma
├── locations.json
├── service-schema-template.json
└── business-profile.json (if business name provided)
```

## Example Workflow

**User**: "Make me a service website for pressure washing in Port Orange, Florida"

1. Collect inputs (niche: pressure washing, area: Port Orange FL, Jina key)
2. Generate design with trust signals, CTAs, blue/orange color scheme
3. Discover 48 locations within 15-mile radius
4. Create 15 pressure washing services schema
5. Set up PostgreSQL with 6 lead capture tables
6. Calculate: 15 × 48 = 720 pages → spawn ~60 agents
7. Generate all service+location JSON files with Unsplash images
8. Build NextJS site with 214+ pages
9. Run Playwright tests (target: 80%+ pass rate)
10. Push to GitHub
11. Report: Repository URL, run instructions, deploy instructions

## Critical Rules

**DO:**
- Collect ALL inputs before starting
- Generate design FIRST if not provided
- Discover locations BEFORE creating pages
- Create service schema BEFORE generating pages
- Spawn ALL page generator agents SIMULTANEOUSLY
- Test with Playwright BEFORE deployment
- Provide clear deployment instructions

**NEVER:**
- Skip input collection phase
- Proceed without Jina API key
- Spawn agents sequentially (must be parallel!)
- Build NextJS site before all pages ready
- Skip Playwright testing
- Leave user without next steps

## Success Criteria

- [ ] Design exists (generated OR user-provided)
- [ ] 20-50+ locations discovered
- [ ] 5-15 services in schema
- [ ] 200+ service+location pages generated
- [ ] All pages have Unsplash images
- [ ] NextJS website built and running
- [ ] Playwright tests pass (80%+ rate)
- [ ] Code committed to git
- [ ] User has deployment instructions
