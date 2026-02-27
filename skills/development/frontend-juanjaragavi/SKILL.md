---
name: frontend
description: Component architecture, UI patterns, design system integration, accessibility standards, and framework-specific conventions for TopNetworks properties built on Next.js 15+ and Astro 5. Use when building pages, components, or layouts for any TopNetworks project.
---

# Frontend — TopNetworks, Inc.

This skill governs all frontend development across TopNetworks properties. Apply it alongside the `brand-identity` skill. Every rule here is derived from production codebases (topfinanzas-us-next, uk-topfinanzas-com, budgetbee-next, kardtrust, mejoresfinanzas).

---

## Scope

**Use for:** Next.js page components, React components, Astro pages, Tailwind CSS styling, MDX content, form handling, navigation, analytics integration, and any client/server component work.

**Not for:** API route logic (see `backend` skill), database queries (see `database` skill), or auth flows (see `authentication` skill).

---

## Framework Versions

| Property                | Framework | Version | Router     |
| ----------------------- | --------- | ------- | ---------- |
| topfinanzas-us-next     | Next.js   | 15.x    | App Router |
| uk-topfinanzas-com      | Next.js   | 15.x    | App Router |
| topfinanzas-mx-next     | Next.js   | 15.x    | App Router |
| budgetbee-next          | Next.js   | 15.x    | App Router |
| kardtrust               | Next.js   | 15.x    | App Router |
| route-genius            | Next.js   | 16.1.x  | App Router |
| mejoresfinanzas         | Astro     | 5.0     | File-based |
| financial-blog-template | Astro     | 5.0     | File-based |

---

## Directory Structure (Next.js App Router)

```
app/
├── layout.tsx              # Root layout — loads fonts, GTM, global providers
├── page.tsx                # Homepage (Server Component)
├── api/                    # API routes (see backend skill)
├── blog/
│   └── page.tsx            # Blog listing — contains allPosts array
├── personal-finance/
│   ├── page.tsx            # Category listing — contains allPosts array
│   └── [slug]/
│       └── page.tsx        # Individual article (Server Component)
├── financial-solutions/
│   ├── page.tsx            # Product listing
│   └── [product-name]/
│       ├── page.tsx        # Benefits page (MANDATORY layout standard)
│       └── requirements/
│           └── page.tsx    # Requirements page (MANDATORY layout standard)
components/
├── layout/
│   ├── header.tsx
│   ├── footer.tsx
│   └── compact-footer.tsx
├── ui/
│   ├── button.tsx          # shadcn/ui base
│   ├── ai-content-disclaimer.tsx
│   └── ...
├── mdx/
│   └── blog-layout.tsx     # Blog article wrapper with sidebar
lib/
├── logger.ts               # Pino logger (NEVER use console.log)
├── utils.ts                # cn() and shared utilities
├── navigation/
│   └── headerNavigation.ts
└── documents/
    └── commit-message.txt  # Git workflow file
```

---

## Component Patterns

### Server vs Client Components

Default to Server Components. Add `"use client"` only when the component requires:

- `useState` / `useReducer` / `useEffect`
- Browser APIs (`window`, `document`, `localStorage`)
- Event handlers (`onClick`, `onChange`, etc.)
- Third-party client-only libraries

```typescript
// ✅ Server Component (default) — no directive needed
export default function ArticlePage() {
  return <article>...</article>;
}

// ✅ Client Component — explicit directive at top of file
"use client";
import { useState } from "react";

export default function FilterPanel() {
  const [active, setActive] = useState(false);
  return <div onClick={() => setActive(!active)}>...</div>;
}
```

### Standard Page Component Shape

```typescript
import type { Metadata } from "next";
import Image from "next/image";
import Link from "next/link";
import { Header } from "@/components/layout/header";
import { CompactFooter } from "@/components/layout/compact-footer";
import { AIContentDisclaimer } from "@/components/ui/ai-content-disclaimer";

export function generateMetadata(): Metadata {
  return {
    title: "Page Title | TopFinanzas US",
    description: "Compelling description (150–160 characters)",
    keywords: "keyword1, keyword2, keyword3",
  };
}

export default function PageNamePage() {
  return (
    <main className="bg-white min-h-screen flex flex-col">
      <Header />
      <article className="bg-white py-8 md:py-12" data-category="personal-finance">
        <div className="container mx-auto px-4">
          <div className="max-w-4xl mx-auto">
            <h1 className="text-3xl md:text-4xl font-bold text-gray-800 mb-6 text-left">
              H1 Title
            </h1>
            <div id="square01" data-topads data-topads-size="square"></div>
            {/* content */}
            <div id="square02" data-topads data-topads-size="square"></div>
            <AIContentDisclaimer />
          </div>
        </div>
      </article>
      <CompactFooter />
    </main>
  );
}
```

### Financial Solutions Layout Standard (MANDATORY)

Product pages (credit cards, loans) follow an exact, non-negotiable element order validated against GA and Google Ads data. **Any deviation negatively impacts SEO and conversion rates.**

**Benefits Page Order:**

1. `<Header />`
2. `<h1>` — product name
3. Opening paragraph (≥2 sentences)
4. First ad unit (`id="square01"`)
5. `<hr />` separator
6. Key Features section (plain text — NO bullet lists, NO grid)
7. "View Requirements" button (links to `/requirements` sub-page)
8. Hero image (`<ResponsiveImage>` component)
9. Content sections with prose
10. Second ad unit (`id="square02"`)
11. Related Articles section
12. Additional features prose
13. Final CTA (external affiliate link)
14. `<AIContentDisclaimer />`
15. `<CompactFooter />`

**Requirements Page Order:**

1. `<Header />`
2. `<h1>` — "Requirements for [Product]"
3. First ad unit (`id="square01"`)
4. Opening paragraph
5. Hero image (`<Image>` — not `<ResponsiveImage>`)
6. How to Qualify section (prose, no bullets)
7. Second ad unit (`id="square02"`)
8. Documentation requirements (prose)
9. Costs and fees (prose)
10. Promotional images (if available)
11. Benefits overview (links back to benefits page)
12. Target audience description
13. Final external CTA button
14. `<AIContentDisclaimer />`
15. `<CompactFooter />`

**FORBIDDEN on all Financial Solutions pages:**

- `<ul>` bullet lists
- CSS grid layouts
- Colored background boxes / highlight cards
- Special CTA boxes with background colors
- Inline SVG icons
- Representative example boxes

**Image component selection:**

- Benefits page hero: use `<ResponsiveImage>` (responsive wrapper component)
- Requirements page hero: use `<Image>` from `next/image` directly

---

## Styling Conventions

### Tailwind CSS

All properties use Tailwind CSS. Version varies: 3.x for Next.js 14/15 projects, 4.x for route-genius.

```typescript
// Always use cn() for conditional class merging
import { cn } from "@/lib/utils";

<div className={cn("base-class", isActive && "active-class", className)} />
```

Standard typography classes (match existing pages):

```
H1: text-3xl md:text-4xl font-bold text-gray-800 mb-6 text-left
H2: text-2xl font-semibold text-gray-800 mb-4
H3: text-xl font-medium text-gray-700 mb-3
Body: text-lg text-gray-700 mb-6 leading-7
Secondary: text-gray-600 leading-6
```

Standard spacing: `mb-6` between paragraphs, `my-8` for sections, `py-8 md:py-12` for page-level padding.

Mobile-first responsive: always define mobile styles first, use `md:` and `lg:` prefixes for larger breakpoints.

### Shadcn/UI + Radix

Used across all Next.js financial properties:

```typescript
// Import from @/components/ui/ — never from shadcn source directly
import { Button } from "@/components/ui/button";
import { Dialog } from "@/components/ui/dialog";
import { Select } from "@/components/ui/select";
```

Do not install new shadcn components without verifying the component doesn't already exist in the project's `components/ui/` directory.

### Icons

Use `lucide-react` exclusively. Do not import from `@heroicons/react`, `react-icons`, or any other icon library.

```typescript
import { ArrowRight, ChevronDown, ExternalLink } from "lucide-react";
```

---

## Image Handling

Always use `next/image` (`<Image>`). Never use bare `<img>` tags.

```typescript
import Image from "next/image";

// Standard usage
<Image
  src="https://storage.googleapis.com/media-topfinanzas-com/images/us/hero.webp"
  alt="Descriptive alt text — not keyword-stuffed"
  width={800}
  height={450}
  className="w-full h-auto rounded-xl"
  priority={false}
  loading="lazy"
/>

// Above-the-fold hero (LCP element)
<Image
  src="..."
  alt="..."
  width={1200}
  height={630}
  priority={true}  // removes loading="lazy"
  className="w-full h-auto"
/>
```

Remote patterns must be configured in `next.config.mjs`:

```javascript
images: {
  remotePatterns: [
    { protocol: "https", hostname: "storage.googleapis.com" },
    { protocol: "https", hostname: "lh3.googleusercontent.com" },
  ],
},
```

---

## Form Handling

Multi-step forms (credit card quizzes, recommenders) follow this pattern:

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const schema = z.object({
  field: z.string().min(1, "Required"),
});

export default function StepForm() {
  const form = useForm({ resolver: zodResolver(schema) });

  function onStepChange() {
    // ALWAYS scroll to top on step transitions
    window.scrollTo(0, 0);
  }
}
```

**Critical form rules:**

- Always call `window.scrollTo(0, 0)` on step transitions
- Use `localStorage` AND cookies for persistence (dual storage)
- Track step progress with a `<ProgressIndicator>` component

---

## Analytics Integration

### GTM (Google Tag Manager)

Loads in `app/layout.tsx` via `@next/third-parties`. Must load before AdZep. Never initialize GTM manually.

```typescript
import { GoogleTagManager } from "@next/third-parties/google";
// In layout:
<GoogleTagManager gtmId={process.env.NEXT_PUBLIC_GTM_ID!} />
```

### TopAds

TopAds ad units activate automatically on navigation. Never call `window.AdZepActivateAds()` manually — this is handled by the `AdZepNavigationHandler` component which uses `usePathname()` hook and `popstate` events.

Ad container pattern:

```html
<div id="square01" data-topads data-topads-size="square"></div>
```

### UTM Tracking

UTM parameters follow strict format: `[country]_tf_[platform]_broad`

- Capture on landing, persist throughout the user journey via cookie
- US example: `us_tf_meta_broad`

---

## Blog Post Synchronization

**CRITICAL — never skip this step.**

When creating, updating, or deleting any blog post, you MUST update ALL of these arrays:

1. `app/blog/page.tsx` — `allPosts` array
2. `app/personal-finance/page.tsx` — `allPosts` array (or the relevant category page)
3. `app/financial-solutions/page.tsx` — `allPosts` array (if applicable)
4. `components/mdx/blog-layout.tsx` — sidebar recent articles
5. `app/page.tsx` — homepage featured posts (for flagship articles)
6. `lib/navigation/headerNavigation.ts` — `featuredPosts` (when strategically justified)

Array entry format:

```typescript
{
  title: "Article Title | TopFinanzas US",
  slug: "article-slug",            // kebab-case, matches directory name
  description: "160-char description",
  image: "https://storage.googleapis.com/media-topfinanzas-com/images/us/article.webp",
  category: "Personal Finance",    // or "Financial Solutions"
  categoryPath: "/personal-finance",
  date: "Month DD, YYYY",          // US format
}
```

---

## Logging

**NEVER use `console.log()`, `console.warn()`, or `console.error()` directly.**

```typescript
// ❌ Wrong
console.log("Data:", data);

// ✅ Correct
import { logger } from "@/lib/logger";
logger.info({ data }, "Data retrieved");
logger.error({ error }, "Operation failed");
```

Log levels: `trace` < `debug` < `info` < `warn` < `error` < `fatal`. Logger is auto-disabled in production unless `NEXT_PUBLIC_ENABLE_LOGGING=true`.

---

## Accessibility

Minimum WCAG AA compliance:

- All interactive elements need `aria-label` when icon-only
- Color contrast ratio ≥ 4.5:1 for text (3:1 for large text)
- All images need non-empty `alt` attributes (empty `alt=""` only for purely decorative images)
- Keyboard navigation: all interactive elements reachable via Tab
- Use semantic HTML: `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<header>`, `<footer>`

---

## Import Organization

Always order imports as:

```typescript
// 1. React
import { useState, useEffect } from "react";

// 2. Next.js
import Link from "next/link";
import Image from "next/image";
import type { Metadata } from "next";

// 3. Third-party
import { Button } from "@/components/ui/button";
import { cn } from "@/lib/utils";

// 4. Local
import { CustomComponent } from "./custom-component";
```

---

## File Naming

- Components: `kebab-case.tsx` (e.g., `credit-card-form.tsx`)
- Pages: `page.tsx` (App Router convention)
- Layouts: `layout.tsx`
- API routes: `route.ts`
- Utilities: descriptive names in `lib/utils/`
- Instruction files: `UPPER_CASE_WITH_UNDERSCORES.instructions.md`

---

## Constraints

- No `any` types without explicit justification
- No inline styles (use Tailwind classes)
- No bare `<img>` tags — always `<Image>`
- No direct `console.*` calls — always use `logger`
- No importing shadcn components directly from source — use `@/components/ui/`
- No `find` pattern for routing — always use Next.js App Router file conventions
- TypeScript strict mode is enabled; all components must be fully typed
