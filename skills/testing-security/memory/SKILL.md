---
name: memory
description: This skill should be used when the user asks to "/gobby memory", "remember", "recall", "forget memory". Manage persistent memories across sessions - store, search, delete, update, and list memories.
category: core
alwaysApply: false
triggers: remember, recall, forget, memory
metadata:
  gobby:
    audience: all
    format_overrides:
      autonomous: full
---

# /gobby memory - Memory Management Skill

This skill manages persistent memories via the gobby-memory MCP server. Parse the user's input to determine which subcommand to execute.

## Tool Schema Reminder

**First time calling a tool this session?** Use `get_tool_schema(server_name, tool_name)` before `call_tool` to get correct parameters. Schemas are cached per session—no need to refetch.

## Subcommands

### `/gobby memory remember <content>` - Store a memory
Call `create_memory` with:
- `content`: (required) The memory content to store
- `memory_type`: Optional type categorization
- `tags`: Comma-separated tags (e.g., "testing,security")
- `project_id`: Optional project scope (defaults to current)

Example: `/gobby memory remember Always use pytest fixtures for test setup`
→ `create_memory(content="Always use pytest fixtures for test setup", tags="testing")`

Example: `/gobby memory remember [critical] Never commit .env files`
→ `create_memory(content="Never commit .env files", tags="critical,security")`

### `/gobby memory recall <query>` - Search/recall memories
Call `search_memories` with:
- `query`: Search query text
- `limit`: Max results (default 10)
- `tags_all`: Require all these tags (comma-separated)
- `tags_any`: Match any of these tags
- `tags_none`: Exclude these tags
- `project_id`: Optional project scope

Returns memories matching the query, ranked by relevance.

Example: `/gobby memory recall testing best practices` → `search_memories(query="testing best practices")`
Example: `/gobby memory recall tag:security` → `search_memories(tags_any="security")`

### `/gobby memory forget <memory-id>` - Delete a memory
Call `delete_memory` with:
- `memory_id`: (required) The memory ID to delete

Example: `/gobby memory forget a1b2c3d4-5678-9abc-def0-1234567890ab` → `delete_memory(memory_id="a1b2c3d4-5678-9abc-def0-1234567890ab")`

### `/gobby memory list` - List all memories
Call `list_memories` with:
- `limit`: Max results (default 20)
- `memory_type`: Filter by type
- `tags_all`: Require all these tags
- `tags_any`: Match any of these tags
- `tags_none`: Exclude these tags
- `project_id`: Optional project scope

Returns all stored memories, most recent first.

Example: `/gobby memory list` → `list_memories(limit="20")`
Example: `/gobby memory list tag:workflow` → `list_memories(tags_any="workflow")`

### `/gobby memory show <memory-id>` - Get memory details
Call `get_memory` with:
- `memory_id`: (required) The memory ID to retrieve

Returns full memory details including content, tags, and metadata.

Example: `/gobby memory show a1b2c3d4-5678-9abc-def0-1234567890ab` → `get_memory(memory_id="a1b2c3d4-5678-9abc-def0-1234567890ab")`

### `/gobby memory update <memory-id>` - Update a memory
Call `update_memory` with:
- `memory_id`: (required) The memory ID to update
- `content`: New content (optional)
- `tags`: New tags (optional)

Example: `/gobby memory update a1b2c3d4-5678-9abc-def0-1234567890ab tags=critical,security` → `update_memory(memory_id="a1b2c3d4-5678-9abc-def0-1234567890ab", tags="critical,security")`

### `/gobby memory related <memory-id>` - Get related memories
Call `get_related_memories` with:
- `memory_id`: (required) The memory ID to find relations for
- `limit`: Max results
- `min_similarity`: Minimum similarity threshold

Returns memories related via cross-references.

Example: `/gobby memory related a1b2c3d4-5678-9abc-def0-1234567890ab` → `get_related_memories(memory_id="a1b2c3d4-5678-9abc-def0-1234567890ab")`

### `/gobby memory stats` - Show memory statistics
Call `memory_stats` to retrieve:
- Total memory count
- Memories by type
- Storage usage
- Recent activity

Example: `/gobby memory stats` → `memory_stats()`

### `/gobby memory graph <query>` - Search knowledge graph
Call `search_knowledge_graph` with:
- `query`: (required) Search query string
- `limit`: Max results (default 10)

Searches the Neo4j knowledge graph for entities matching the query.

Example: `/gobby memory graph authentication` → `search_knowledge_graph(query="authentication")`

## Response Format

After executing the appropriate MCP tool, present the results clearly:
- For remember/create: Confirm storage with memory ID
- For recall: List matching memories with ID, content snippet, and relevance
- For forget/delete: Confirm deletion
- For list: Display memories with ID, content, tags, and creation date
- For show: Full memory details
- For update: Confirm update
- For related: List related memories with similarity scores
- For stats: Show statistics in a readable summary
- For graph: Show matching entities and relationships

## Tag Extraction

When storing memories, extract implicit tags from content:
- `[tag]` syntax → explicit tag
- `testing`, `test` → tag: testing
- `security`, `auth` → tag: security
- `workflow`, `process` → tag: workflow
- `code`, `implementation` → tag: code

## Error Handling

If the subcommand is not recognized, show available subcommands:
- remember, recall, forget, list, show, update, related, stats, graph
