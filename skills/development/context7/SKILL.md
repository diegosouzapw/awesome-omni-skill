---
name: Context7
description: Up-to-date library documentation and code examples from Context7. USE WHEN looking up API docs, library documentation, framework guides, code examples, OR needing version-specific technical documentation. Prevents hallucinated APIs.
---

# Context7

Query up-to-date, version-specific documentation and code examples directly from source libraries via Context7's documentation aggregation platform.

## Why Context7

| Benefit | Description |
|---------|-------------|
| **Current APIs** | No hallucinated or outdated patterns - documentation comes from actual sources |
| **Version-Specific** | Gets docs for exact library versions you're using |
| **Code Examples** | Real, working code extracted from actual documentation |
| **Broad Coverage** | 1000+ libraries including React, Next.js, Vue, Go, Python, Kubernetes, etc. |

## Setup

```bash
# Navigate to tools directory and install dependencies
cd ~/.claude/skills/Context7/Tools
bun install

# Optional: Set API key for higher rate limits
export CONTEXT7_API_KEY="ctx7sk_your_key_here"  # Get at context7.com/dashboard
```

## Available CLI Tools

| Tool | Purpose | Command |
|------|---------|---------|
| `lookup` | Full lookup (resolve + query) | `bun src/cli/lookup.ts <library> <query>` |
| `resolve` | Find library ID | `bun src/cli/resolve.ts <library> [query]` |
| `query` | Query docs by ID | `bun src/cli/query.ts <library_id> <query>` |

## Quick Reference

### Full Lookup (Recommended)

One command to resolve library and query documentation:

```bash
cd ~/.claude/skills/Context7/Tools
bun src/cli/lookup.ts react "useEffect cleanup function"
bun src/cli/lookup.ts next.js "app router middleware"
bun src/cli/lookup.ts kubernetes "deployment rolling update"
```

### Step-by-Step (When Needed)

#### Step 1: Resolve Library ID

```bash
bun src/cli/resolve.ts react
bun src/cli/resolve.ts next.js "authentication"
```

Returns: Context7-compatible library ID like `/facebook/react` or `/vercel/next.js`

#### Step 2: Query Documentation

```bash
bun src/cli/query.ts /facebook/react "useEffect cleanup"
bun src/cli/query.ts /vercel/next.js "middleware configuration"
```

Returns: Relevant documentation snippets and code examples

## Common Library IDs

| Library | Context7 ID | CLI Shortcut |
|---------|-------------|--------------|
| React | `/facebook/react` | `react` |
| Next.js | `/vercel/next.js` | `next.js`, `nextjs` |
| Vue | `/vuejs/vue` | `vue` |
| Kubernetes | `/kubernetes/kubernetes` | `kubernetes`, `k8s` |
| Go stdlib | `/golang/go` | `go`, `golang` |
| Python | `/python/cpython` | `python` |
| Node.js | `/nodejs/node` | `node`, `nodejs` |
| TypeScript | `/microsoft/typescript` | `typescript`, `ts` |
| Prisma | `/prisma/prisma` | `prisma` |
| Tailwind | `/tailwindlabs/tailwindcss` | `tailwind`, `tailwindcss` |

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **ResolveLibrary** | "find library ID", "resolve library" | `Workflows/ResolveLibrary.md` |
| **QueryDocs** | "lookup docs", "get documentation", "code examples" | `Workflows/QueryDocs.md` |
| **FullLookup** | "help me with [library]", "how do I use [feature]" | `Workflows/FullLookup.md` |

## Examples

### Example 1: React Hooks Documentation

```bash
cd ~/.claude/skills/Context7/Tools
bun src/cli/lookup.ts react "useEffect cleanup function"
```

Output includes current React docs with cleanup pattern examples.

### Example 2: Kubernetes Deployment Spec

```bash
bun src/cli/lookup.ts kubernetes "deployment spec rolling update strategy"
```

Output includes current K8s API reference for Deployment.

### Example 3: Next.js App Router

```bash
bun src/cli/lookup.ts next.js "middleware authentication app router"
```

Output includes latest Next.js middleware documentation.

### Example 4: Using in Claude Code Session

When you need documentation during a coding session:

```
User: "How do I implement server-side data fetching in Next.js 14?"

Claude runs:
  cd ~/.claude/skills/Context7/Tools && bun src/cli/lookup.ts next.js "server components data fetching"

Then synthesizes response with current patterns (Server Components, not old getServerSideProps)
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CONTEXT7_API_KEY` | API key for higher rate limits | None (uses public rate limits) |

Get your API key at [context7.com/dashboard](https://context7.com/dashboard)

## Tips

- **Be specific** in your query for better results
- **Max 3 calls** per question - if you can't find it after 3 tries, use best available info
- **Include version** in query if you need specific version docs (e.g., "React 18 concurrent features")
- **Combine with local context** - use Context7 to verify APIs, then apply to your codebase
- **Known IDs skip API** - common libraries like `react`, `next.js` use cached IDs to skip the resolve step

## Project Structure

```
Tools/
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
└── src/
    ├── index.ts          # Library exports
    ├── lib/
    │   └── context7.ts   # Core API client
    └── cli/
        ├── lookup.ts     # Full lookup command
        ├── resolve.ts    # Library ID resolver
        └── query.ts      # Documentation query
```

## API Reference

The TypeScript client can also be imported programmatically:

```typescript
import { Context7Client, getKnownLibraryId } from "./src/index.js";

const client = new Context7Client({ apiKey: process.env.CONTEXT7_API_KEY });

// Full lookup
const result = await client.lookup("react", "useEffect hooks");
console.log(result.rawContent);

// Or step by step
const { bestMatch } = await client.resolveLibrary("react");
const docs = await client.queryDocs(bestMatch.id, "useEffect cleanup");
```
