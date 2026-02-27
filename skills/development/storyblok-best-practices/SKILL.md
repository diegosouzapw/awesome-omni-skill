---
name: storyblok-best-practices
description: Comprehensive Storyblok CMS development best practices for agency developers. Covers content modeling, SDK integration (React, Vue, Nuxt, Next.js), Visual Editor configuration, field plugins, API usage, internationalization, webhooks, and deployment patterns. Triggers on tasks involving Storyblok components, Visual Editor setup, content fetching, field plugin development, or headless CMS integration.
license: MIT
metadata:
  author: storyblok-engineering
  version: "1.0.0"
---

# Storyblok Best Practices

Comprehensive best practices guide for Storyblok CMS development, designed for AI agents and LLMs helping agency developers. Contains 40 rules across 24 categories, prioritized by impact to guide automated code generation and content architecture decisions.

## When to Apply

Reference these guidelines when:
- Designing Storyblok content models and component schemas
- Integrating Storyblok with React, Vue, Nuxt, or Next.js
- Configuring Visual Editor and real-time preview
- Building custom field plugins with @storyblok/field-plugin
- Implementing Content Delivery API or Management API
- Setting up internationalization (i18n) with field or folder translations
- Configuring webhooks for cache invalidation and automation
- Managing multi-space architectures and CI/CD workflows
- Handling rich text fields with custom resolvers
- Optimizing images and performance with Image Service
- Building tool plugins or space plugins (sidebar apps)
- Setting up preview/production environments

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Content Modeling | CRITICAL | `model-` |
| 2 | SDK Integration | CRITICAL | `sdk-` |
| 3 | Visual Editor | CRITICAL | `editor-` |
| 4 | Performance & Caching | CRITICAL | `perf-` |
| 5 | Security & Authentication | CRITICAL | `security-` |
| 6 | Field Plugins | HIGH | `plugin-` |
| 7 | API Development | HIGH | `api-` |
| 8 | Internationalization | HIGH | `i18n-` |
| 9 | Next.js Integration | HIGH | `nextjs-` |
| 10 | Nuxt Integration | HIGH | `nuxt-` |
| 11 | App Development | HIGH | `app-` |
| 12 | Space & Asset Management | HIGH | `space-` |
| 13 | Webhooks & Automation | MEDIUM-HIGH | `webhook-` |
| 14 | Workflows & Publishing | MEDIUM-HIGH | `workflow-` |
| 15 | CLI & DevOps | MEDIUM | `cli-` |
| 16 | Testing & Quality | MEDIUM | `test-` |
| 17 | Rich Text & Media | MEDIUM | `richtext-` |
| 18 | Common Patterns | HIGH | `pattern-` |
| 19 | SEO & Structured Data | HIGH | `seo-` |
| 20 | E-commerce Integration | HIGH | `ecommerce-` |
| 21 | Content Migration | MEDIUM-HIGH | `migration-` |
| 22 | AI Features | MEDIUM | `ai-` |
| 23 | Collaboration | MEDIUM | `collaboration-` |
| 24 | Custom Apps | MEDIUM | `app-` |

## Quick Reference

### 1. Content Modeling (CRITICAL)

- `model-component-structure` - Follow atomic design for components
- `model-field-configuration` - Configure field types with validation
- `model-block-nesting` - Manage block nesting with restrictions
- `model-naming-conventions` - Use consistent naming (snake_case)

### 2. SDK Integration (CRITICAL)

- `sdk-storyblok-editable` - Always apply storyblokEditable() to components
- `sdk-component-registration` - Register all components globally
- `sdk-richtext-rendering` - Render rich text with custom resolvers

### 3. Visual Editor (CRITICAL)

- `editor-bridge-setup` - Configure Storyblok Bridge correctly
- `editor-draft-published` - Handle draft/published content properly

### 4. Performance & Caching (CRITICAL)

- `perf-image-optimization` - Use Image Service for optimized delivery
- `perf-cache-invalidation` - Implement proper cache invalidation with cv parameter
- `perf-monitoring` - API and content performance monitoring

### 5. Security & Authentication (CRITICAL)

- `security-token-handling` - Secure API token management
- `security-roles-permissions` - RBAC and access control configuration

### 6. Field Plugins (HIGH)

- `plugin-field-development` - Build field plugins with @storyblok/field-plugin SDK

### 7. API Development (HIGH)

- `api-content-delivery` - Use Content Delivery API correctly with pagination
- `api-graphql` - Use GraphQL API for optimized queries
- `api-management` - Use Management API for CRUD operations

### 8. Internationalization (HIGH)

- `i18n-field-translation` - Implement field-level translations correctly
- `i18n-folder-level` - Folder-level translations with Dimensions app

### 9. Next.js Integration (HIGH)

- `nextjs-app-router` - Integrate with Next.js App Router and RSC

### 10. Nuxt Integration (HIGH)

- `nuxt-integration` - Configure @storyblok/nuxt module correctly

### 11. App Development (HIGH)

- `app-tool-plugins` - Build tool and space plugins with App Bridge

### 12. Space & Asset Management (HIGH)

- `space-asset-management` - Organize assets and datasources
- `space-multi-environment` - Multi-space architecture for enterprise

### 13. Webhooks & Automation (MEDIUM-HIGH)

- `webhook-configuration` - Configure webhooks for content events

### 14. Workflows & Publishing (MEDIUM-HIGH)

- `workflow-releases` - Use Releases for scheduled publishing
- `workflow-pipelines` - Pipeline stages for content staging

### 15. CLI & DevOps (MEDIUM)

- `cli-component-sync` - Use CLI for component schema management

### 16. Testing & Quality (MEDIUM)

- `test-preview-environments` - Set up preview and production environments
- `test-unit-integration` - Unit, integration, and E2E testing strategies

### 17. Rich Text & Media (MEDIUM)

- `richtext-media-handling` - Handle rich text media correctly

### 18. Common Patterns (HIGH)

- `pattern-typescript` - Generate and use TypeScript types
- `pattern-error-handling` - Implement robust error handling
- `pattern-debugging` - Debugging and troubleshooting guide
- `pattern-relations-references` - Story links, relations, and reference resolution

### 19. SEO & Structured Data (HIGH)

- `seo-structured-data` - JSON-LD, Open Graph, and sitemap configuration

### 20. E-commerce Integration (HIGH)

- `ecommerce-integration` - Commerce platform integration patterns

### 21. Content Migration (MEDIUM-HIGH)

- `migration-patterns` - Content migration strategies and tooling

### 22. AI Features (MEDIUM)

- `ai-content-features` - Storyblok AI capabilities and RAG preparation

### 23. Collaboration (MEDIUM)

- `collaboration-realtime` - Real-time editing, comments, and conflict resolution

### 24. Custom Apps (MEDIUM)

- `app-custom-sidebar` - Sidebar apps and tool plugins beyond field plugins

## Core Principles

### Visual Editor First

Storyblok's Visual Editor is the primary editing experience:
- Always apply `storyblokEditable()` to components
- Configure the Storyblok Bridge for live editing
- Test components in the Visual Editor, not just code

### Content Delivery vs Management API

| API | Purpose | Token | Cache |
|-----|---------|-------|-------|
| Content Delivery | Fetch content | Public/Preview | CDN |
| Management | CRUD operations | Personal/OAuth | None |
| GraphQL | Read-only queries | Public/Preview | CDN |

### Framework Integration

| Framework | SDK | Special Features |
|-----------|-----|------------------|
| React | `@storyblok/react` (v5+) | useStoryblokState, RSC support, StoryblokServerComponent |
| Vue | `@storyblok/vue` | v-editable directive |
| Nuxt | `@storyblok/nuxt` (v9+ for Nuxt 4) | Auto-registration, useAsyncStoryblok, deep option |
| Next.js | `@storyblok/react` | Draft mode, ISR, App Router |
| Astro | `@storyblok/astro` | Hybrid rendering |

### Token Security

Never expose these in client-side code:
- Preview tokens (show draft content)
- Personal access tokens (full write access)
- OAuth tokens (scoped access)

Safe for client-side:
- Public tokens (read-only published content)

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/model-component-structure.md
rules/sdk-storyblok-editable.md
rules/editor-bridge-setup.md
rules/perf-image-optimization.md
rules/security-token-handling.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Configuration tables and references

## Key Resources

- [Storyblok Documentation](https://www.storyblok.com/docs)
- [React SDK](https://www.storyblok.com/docs/packages/storyblok-react)
- [Nuxt Module](https://www.storyblok.com/docs/packages/storyblok-nuxt)
- [Field Plugin SDK](https://www.storyblok.com/docs/packages/storyblok-field-plugin)
- [Storyblok CLI](https://www.storyblok.com/docs/packages/storyblok-cli)
- [Content Delivery API](https://www.storyblok.com/docs/api/content-delivery/v2)
- [Management API](https://www.storyblok.com/docs/api/management)
