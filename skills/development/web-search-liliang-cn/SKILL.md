---
name: web-search
description: Perform comprehensive web searches and research using the MCP Web Search Server.
user-invocable: true
---

# Web Search Assistant

You have access to powerful web search tools via the `mcp-websearch-server`. Use these tools to answer user queries with up-to-date information, perform deep research, and verify facts.

## Available Tools

1.  **`websearch_basic`**: 
    -   **Use for**: Quick lookups, fact-checking, or finding a specific URL.
    -   **Returns**: Titles, URLs, and short snippets.
    -   **Cost**: Low latency.

2.  **`websearch_with_content`**: 
    -   **Use for**: Deep dives into specific pages or when snippets are insufficient.
    -   **Returns**: Full extracted text content from the page (intelligent extraction).
    -   **Cost**: Medium latency (fetches page).

3.  **`websearch_multi_engine`**: 
    -   **Use for**: Broad research on complex or controversial topics.
    -   **Returns**: Aggregated results from Bing, Brave, and DuckDuckGo.
    -   **Cost**: Higher latency (multiple searches).

4.  **`websearch_ai_summary`**: 
    -   **Use for**: Getting a pre-digested, AI-ready summary of a topic.
    -   **Returns**: Structured markdown optimized for LLM analysis.

## Recommended Workflows

### 🔍 Deep Research
When the user asks for in-depth research on a topic:
1.  **Broad Sweep**: Use `websearch_multi_engine` with a query like "overview of [topic]" or specific sub-questions.
2.  **Identify Sources**: Look at the results. If a result looks promising but the snippet is cut off, use `websearch_with_content` on that specific URL (or use the tool if it supports URL input, otherwise search specifically for that page).
3.  **Synthesize**: Combine information from multiple sources. Cite your sources using the URLs provided.

### ✅ Fact Checking
When the user asks "Is [X] true?":
1.  **Quick Check**: Use `websearch_basic` with a precise query.
2.  **Verify**: If results are conflicting, use `websearch_multi_engine` to see different perspectives.

### 📰 Latest News
When the user asks for recent events:
1.  Use `websearch_basic` or `websearch_multi_engine` with "latest news [topic]".
2.  Prioritize results with recent dates in the snippet (if visible) or title.

## Tips
-   Always cite the source (URL) when providing information found via search.
-   If the first search yields no results, try a broader keyword or a different search tool (e.g., switch from `websearch_basic` to `websearch_multi_engine`).
-   For technical documentation or coding questions, prefer `websearch_with_content` to get code examples and full context.
