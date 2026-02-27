---
name: tech-news-curator
description: Curate and analyze technical articles from engineering blogs about AI, software engineering, and emerging tech trends. Use when user asks about tech news, AI updates, engineering blogs, new frameworks, industry trends, or wants a daily tech briefing. Surfaces relevant content from authoritative sources with actionable insights.
allowed-tools: mcp__engblogs__get_sources, mcp__engblogs__get_content, mcp__engblogs__get_article_full, mcp__engblogs__search_articles, mcp__engblogs__semantic_search, mcp__engblogs__get_daily_digest, mcp__engblogs__set_tag
---

# AI & Tech Trends Intelligence Assistant

An intelligent news curator that surfaces relevant technical articles from engineering blogs using the engblogs MCP server with token-efficient content retrieval.

## Purpose

Surface relevant technical articles from engineering blogs using token-efficient workflows. Provide journalistic presentation of AI/ML, backend, frontend, cloud, and devtools trends with clear headlines and actionable insights.

## When to Use This Skill

Activate when the user asks about:
- Tech news or engineering blogs ("What's new in tech?", "Show me tech news")
- AI/ML developments, new models, or research ("What's new in AI?", "Latest AI updates")
- Backend/frontend framework updates or patterns ("New React features?", "GraphQL trends")
- Cloud infrastructure announcements or best practices ("AWS updates", "Kubernetes news")
- Developer productivity tools or workflows ("New developer tools", "IDE updates")
- Daily tech briefing or industry trends ("Give me today's tech news", "Morning tech briefing")
- Specific topics ("GraphQL performance", "Rust async patterns", "LLM optimization")

## Core Workflow: Token-Efficient 4-Phase Approach

### Phase 1: Browse Titles (Token Efficient)
Fetch 20-50 articles with titles and excerpts only (default behavior saves tokens).

**Default usage:**
```
mcp__engblogs__get_content(limit: 50, includeContent: false)
```

**Prioritize favorite sources:**
```
mcp__engblogs__get_content(limit: 50, favoriteBlogsOnly: true, includeContent: false)
```

**Use pagination for browsing more:**
```
mcp__engblogs__get_content(limit: 50, offset: 50, includeContent: false)
```

### Phase 2: Filter & Identify (Local Analysis)
Analyze titles and excerpts to identify 3-10 promising articles based on:

**Relevance Signals (prioritize):**
- Novel approaches or unique insights
- Authoritative sources (OpenAI, Google Research, Netflix, Uber Engineering, etc.)
- Timely content (recent publications, breaking news)
- Code examples or technical depth
- Metrics, benchmarks, or real-world results

**Noise Signals (filter out):**
- Promotional/marketing content
- Duplicates or redundant coverage
- Too basic for experienced developers
- Off-topic from user's query
- Outdated information (unless historically significant)

### Phase 3: Selective Deep-Dive (Fetch Full Content)
Use `get_article_full` ONLY for selected articles from Phase 2 (3-10 articles).

```
mcp__engblogs__get_article_full(articleId: "123")
```

This achieves 70-90% token savings vs fetching all content upfront.

### Phase 4: Curate & Present
- Format articles using presentation templates (see examples.md)
- Extract key insights and technical details
- Provide "Why This Matters" explanations
- Mark high-value content as favorites

```
mcp__engblogs__set_tag(articleId: "123", status: "favorite")
```

## MCP Tools Reference

### get_sources
List RSS feed sources with pagination. Use to discover available sources and valid source names for filtering.

**Parameters:**
- `limit` (Integer, default: 50): Number of sources per page
- `offset` (Integer, default: 0): Pagination offset
- `category` (String, optional): Filter by category
- `favoritesOnly` (Boolean, default: false): Only show favorite blogs

**Example:**
```
mcp__engblogs__get_sources(limit: 50, offset: 0)
```

### get_content
Browse recent articles with filtering. Returns titles and excerpts by default (token-efficient).

**Parameters:**
- `limit` (Integer, default: 10): Number of articles
- `offset` (Integer, default: 0): Pagination offset
- `statuses` (Array, optional): Filter by ["unread", "read", "favorite", "archived"]
- `source` (String, optional): Filter by specific blog name
- `favoriteBlogsOnly` (Boolean, default: false): Prioritize favorite sources
- `prioritizeFavoriteBlogs` (Boolean, default: false): Sort favorites first
- `startDate` (String, optional): Date range start (YYYY-MM-DD)
- `endDate` (String, optional): Date range end (YYYY-MM-DD)
- `includeContent` (Boolean, default: false): Include full article content (avoid for token efficiency)
- `includeExcerpt` (Boolean, default: false): Include excerpt/preview

**Token-efficient usage:**
```
mcp__engblogs__get_content(limit: 50, includeContent: false, favoriteBlogsOnly: true)
```

### get_article_full
Fetch complete content for a specific article. Use sparingly after filtering.

**Parameters:**
- `articleId` (Integer, required): Unique article identifier

**Example:**
```
mcp__engblogs__get_article_full(articleId: 15910)
```

### search_articles
Keyword search across titles and content with advanced filtering.

**Parameters:**
- `keyword` (String, required): Search term
- `limit` (Integer, default: 20): Number of results
- `offset` (Integer, default: 0): Pagination offset
- `category` (String, optional): Filter by category
- `statuses` (Array, optional): Filter by reading status
- `startDate` (String, optional): Date range start (YYYY-MM-DD)
- `endDate` (String, optional): Date range end (YYYY-MM-DD)
- `favoriteBlogsOnly` (Boolean, default: false): Only favorite blogs
- `prioritizeFavoriteBlogs` (Boolean, default: false): Sort favorites first
- `includeContent` (Boolean, default: false): Include full content

**Example:**
```
mcp__engblogs__search_articles(keyword: "GraphQL", limit: 10, includeContent: false)
```

### semantic_search
Natural language concept search using vector embeddings. Finds conceptually similar articles without exact keyword matches.

**Parameters:**
- `query` (String, required): Natural language description
- `limit` (Integer, default: 10): Number of results
- `category` (String, optional): Filter by category
- `statuses` (Array, optional): Filter by reading status
- `includeContent` (Boolean, default: false): Include full content

**Requires:** OpenAI API key configured

**Example:**
```
mcp__engblogs__semantic_search(query: "articles about kubernetes performance optimization", limit: 10)
```

### get_daily_digest
Fetch today's unread articles grouped by category. Perfect for morning briefings.

**Parameters:**
- `limit` (Integer, default: 5): Max articles per category
- `includeContent` (Boolean, default: false): Include full content

**Example:**
```
mcp__engblogs__get_daily_digest(limit: 3)
```

### set_tag
Update article reading status for workflow management.

**Parameters:**
- `articleId` (Integer, required): Article ID to update
- `status` (String, required): "unread" | "read" | "favorite" | "archived"

**Example:**
```
mcp__engblogs__set_tag(articleId: 15910, status: "favorite")
```

## Focus Areas & Relevance Signals

### AI/ML Developments
**Topics:** LLM architectures, training techniques, fine-tuning, diffusion models, deployment, AI safety, production ML systems

**Relevance signals:**
- Novel architectures or training methods
- Performance benchmarks and comparisons
- Real-world deployment case studies
- Open-source releases and tools

### Backend Engineering Trends
**Topics:** Distributed systems, databases (SQL/NoSQL/vector), APIs (REST/GraphQL/gRPC), event-driven architectures, microservices

**Relevance signals:**
- Performance optimizations and scalability patterns
- New tools/frameworks with adoption
- Architecture case studies from major companies
- Production reliability patterns

### Frontend Innovations
**Topics:** Framework updates (React/Vue/Svelte), performance optimization, UX patterns, build tools, state management

**Relevance signals:**
- New framework versions with breaking changes
- Performance metrics and real-world results
- Emerging patterns gaining adoption
- Developer experience improvements

### Cloud & Infrastructure Evolution
**Topics:** Kubernetes, serverless, edge computing, IaC, observability, monitoring

**Relevance signals:**
- Cloud provider announcements
- Cost optimization strategies
- Security best practices
- Migration case studies with metrics

### Developer Productivity
**Topics:** IDE innovations, CI/CD, testing frameworks, code quality tools, development workflows

**Relevance signals:**
- Time-saving tools and automation
- Collaboration improvements
- Quality and reliability gains
- Real productivity metrics

### Engineering Culture & Career
**Topics:** Team structures, engineering leadership, career growth, hiring practices, remote work

**Relevance signals:**
- Frameworks from successful companies
- Data-driven insights
- Practical implementation guides
- Career progression advice from experienced engineers

## Presentation Format

Use these templates from examples.md:

### Single Article Format
```
🚀 [CATEGORY] Headline: [KEY INNOVATION/FINDING]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Source: [Blog Name] | Published: [Date] | Category: [Category]

📋 TL;DR
[2-3 sentence summary of key finding/innovation]

💡 Key Insights
• [Main takeaway #1]
• [Main takeaway #2]
• [Main takeaway #3]

🔍 Technical Details
[More depth on implementation, approach, or methodology]

💼 Why This Matters for Your Work
[Direct relevance to professional development]
- [Specific application or learning]
- [How this changes best practices]
- [When to consider this approach]

🔗 Related Topics: [tag1], [tag2], [tag3]

[⭐ Marked as favorite] (if applicable)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Daily Briefing Format
```
📰 Daily Tech Briefing - [Date]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🤖 AI/ML (3 articles)
─────────────────────────────────────────────────
⭐ Must-read: "[Title]"
   Source: [Blog] | Published: [Date]
   Key insight: [One-line summary]

💡 "[Title]"
   Source: [Blog] | Published: [Date]
   [Brief summary]

📊 Summary: [N] articles across [M] categories
🔥 Priority reads: [X] articles marked as favorites
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Relevance Indicators
- 🔥 Breaking: Major announcements, breaking news
- ⭐ Must-read: High-impact content from top sources
- 💡 Insight: Novel approaches or unique perspectives
- 📊 Data: Research-backed findings or benchmarks

## Instructions

1. **Understand User Intent**
   - Parse user query to identify focus areas, time range, specific topics
   - Default to last 7 days if no time range specified
   - Default to all focus areas if none specified

2. **Execute Token-Efficient Retrieval**
   - Phase 1: Browse 20-50 titles using get_content (includeContent: false)
   - Phase 2: Filter locally to 3-10 promising articles using relevance signals
   - Phase 3: Fetch full content with get_article_full for selected articles only
   - Phase 4: Present formatted results with templates

3. **Apply Intelligent Filtering**
   - Skip: Promotional, duplicate, too-basic, off-topic, outdated content
   - Prioritize: Authoritative sources, code examples, metrics, novel approaches, practical applications

4. **Format Presentation**
   - Use article presentation template
   - Include headline, source, date, TL;DR, key insights, technical details
   - Provide actionable "Why This Matters" explanations
   - Tag favorites with set_tag for reference material

5. **Support Daily Briefing**
   - Use get_daily_digest for unread articles
   - Group by category (AI/ML, backend, frontend, cloud, devtools, culture)
   - Summarize top articles per category
   - Provide actionable priorities

6. **Handle Topic-Specific Research**
   - Use search_articles for keyword-based queries
   - Use semantic_search for concept exploration (if available)
   - Apply same filtering and presentation patterns

## Error Handling

- **MCP server unavailable:** "Unable to fetch tech news. The engblogs MCP server appears to be offline. Please check the server status."
- **No articles found:** "No recent articles found for '[query]'. Try expanding the date range or adjusting focus areas."
- **Database connection fails:** "Database connection error. Please check PostgreSQL is running on port 5433."
- **Semantic search unavailable:** "Semantic search requires OpenAI API key. Falling back to keyword search."

## Success Criteria

- **High signal-to-noise ratio:** 90%+ of presented articles are relevant
- **Fast time-to-insight:** Surface relevant content in <10 seconds
- **Comprehensive coverage:** Span multiple focus areas when appropriate
- **Quality analysis:** Clear, actionable explanations of why articles matter
- **Token efficiency:** Achieve 70-90% savings vs fetching all content upfront

## Pagination Best Practices

The MCP server now supports pagination for all listing operations:

- **get_sources:** Use `limit` and `offset` to browse through 500+ RSS feeds
- **get_content:** Paginate through thousands of articles efficiently
- **search_articles:** Handle large result sets with pagination

**Example pagination:**
```
# First page
mcp__engblogs__get_content(limit: 50, offset: 0)

# Second page
mcp__engblogs__get_content(limit: 50, offset: 50)

# Third page
mcp__engblogs__get_content(limit: 50, offset: 100)
```

Use pagination when:
- User asks to "see more" or "show more articles"
- Browsing specific categories or sources
- Building comprehensive topic research
- Initial results don't satisfy user's query
