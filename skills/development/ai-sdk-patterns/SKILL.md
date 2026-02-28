---
name: ai-sdk-patterns
description: Vercel AI SDK tool patterns for dx-toolkit - input schemas for smart queries, API key handling, raw response returns
---

# Vercel AI SDK Tool Patterns

Package-specific patterns for `@youdotcom-oss/ai-sdk-plugin`. Use these patterns when creating AI SDK tools.

---

## Tool Function Structure

Each tool function follows this pattern:

```typescript
export const youToolName = (config: YouToolsConfig = {}) => {
  const apiKey = config.apiKey ?? process.env.YDC_API_KEY;

  return tool({
    description: 'Tool description for AI model',
    inputSchema: ZodSchema,
    execute: async (params) => {
      if (!apiKey) {
        throw new Error('YDC_API_KEY is required');
      }

      const response = await callApiUtility({
        params,
        YDC_API_KEY: apiKey,
        getUserAgent,
      });

      // Return raw API response for maximum flexibility
      return response;
    },
  });
};
```

### Key Components

- `config` - Optional configuration with API key
- `tool()` - AI SDK tool wrapper
- `inputSchema` - Zod schema that enables AI to construct intelligent queries
- `execute()` - Async function that calls You.com API
- Returns raw API response for maximum flexibility

## Input Schemas Enable Smart Queries

The most critical pattern is providing comprehensive input schemas. These schemas allow AI agents to construct intelligent, well-formed queries with all available parameters.

### Always use schemas from `@youdotcom-oss/mcp`

```typescript
// ✅ Import from @youdotcom-oss/mcp
import { SearchQuerySchema } from '@youdotcom-oss/mcp';

export const youSearch = (config: YouToolsConfig = {}) => {
  return tool({
    description: '...',
    inputSchema: SearchQuerySchema,  // Enables AI to use all search parameters
    execute: async (params) => { ... },
  });
};

// ❌ Don't duplicate or simplify schemas
const MySearchSchema = z.object({ query: z.string() });  // Missing filters!
inputSchema: MySearchSchema
```

**Why this matters:**
- Rich schemas enable AI to use advanced query parameters (filters, freshness, country, etc.)
- AI can construct more intelligent queries based on user intent
- Prevents duplicating schema definitions across packages
- Ensures consistency with MCP server schemas

### Schema Examples

**SearchQuerySchema** - Enables AI to use search filters:
```typescript
// AI can automatically add filters based on user intent:
// "Latest news about AI" → { query: "AI", freshness: "day" }
// "Python tutorials" → { query: "Python tutorials" }
```

**ExpressAgentInputSchema** - Enables AI to control agent behavior:
```typescript
// AI can enable/disable web search based on query type:
// "What is 2+2?" → { input: "What is 2+2?", tools: [] }  // No web needed
// "Latest GDP data" → { input: "Latest GDP data", tools: ["search"] }
```

**ContentsQuerySchema** - Enables AI to extract from multiple URLs:
```typescript
// AI can extract from multiple pages and choose format:
// { urls: ["url1", "url2"], format: "markdown" }
```

## API Key Handling

### Always provide environment variable fallback

```typescript
// ✅ Automatic environment variable fallback
const apiKey = config.apiKey ?? process.env.YDC_API_KEY;

// Usage - no explicit apiKey needed if YDC_API_KEY is set
const search = youSearch();

// Or override with explicit config
const search = youSearch({ apiKey: 'custom-key' });
```

### Always validate API key before API calls

```typescript
// ✅ Check API key in execute function
execute: async (params) => {
  if (!apiKey) {
    throw new Error('YDC_API_KEY is required');
  }
  const response = await callApi(...);
}

// ❌ Don't skip validation
execute: async (params) => {
  const response = await callApi(...); // May fail with unclear error
}
```

**Why this pattern?**
- Provides clear error message when API key is missing
- Fails fast before making API calls
- Helps users identify configuration issues quickly

## Response Format

### Always return raw API response

```typescript
// ✅ Return raw API response
execute: async (params) => {
  const response = await fetchSearchResults({
    searchQuery: params,
    YDC_API_KEY: apiKey,
    getUserAgent,
  });

  return response;  // Raw response for maximum flexibility
}

// ❌ Don't format or transform responses
return {
  text: formatResponse(response),
  data: response,
};
```

**Why raw responses?**
- Maximum flexibility for AI SDK to process results
- No information loss from formatting
- AI SDK handles presentation layer
- Easier to debug (see actual API response)

## Tool Descriptions

Write descriptions that guide AI behavior and explain when to use the tool:

```typescript
// ✅ Clear guidance for AI model
description: 'Search the web for current information, news, articles, and content using You.com. Returns web results with snippets and news articles. Use this when you need up-to-date information or facts from the internet.'

// ✅ Explains tool purpose and use case
description: 'Fast AI agent powered by You.com that provides quick answers with optional web search. Use this for straightforward queries that benefit from real-time web information.'

// ❌ Too brief
description: 'Search the web'

// ❌ For humans instead of AI
description: 'This tool allows you to search'
```

## Best Practices

1. **Use comprehensive input schemas** - Enable AI to construct intelligent queries with all available parameters
2. **Always use schemas from @youdotcom-oss/mcp** - Don't duplicate, import from shared package
3. **Return raw API responses** - Maximum flexibility, no information loss
4. **Validate API keys** - Fail fast with clear error messages
5. **Use environment variable fallback** - Automatic YDC_API_KEY detection
6. **Write clear tool descriptions** - Guide AI on when and how to use tools

## Related Resources

- Vercel AI SDK Documentation: https://sdk.vercel.ai/docs
- Package AGENTS.md: `packages/ai-sdk-plugin/AGENTS.md`
- MCP Package (schemas): `packages/mcp/`
- API Documentation: `packages/ai-sdk-plugin/docs/API.md`
