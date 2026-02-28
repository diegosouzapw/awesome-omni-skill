---
name: nextjs
description: Next.js framework best practices including App Router, data fetching, and performance optimization.
globs: ["**/next.config.*", "**/app/**/*.tsx", "**/pages/**/*.tsx", "**/middleware.ts"]
priority: 90
tags: ["framework"]
---

# Next.js Best Practices

## App Router (v13+)
- Use Server Components by default
- Add 'use client' only when needed
- Use loading.tsx for loading states
- Use error.tsx for error boundaries
- Use layout.tsx for shared layouts
- Use route groups (folder) for organization

## Data Fetching
- Fetch in Server Components when possible
- Use React Suspense for streaming
- Use generateStaticParams for static generation
- Use revalidate for ISR
- Use Next.js cache() for request deduplication

## Performance
- Use next/image for optimized images
- Use next/font for font optimization
- Use next/link for client-side navigation
- Implement proper caching headers
- Use dynamic imports for code splitting

## API Routes
- Use Route Handlers in app directory
- Validate inputs thoroughly
- Handle errors gracefully
- Use middleware for auth/logging

## SEO
- Use generateMetadata for dynamic SEO
- Implement proper Open Graph tags
- Use semantic HTML
- Add structured data (JSON-LD)
