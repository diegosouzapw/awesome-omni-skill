---
name: web-search
description: Search the web for real-time information. Note: International search APIs (DuckDuckGo, Brave, Google) are blocked in China. Use Bing or Baidu API instead.
---
# web-search

@command(web_search)
Usage: web_search --query <query>
Run: |
  # URL encode the query
  QUERY=$(echo "$1" | jq -sRr @uri 2>/dev/null || echo "$1" | sed 's/ /+/g')

  # Option 1: Bing Search API (Recommended for China)
  # Get API key: https://azure.microsoft.com/en-us/services/cognitive-services/bing-web-search-api/
  BING_API_KEY="${BING_SEARCH_API_KEY:-}"
  if [ -n "$BING_API_KEY" ]; then
    result=$(curl -s --connect-timeout 10 "https://api.bing.microsoft.com/v7.0/search?q=${QUERY}&count=5" \
      -H "Ocp-Apim-Subscription-Key: ${BING_API_KEY}" 2>/dev/null | \
      jq -r '.webPages.value[] | "\(.title)\n\(.url)\n\(.snippet)\n---"' 2>/dev/null)

    if [ -n "$result" ] && [ "$result" != "---" ]; then
      echo "$result"
      exit 0
    fi
  fi

  # Option 2: Brave Search API (Requires proxy in China)
  # Note: This key is configured in ~/.openclaw/openclaw.json: BSAyEBmSu2o9kfTFDMTo5D2K7-Oy9JT
  BRAVE_API_KEY="${BRAVE_SEARCH_API_KEY:-BSAyEBmSu2o9kfTFDMTo5D2K7-Oy9JT}"
  result=$(curl -s --connect-timeout 10 "https://api.search.brave.com/res/v1/web/search?q=${QUERY}&count=5" \
    -H "Accept: application/json" \
    -H "X-Subscription-Token: ${BRAVE_API_KEY}" 2>/dev/null | \
    jq -r '.web.results[] | "\(.title)\n\(.url)\n\(.description)\n---"' 2>/dev/null)

  if [ -n "$result" ] && [ "$result" != "---" ]; then
    echo "$result"
    exit 0
  fi

  # Option 3: Try DuckDuckGo (will likely timeout in China)
  result=$(curl -s --connect-timeout 5 "https://api.duckduckgo.com/?q=${QUERY}&format=json&no_html=1&skip_disambig=0" 2>/dev/null)
  if [ -n "$result" ] && echo "$result" | jq -e '.AbstractText != null' > /dev/null 2>&1; then
    echo "$result" | jq -r '.AbstractText'
    exit 0
  fi

  # Option 4: Bing Web Search (HTML fallback)
  result=$(curl -s --connect-timeout 10 "https://cn.bing.com/search?q=${QUERY}" 2>/dev/null | \
    grep -oP '(?<=<h2><a href=")[^"]*' | \
    head -5)

  if [ -n "$result" ]; then
    echo "$result" | sed 's/$/\n/'
    exit 0
  fi

  # All failed
  echo "Error: No working search API found."
  echo ""
  echo "Current search providers tried:"
  echo "  1. Bing Search API (if BING_SEARCH_API_KEY is set)"
  echo "  2. Brave Search API (requires proxy in China)"
  echo "  3. DuckDuckGo API (blocked in China)"
  echo "  4. Bing Web Search (HTML scraping)"
  echo ""
  echo "To enable web search in China:"
  echo "  Option A: Use Bing Search API"
  echo "    1. Get API key: https://azure.microsoft.com/en-us/services/cognitive-services/bing-web-search-api/"
  echo "    2. Set environment variable: export BING_SEARCH_API_KEY='your-key'"
  echo ""
  echo "  Option B: Use Brave Search API with proxy"
  echo "    1. Set up proxy: export https_proxy='http://your-proxy:port'"
  echo "    2. Brave API key is already configured: BSAyEBmSu2o9kfTFDMTo5D2K7-Oy9JT"
  echo ""
  echo "Note: DuckDuckGo, Brave, and Google APIs are blocked in China without proxy."
  exit 1