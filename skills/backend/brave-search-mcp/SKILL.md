---
name: brave-search-mcp
description: Official Brave Search MCP Server for web search, image search, news search, video search, and local POI search. Privacy-focused search API with AI-powered summarization. Connect AI agents to comprehensive search capabilities without Google tracking. Supports web navigation, research, fact-checking, and content discovery. Use when agents need to search the internet, find current information, research topics, verify facts, discover images/videos, or locate businesses/places.
---

# Brave Search MCP Server

> **Privacy-First Search for AI Agents**

Official MCP server from Brave integrating the [Brave Search API](https://brave.com/search/api/). Provides comprehensive search capabilities including web, images, videos, news, and local points of interest with AI-powered summarization.

## Why Brave Search?

### 🔒 Privacy-Focused
No user tracking, no profiling, no search history surveillance. Unlike Google, Brave Search doesn't build profiles or track behavior.

### 🤖 AI-Native Features
- AI-powered summarization (summarizer tool)
- Structured data for agent consumption
- Rich context in search results

### 🌐 Comprehensive Coverage
- **Web Search** - General internet search
- **Image Search** - Visual content discovery
- **Video Search** - Video content from multiple platforms
- **News Search** - Current events and journalism
- **Local POI** - Businesses, restaurants, services near any location

## Installation

```bash
# Official Brave Search MCP Server
npm install -g @brave/brave-search-mcp-server

# Or via GitHub
git clone https://github.com/brave/brave-search-mcp-server
cd brave-search-mcp-server
npm install
npm run build
```

## Configuration

Add to your MCP client config:

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@brave/brave-search-mcp-server"],
      "env": {
        "BRAVE_API_KEY": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

### Get API Key

1. Visit https://brave.com/search/api/
2. Sign up for Brave Search API
3. Free tier: 2,000 queries/month
4. Paid plans available for higher volume

## Available Tools

### 1. Web Search (`brave_web_search`)

General purpose internet search.

**Agent Usage:**
```
"Search for recent developments in quantum computing"
"Find tutorials on React hooks"
"What are the best practices for Docker security?"
```

**Parameters:**
- `query` (required) - Search terms
- `count` (optional) - Number of results (default 10, max 20)
- `offset` (optional) - Pagination offset

### 2. Local Search (`brave_local_search`)

Find businesses, restaurants, services near a location.

**Agent Usage:**
```
"Find coffee shops near San Francisco"
"Pizza restaurants in Brooklyn"
"Gas stations near Times Square"
```

**Parameters:**
- `query` (required) - What to search for
- `location` (optional) - City, address, coordinates

### 3. Image Search (`brave_image_search`)

Visual content discovery.

**Agent Usage:**
```
"Find images of the Golden Gate Bridge"
"Product photography for smartphones"
"Infographics about climate change"
```

### 4. Video Search (`brave_video_search`)

Video content from YouTube, Vimeo, and other platforms.

**Agent Usage:**
```
"Tutorial videos on machine learning"
"Keynotes from recent tech conferences"
"Documentary about space exploration"
```

### 5. News Search (`brave_news_search`)

Current events and journalism.

**Agent Usage:**
```
"Latest news about AI regulation"
"Recent developments in renewable energy"
"Tech industry news this week"
```

### 6. Summarizer (`brave_web_search` with summarizer)

AI-powered summarization of search results.

**Agent Usage:**
```
"Summarize current state of quantum computing research"
"Give me a summary of recent climate policy changes"
```

## Use Cases for Agents

### Research Assistant
```
Agent: "What are the latest findings on CRISPR gene editing?"
Brave Search: Returns recent articles, papers, news with summary
```

### Fact Checking
```
Agent: "Is it true that coffee improves cognitive function?"
Brave Search: Provides sources, studies, verification
```

### Local Discovery
```
Agent: "Find highly-rated sushi restaurants in Seattle"
Brave Search: Returns businesses with ratings, addresses, hours
```

### Content Discovery
```
Agent: "Find video tutorials on Kubernetes deployment"
Brave Search: Returns relevant videos from multiple platforms
```

### News Monitoring
```
Agent: "What's happening with Tesla this week?"
Brave Search: Recent news articles, announcements, coverage
```

## Example Agent Workflow

```
Human: "I'm planning a trip to Tokyo. Help me prepare."

Agent:
1. brave_web_search("Tokyo travel guide 2026")
2. brave_web_search("Tokyo weather forecast")
3. brave_local_search("best ramen restaurants Tokyo")
4. brave_image_search("Tokyo metro map")
5. brave_news_search("Tokyo events 2026")

Agent: "Here's your Tokyo trip prep:
- Weather: [from search results]
- Top ramen spots: [from local search]
- Metro map: [image links]
- Current events: [from news search]"
```

## vs Google Search

| Feature | Brave Search | Google Search |
|---------|--------------|---------------|
| **Privacy** | ✅ No tracking | ❌ Extensive tracking |
| **AI Summary** | ✅ Built-in | ⚠️ Limited |
| **API Cost** | ✅ 2K free/month | ❌ Expensive |
| **Speed** | ✅ Fast | ✅ Fast |
| **Coverage** | ✅ Independent index | ✅ Comprehensive |
| **Agent-Friendly** | ✅ Structured data | ⚠️ Limited |

## Rate Limits

**Free Tier:**
- 2,000 queries/month
- 1 query/second
- Web, image, video, news, local search

**Pro Tier:**
- Higher volume available
- Dedicated support
- See https://brave.com/search/api/ for pricing

## Privacy Guarantee

From Brave:
> "Brave Search does not collect personal data, build user profiles, or track individual searches. Your queries are anonymous."

## Resources

- **Official MCP Server**: https://github.com/brave/brave-search-mcp-server
- **API Documentation**: https://brave.com/search/api/
- **API Key Signup**: https://brave.com/search/api/
- **Brave Search**: https://search.brave.com

## Advanced: Custom Configuration

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "node",
      "args": ["/path/to/brave-search-mcp-server/build/index.js"],
      "env": {
        "BRAVE_API_KEY": "YOUR_API_KEY",
        "DEFAULT_COUNT": "15",
        "ENABLE_SUMMARIZER": "true"
      }
    }
  }
}
```

---

**The search tool every agent needs**: Privacy-first, AI-native, comprehensive coverage. Install once, search forever.
