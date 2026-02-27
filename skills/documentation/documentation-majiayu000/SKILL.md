---
name: documentation
description: Updating user guides, CLI reference, and API documentation for the B2C CLI project
---

# Documentation

This skill covers updating documentation for the B2C CLI project.

## Documentation Structure

The project has three types of documentation:

```
docs/
├── guide/              # User guides (manually written)
│   ├── index.md
│   ├── installation.md
│   ├── authentication.md
│   └── configuration.md
├── cli/                # CLI reference (manually written)
│   ├── index.md
│   ├── code.md
│   ├── webdav.md
│   ├── jobs.md
│   └── ...
├── api/                # API reference (auto-generated)
│   └── *.md
└── .vitepress/         # Vitepress configuration
    └── config.mts
```

## Documentation Types

### 1. User Guides (`docs/guide/`)

Purpose: Help users get started and understand concepts.

When to update:
- New features that need explanation
- Changes to installation or setup process
- New authentication methods
- Configuration changes

Example structure:

```markdown
# Getting Started

Introduction to the topic.

## Prerequisites

- Node.js 22+
- pnpm

## Installation

\`\`\`bash
pnpm install -g @salesforce/b2c-cli
\`\`\`

## Next Steps

- [Authentication](./authentication.md)
- [Configuration](./configuration.md)
```

### 2. CLI Reference (`docs/cli/`)

Purpose: Document command syntax, flags, and usage examples.

When to update:
- New commands added
- Flags added, removed, or changed
- Command behavior changes
- New examples needed

Structure per command topic:

```markdown
# Code Commands

Commands for managing code versions and cartridge deployment.

## b2c code deploy

Deploy cartridges to a B2C Commerce instance.

### Usage

\`\`\`bash
b2c code deploy [PATH] [FLAGS]
\`\`\`

### Arguments

| Argument | Description | Required | Default |
|----------|-------------|----------|---------|
| PATH | Path to cartridges directory | No | . |

### Flags

| Flag | Short | Description | Default |
|------|-------|-------------|---------|
| --server | -s | Instance hostname | - |
| --code-version | -v | Code version name | - |
| --cartridge | -c | Include specific cartridges | - |
| --exclude-cartridge | -x | Exclude cartridges | - |

### Examples

\`\`\`bash
# Deploy all cartridges in current directory
b2c code deploy --server dev01.example.com --code-version v1

# Deploy specific cartridges
b2c code deploy ./cartridges -c app_storefront -c app_custom

# Deploy excluding certain cartridges
b2c code deploy -x test_cartridge -x bm_extensions
\`\`\`

### Authentication

Requires WebDAV credentials (username/password) or OAuth.
```

### 3. API Reference (`docs/api/`)

Purpose: Document the SDK programmatic API.

This is **auto-generated** from TypeScript JSDoc comments using TypeDoc.

Never edit files in `docs/api/` directly. Instead:

1. Update JSDoc comments in SDK source files
2. Run `pnpm run docs:api` to regenerate

## Writing JSDoc for API Docs

### Module-Level Documentation

Add to barrel files (`index.ts`):

```typescript
/**
 * Authentication strategies for B2C Commerce APIs.
 *
 * This module provides various authentication mechanisms:
 * - OAuth 2.0 client credentials
 * - Basic authentication
 * - API key authentication
 *
 * @example
 * ```typescript
 * import { OAuthStrategy } from '@salesforce/b2c-tooling-sdk/auth';
 *
 * const auth = new OAuthStrategy({
 *   clientId: 'my-client',
 *   clientSecret: 'my-secret',
 * });
 * ```
 *
 * @module auth
 */
```

### Class Documentation

```typescript
/**
 * Client for WebDAV file operations on B2C Commerce instances.
 *
 * Supports uploading, downloading, and managing files in various
 * WebDAV roots (Cartridges, IMPEX, Logs, etc.).
 *
 * @example
 * ```typescript
 * const client = new WebDavClient(hostname, auth);
 * await client.put('Cartridges/v1/app_custom/file.js', content);
 * ```
 */
export class WebDavClient {
  /**
   * Creates a new WebDAV client.
   *
   * @param hostname - The B2C Commerce instance hostname
   * @param auth - Authentication strategy to use
   */
  constructor(hostname: string, auth: AuthStrategy) {}
}
```

### Function Documentation

```typescript
/**
 * Deploys cartridges to a B2C Commerce instance.
 *
 * Discovers cartridges in the specified path, creates a ZIP archive,
 * and uploads via WebDAV.
 *
 * @param instance - The B2C instance to deploy to
 * @param cartridgePath - Path containing cartridge directories
 * @param options - Deployment options
 * @returns Deployment result with uploaded cartridges
 *
 * @example
 * ```typescript
 * const result = await deployCartridges(instance, './cartridges', {
 *   include: ['app_storefront'],
 * });
 * console.log(result.cartridges);
 * ```
 *
 * @throws {DeploymentError} If deployment fails
 */
export async function deployCartridges(
  instance: B2CInstance,
  cartridgePath: string,
  options?: DeployOptions
): Promise<DeployResult> {}
```

### Type Documentation

```typescript
/**
 * Configuration for OAuth authentication.
 */
export interface OAuthConfig {
  /** OAuth client ID */
  clientId: string;

  /** OAuth client secret */
  clientSecret: string;

  /** OAuth scopes to request */
  scopes?: string[];
}
```

## Building Documentation

```bash
# Generate API docs from JSDoc
pnpm run docs:api

# Start dev server (includes API generation)
pnpm run docs:dev

# Build static site
pnpm run docs:build

# Preview built site
pnpm run docs:preview
```

## TypeDoc Configuration

Located in `typedoc.json`:

```json
{
  "entryPoints": [
    "packages/b2c-tooling-sdk/src/auth/index.ts",
    "packages/b2c-tooling-sdk/src/clients/index.ts",
    "packages/b2c-tooling-sdk/src/operations/code/index.ts"
  ],
  "out": "docs/api",
  "plugin": [
    "typedoc-plugin-markdown",
    "typedoc-vitepress-theme"
  ],
  "exclude": ["**/*.generated.ts"]
}
```

When adding new SDK modules, add their barrel file to `entryPoints`.

## Vitepress Configuration

Located in `docs/.vitepress/config.mts`:

```typescript
export default defineConfig({
  title: 'B2C CLI',
  base: '/b2c-developer-tooling/',

  themeConfig: {
    nav: [
      { text: 'Guide', link: '/guide/' },
      { text: 'CLI Reference', link: '/cli/' },
      { text: 'API Reference', link: '/api/' },
    ],

    sidebar: {
      '/guide/': [
        {
          text: 'Getting Started',
          items: [
            { text: 'Installation', link: '/guide/installation' },
            { text: 'Authentication', link: '/guide/authentication' },
          ],
        },
      ],
      '/cli/': [
        {
          text: 'Commands',
          items: [
            { text: 'code', link: '/cli/code' },
            { text: 'webdav', link: '/cli/webdav' },
          ],
        },
      ],
    },
  },
});
```

When adding new CLI commands or guide pages, update the sidebar config.

## Claude Code Skills (Plugin)

The `plugins/b2c-cli/skills/` directory contains skills that teach Claude about using the CLI commands. These are distributed via the plugin.

When to update:
- New CLI commands added
- Existing commands changed
- New usage patterns

Skill format:

```markdown
---
name: b2c-<topic>
description: Brief description
---

# B2C <Topic> Skill

Overview of the command topic.

## Examples

### <Use Case>

\`\`\`bash
# Comment explaining the command
b2c <topic> <command> [args] [flags]
\`\`\`

### <Another Use Case>

\`\`\`bash
b2c <topic> <command> --flag value
\`\`\`
```

## Documentation Update Checklist

### When Adding a CLI Command

1. Update `docs/cli/<topic>.md` with command documentation
2. Update `docs/.vitepress/config.mts` sidebar if new topic
3. Update `plugins/b2c-cli/skills/b2c-<topic>/SKILL.md` with examples

### When Adding an SDK Module

1. Write module-level JSDoc in barrel file
2. Write JSDoc for all public classes, functions, types
3. Add entry point to `typedoc.json` if new module
4. Run `pnpm run docs:api` to regenerate

### When Changing CLI Behavior

1. Update affected examples in `docs/cli/*.md`
2. Update affected examples in `plugins/b2c-cli/skills/*/SKILL.md`
3. Update guide pages if conceptual changes

### When Adding Configuration Options

1. Update `docs/guide/configuration.md`
2. Update relevant CLI command docs with new flags
3. Update skills with new flag examples

## Navigation Structure

**Top Navigation:**
- Guide (`/guide/`)
- CLI Reference (`/cli/`)
- API Reference (`/api/`)

**Sidebar:**
- Contextual based on section
- API reference sidebar auto-generated from TypeDoc

## Style Guidelines

- Use code blocks with language hints (```bash, ```typescript)
- Include practical examples for every command/function
- Keep flag tables consistent across command docs
- Use relative links for internal references
- Avoid emojis unless specifically requested
