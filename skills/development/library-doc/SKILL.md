---
name: library-doc
description: Index and search library documentation locally for offline use. Invoke when user asks to index docs, search library topics, or list indexed libraries.
allowed-tools: WebSearch, WebFetch, Bash, Read
user-invocable: true
---

# Library Documentation Indexer

## Commands

```bash
# List
python3 ./scripts/list.py                         # all libraries
python3 ./scripts/list.py <library>               # docs in a library
python3 ./scripts/list.py --delete <library>      # delete a library

# Search
python3 ./scripts/search.py "<query>"                       # all libraries
python3 ./scripts/search.py "<query>" --library <library>   # specific library
python3 ./scripts/search.py "<query>" --limit <n>           # limit results

# Read
python3 ./scripts/read.py <library> "<title>"               # by title (exact)
python3 ./scripts/read.py <library> <id>                    # by document ID
python3 ./scripts/read.py <library> "<title>" --lines 1-100 # specific lines

# All commands support --json
```

## Search → Read Workflow

```bash
# 1. Search
python3 ./scripts/search.py "<query>" --library <library>

# 2. Read (use title or ID from search results)
python3 ./scripts/read.py <library> "<title>"
python3 ./scripts/read.py <library> <id>

# 3. For long docs, read in chunks
python3 ./scripts/read.py <library> "<title>" --lines 1-100
python3 ./scripts/read.py <library> "<title>" --lines 100-200
```

## Indexing a Library

### Check for preset first

```bash
python3 ./scripts/presets.py list              # List all available presets
python3 ./scripts/presets.py show <library>    # Show preset details
```

If a preset exists, use it directly (see "Using Presets" below).

### Manual indexing (no preset)

#### Step 1: Clarify what to index

If ambiguous, ask the user:
- Library/framework (ex: React, FastAPI)
- Web API or spec (ex: WebSocket API)
- Different implementations across languages

#### Step 2: Find docs source

Search: `"<library> documentation github"` or `"<library> docs site:github.com"`

Look for:
- Official repo with `docs/`, `content/`, or `pages/` folder
- Dedicated docs repo (`<lib>-docs`, `<lib>.dev`)
- README-only projects

For specific versions: check releases/tags (`v15.0.0`, `15.x`, `docs-v15`)

### Step 3: Identify structure

- **Docs path**: `docs/`, `content/`, `src/content/`, etc.
- **File types**: `.md`, `.mdx`, `.rst`
- **Base URL**: live docs URL (e.g., `https://<library>.dev/docs`)

### Step 4: Clone and index

Naming: `<library>` for latest, `<library>-<version>` for specific version.

CRITICAL: Always use sparse checkout to only download docs.

```bash
cd /tmp && rm -rf <lib>-docs

# clone with sparse checkout
git clone --depth 1 --filter=blob:none --sparse <repo-url> <lib>-docs
cd <lib>-docs && git sparse-checkout set <docs-path>

# verify docs exist
find <docs-path> -type f \( -name "*.md" -o -name "*.mdx" \) | wc -l

# index
find <docs-path> -type f \( -name "*.md" -o -name "*.mdx" \) | \
  python3 ./scripts/index.py <lib> --batch --base-url "<base-url>" --strip-prefix "<docs-path>/"

# cleanup and verify
rm -rf /tmp/<lib>-docs
python3 ./scripts/list.py <lib>
```

## Common Patterns

### Standard docs folder

```bash
cd /tmp && rm -rf <lib>-docs
git clone --depth 1 --filter=blob:none --sparse <repo-url> <lib>-docs
cd <lib>-docs && git sparse-checkout set docs
find docs -type f \( -name "*.md" -o -name "*.mdx" \) | \
  python3 ./scripts/index.py <lib> --batch --base-url "<base-url>" --strip-prefix "docs/"
rm -rf /tmp/<lib>-docs
```

### README-only project

```bash
cd /tmp && rm -rf <lib>-repo
git clone --depth 1 <repo-url> <lib>-repo
cat <lib>-repo/README.md | python3 ./scripts/index.py <lib> -f "README.md" -u "<repo-url>#readme"
rm -rf /tmp/<lib>-repo
```

### Specific version

```bash
cd /tmp && rm -rf <lib>-docs
git clone --branch <tag> --depth 1 --filter=blob:none --sparse <repo-url> <lib>-docs
cd <lib>-docs && git sparse-checkout set <docs-path>
find <docs-path> -type f \( -name "*.md" -o -name "*.mdx" \) | \
  python3 ./scripts/index.py <lib>-<version> --batch --base-url "<base-url>" --strip-prefix "<docs-path>/"
rm -rf /tmp/<lib>-docs
```

## Using Presets

For popular libraries, use the preset for correct repo/path/URL:

```bash
python3 ./scripts/presets.py show react   # Show preset config

# Then use the preset values:
cd /tmp && rm -rf react-docs
git clone --depth 1 --filter=blob:none --sparse https://github.com/reactjs/react.dev react-docs
cd react-docs && git sparse-checkout set src/content
find src/content -type f \( -name "*.md" -o -name "*.mdx" \) | \
  python3 ./scripts/index.py react --batch --base-url "https://react.dev" --strip-prefix "src/content/"
rm -rf /tmp/react-docs
```

Available presets: react, nextjs, vue, svelte, nuxt, angular, astro, remix, tailwind, shadcn, express, fastapi, django, flask, hono, nestjs, rails, laravel, bun, node, deno, typescript, python, rust, go, prisma, drizzle, supabase, redux, zustand, tanstack-query, vitest, playwright, jest, pytest, vite, langchain, openai, anthropic, docker, kubernetes, and more.

## Troubleshooting

- **0 docs indexed**: check docs path exists after sparse checkout
- **No .md files**: try `.mdx`, `.rst`, or check actual extension
- **Wrong URLs**: check live docs site structure
- **Re-indexing**: just run again, documents are upserted by path
- **Large doc chunking**: files >8KB auto-split by ## headings; use --no-chunk to disable
