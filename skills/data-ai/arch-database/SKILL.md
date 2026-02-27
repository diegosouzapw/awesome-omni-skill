---
name: arch-database
cluster: se-architecture
description: "DB architecture: relational vs document vs graph vs vector, schema design, indexing, replication, sharding"
tags: ["database","schema","indexing","architecture"]
dependencies: []
composes: []
similar_to: []
called_by: []
authorization_required: false
scope: general
model_hint: claude-sonnet
embedding_hint: "database schema indexing replication sharding polyglot sql nosql graph"
---

# arch-database

## Purpose
This skill helps the AI agent advise on database architecture decisions, including selecting between relational, document, graph, and vector databases; designing schemas; optimizing indexing; and planning replication and sharding for scalable systems.

## When to Use
Use this skill when architecting a new application backend, migrating databases, handling high-traffic data needs, or resolving performance issues. For example, choose it for e-commerce apps needing transactions (relational) versus social graphs (graph DBs). Avoid for low-level coding tasks like writing SQL queries; pair with query-focused skills instead.

## Key Capabilities
- Compare database types: Evaluate relational (e.g., PostgreSQL) vs. document (e.g., MongoDB) based on data structure and queries.
- Design schemas: Generate ER diagrams or JSON schemas with constraints, e.g., defining primary keys and relationships.
- Optimize indexing: Recommend indexes like B-tree for relational or full-text for document DBs to reduce query times.
- Handle replication and sharding: Suggest setups like master-slave for fault tolerance or horizontal sharding for load balancing.
- Support polyglot persistence: Advise on mixing SQL and NoSQL in a system, e.g., using Redis for caching with PostgreSQL.

## Usage Patterns
Invoke this skill via OpenClaw prompts prefixed with "arch-database:", e.g., "arch-database: compare relational and graph for a social network." For programmatic use, call the OpenClaw API endpoint `/api/skills/arch-database` with a JSON payload. Always include context like app requirements (e.g., read-heavy vs. write-heavy). If using in a script, wrap calls in error checks to handle API failures. For multi-step tasks, chain with other se-architecture skills, like starting with schema design then moving to indexing.

## Common Commands/API
Use OpenClaw CLI for quick interactions: `openclaw run arch-database --compare relational document --app-type social` (outputs pros/cons). For API, POST to `https://api.openclaw.ai/v1/skills/arch-database` with body like:
```json
{
  "action": "design-schema",
  "params": {"db-type": "relational", "tables": ["users", "posts"]}
}
```
Require authentication via header: `Authorization: Bearer $OPENCLAW_API_KEY`. Common flags: `--verbose` for detailed output, `--output json` for structured results. For config files, use YAML like:
```yaml
db-arch:
  type: graph
  schema: {nodes: users, edges: friendships}
```
To generate indexing advice, run: `openclaw run arch-database --index-suggest --query "SELECT * FROM users WHERE name LIKE '%john%'"`.

## Integration Notes
Integrate by setting environment variables for API access, e.g., export `OPENCLAW_API_KEY=your-secret-key`. When embedding in larger workflows, use OpenClaw's SDK: import openclaw; client = openclaw.Client(api_key=os.environ['OPENCLAW_API_KEY']); response = client.invoke('arch-database', {'action': 'replication-plan', 'replicas': 3}). For polyglot setups, ensure compatibility by specifying DB drivers in your app config, like adding "postgresql" and "neo4j" to a Node.js project's package.json. Test integrations in a sandbox environment before production.

## Error Handling
When invoking, check for errors like invalid parameters (e.g., API returns 400 if db-type is misspelled). Handle with try-except in code: try: response = client.invoke('arch-database', params) except openclaw.APIError as e: print(f"Error: {e.status_code} - {e.message}"). For common issues, retry on 5xx errors with exponential backoff. If the skill returns "incompatible architecture," refine your input (e.g., specify data volume). Log all responses for debugging, and use the `--debug` flag in CLI to get detailed traces.

## Concrete Usage Examples
1. **Example 1: Compare DB types for a user profile system**  
   Prompt: "arch-database: compare relational and document for a system with 1M user profiles, frequent updates."  
   Expected: The agent outputs: "Use relational (e.g., MySQL) for structured data and ACID compliance; document (e.g., MongoDB) for flexible schemas. Recommendation: Relational with indexing on user_id."  
   Follow up: Use the output to generate a schema via API: POST to /api/skills/arch-database with {"action": "design-schema", "db-type": "relational"}.

2. **Example 2: Design schema with sharding for a social network**  
   Command: `openclaw run arch-database --design-schema --db-type graph --sharding key-based --entities users,posts`  
   Expected: Agent responds with: "Schema: Nodes: users {id, name}; Edges: friendships {from, to}. Sharding: Shard by user ID for even distribution."  
   Integrate: Export as YAML and apply in your app's deployment script.

## Graph Relationships
- Related to: se-architecture cluster (e.g., links to data-modeling for schema refinement).
- Connected via: tags like "database" to query-optimization skill for indexing follow-ups.
- Dependencies: Requires se-architecture base for replication advice; outputs can feed into deployment skills.
