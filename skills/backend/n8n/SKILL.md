---
name: n8n
description: N8N workflow automation platform. Use for workflow automation, integrations, nodes, credentials, API development, RAG/AI workflows, and low-code automation solutions.
---

# N8N Skill

Comprehensive assistance with n8n development, generated from official documentation.

## When to Use This Skill

This skill should be triggered when:
- Building or debugging n8n workflows
- Working with n8n nodes (core, integration, or AI nodes)
- Implementing RAG (Retrieval-Augmented Generation) workflows with vector stores
- Using n8n's REST API for programmatic workflow management
- Writing custom JavaScript/Python code in n8n Code nodes
- Setting up n8n authentication and credentials
- Handling API rate limits in workflows
- Working with Luxon for date/time operations
- Using expressions to manipulate data between nodes
- Deploying or hosting n8n (Docker, self-hosted, or cloud)
- Creating AI agents with LangChain integration
- Integrating with OpenAI, ChatGPT, or other LLM services

## Quick Reference

### RAG Workflow - Inserting Data

```javascript
// Basic RAG workflow structure:
// 1. Fetch source data (HTTP Request, Google Drive, etc.)
// 2. Insert into Vector Store node with "Insert Documents" operation
// 3. Select embedding model (e.g., text-embedding-ada-002)
// 4. Add Default Data Loader to chunk content
// 5. Optional: Add metadata for better filtering

// Chunking strategies:
// - Character Text Splitter: splits by character length
// - Recursive Character Text Splitter: splits by Markdown/HTML/code (recommended)
// - Token Text Splitter: splits by token count

// Small chunks (200-500 tokens): fine-grained retrieval
// Large chunks: more context but potentially noisy
```

### RAG Workflow - Querying with Agent

```javascript
// Using an agent to query vector store:
// 1. Add an Agent node to your workflow
// 2. Connect Vector Store as a tool
// 3. Give it a clear description (helps agent decide when to use it)
// 4. Set limit for chunks to return
// 5. Enable "Include Metadata" for extra context
// 6. Use same embedding model as when inserting data

// Cost-saving tip: Use Vector Store Question Answer tool first
// to retrieve relevant data, then pass to expensive model
```

### Code Node - Data Transformation

```javascript
// Finding data in nested structures
const slackUser = $("Mock Slack").all()[0];
const notionUsers = $input.all();
const slackUserEmail = slackUser.json.email;

const notionUser = notionUsers.find(
  (user) => user.json.person && user.json.person.email === slackUserEmail
);

return notionUser ? [{ json: { notionId: notionUser.json.id } }] : [];
```

### Code Node - Joining Items into Text

```javascript
// Return a single line of text with all usernames, comma-separated and quoted
const items = $input.all();
const usernames = items.map((item) => `"${item.json.username}"`);
const result = usernames.join(", ");
return [{ json: { usernames: result } }];
```

### Code Node - Data Aggregation

```javascript
// Count submissions by type and list top 5 by votes
const submissions = $input.all();

let ideaCount = 0, featureCount = 0, bugCount = 0;

submissions.forEach((submission) => {
  switch (submission.json.property_type[0]) {
    case "Idea": ideaCount++; break;
    case "Feature": featureCount++; break;
    case "Bug": bugCount++; break;
  }
});

// Sort by votes and take top 5
const topSubmissions = submissions
  .sort((a, b) => b.json.property_votes - a.json.property_votes)
  .slice(0, 5);

return [{ json: { ideaCount, featureCount, bugCount, topSubmissions } }];
```

### Luxon Date/Time - Current Time

```javascript
// Get current datetime
{{$now}}
// Returns ISO formatted: 2022-03-09T14:02:37.065+00:00

// Get today's date (rounded down to day)
{{$today}}
// Equivalent to: DateTime.now().set({ hour: 0, minute: 0, second: 0, millisecond: 0 })

// String concatenation behavior
{{"Today's date is " + $now}}
// Returns: "Today's date is 1646834498755" (unix timestamp)
```

### Expressions - Referencing Node Data

```javascript
// Reference data from specific nodes using dot notation
// For each person, extract first name and job title
const items = $input.all();
const newItems = items.map((item) => {
  const firstName = item.json.personal_info.first_name;
  const jobTitle = item.json.work_info.job_title;
  return { json: { firstName, jobTitle } };
});
return newItems;
```

### n8n API - Authentication

```bash
# Authenticate API calls with X-N8N-API-KEY header
curl -X GET \
  https://your-instance.com/api/v1/workflows?active=true \
  -H "X-N8N-API-KEY: your_api_key_here"

# Create API key:
# 1. Go to Settings > n8n API
# 2. Create an API key with label and expiration
# 3. Set scopes (enterprise only)
# 4. Copy and use key in X-N8N-API-KEY header
```

### Docker Compose - Production Setup

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      - N8N_HOST=${SUBDOMAIN}.${DOMAIN_NAME}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - N8N_RUNNERS_ENABLED=true
      - NODE_ENV=production
      - WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}/
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./local-files:/files

volumes:
  n8n_data:
```

### Handling API Rate Limits

```javascript
// Method 1: Enable "Retry On Fail" in node settings
// - Automatically retries on failure with pause between attempts

// Method 2: Use Loop Over Items + Wait nodes
// 1. Loop Over Items: batch requests into smaller chunks
// 2. Wait node: pause between batches
// 3. Process each batch through API node

// Rate limit error: HTTP 429 "Too many requests"
// Check API docs for specific limits (per minute/day)
```

## Reference Files

This skill includes comprehensive documentation in `references/`:

### advanced.md - Advanced Documentation (39 pages)
- **RAG (Retrieval-Augmented Generation)**: Complete guide to implementing RAG workflows with vector stores, embedding models, and chunking strategies
- **AI Tools**: Understanding and using tools in AI agents (Wikipedia, SerpAPI, Call n8n Workflow Tool, Custom Code Tool, HTTP Request Tool)
- **OpenAI Integration**: File operations, conversation management, and text operations with ChatGPT/GPT models

### api.md - API Documentation (12 pages)
- **REST API Overview**: Introduction to n8n's public API for programmatic workflow management
- **Authentication**: API key creation, scopes, and usage with X-N8N-API-KEY header
- **API Playground**: Using Scalar's interactive playground and built-in Swagger UI
- **Monitoring Endpoints**: `/healthz`, `/healthz/readiness`, and `/metrics` for instance monitoring
- **Base URL Configuration**: Setting VUE_APP_URL_BASE_API for separate front-end/back-end hosting

### code.md - Code Documentation (26 pages)
- **AI Coding with GPT**: Using ChatGPT to generate JavaScript in Code nodes (Cloud only)
- **Writing Good Prompts**: Best practices for AI code generation with examples
- **Data Structure**: Understanding n8n's data format for custom code
- **Expressions Cookbook**: Common expression patterns and recipes
- **Luxon Date/Time**: Working with dates using the Luxon library
- **Convenience Methods**: Built-in helper methods like `$evaluateExpression()` and `$ifEmpty()`

### getting_started.md - Getting Started
- Initial setup and configuration
- Basic workflow concepts
- First workflow creation

### hosting.md - Hosting Documentation
- Docker deployment options
- Self-hosted installation
- Traefik SSL/TLS configuration
- Environment variables for production

### integrations.md - Integrations
- Node types and usage
- Credential management
- API connection patterns

### workflows.md - Workflows Documentation
- Workflow design patterns
- Data flow between nodes
- Debugging and testing

### other.md - Other Documentation
- Miscellaneous topics
- Advanced configuration
- Troubleshooting

## Working with This Skill

### For Beginners
- Start with **getting_started.md** for foundational concepts
- Review **workflows.md** to understand how nodes connect and data flows
- Use the Quick Reference examples above for common patterns
- Learn about n8n's data structure (arrays of JSON objects)

### For AI/LLM Workflows
- Read **advanced.md** for RAG implementation details
- Understand vector stores, embedding models, and chunking strategies
- Learn about AI agents and tools
- Use the RAG examples in Quick Reference as templates

### For Custom Code
- Study **code.md** for JavaScript/Python in Code nodes
- Learn AI-assisted code generation with ChatGPT (Cloud only)
- Master Luxon for date/time operations
- Review expression syntax and built-in methods

### For API Integration
- Check **api.md** for REST API authentication and endpoints
- Use the API playground to test calls
- Learn about rate limit handling
- Understand pagination and monitoring

### For Deployment
- See **hosting.md** for Docker Compose configurations
- Configure SSL/TLS with Traefik
- Set environment variables for production
- Enable metrics and health check endpoints

## Key Concepts

### Data Structure
n8n uses a specific data format between nodes:
- Each node outputs an array of items
- Each item has a `json` property containing the data
- Access data with `$input.all()` or `$("NodeName").all()`
- Transform data to maintain this structure

### Expressions
- JavaScript expressions wrapped in `{{  }}`
- Access node data with `$input`, `$json`, `$node`
- Use Luxon for dates: `$now`, `$today`
- Built-in methods: `$evaluateExpression()`, `$ifEmpty()`

### Vector Stores & RAG
- **Vector Store**: Database for high-dimensional vectors representing text/data
- **Embedding Model**: Converts text chunks to vectors (e.g., text-embedding-ada-002)
- **Chunking**: Splitting documents into smaller pieces for better retrieval
- **Semantic Search**: Find relevant content by meaning, not keywords

### AI Agents & Tools
- **Agent**: AI that can use tools to interact with external systems
- **Tools**: Interfaces for agents (workflows, APIs, code, databases)
- **Call n8n Workflow Tool**: Load any workflow as a tool for agents
- **Context Passing**: Provide relevant data to agents for better responses

### API Authentication
- Create API keys in Settings > n8n API
- Use `X-N8N-API-KEY` header for authentication
- Enterprise: Scope keys to limit access
- Non-enterprise: Keys have full account access

### Rate Limits
- **Identification**: Error 429 "Too many requests"
- **Retry On Fail**: Automatic retry with delays
- **Batching**: Use Loop Over Items + Wait to chunk requests
- **Documentation**: Check API docs for specific limits

## Resources

### Official Documentation
All reference files are extracted from official n8n documentation and include:
- Detailed explanations with context
- Real code examples with language annotations
- Links to original documentation
- Table of contents for navigation

### Common Patterns
The Quick Reference section contains practical patterns for:
- RAG workflow implementation
- Code node data transformations
- Date/time operations with Luxon
- API authentication and calls
- Docker deployment configurations

### Helper Scripts
Add automation scripts to `scripts/` directory for:
- Workflow backup/restore
- Bulk credential management
- API testing utilities

### Templates & Examples
Add to `assets/` directory:
- Workflow templates (JSON)
- Docker Compose configurations
- Common integration patterns

## Notes

- This skill was automatically generated from official n8n documentation
- Code examples are extracted from real documentation and tested patterns
- AI code generation is Cloud-only (not available in self-hosted)
- Python support is limited to Code node, not available in expressions
- Vector stores and RAG features require appropriate n8n plan
- API access requires upgrade from free trial

## Best Practices

### Writing Code in Code Node
1. Use AI assistance to generate initial code (Cloud only)
2. Provide clear prompts with expected input/output
3. Reference node data explicitly with dot notation
4. Always return data in n8n format: `[{ json: { ... } }]`
5. Handle null/undefined values in nested structures

### Building RAG Workflows
1. Choose appropriate embedding model (smaller = faster, larger = more accurate)
2. Select optimal chunk size (200-500 tokens for fine-grained retrieval)
3. Use Recursive Character Text Splitter for most use cases
4. Add metadata to chunks for better filtering
5. Save costs by using QA tool before expensive models

### API Integration
1. Always handle rate limits (Retry On Fail or batching)
2. Use specific error handling for API failures
3. Test with API playground before deploying
4. Store credentials securely in n8n credential store
5. Monitor with `/metrics` and `/healthz` endpoints

### Deployment
1. Use Docker Compose for production
2. Configure Traefik for automatic SSL/TLS
3. Set `GENERIC_TIMEZONE` for consistent date/time
4. Mount volumes for data persistence
5. Enable `N8N_RUNNERS_ENABLED` for better performance

## Updating

To refresh this skill with updated documentation:
1. Re-run the scraper with the same configuration
2. The skill will be rebuilt with the latest information
3. Quick Reference examples will be updated with new patterns
