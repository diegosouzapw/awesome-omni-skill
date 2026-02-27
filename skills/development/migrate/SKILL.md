---
name: migrate
description: Guide migration to Astro from other frameworks or between Astro versions. Use when converting Next.js, Nuxt, Gatsby projects or upgrading Astro.
---

# Migrate to Astro

## When to Use

- Migrating from Next.js, Nuxt, Gatsby, or other frameworks
- Upgrading between major Astro versions
- Converting a static HTML site to Astro
- Moving from Create React App to Astro

## Instructions

### 1. Identify Migration Path

| From | Complexity | Key Changes |
|------|------------|-------------|
| **Static HTML** | Low | Wrap in `.astro`, add layouts |
| **Next.js (Pages)** | Medium | Route structure similar, convert JSX |
| **Next.js (App)** | Medium-High | Server components map well to Astro |
| **Nuxt** | Medium | Vue components work with integration |
| **Gatsby** | Medium | GraphQL → Content Collections |
| **Create React App** | Medium | Add routing, convert to islands |
| **Astro v3 → v4** | Low | Minor breaking changes |
| **Astro v4 → v5** | Low-Medium | Content layer changes |

### 2. Project Setup

Create new Astro project alongside existing:

```bash
npm create astro@latest my-astro-site
```

Or convert in-place by installing Astro:

```bash
npm install astro
```

Update `package.json` scripts:

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview"
  }
}
```

### 3. Migration Patterns

#### Static HTML → Astro

1. Rename `.html` to `.astro`
2. Move to `src/pages/`
3. Extract repeated sections to `src/layouts/`
4. Convert to components as needed

```astro
---
// src/layouts/Layout.astro
const { title } = Astro.props
---
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>{title}</title>
  </head>
  <body>
    <slot />
  </body>
</html>
```

#### Next.js → Astro

**Pages Router:**

| Next.js | Astro |
|---------|-------|
| `pages/index.js` | `src/pages/index.astro` |
| `pages/blog/[slug].js` | `src/pages/blog/[slug].astro` |
| `getStaticProps` | Frontmatter code block |
| `getStaticPaths` | `getStaticPaths()` export |
| `_app.js` | `src/layouts/Layout.astro` |
| `next/image` | `astro:assets` Image component |
| `next/link` | Standard `<a>` tags |

**Convert a Next.js page:**

```jsx
// Next.js: pages/blog/[slug].js
export async function getStaticPaths() {
  const posts = await getPosts()
  return { paths: posts.map(p => ({ params: { slug: p.slug } })) }
}

export async function getStaticProps({ params }) {
  const post = await getPost(params.slug)
  return { props: { post } }
}

export default function Post({ post }) {
  return <article><h1>{post.title}</h1></article>
}
```

```astro
---
// Astro: src/pages/blog/[slug].astro
import { getCollection } from 'astro:content'

export async function getStaticPaths() {
  const posts = await getCollection('blog')
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }))
}

const { post } = Astro.props
const { Content } = await post.render()
---

<article>
  <h1>{post.data.title}</h1>
  <Content />
</article>
```

**Keep React components:**

```bash
npx astro add react
```

Then use with client directives:

```astro
---
import InteractiveWidget from '../components/InteractiveWidget.tsx'
---

<InteractiveWidget client:visible />
```

#### Nuxt → Astro

```bash
npx astro add vue
```

| Nuxt | Astro |
|------|-------|
| `pages/` | `src/pages/` (similar) |
| `components/` | `src/components/` |
| `layouts/` | `src/layouts/` |
| `nuxt.config.ts` | `astro.config.mjs` |
| Auto-imports | Explicit imports |

Vue components work directly:

```astro
---
import MyVueComponent from '../components/MyVueComponent.vue'
---

<MyVueComponent client:load />
```

#### Gatsby → Astro

| Gatsby | Astro |
|--------|-------|
| GraphQL queries | Content Collections |
| `gatsby-image` | `astro:assets` Image |
| `gatsby-plugin-*` | Astro integrations |
| MDX pages | MDX in content collections |

**Convert GraphQL to Content Collections:**

```javascript
// Gatsby: gatsby-node.js with GraphQL
// → Astro: src/content/config.ts with Zod schema

import { defineCollection, z } from 'astro:content'

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    date: z.coerce.date(),
    // Match your Gatsby frontmatter fields
  }),
})

export const collections = { blog }
```

### 4. Astro Version Upgrades

**Upgrade command:**

```bash
npx @astrojs/upgrade
```

**v4 → v5 Key Changes:**

- Content collections now use "Content Layer" API
- `astro:content` imports updated
- Some deprecated features removed

Check migration guide: [docs.astro.build/guides/upgrade-to/v5](https://docs.astro.build/en/guides/upgrade-to/v5/)

### 5. Common Migration Tasks

#### Move static assets

```
public/images/ → public/images/  (same)
src/assets/   → src/assets/      (processed by Vite)
```

#### Update image imports

```astro
---
import { Image } from 'astro:assets'
import heroImage from '../assets/hero.jpg'
---

<Image src={heroImage} alt="Hero" width={800} height={400} />
```

#### Convert CSS

- Global CSS → Import in layout
- CSS Modules → Work as-is
- Styled Components → Use with React integration
- Tailwind → `npx astro add tailwind`

#### Environment variables

```
// Next.js
NEXT_PUBLIC_API_URL

// Astro
PUBLIC_API_URL  // Client-side
API_SECRET      // Server-side only
```

### 6. Verification Checklist

- [ ] All routes render correctly
- [ ] Images load and are optimized
- [ ] Interactive components hydrate
- [ ] SEO meta tags present
- [ ] Build completes without errors
- [ ] Links work (no 404s)
- [ ] Forms submit correctly
- [ ] Performance improved or maintained

## Resources

- [Official Migration Guides](https://docs.astro.build/en/guides/migrate-to-astro/)
- [From Next.js](https://docs.astro.build/en/guides/migrate-to-astro/from-nextjs/)
- [From Nuxt](https://docs.astro.build/en/guides/migrate-to-astro/from-nuxt/)
- [From Gatsby](https://docs.astro.build/en/guides/migrate-to-astro/from-gatsby/)
