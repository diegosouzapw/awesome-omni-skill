---
name: astro-setup
description: Astro project initialization and configuration patterns. Use when setting up new Astro projects or configuring Astro features.
---

# Astro Setup Skill

## Project Initialization

```bash
npm create astro@latest project-name -- --template minimal --typescript strict
cd project-name
```

## Essential Integrations

```bash
npx astro add react
npx astro add mdx
npx astro add tailwind
```

## Configuration Pattern

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import mdx from '@astrojs/mdx';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  site: 'https://username.github.io',
  base: '/repo-name', // or '/' for user pages
  integrations: [react(), mdx(), tailwind()],
  output: 'static', // Required for GitHub Pages
  image: {
    service: {
      entrypoint: 'astro/assets/services/sharp'
    }
  }
});
```

## Content Collections

Content collections go in `src/content/` with a `config.ts` file defining schemas using Zod.

## Best Practices

- Use `output: 'static'` for GitHub Pages
- Configure `site` and `base` for proper asset paths
- Enable image optimization
- Keep integrations minimal
