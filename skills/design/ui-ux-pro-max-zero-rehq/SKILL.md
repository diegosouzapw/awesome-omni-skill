---
name: ui-ux-pro-max
description: 57 UI styles + 95 color palettes + 56 font pairings + 24 charts + 98 UX guidelines. Motor BM25 para búsqueda inteligente.
compatibility: opencode
trigger_keywords: ["design", "UI", "UX", "landing page", "dashboard", "color palette", "typography"]
source: NextLevelBuilder - UI/UX Pro Max Skill
---

# UI/UX Pro Max - Design Intelligence

Searchable database de diseño profesional con **300+ recursos curados**: 57 UI styles, 95 color palettes, 56 font pairings, 24 chart types, 98 UX guidelines, y 11 stack-specific guides.

## Features Principales

- **57 UI Styles** - Glassmorphism, Claymorphism, Minimalism, Brutalism, Neumorphism, Bento Grid, Dark Mode, etc.
- **95 Color Palettes** - Paletas específicas por industria (SaaS, E-commerce, Healthcare, Fintech, Beauty, etc.)
- **56 Font Pairings** - Combinaciones tipográficas curadas con Google Fonts imports
- **24 Chart Types** - Recomendaciones para dashboards y analytics
- **98 UX Guidelines** - Best practices, anti-patterns, accessibility
- **11 Tech Stacks** - React, Next.js, Vue, Nuxt.js, Nuxt UI, Svelte, SwiftUI, React Native, Flutter, HTML+Tailwind, shadcn/ui
- **Motor de Búsqueda BM25** - Búsqueda híbrida BM25 + regex para matches precisos

## Cuándo Usar

AUTO-TRIGGER cuando:
- "Build a landing page for..."
- "Create a dashboard for..."
- "Design a [product type] website"
- "Make a mobile app UI for..."
- Task menciona: color palette, typography, font pairing, UI style, layout

## Prerequisitos

**Python 3.x requerido** para el motor de búsqueda:

```bash
# Verificar si Python está instalado
python3 --version || python --version

# macOS
brew install python3

# Ubuntu/Debian
sudo apt update && sudo apt install python3

# Windows
winget install Python.Python.3.12
```

---

## Workflow de Uso

### Step 1: Analizar User Requirements

Extraer info clave del request:
- **Product type**: SaaS, e-commerce, portfolio, dashboard, landing page, etc.
- **Style keywords**: minimal, playful, professional, elegant, dark mode, etc.
- **Industry**: healthcare, fintech, gaming, education, etc.
- **Stack**: React, Vue, Next.js, o default a `html-tailwind`

### Step 2: Search Relevant Domains

Usar `search.py` múltiples veces para gather comprehensive info:

```bash
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "<keyword>" --domain <domain> [-n <max_results>]
```

**Orden recomendado de búsqueda**:
1. **product** - Style recommendations para product type
2. **style** - Detailed style guide (colors, effects, frameworks)
3. **typography** - Font pairings con Google Fonts imports
4. **color** - Color palette (Primary, Secondary, CTA, Background, Text, Border)
5. **landing** - Page structure (si landing page)
6. **chart** - Chart recommendations (si dashboard/analytics)
7. **ux** - Best practices y anti-patterns
8. **stack** - Stack-specific guidelines (default: html-tailwind)

### Step 3: Stack Guidelines

Si user NO especifica stack, **default to `html-tailwind`**.

```bash
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "<keyword>" --stack html-tailwind
```

**Available stacks**: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`

---

## Available Domains

| Domain | Use For | Example Keywords |
|--------|---------|------------------|
| `product` | Product type recommendations | SaaS, e-commerce, portfolio, healthcare, beauty |
| `style` | UI styles, colors, effects | glassmorphism, minimalism, dark mode, brutalism |
| `typography` | Font pairings, Google Fonts | elegant, playful, professional, modern |
| `color` | Color palettes by product type | saas, ecommerce, healthcare, beauty, fintech |
| `landing` | Page structure, CTA strategies | hero, testimonial, pricing, social-proof |
| `chart` | Chart types, library recommendations | trend, comparison, timeline, funnel, pie |
| `ux` | Best practices, anti-patterns | animation, accessibility, z-index, loading |
| `prompt` | AI prompts, CSS keywords | (style name) |

---

## Available Stacks

| Stack | Focus |
|-------|-------|
| `html-tailwind` | Tailwind utilities, responsive, a11y (DEFAULT) |
| `react` | State, hooks, performance, patterns |
| `nextjs` | SSR, routing, images, API routes |
| `vue` | Composition API, Pinia, Vue Router |
| `svelte` | Runes, stores, SvelteKit |
| `swiftui` | Views, State, Navigation, Animation |
| `react-native` | Components, Navigation, Lists |
| `flutter` | Widgets, State, Layout, Theming |
| `shadcn` | shadcn/ui components, theming, forms, patterns |
| `nuxt-ui` | Nuxt UI components, Nuxt.js ecosystem |
| `nuxtjs` | Nuxt.js patterns, SSR, Vue Router |

---

## Data Intelligence: 57 UI Styles

Los 57 estilos UI cubren todo el espectro de diseño moderno:

**Popular Styles**:
1. **Glassmorphism** - Frosted glass effect, backdrop-blur, transparency
2. **Claymorphism** - Soft 3D, inflated shapes, inner shadows
3. **Minimalism** - Clean, white space, simple typography
4. **Brutalism** - Raw, bold typography, asymmetric layouts
5. **Neumorphism** - Soft shadows, subtle 3D, light backgrounds
6. **Bento Grid** - Card-based layout, organized sections
7. **Dark Mode** - Dark backgrounds, high contrast, OLED-friendly
8. **Flat Design** - No shadows, simple shapes, solid colors
9. **Material Design** - Elevation, paper metaphor, ripple effects
10. **Skeuomorphism** - Real-world textures, realistic shadows

**Specialty Styles**:
11. **Abstract Geometric** - Shapes, patterns, colorful
12. **Art Deco** - Luxurious, geometric patterns, gold accents
13. **Cyberpunk** - Neon, futuristic, glitch effects
14. **Memphis Design** - Bold colors, geometric shapes, playful
15. **Swiss Style** - Grid-based, sans-serif, asymmetric
16. **Retro/Vintage** - Old aesthetics, warm colors, nostalgia
17. **Organic/Nature** - Natural forms, earth tones, flowing shapes
18. **Maximalism** - Bold, abundant, layered, colorful
19. **Monochrome** - Single color palette, high contrast
20. **Gradient Mesh** - Colorful gradients, smooth transitions

**Y 37 más** (total 57) documentados en data/styles.csv

---

## Data Intelligence: 95 Color Palettes

Paletas específicas por industria y product type:

### Top 10 Product Types con Palettes

| Product Type | Primary | Secondary | CTA | Background | Text | Border |
|-------------|---------|-----------|-----|------------|------|--------|
| **SaaS (General)** | #2563EB | #3B82F6 | #F97316 | #F8FAFC | #1E293B | #E2E8F0 |
| **E-commerce** | #3B82F6 | #60A5FA | #F97316 | #F8FAFC | #1E293B | #E2E8F0 |
| **Healthcare** | #0891B2 | #22D3EE | #059669 | #ECFEFF | #164E63 | #A5F3FC |
| **Educational** | #4F46E5 | #818CF8 | #F97316 | #EEF2FF | #1E1B4B | #C7D2FE |
| **Fintech** | #F59E0B | #FBBF24 | #8B5CF6 | #0F172A | #F8FAFC | #334155 |
| **Gaming** | #7C3AED | #A78BFA | #F43F5E | #0F0F23 | #E2E8F0 | #4C1D95 |
| **Beauty/Spa** | #10B981 | #34D399 | #8B5CF6 | #ECFDF5 | #064E3B | #A7F3D0 |
| **Luxury** | #1C1917 | #44403C | #CA8A04 | #FAFAF9 | #0C0A09 | #D6D3D1 |
| **Real Estate** | #0F766E | #14B8A6 | #0369A1 | #F0FDFA | #134E4A | #99F6E4 |
| **Travel/Tourism** | #EC4899 | #F472B6 | #06B6D4 | #FDF2F8 | #831843 | #FBCFE8 |

**Y 85 palettes más** para: Micro SaaS, B2B Service, Analytics, Creative Agency, Portfolio, Social Media, Productivity, AI/Chatbot, NFT/Web3, Creator Economy, Sustainability, Mental Health, Smart Home, Legal, Insurance, Banking, Music/Video Streaming, Job Board, Marketplace, Logistics, Agriculture, Construction, Photography, Childcare, Medical, Dental, Veterinary, Florist, Coffee Shop, Airline, Consulting, Marketing, Newsletter, Sports, Museum, Language Learning, Cybersecurity, Developer Tools, Biotech, Space Tech, Architecture, Quantum Computing, Climate Tech, y más.

---

## Data Intelligence: 56 Font Pairings

Combinaciones tipográficas curadas con Google Fonts imports:

### Top 10 Font Pairings

| Pairing | Use For | Heading Font | Body Font | Google Fonts Import |
|---------|---------|--------------|-----------|---------------------|
| **Professional** | SaaS, Business | Inter (700) | Inter (400) | `@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');` |
| **Elegant** | Luxury, Beauty | Playfair Display (700) | Lato (400) | `@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&family=Lato:wght@400&display=swap');` |
| **Modern Tech** | Startups, Tech | Montserrat (700) | Open Sans (400) | `@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@700&family=Open+Sans:wght@400&display=swap');` |
| **Minimalist** | Portfolio, Design | Raleway (600) | Nunito (400) | `@import url('https://fonts.googleapis.com/css2?family=Raleway:wght@600&family=Nunito:wght@400&display=swap');` |
| **Playful** | Kids, Creative | Fredoka One (400) | Poppins (400) | `@import url('https://fonts.googleapis.com/css2?family=Fredoka+One&family=Poppins:wght@400&display=swap');` |
| **Editorial** | News, Blog | Merriweather (700) | Roboto (400) | `@import url('https://fonts.googleapis.com/css2?family=Merriweather:wght@700&family=Roboto:wght@400&display=swap');` |
| **Corporate** | Finance, Legal | Roboto Slab (700) | Source Sans Pro (400) | `@import url('https://fonts.googleapis.com/css2?family=Roboto+Slab:wght@700&family=Source+Sans+Pro:wght@400&display=swap');` |
| **Creative Agency** | Art, Design | Oswald (700) | PT Sans (400) | `@import url('https://fonts.googleapis.com/css2?family=Oswald:wght@700&family=PT+Sans:wght@400&display=swap');` |
| **Friendly** | Service, Community | Quicksand (700) | Karla (400) | `@import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@700&family=Karla:wght@400&display=swap');` |
| **Luxury Fashion** | High-end, Brands | Bodoni Moda (700) | Cormorant Garamond (400) | `@import url('https://fonts.googleapis.com/css2?family=Bodoni+Moda:wght@700&family=Cormorant+Garamond:wght@400&display=swap');` |

**Y 46 pairings más** documentados en data/typography.csv

---

## Data Intelligence: 24 Chart Types

Recommendations para visualización de datos:

### Top 10 Chart Types

| Data Type | Best Chart | Color Guidance | Library | Interactive Level |
|-----------|------------|----------------|---------|-------------------|
| **Trend Over Time** | Line Chart | Primary: #0080FF. Fill: 20% opacity | Chart.js, Recharts | Hover + Zoom |
| **Compare Categories** | Bar Chart | Each bar: distinct color. Sorted: descending | Chart.js, Recharts | Hover + Sort |
| **Part-to-Whole** | Pie/Donut | 5-6 max. Contrasting palette | Chart.js, Recharts | Hover + Drill |
| **Correlation** | Scatter Plot | Gradient blue-red. Opacity 0.6-0.8 | D3.js, Plotly | Hover + Brush |
| **Heatmap** | Heat Map | Cool (blue) to Hot (red). Clear legend | D3.js, Plotly | Hover + Zoom |
| **Geographic** | Choropleth Map | Regional gradient or categorized | D3.js, Mapbox | Pan + Zoom + Drill |
| **Funnel/Flow** | Funnel Chart | Gradient start→end. Show conversion % | D3.js, Recharts | Hover + Drill |
| **Performance** | Gauge/Bullet | Red→Yellow→Green gradient | D3.js, ApexCharts | Hover |
| **Forecast** | Line with Band | Actual: solid. Forecast: dashed + band | Chart.js, ApexCharts | Hover + Toggle |
| **Hierarchical** | Treemap | Parent: distinct. Children: lighter shades | D3.js, Recharts | Hover + Drilldown |

**Y 14 tipos más**: Sankey, Waterfall, Radar, Candlestick, Network Graph, Box Plot, Waffle Chart, Sunburst, Decomposition Tree, 3D Spatial, Streaming Area, Sentiment Word Cloud, Process Map, y más.

---

## Data Intelligence: 98 UX Guidelines

Best practices y anti-patterns para UX profesional.

### Common Rules for Professional UI

#### Icons & Visual Elements

| Rule | Do | Don't |
|------|----|----- |
| **No emoji icons** | Use SVG icons (Heroicons, Lucide, Simple Icons) | Use emojis like 🎨 🚀 ⚙️ as UI icons |
| **Stable hover states** | Use color/opacity transitions on hover | Use scale transforms that shift layout |
| **Correct brand logos** | Research official SVG from Simple Icons | Guess or use incorrect logo paths |
| **Consistent icon sizing** | Use fixed viewBox (24x24) with w-6 h-6 | Mix different icon sizes randomly |

#### Interaction & Cursor

| Rule | Do | Don't |
|------|----|----- |
| **Cursor pointer** | Add `cursor-pointer` to all clickable/hoverable cards | Leave default cursor on interactive elements |
| **Hover feedback** | Provide visual feedback (color, shadow, border) | No indication element is interactive |
| **Smooth transitions** | Use `transition-colors duration-200` | Instant state changes or too slow (>500ms) |

#### Light/Dark Mode Contrast

| Rule | Do | Don't |
|------|----|----- |
| **Glass card light mode** | Use `bg-white/80` or higher opacity | Use `bg-white/10` (too transparent) |
| **Text contrast light** | Use `#0F172A` (slate-900) for text | Use `#94A3B8` (slate-400) for body text |
| **Muted text light** | Use `#475569` (slate-600) minimum | Use gray-400 or lighter |
| **Border visibility** | Use `border-gray-200` in light mode | Use `border-white/10` (invisible) |

#### Layout & Spacing

| Rule | Do | Don't |
|------|----|----- |
| **Floating navbar** | Add `top-4 left-4 right-4` spacing | Stick navbar to `top-0 left-0 right-0` |
| **Content padding** | Account for fixed navbar height | Let content hide behind fixed elements |
| **Consistent max-width** | Use same `max-w-6xl` or `max-w-7xl` | Mix different container widths |

### Pre-Delivery Checklist

**Visual Quality**:
- [ ] No emojis used as icons (use SVG instead)
- [ ] All icons from consistent icon set (Heroicons/Lucide)
- [ ] Brand logos are correct (verified from Simple Icons)
- [ ] Hover states don't cause layout shift
- [ ] Use theme colors directly (bg-primary) not var() wrapper

**Interaction**:
- [ ] All clickable elements have `cursor-pointer`
- [ ] Hover states provide clear visual feedback
- [ ] Transitions are smooth (150-300ms)
- [ ] Focus states visible for keyboard navigation

**Light/Dark Mode**:
- [ ] Light mode text has sufficient contrast (4.5:1 minimum)
- [ ] Glass/transparent elements visible in light mode
- [ ] Borders visible in both modes
- [ ] Test both modes before delivery

**Layout**:
- [ ] Floating elements have proper spacing from edges
- [ ] No content hidden behind fixed navbars
- [ ] Responsive at 320px, 768px, 1024px, 1440px
- [ ] No horizontal scroll on mobile

**Accessibility**:
- [ ] All images have alt text
- [ ] Form inputs have labels
- [ ] Color is not the only indicator
- [ ] `prefers-reduced-motion` respected

---

## Example Workflow

**User request:** "Build a landing page for a professional beauty spa service"

**AI workflow**:

```bash
# 1. Search product type
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "beauty spa wellness service" --domain product

# 2. Search style
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "elegant minimal soft" --domain style

# 3. Search typography
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "elegant luxury" --domain typography

# 4. Search color palette
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "beauty spa wellness" --domain color

# 5. Search landing page structure
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "hero-centric social-proof" --domain landing

# 6. Search UX guidelines
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "animation" --domain ux
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "accessibility" --domain ux

# 7. Search stack guidelines (default: html-tailwind)
python3 .opencode/skill/ui-ux-pro-max/scripts/search.py "layout responsive" --stack html-tailwind
```

**Then**: Synthesize all search results and implement the design with:
- Soft pastels (Pink #FFB6C1, Sage #90EE90) + Cream + Gold accents
- Playfair Display (headings) + Lato (body)
- Hero-centric layout with social proof section
- Glassmorphism or Minimalism style
- Smooth animations respecting prefers-reduced-motion
- Responsive Tailwind utilities

---

## Tips for Better Results

1. **Be specific with keywords** - "healthcare SaaS dashboard" > "app"
2. **Search multiple times** - Different keywords reveal different insights
3. **Combine domains** - Style + Typography + Color = Complete design system
4. **Always check UX** - Search "animation", "z-index", "accessibility" for common issues
5. **Use stack flag** - Get implementation-specific best practices
6. **Iterate** - If first search doesn't match, try different keywords

---

## Integration con Orchestrator

**AUTO-TRIGGER** cuando:
- Task menciona: "landing page", "dashboard", "design", "UI", "color palette", "typography"
- Files touched: páginas, componentes UI, layouts
- Product type mencionado: SaaS, e-commerce, healthcare, etc.

**Output esperado**:
```
✅ Design system found

STYLE: Minimalism
- Clean white space
- Simple typography
- Focused content

COLORS:
- Primary: #2563EB (Trust Blue)
- Secondary: #3B82F6
- CTA: #F97316 (Orange)
- Background: #F8FAFC
- Text: #1E293B

TYPOGRAPHY:
- Heading: Inter 700
- Body: Inter 400
- Import: @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');

STRUCTURE: Hero-centric landing
- Above fold: Value prop + CTA
- Social proof: Testimonials
- Pricing section
- FAQ
- Footer with links

UX GUIDELINES:
- No emoji icons (use Lucide)
- Cursor pointer on interactive elements
- Smooth transitions (200ms)
- Light mode contrast 4.5:1 minimum
- Responsive: 320px, 768px, 1024px, 1440px

Implementing design...
```

---

## Search Engine: BM25 + Regex Hybrid

El motor de búsqueda combina:
- **BM25 ranking** para relevancia semántica
- **Regex matching** para keywords exactos
- **Domain auto-detection** cuando `--domain` se omite

**Performance**:
- Búsqueda en <100ms para datasets de 100+ entries
- Resultados rankeados por relevancia
- Soporta multiple keywords y boolean logic

---

## Stack-Specific Guides

Cada stack tiene guías específicas para implementación:

### html-tailwind (DEFAULT)
- Tailwind utilities
- Responsive design
- Accessibility (a11y)
- Dark mode con `dark:` prefix

### react
- State management (useState, useContext)
- Hooks patterns
- Performance optimization
- Component composition

### nextjs
- SSR/SSG patterns
- Image optimization (next/image)
- API routes
- File-based routing

### vue
- Composition API
- Pinia store
- Vue Router
- Reactivity system

### svelte
- Svelte Runes ($state, $derived)
- Stores
- SvelteKit routing
- Reactive declarations

### swiftui
- Views composition
- @State, @Binding
- Navigation
- Animations

### react-native
- Native components
- Navigation (React Navigation)
- FlatList performance
- Platform-specific code

### flutter
- Widgets tree
- State management (Provider, Bloc)
- Layout system
- Material/Cupertino theming

### shadcn
- shadcn/ui components
- Theming with CSS variables
- Form validation (React Hook Form + Zod)
- Dark mode patterns

---

## Referencias

- **Source**: NextLevelBuilder - UI/UX Pro Max Skill
- **Repo**: https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
- **Total Resources**: 300+ curated design resources
- **Data Format**: CSV databases for easy searching
- **Search Engine**: BM25 + regex hybrid (Python 3.x required)

---

**Version**: 1.0 (Expanded)
**Maintainer**: NextLevelBuilder
**Last Updated**: 2026-01-15
**Total**: 57 styles + 95 palettes + 56 fonts + 24 charts + 98 UX rules + 11 stacks
