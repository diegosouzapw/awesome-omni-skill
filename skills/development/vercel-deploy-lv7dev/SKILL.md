---
name: vercel-deploy
description: Deploy applications and websites to Vercel instantly. Automatically detects frameworks from package.json and returns both a preview URL and a claimable URL for transferring ownership. Use when deploying to Vercel, getting a preview URL, or sharing a live version of an app.
---

# Vercel Deploy (Claimable)

## Purpose

Instantly deploy any web application to Vercel without authentication. Returns a live preview URL and a claimable URL for the user to transfer ownership to their Vercel account.

## When to Use This Skill

- User asks to "deploy to Vercel"
- User wants a preview URL for their app
- User wants to share a live version
- User says "deploy this" and the project has a package.json

## Supported Frameworks

Automatically detected from `package.json`:
- **React**: Next.js, Remix, Create React App, Gatsby
- **Vue**: Nuxt, Vue CLI
- **Svelte**: SvelteKit, Svelte
- **Angular**: Angular CLI
- **Backend**: Express, Fastify, Hono
- **Build Tools**: Vite, Astro
- **Static**: Plain HTML/CSS

## How to Deploy

### Step 1: Package the Project

```bash
# Create a tarball of the project, excluding unnecessary files
tar -czf /tmp/deploy.tar.gz \
  --exclude='node_modules' \
  --exclude='.git' \
  --exclude='.next' \
  --exclude='dist' \
  --exclude='.env' \
  --exclude='.env.local' \
  -C /path/to/project .
```

### Step 2: Detect Framework

Read `package.json` and identify the framework from dependencies:
- `next` -> Next.js
- `nuxt` -> Nuxt
- `@sveltejs/kit` -> SvelteKit
- `vite` -> Vite
- `remix` -> Remix
- `astro` -> Astro
- `gatsby` -> Gatsby
- `express` -> Express
- `fastify` -> Fastify

### Step 3: Deploy via Vercel API

```bash
# Deploy using Vercel CLI (if available)
npx vercel deploy --yes

# Or use the Vercel API directly
curl -X POST "https://api.vercel.com/v13/deployments" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "my-project",
    "framework": "nextjs",
    "files": [...]
  }'
```

### Step 4: Return URLs

After deployment, provide:
1. **Preview URL**: `https://my-project-abc123.vercel.app` — the live site
2. **Claim URL**: `https://vercel.com/claim/...` — transfer to user's account

## Static HTML Projects

For projects without `package.json`:
1. If there's a single HTML file, rename it to `index.html`
2. Deploy as a static site

## Important Notes

- **Never deploy `.env` files** — they contain secrets
- **Exclude `node_modules`** — Vercel installs dependencies
- **Exclude `.git`** — not needed for deployment
- If deployment fails due to network restrictions, the user may need to allow `*.vercel.com`

## Example Workflow

```
User: "Deploy this to Vercel"

1. Check for package.json to detect framework
2. Create tarball excluding node_modules, .git, .env
3. Deploy via Vercel CLI or API
4. Return preview URL and claim URL to user
```
