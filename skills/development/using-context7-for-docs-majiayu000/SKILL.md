---
name: using-context7-for-docs
description: Use when researching library documentation with Context7 MCP tools for official patterns and best practices
---

# Using Context7 for Documentation

Use this skill when researching library documentation with Context7 MCP tools for official patterns and best practices.

## Core Principles

- Always resolve library ID first (unless user provides exact ID)
- Use topic parameter to focus documentation
- Paginate when initial results insufficient
- Prioritize high benchmark scores and reputation

## Workflow

### 1. Resolve Library ID

**Use `resolve-library-id`** before fetching docs:

```python
# Search for library
result = resolve_library_id(libraryName="react")

# Returns matches with:
# - Context7 ID (e.g., "/facebook/react")
# - Description
# - Code snippet count
# - Source reputation (High/Medium/Low)
# - Benchmark score (0-100, higher is better)
```

**Selection criteria:**
1. Exact name match preferred
2. Higher documentation coverage (more snippets)
3. High/Medium reputation sources
4. Higher benchmark scores (aim for 80+)

**Example output:**

```markdown
Selected: /facebook/react
Reason: Official React repository, High reputation, 850 snippets, Benchmark: 95
```

### 2. Fetch Documentation

**Use `get-library-docs`** with resolved ID:

```python
# Get focused documentation
docs = get_library_docs(
    context7CompatibleLibraryID="/facebook/react",
    topic="hooks",
    page=1
)
```

**Topic parameter:**
- Focuses results on specific area
- Examples: "hooks", "routing", "authentication", "testing"
- More specific = better results

**Pagination:**
- Default `page=1` returns first batch
- If insufficient, try `page=2`, `page=3`, etc.
- Maximum `page=10`

### 3. Version-Specific Docs

**Include version in ID** when needed:

```python
# Specific version
docs = get_library_docs(
    context7CompatibleLibraryID="/vercel/next.js/v14.3.0-canary.87",
    topic="server components"
)
```

Use when:
- Project uses specific version
- Breaking changes between versions
- Need migration guidance

## Reporting Format

Structure findings as:

```markdown
## Library Documentation Findings

### Library: React 18
**Context7 ID:** /facebook/react
**Benchmark Score:** 95

### Relevant APIs

**useEffect Hook** (Official pattern)
```javascript
// Recommended: Cleanup pattern
useEffect(() => {
  const subscription = api.subscribe()
  return () => subscription.unsubscribe()
}, [dependencies])
```
Source: React docs, hooks section

### Best Practices

1. **Dependency Arrays**
   - Always specify dependencies
   - Use exhaustive-deps ESLint rule
   - Avoid functions in dependencies

2. **Performance**
   - Prefer useMemo for expensive calculations
   - useCallback for function props
   - React.memo for component memoization

### Migration Notes
- React 18 introduces concurrent features
- Automatic batching now default
- Upgrade guide: /facebook/react/v18/migration
```

## Common Libraries

**Frontend:**
- React: `/facebook/react`
- Next.js: `/vercel/next.js`
- Vue: `/vuejs/vue`
- Svelte: `/sveltejs/svelte`

**Backend:**
- Express: `/expressjs/express`
- FastAPI: `/tiangolo/fastapi`
- Django: `/django/django`

**Tools:**
- TypeScript: `/microsoft/typescript`
- Vite: `/vitejs/vite`
- Jest: `/jestjs/jest`

## Anti-Patterns

❌ **Don't:** Skip resolve-library-id step
✅ **Do:** Always resolve first (unless user provides exact ID)

❌ **Don't:** Use vague topics like "general"
✅ **Do:** Use specific topics: "authentication", "state management"

❌ **Don't:** Accept low benchmark scores (<50) without checking alternatives
✅ **Do:** Prefer high-quality sources (benchmark 80+)

❌ **Don't:** Cite docs without library version
✅ **Do:** Include version in findings

## Example Session

```python
# 1. Resolve library
result = resolve_library_id(libraryName="fastapi")
# → Selected: /tiangolo/fastapi (Benchmark: 92, High reputation)

# 2. Get auth documentation
docs = get_library_docs(
    context7CompatibleLibraryID="/tiangolo/fastapi",
    topic="authentication",
    page=1
)
# → Got OAuth2, JWT patterns, security best practices

# 3. Need more detail on dependencies
docs2 = get_library_docs(
    context7CompatibleLibraryID="/tiangolo/fastapi",
    topic="dependency injection",
    page=1
)
# → Got Depends() patterns, testing with overrides

# 4. Check pagination if needed
if insufficient:
    docs3 = get_library_docs(
        context7CompatibleLibraryID="/tiangolo/fastapi",
        topic="authentication",
        page=2  # Next page
    )
```

## Quality Indicators

**High-quality results have:**
- ✅ Benchmark score 80+
- ✅ High/Medium source reputation
- ✅ Recent documentation (check dates)
- ✅ Official repositories
- ✅ Code examples with explanation

**Consider alternatives if:**
- ❌ Benchmark score <50
- ❌ Low reputation source
- ❌ Very few code snippets (<10)
- ❌ Unofficial/outdated sources
