---
name: "Academic Researcher"
description: "Academic paper search across 14+ scholarly platforms including arXiv, PubMed, Google Scholar, Web of Science, Semantic Scholar, Sci-Hub, and more. Use for literature review, research discovery, and citation management."
---

# Academic Researcher

## What This Skill Does

Provides unified access to **14 academic platforms** through a single MCP interface. Enables searching, retrieving, and analyzing scholarly papers with standardized data models and security features. Powered by `paper-search-mcp-nodejs` v0.2.5.

## Supported Platforms

| Platform | Search | Fetch | Advanced | Rate Limit |
|----------|--------|-------|----------|------------|
| arXiv | ✅ | ✅ | Query syntax | 3 req/s |
| Web of Science | ✅ | ✅ | WQL support | API key required |
| PubMed | ✅ | ✅ | Boolean ops | 3 req/s |
| Google Scholar | ✅ | ❌ | Auto-retry | IP-based |
| Semantic Scholar | ✅ | ✅ | GraphQL | 100 req/5min |
| Sci-Hub | ❌ | ✅ | Mirror mgmt | Legal caution ⚠️ |
| ScienceDirect | ✅ | ✅ | Facets | API key required |
| Springer Nature | ✅ | ✅ | Dual API | API key required |
| Wiley | ✅ | ✅ | Full metadata | API key required |
| Scopus | ✅ | ✅ | Advanced | API key required |
| Crossref | ✅ | ✅ | Works API | Open access |
| bioRxiv | ✅ | ✅ | Pre-prints | Open access |
| medRxiv | ✅ | ✅ | Medical pre-prints | Open access |
| IACR ePrint | ✅ | ✅ | Cryptography | Open access |

## Prerequisites

- Node.js 18+
- **Globally installed**: `npm install -g paper-search-mcp-nodejs`
- API keys for premium platforms (optional)

## Quick Start

### Add to Claude Code (MCP Integration)

```bash
# The package is already globally installed via:
# npm install -g paper-search-mcp-nodejs

# Add MCP server to Claude Code
claude mcp add paper-search npx paper-search-mcp-nodejs

# Verify installation
npx paper-search-mcp-nodejs --version  # Should show v0.2.5
```

### Configure API Keys

Create `.env` file in your working directory:

```bash
# Optional: Premium platform API keys
WOS_API_KEY=your_web_of_science_key
PUBMED_API_KEY=your_pubmed_key
SCIENCEDIRECT_API_KEY=your_elsevier_key
SPRINGER_API_KEY=your_springer_key
WILEY_API_KEY=your_wiley_key
SCOPUS_API_KEY=your_scopus_key

# Optional: Sci-Hub mirror override
SCIHUB_MIRROR=https://sci-hub.se
```

## Available MCP Tools (17 Total)

### Universal Search Tools

#### 1. `search_papers`
Search across any supported platform with unified interface.

```typescript
// Example: Search arXiv
{
  "platform": "arxiv",
  "query": "quantum computing",
  "maxResults": 10
}

// Example: Search PubMed with filters
{
  "platform": "pubmed",
  "query": "CRISPR gene editing",
  "filters": {
    "year": "2024",
    "species": "human"
  },
  "maxResults": 20
}

// Example: Web of Science advanced search
{
  "platform": "wos",
  "query": "TS=(machine learning) AND PY=(2023-2024)",
  "filters": {
    "documentType": "article"
  }
}
```

#### 2. `fetch_paper`
Retrieve full paper metadata or PDF by identifier.

```typescript
// By DOI
{
  "doi": "10.1038/s41586-019-1666-5",
  "platform": "crossref"
}

// By arXiv ID
{
  "arxivId": "2301.07041",
  "platform": "arxiv"
}

// By PubMed ID
{
  "pmid": "12345678",
  "platform": "pubmed"
}
```

### Platform-Specific Tools

#### 3. `search_arxiv`
Advanced arXiv search with category filtering.

```typescript
{
  "query": "ti:transformer AND cat:cs.LG",
  "maxResults": 50,
  "sortBy": "submittedDate",
  "sortOrder": "descending"
}
```

#### 4. `search_pubmed`
PubMed/MEDLINE search with MeSH terms.

```typescript
{
  "query": "(cancer[MeSH]) AND (immunotherapy[Title])",
  "filters": {
    "publicationType": "Clinical Trial",
    "minDate": "2020/01/01",
    "maxDate": "2024/12/31"
  }
}
```

#### 5. `search_google_scholar`
Google Scholar search with citation data.

```typescript
{
  "query": "deep learning natural language processing",
  "filters": {
    "yearLow": 2020,
    "yearHigh": 2024
  }
}
```

⚠️ **Rate Limit Warning**: Google Scholar enforces strict IP-based limits. Use sparingly.

#### 6. `search_semantic_scholar`
Semantic Scholar with citation graphs.

```typescript
{
  "query": "attention is all you need",
  "fields": ["title", "authors", "citationCount", "influentialCitationCount"],
  "limit": 100
}
```

#### 7. `search_wos` (Web of Science)
Advanced WoS search with citation analytics.

```typescript
{
  "query": "TS=(climate change) AND CU=(China)",
  "filters": {
    "databaseId": "WOS",
    "timespan": "2020-2024"
  }
}
```

#### 8. `search_sciencedirect`
Elsevier ScienceDirect full-text search.

```typescript
{
  "query": "CRISPR Cas9",
  "filters": {
    "date": "2024",
    "contentType": "Journal",
    "openAccess": true
  }
}
```

#### 9. `search_springer`
Springer Nature search across journals and books.

```typescript
{
  "query": "quantum cryptography",
  "filters": {
    "subject": "Physics",
    "year": "2024"
  }
}
```

#### 10. `search_wiley`
Wiley Online Library search.

```typescript
{
  "query": "protein folding",
  "filters": {
    "publicationYear": "2024"
  }
}
```

#### 11. `search_scopus`
Scopus citation database search.

```typescript
{
  "query": "TITLE-ABS-KEY(machine learning healthcare)",
  "filters": {
    "doctype": "ar",  // Article
    "pubyear": "2024"
  }
}
```

#### 12. `search_crossref`
Crossref DOI search and metadata.

```typescript
{
  "query": "neural networks",
  "filters": {
    "type": "journal-article",
    "from-pub-date": "2024-01-01"
  },
  "rows": 100
}
```

#### 13. `search_biorxiv`
bioRxiv preprint search (biology).

```typescript
{
  "query": "COVID-19 vaccine",
  "filters": {
    "category": "immunology"
  }
}
```

#### 14. `search_medrxiv`
medRxiv preprint search (medicine).

```typescript
{
  "query": "cancer therapy",
  "maxResults": 50
}
```

#### 15. `search_iacr`
IACR ePrint archive (cryptography).

```typescript
{
  "query": "zero knowledge proof",
  "maxResults": 30
}
```

### Utility Tools

#### 16. `fetch_scihub_pdf`
Download PDF via Sci-Hub (legal caution required).

```typescript
{
  "doi": "10.1016/j.cell.2024.01.001"
}
```

⚠️ **Legal Warning**: Sci-Hub access may violate copyright laws in many jurisdictions. Use only for:
- Papers you have legal access to
- Fair use/educational purposes where permitted
- Jurisdictions where Sci-Hub is legal

#### 17. `get_paper_recommendations`
Get citation-based paper recommendations.

```typescript
{
  "doi": "10.1038/nature12373",
  "platform": "semantic_scholar",
  "limit": 10
}
```

## Security Features (v0.2.5)

### 1. DOI Validation
Automatic validation of DOI formats to prevent injection attacks.

### 2. Query Sanitization
SQL injection and command injection protection for all queries.

### 3. Sensitive Data Masking
API keys automatically masked in logs and error messages.

### 4. Rate Limiting
Built-in token bucket rate limiting for all platforms.

## Common Workflows

### Literature Review
```typescript
// 1. Broad search across multiple platforms
await search_papers({
  platform: "semantic_scholar",
  query: "transformer models NLP",
  maxResults: 100
});

// 2. Filter by citations
const highImpact = results.filter(p => p.citationCount > 100);

// 3. Get related papers
for (const paper of highImpact) {
  await get_paper_recommendations({
    doi: paper.doi,
    limit: 5
  });
}
```

### Systematic Review
```typescript
// Search across multiple databases
const platforms = ["pubmed", "wos", "scopus", "crossref"];

for (const platform of platforms) {
  await search_papers({
    platform,
    query: "(systematic review) AND (machine learning diagnosis)",
    filters: { year: "2020-2024" }
  });
}
```

### Citation Analysis
```typescript
// Get paper with citations
const paper = await search_semantic_scholar({
  query: "BERT: Pre-training of Deep Bidirectional Transformers",
  fields: ["citationCount", "influentialCitationCount", "references", "citations"]
});

// Analyze citation network
console.log(`Total citations: ${paper.citationCount}`);
console.log(`Influential citations: ${paper.influentialCitationCount}`);
```

### PDF Retrieval
```typescript
// Try open access first
let pdf = await fetch_paper({
  doi: "10.1038/nature12373",
  platform: "crossref"
});

// Fallback to Sci-Hub if needed (legal caution)
if (!pdf.openAccess) {
  pdf = await fetch_scihub_pdf({
    doi: "10.1038/nature12373"
  });
}
```

## Platform-Specific Advanced Features

### Springer Dual API Support
Springer supports both REST and GraphQL APIs:

```typescript
// REST API (default)
await search_springer({
  query: "quantum computing",
  api: "rest"
});

// GraphQL API (more flexible)
await search_springer({
  query: "quantum computing",
  api: "graphql",
  fields: ["title", "creators", "doi", "abstract"]
});
```

### Web of Science Advanced Search (WQL)
```typescript
// Topic + Author + Year
await search_wos({
  query: "TS=(deep learning) AND AU=(LeCun) AND PY=(2015-2024)"
});

// Citation count filter
await search_wos({
  query: "TS=(CRISPR) AND TC >= 100"  // 100+ citations
});
```

### Sci-Hub Mirror Management
```typescript
// Check mirror status
const mirrors = [
  "https://sci-hub.se",
  "https://sci-hub.st",
  "https://sci-hub.ru"
];

// Tool automatically tries mirrors in order
// Override via SCIHUB_MIRROR env variable
```

## Rate Limits and Best Practices

| Platform | Limit | Recommendation |
|----------|-------|----------------|
| arXiv | 3 req/s | Batch queries, cache results |
| PubMed | 3 req/s | Use E-utilities efficiently |
| Google Scholar | IP-based | Minimize usage, add delays |
| Semantic Scholar | 100 req/5min | Use GraphQL for efficiency |
| Web of Science | API key dependent | Check your plan limits |
| Crossref | No strict limit | Be respectful, ~50 req/s max |
| Sci-Hub | Varies by mirror | Use sparingly, legal caution |

### Best Practices:
1. **Cache results**: Store search results locally to avoid re-querying
2. **Batch operations**: Group related queries together
3. **Use filters**: Narrow searches to reduce API calls
4. **Respect rate limits**: Add delays between requests
5. **Check open access**: Try open platforms before premium APIs
6. **API key management**: Store keys in `.env`, never commit to git

## Data Model

All platforms return standardized `Paper` objects:

```typescript
interface Paper {
  // Identifiers
  doi?: string;
  arxivId?: string;
  pmid?: string;
  pmcid?: string;

  // Core metadata
  title: string;
  authors: Author[];
  abstract?: string;

  // Publication info
  journal?: string;
  volume?: string;
  issue?: string;
  pages?: string;
  year?: number;
  publishedDate?: string;

  // Access
  pdfUrl?: string;
  openAccess?: boolean;

  // Metrics
  citationCount?: number;
  influentialCitationCount?: number;

  // Platform-specific
  platform: string;
  sourceData?: Record<string, any>;
}
```

## Error Handling

```typescript
try {
  const results = await search_papers({
    platform: "arxiv",
    query: "quantum computing"
  });
} catch (error) {
  if (error.code === 'RATE_LIMIT_EXCEEDED') {
    // Wait and retry
    await sleep(5000);
    retry();
  } else if (error.code === 'INVALID_API_KEY') {
    // Check API key configuration
    console.error('API key invalid or missing');
  } else if (error.code === 'PLATFORM_UNAVAILABLE') {
    // Try alternative platform
    await search_papers({ platform: "crossref", query });
  }
}
```

## Troubleshooting

### Issue: API key not recognized
```bash
# Check environment variables
echo $WOS_API_KEY
echo $PUBMED_API_KEY

# Verify .env file location (should be in working directory)
ls -la .env

# Test API key directly
curl -H "Authorization: Bearer $WOS_API_KEY" "https://api.clarivate.com/..."
```

### Issue: Rate limit exceeded
```bash
# Add delays between requests
# Use batch operations
# Cache results locally
# Consider upgrading API plan
```

### Issue: Google Scholar blocking
```bash
# Google Scholar is IP-sensitive:
# - Reduce request frequency
# - Use rotating proxies (if permitted)
# - Consider Semantic Scholar as alternative
# - Wait 24-48 hours if blocked
```

### Issue: Sci-Hub mirror down
```bash
# Tool automatically tries multiple mirrors
# Override with specific mirror:
export SCIHUB_MIRROR=https://sci-hub.st

# Check mirror status manually
curl -I https://sci-hub.se
```

### Issue: Empty results
```bash
# Check query syntax for platform
# Verify filters are not too restrictive
# Try broader search terms
# Check if platform is accessible (API key, network)
```

## Compliance and Legal Notice

### Sci-Hub Usage
⚠️ **Important**: Sci-Hub access may violate copyright laws in many countries. Use responsibly:
- Only for papers you have legal access to
- Educational/fair use purposes where permitted
- Check local laws and institutional policies
- Consider legal alternatives first (open access, institutional access)

### Google Scholar Terms
⚠️ **Important**: Google Scholar prohibits automated scraping:
- Use minimally and responsibly
- Add delays between requests
- Consider official APIs or Semantic Scholar as alternatives
- Respect robots.txt and rate limits

### API Terms of Service
Each platform has its own terms:
- **Web of Science**: Check your license agreement
- **Scopus**: Institutional access required
- **Elsevier/ScienceDirect**: API agreement needed
- **Springer Nature**: Check API usage limits

## Additional Resources

- **GitHub**: https://github.com/your-repo/paper-search-mcp-nodejs
- **Documentation**: Full API docs in package README
- **npm Package**: https://www.npmjs.com/package/paper-search-mcp-nodejs
- **Version**: v0.2.5 (latest)
- **MCP Protocol**: https://modelcontextprotocol.io/

## Updating the Tool

```bash
# Update to latest version
npm update -g paper-search-mcp-nodejs

# Reinstall MCP server (if needed)
claude mcp remove paper-search
claude mcp add paper-search npx paper-search-mcp-nodejs

# Verify version
npx paper-search-mcp-nodejs --version
```

## Related Skills

- `agentdb-vector-search`: For semantic search over downloaded papers
- `reasoningbank-intelligence`: For analyzing research patterns
- `github-code-review`: For reviewing code in academic software

## Support

For issues or questions:
1. Check the troubleshooting section
2. Review platform-specific documentation
3. Check GitHub issues
4. Verify API keys and configuration
5. Test with minimal examples

---

**Pro Tips**:
- Start with open platforms (arXiv, Crossref, bioRxiv) - no API keys needed
- Use Semantic Scholar for citation analysis - excellent free API
- Cache results locally to minimize API calls
- Combine multiple platforms for comprehensive coverage
- Always check open access status before using Sci-Hub
