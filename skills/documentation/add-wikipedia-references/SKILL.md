---
name: add-wikipedia-references
description: Add Wikipedia reference links to concepts that don't have one. Searches for relevant Wikipedia articles and adds them to the references array.
allowed-tools: Bash, Read, Edit, Grep, Glob, WebSearch, WebFetch, mcp__playwright__browser_navigate, mcp__playwright__browser_snapshot, mcp__playwright__browser_close
---

# Add Wikipedia References

Add Wikipedia links to concepts missing them.

## Find Concepts Without Wikipedia

```bash
# List concepts without Wikipedia
grep -L '"url": "https://en.wikipedia.org' src/data/concepts/*.json | xargs -n1 basename | sed 's/.json$//'

# Count
grep -L '"url": "https://en.wikipedia.org' src/data/concepts/*.json | wc -l

# First 20 for batch processing
grep -L '"url": "https://en.wikipedia.org' src/data/concepts/*.json | head -20 | xargs -n1 basename | sed 's/.json$//'
```

## Workflow Per Concept

1. **Read concept** - get name, aliases
2. **Search Wikipedia** - use concept name, then aliases if needed
3. **Verify relevance** - article must match concept's meaning/domain
4. **Add reference**:

```json
{
  "references": [
    {
      "title": "Article Name - Wikipedia",
      "url": "https://en.wikipedia.org/wiki/Article_Name",
      "type": "website"
    }
  ]
}
```

## Search Strategy

1. Primary: exact concept name + "wikipedia"
2. Fallback: aliases, broader terms, concept with context
3. Handle disambiguation pages: choose most relevant article
4. Use final URL after redirects

## Skip When

- No relevant Wikipedia article exists
- Concept too niche (proprietary methods, very recent concepts)
- Article doesn't match concept's domain (e.g., "Flow" in wrong field)

## Reference Format

- **title**: `"Article Name - Wikipedia"`
- **url**: canonical URL with underscores (`https://en.wikipedia.org/wiki/Article_Name`)
- **type**: `"website"`

## Batch Processing

```bash
# Batch 1-20
grep -L '"url": "https://en.wikipedia.org' src/data/concepts/*.json | head -20

# Batch 21-40
grep -L '"url": "https://en.wikipedia.org' src/data/concepts/*.json | tail -n +21 | head -20
```

For large batches: spawn sub-agents (5-10 concepts each).

## Verify

```bash
npm run build 2>&1 | tail -10
```
