---
name: context7
description: Intelligent usage of Context7 MCP for fetching current, version-specific library documentation. Automatically invoked when working with external libraries, frameworks, or APIs where up-to-date documentation improves code quality.
---

# Context7 Documentation Assistant

This skill teaches Codex when and how to effectively use the Context7 MCP server to fetch current library documentation.

## Core Principle

Context7 bridges the gap between Codex's training data (which may lag behind) and the current state of fast-moving libraries. Use it proactively when library-specific knowledge matters.

## Automatic Invocation Triggers

ALWAYS use Context7 when:

1. **Library mentioned by name** in user's request
   - "implement Next.js middleware auth"
   - "use Prisma connection pooling"
   - "Redis pub/sub with ioredis"

2. **Code involves library-specific APIs**
   - Importing third-party packages
   - Using framework-specific patterns
   - Configuring library-specific options

3. **Fast-moving ecosystems** (docs change monthly/quarterly)
   - Next.js, React, Tailwind CSS, Vite
   - Express, Fastify, NestJS
   - Modern testing frameworks (vitest, jest, playwright)
   - ORMs and database libraries (Prisma, Drizzle, Sequelize)

4. **Version-sensitive scenarios**
   - Breaking changes between major versions
   - Deprecated APIs
   - New features in recent releases

## When NOT to Use Context7

Skip Context7 for:

- **Standard library functions** (Node.js core modules, browser APIs)
- **Stable, well-known patterns** (basic SQL, HTTP concepts, Unix commands)
- **General programming concepts** (algorithms, data structures, design patterns)
- **Historical/legacy code** where training data is sufficient
- **Non-library work** (pure business logic, mathematical computations)

## Invocation Workflow

### Step 1: Detect library reference
```
User mentions: "PostgreSQL connection pooling with node-postgres"
→ Library detected: pg (node-postgres)
```

### Step 2: Resolve library ID
```
Call resolve-library-id with query: "pg" or "node-postgres"
→ Receive Context7 ID: "@node-postgres/pg" (or similar)
```

### Step 3: Fetch targeted documentation
```
Call get-library-docs with:
- library_id: "@node-postgres/pg"
- query: "connection pooling" (specific to user's need)
```

### Step 4: Apply documentation
Use the fetched docs to:
- Generate current, correct code
- Cite version-specific patterns
- Avoid deprecated approaches

## Token Efficiency Strategies

1. **One library at a time** - Don't bulk-fetch multiple libraries
2. **Specific queries** - "FastAPI dependency injection" not just "FastAPI"
3. **Read carefully** - Context7 docs are pre-filtered and relevant
4. **Session memory** - Remember library IDs within a conversation
5. **Lazy loading** - Only fetch when about to generate library-specific code

## Error Recovery Patterns

### Library not found
```
Try variations:
- "react-query" → "tanstack-query"
- "nextjs/middleware" → "nextjs"
- "node-redis" → "redis"
- "ioredis" → "redis"

Fallback: Training knowledge + note to user about docs availability
```

### Docs seem incomplete
```
Strategies:
1. Refine query to be more specific
2. Try related library in ecosystem
3. Combine Context7 docs with web_search for very new features
```

### Rate limiting
```
Without API key: Conservative usage, batch-related queries
With API key: Normal operation
```

## Common Library Patterns

### CLI & Logging
- `commander` - CLI applications (multi-command, config files)
- `yargs` - CLI argument parsing
- `pino` - High-performance logging (multi-transport, structured)
- `winston` - Flexible logging transports

### Frontend Frameworks & UI
- `react` - Core UI framework (hooks, concurrent APIs)
- `next` - App Router, middleware, server actions
- `vue` / `nuxt` - Vue ecosystem patterns
- `tailwindcss` - Utility-first styling (config changes)

### Data Validation & Serialization
- `zod` - Runtime schema validation (1.x vs 3.x changes)
- `yup` - Validation schemas

### Database & Caching
- `prisma` - ORM with schema-driven workflows
- `drizzle-orm` - Type-safe SQL
- `pg` - PostgreSQL (pooling, transactions)
- `redis` / `ioredis` - Redis (queuing, caching)

### Development Tools
- `vitest` - Testing framework
- `jest` - Unit testing
- `playwright` / `cypress` - E2E testing
- `eslint` / `typescript` - Linting & typing

### Utilities
- `dotenv` - Environment management
- `axios` - HTTP client

## Integration with Other Skills

### When building frontend apps
```
1. Context7 for React/Next.js routing and data-fetching patterns
2. Context7 for Tailwind CSS configuration and plugins
3. Combine with your UI conventions and design system
```

### When building API services
```
1. Context7 for Express/Fastify/NestJS middleware patterns
2. Context7 for Prisma/Drizzle query patterns
3. Your project structure and conventions
```

### When writing tests
```
1. Context7 for Vitest/Jest features (mocking, config, setup)
2. Context7 for Playwright/Cypress E2E patterns
3. Your testing patterns for organization
```

### When debugging
```
1. Examine error messages for library clues
2. Context7 for library-specific debugging techniques
3. Stack traces often reveal version-specific issues
```

## Best Practices

1. **Be proactive** - Don't wait to be asked; invoke when you see library names
2. **Version awareness** - Check package.json/requirements.txt first
3. **Cite sources** - Mention you're using Context7 docs briefly
4. **Verify assumptions** - Training data may be outdated for fast-moving libs
5. **Combine approaches** - Context7 + training knowledge + web search as needed

## Example Decision Trees

### User: "Set up PostgreSQL connection pooling"
```
Decision: Use Context7
Reason: PostgreSQL drivers have version-specific pooling APIs
Steps:
1. Detect library: pg or Prisma (check user's environment)
2. Resolve ID
3. Fetch pooling docs
4. Generate current best-practice code
```

### User: "Explain bubble sort algorithm"
```
Decision: Skip Context7
Reason: Algorithmic concept, no library involved
Action: Use training knowledge directly
```

### User: "Create a commander CLI that loads config from YAML and has multiple commands"
```
Decision: Use Context7
Reason: commander has specific patterns for multi-command + config
Steps:
1. Resolve commander library ID
2. Fetch docs on command groups and configuration
3. Generate current commander patterns
```

### User: "Build a Next.js app with App Router and middleware"
```
Decision: Use Context7
Reason: Next.js routing and middleware APIs change across versions
Steps:
1. Check package.json for Next.js version
2. Resolve next library ID
3. Fetch current App Router + middleware patterns
4. Generate version-appropriate code
```

### User: "Set up pino with multiple transports for different modules"
```
Decision: Use Context7
Reason: pino has specific multi-transport configuration patterns
Steps:
1. Resolve pino library ID
2. Fetch multi-transport setup docs
3. Generate proper transport configuration
```

## Monitoring Your Usage

After using Context7, briefly note:
- ✓ Library ID resolved successfully
- ✓ Docs were relevant and current
- ⚠ Fell back to training knowledge (docs unavailable)
- ⚠ Combined Context7 + web search (very new features)

This helps users understand your information sources.

## Special Considerations for Node/Web Workflows

### Frontend frameworks
- Use Context7 for: React/Next/Vue/Nuxt routing and data-fetching specifics
- Use training for: general UI concepts and patterns
- Check versions especially for breaking changes (Next.js App Router, React APIs)
- Version-specific APIs matter for framework routing, data fetching, and server components

### CLI applications
- Multi-command structures: command groups, shared options
- Configuration handling: YAML/TOML loading patterns
- Use Context7 for: commander/yargs-specific patterns, not generic CLI concepts

### Logging
- Multi-transport setups for different modules/projects
- Structured logging for Node services
- Rotation and retention policies
- Use Context7 for: pino/winston configuration, not generic logging

### Database-heavy work
- PostgreSQL/MySQL features and drivers change frequently
- Use Context7 for: pg/mysql2 connection patterns, pooling, transactions
- Critical for: JSON operations, performance features
- Migration and schema patterns for Prisma/Drizzle

### Data validation
- zod/yup APIs and best practices evolve
- Always check versions before generating schemas
- Use Context7 for: version-specific validation patterns and transforms

### Development tooling
- vitest/jest: config, mocks, and environment settings change over versions
- eslint/typescript: config formats and rules evolve
- Use Context7 for: current best practices, not outdated patterns

## Meta: When to Load This Skill

This skill loads automatically when:
- User mentions library/framework names
- Code generation involves third-party packages
- Documentation quality affects success

Token cost: ~50 tokens until loaded, ~1200 tokens when active
