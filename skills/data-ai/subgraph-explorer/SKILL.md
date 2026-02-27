---
name: subgraph-explorer
description: Explore and query blockchain subgraphs through a private MCP server running in Docker. Use this skill when exploring GraphQL subgraphs, querying blockchain data from subgraphs (NFT transfers, DEX swaps, DeFi metrics), examining subgraph schemas, or exporting discovered queries for project use. The skill manages Docker-based MCP server interaction and provides utilities for query development and export.
---

# Subgraph Explorer

## Overview

This skill enables exploration and querying of blockchain subgraphs through a private MCP server. It provides tools for managing the Docker-based server, exploring GraphQL schemas, executing queries against configured subgraphs, and exporting discovered queries for project integration.

## Quick Start

### Starting the MCP Server

Before using subgraph exploration features, ensure the MCP server is running:

```bash
bash scripts/start_mcp_server.sh
```

This starts the Docker container with:
- **SSE endpoint**: `http://localhost:8000` (for MCP communication)
- **Metrics endpoint**: `http://localhost:9091/metrics` (for monitoring)

Check server status:
```bash
bash scripts/check_mcp_status.sh
```

Stop the server:
```bash
bash scripts/stop_mcp_server.sh
```

**Note**: The scripts default to `~/Workspace/subgraph-mcp` as the project path. Set `SUBGRAPH_MCP_PATH` environment variable to override.

### MCP Server Connection

The MCP server runs in SSE mode and exposes the following tools via HTTP:

**Registry-based tools:**
- `list_subgraphs` - List all configured subgraphs
- `search_subgraphs_by_keyword` - Search subgraphs by keyword
- `get_schema_by_id` - Get GraphQL schema for a configured subgraph
- `execute_query_by_id` - Execute query against a configured subgraph
- `get_query_examples_by_id` - Get query examples for a subgraph
- `get_subgraph_guidance_by_id` - Get subgraph-specific guidance

**Ad-hoc tools:**
- `get_schema_by_url` - Get schema from any GraphQL endpoint (no registry needed)
- `execute_query_by_url` - Execute query against any GraphQL endpoint (no registry needed)

To interact with the MCP server, use the WebFetch tool to make HTTP requests to the SSE endpoint at `http://localhost:8000`.

## Core Workflows

### 1. Exploring Configured Subgraphs

When exploring subgraphs in the registry (`subgraphs.json`):

**Step 1: List or Search**
- Use `list_subgraphs` to see all available subgraphs
- Use `search_subgraphs_by_keyword` to find specific subgraphs by name/description

**Step 2: Understand the Schema**
- Use `get_schema_by_id` to retrieve the GraphQL schema
- Examine entity types, fields, and relationships
- Check `get_query_examples_by_id` for pre-built query templates
- Review `get_subgraph_guidance_by_id` for subgraph-specific tips

**Step 3: Execute Queries**
- Use `execute_query_by_id` to run GraphQL queries
- Start with simple queries and iterate
- Apply pagination for large result sets
- Reference `references/graphql_patterns.md` for common patterns

**Step 4: Export Useful Queries**
- Use `scripts/export_query.py` to save queries for project use
- Choose format: JavaScript, Python, GraphQL, or JSON

### 2. Ad-hoc Subgraph Exploration

For exploring subgraphs not in the registry:

**Direct URL Access:**
- Use `get_schema_by_url` with the GraphQL endpoint URL
- Optionally provide `auth_header` if authentication is required
- Use `execute_query_by_url` to run queries directly

Example workflow:
1. Get schema: `get_schema_by_url(url="https://example.com/graphql")`
2. Analyze available entities and fields
3. Build query based on schema
4. Execute: `execute_query_by_url(url="https://example.com/graphql", query="...", variables={...})`

### 3. Query Development Process

**Iterative Query Building:**

1. **Start Simple**: Query a single entity to understand data structure
   ```graphql
   query SimpleQuery {
     entity(id: "0x123") {
       id
       name
     }
   }
   ```

2. **Add Fields**: Gradually add more fields as needed
   ```graphql
   query ExpandedQuery {
     entity(id: "0x123") {
       id
       name
       timestamp
       relatedData {
         field1
         field2
       }
     }
   }
   ```

3. **Apply Filters**: Use `where` clauses for specific criteria
   ```graphql
   query FilteredQuery($minValue: String!) {
     entities(where: { value_gte: $minValue }, first: 100) {
       id
       value
       timestamp
     }
   }
   ```

4. **Optimize**: Use aggregated fields instead of large arrays
   - Prefer: `contract.holders` (pre-calculated count)
   - Avoid: Counting all `tokens` manually

**Reference**: See `references/graphql_patterns.md` for comprehensive query patterns including pagination, filtering, aggregation, and performance optimization.

## Exporting Queries

Use the export utility to save discovered queries for project integration:

### JavaScript/TypeScript Export
```bash
python3 scripts/export_query.py queries/getLatestSwaps.js --format js --name GetLatestSwaps --description "Fetch latest DEX swaps"
```

Then paste your GraphQL query when prompted.

Output:
```javascript
/**
 * Fetch latest DEX swaps
 */
export const GetLatestSwaps = `
  query GetLatestSwaps($first: Int!) {
    swaps(first: $first, orderBy: timestamp, orderDirection: desc) {
      id
      timestamp
      amountUSD
      pair {
        token0 { symbol }
        token1 { symbol }
      }
    }
  }
`;
```

### Python Export
```bash
python3 scripts/export_query.py queries/get_latest_swaps.py --format py --name get_latest_swaps
```

### GraphQL File Export
```bash
python3 scripts/export_query.py queries/latest_swaps.graphql --format graphql
```

### JSON Export (with metadata)
```bash
python3 scripts/export_query.py queries/latest_swaps.json --format json --name GetLatestSwaps --variables '{"first": 100}'
```

## Understanding Subgraph Data

### Common Entity Types

**DEX/Trading Subgraphs:**
- `Swap` - Individual trade transactions
- `Pair` - Trading pairs with liquidity and volume
- `Token` - Token information and metadata
- `DayData` / `PairDayData` - Aggregated daily metrics

**NFT Subgraphs:**
- `ERC721Transfer` / `ERC1155Transfer` - NFT transfer events
- `Account` - User accounts with balances
- `ERC721Token` / `ERC1155Token` - Individual NFT tokens
- `ERC721Contract` - NFT collection contracts

**Common Patterns:**
- Most entities have `id`, `timestamp`, and `blockNumber` fields
- Use relationship fields (e.g., `pair { token0 { symbol } }`) to navigate connections
- Aggregated fields (e.g., `totalSupply`, `holders`) provide pre-calculated stats

### Data Considerations

**Time-based Data:**
- Daily aggregates typically reset at midnight UTC
- For "today's" data, consider querying both current day and previous day
- Calculate partial day metrics: `(yesterday_value * hours_passed / 24) + today_value`

**Pagination:**
- Maximum `first` parameter is typically 1000, recommended 100
- Use `skip` for offset-based pagination
- Use cursor-based pagination (`id_gt`) for large datasets

**Performance:**
- Avoid deep nesting of relationships
- Use aggregated fields when available
- Apply specific filters to reduce result sets
- Request only needed fields, not entire objects

## Troubleshooting

### MCP Server Issues

**Container won't start:**
- Check if ports 8000 or 9091 are already in use
- Verify `subgraphs.json` exists in the subgraph-mcp directory
- View logs: `docker logs subgraph-mcp-server`

**Server not responding:**
- Run: `bash scripts/check_mcp_status.sh`
- Verify Docker is running
- Check firewall settings for localhost access

**Configuration errors:**
- Verify `SUBGRAPH_MCP_PATH` points to correct directory
- Ensure `subgraphs.json` is valid JSON
- Check subgraph URLs are accessible

### Query Issues

**Schema introspection fails:**
- Verify the subgraph endpoint is accessible
- Check authentication headers if required
- Ensure the endpoint is a valid GraphQL API

**Query timeout:**
- Simplify the query (reduce nesting, fewer fields)
- Add more specific filters
- Reduce `first` parameter value
- Use pagination for large datasets

**Type errors:**
- Check field types in schema (String vs Int vs BigInt)
- Ensure variable types match schema requirements
- Use quotes for String types in variables: `{"id": "0x123"}`

**Empty results:**
- Verify entity IDs are correct (case-sensitive)
- Check filter conditions aren't too restrictive
- For time-based queries, verify timestamp format (usually Unix seconds)
- Confirm data exists in the subgraph (check subgraph sync status)

## Resources

### scripts/

**Docker Management:**
- `start_mcp_server.sh` - Start the MCP server in Docker
- `stop_mcp_server.sh` - Stop the MCP server
- `check_mcp_status.sh` - Check server status and health

**Query Export:**
- `export_query.py` - Export GraphQL queries to various formats (JS, Python, GraphQL, JSON)

### references/

- `graphql_patterns.md` - Comprehensive guide to GraphQL query patterns for subgraphs
  - Pagination strategies
  - Filtering patterns
  - Aggregation techniques
  - Performance optimization
  - Common query scenarios

## Tips

- **Always start with schema exploration** - Use `get_schema_by_id` or `get_schema_by_url` first
- **Check for query examples** - Use `get_query_examples_by_id` for configured subgraphs
- **Read subgraph guidance** - Use `get_subgraph_guidance_by_id` for subgraph-specific tips
- **Test queries incrementally** - Build complex queries step by step
- **Export working queries** - Save successful queries for reuse in projects
- **Monitor performance** - Check metrics endpoint (`http://localhost:9091/metrics`) for server health
- **Use aggregated data** - Prefer pre-calculated fields over manual aggregation
- **Consider UTC timezone** - Daily data typically resets at midnight UTC
