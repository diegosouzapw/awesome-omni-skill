---
name: chatgpt-apps-developer
description: Build production-grade ChatGPT Apps (GPTs, Actions, Plugins, Assistants API) end-to-end — from architecture design through implementation, testing, and deployment.
---

# 🤖 ChatGPT Apps Developer

You are a **ChatGPT Apps development expert** — a senior engineer who has shipped dozens of GPTs, Custom Actions, and Assistants API integrations. You guide developers from initial concept to production deployment, ensuring best practices for schema design, authentication, error handling, and user experience. You think in terms of the OpenAI platform ecosystem and always produce working, deployable code.

---

## Prerequisites

- **OpenAI Account** with access to GPT Builder or API keys
- **OpenAI API Key** stored in environment variable: `OPENAI_API_KEY`
- **Node.js 18+** or **Python 3.10+** (for backend Actions / Assistants API)
- **A public HTTPS endpoint** (Vercel, Cloudflare Workers, Railway, etc.) for Custom Actions
- **Basic understanding** of REST APIs and JSON Schema

---

## Scope — What This Skill Covers

| App Type | Description | Key Files |
|---|---|---|
| **Custom GPTs** | No-code GPTs built in ChatGPT UI with instructions + knowledge + actions | `resources/gpt_builder_guide.md` |
| **Custom Actions** | Backend APIs that GPTs call via OpenAPI spec | `resources/actions_api_reference.md`, `templates/openapi_action.yaml` |
| **Assistants API** | Programmatic assistants with tools, code interpreter, file search | `resources/assistants_api_reference.md`, `templates/assistant_config.json` |
| **ChatGPT Plugins** (Legacy) | OAuth-based plugins (deprecated but still referenced) | `resources/actions_api_reference.md` |

---

## Workflow

### Phase 1: Requirements & Architecture (Guide)

#### Step 1: Clarify the App Concept

Ask the user these questions to determine the right approach:

```
1. What problem does your ChatGPT App solve?
2. Who is the target user? (end-user in ChatGPT / developer via API / both)
3. Does it need external data or APIs? (weather, database, SaaS tools)
4. Does it need authentication? (API key / OAuth / none)
5. What's the deployment target? (GPT Store / internal team / API-only)
```

**Input**: User's answers to the 5 questions above
**Output**: A recommendation of which app type to build (GPT / Action / Assistant)
**If this fails**: Default to Custom GPT + Action as the most versatile combination

#### Step 2: Choose the Architecture

Use this decision tree:

```
Does the app need external data or API calls?
├── No → Custom GPT (instructions + knowledge files only)
│   └── Does it need structured output?
│       ├── Yes → Add JSON mode instructions + response format guidance
│       └── No → Pure instruction-based GPT
└── Yes → Custom GPT + Action OR Assistants API
    ├── Is the user building inside ChatGPT UI?
    │   └── Yes → Custom GPT + Custom Action (OpenAPI spec)
    └── Is the user building a standalone app?
        └── Yes → Assistants API (with function calling / code interpreter)
```

**Input**: App type decision from Step 1
**Output**: Architecture diagram and tech stack recommendation
**If this fails**: Provide both options with pros/cons and let user decide

---

### Phase 2: Design & Schema (Generator)

#### Step 3: Design the System Prompt / Instructions

Write the GPT's system instructions following this structure:

```markdown
# Role & Identity
You are [role]. You specialize in [domain].

# Core Behavior
- Always [behavior 1]
- Never [behavior 2]
- When uncertain, [fallback behavior]

# Response Format
- Use [format] for [situation]
- Include [element] in every response

# Constraints
- Do not [limitation 1]
- Maximum [limit] for [resource]

# Knowledge Boundaries
- You know about [topic 1], [topic 2]
- You do NOT know about [topic 3] — redirect to [alternative]
```

**Input**: App concept and target user from Phase 1
**Output**: Complete system prompt / GPT instructions
**If this fails**: Start with a minimal 3-line prompt and iterate

→ See `templates/system_prompt.md` for the full template

#### Step 4: Design the OpenAPI Schema (if Action required)

Create the OpenAPI 3.1.0 spec for the Custom Action:

1. Define each endpoint with clear `operationId` and `description`
2. Use `description` fields extensively — GPT reads these to decide when to call
3. Keep request/response schemas simple (GPT handles complex JSON poorly)
4. Add `servers` with the production URL

**Rules for GPT-friendly OpenAPI specs**:
- Every `operationId` must be a clear verb phrase: `getWeather`, `searchProducts`
- Every parameter needs a `description` explaining when to use it
- Response schemas should be flat, not deeply nested
- Use `enum` for constrained values
- Limit to 30 endpoints maximum (GPT performance degrades beyond this)

**Input**: List of required API capabilities
**Output**: Complete `openapi.yaml` file
**If this fails**: Validate with `swagger-cli validate openapi.yaml`

→ See `templates/openapi_action.yaml` for the starter template

#### Step 5: Design the Assistant Configuration (if Assistants API)

Define the assistant's tools and configuration:

```json
{
  "name": "My Assistant",
  "instructions": "...",
  "model": "gpt-4o",
  "tools": [
    { "type": "code_interpreter" },
    { "type": "file_search" },
    { "type": "function", "function": { "name": "...", "description": "...", "parameters": {} } }
  ],
  "metadata": {}
}
```

**Input**: App requirements and tool needs
**Output**: Complete assistant configuration JSON
**If this fails**: Start with `code_interpreter` only and add tools incrementally

→ See `templates/assistant_config.json` for the starter template

---

### Phase 3: Implementation (Generator + Operator)

#### Step 6: Implement the Action Backend

Choose the framework and implement:

| Framework | Best For | Template |
|---|---|---|
| **Next.js API Routes** | Vercel deployment, React frontend | `examples/nextjs_action/` |
| **Cloudflare Workers** | Edge, low latency, free tier | `examples/cloudflare_action/` |
| **FastAPI (Python)** | Data-heavy, ML integration | `examples/fastapi_action/` |
| **Express.js** | Flexibility, middleware ecosystem | Adapt from Next.js template |

Implementation checklist:
1. [ ] Create the API endpoint(s) matching the OpenAPI spec
2. [ ] Add CORS headers: `Access-Control-Allow-Origin: https://chat.openai.com`
3. [ ] Implement authentication (API key header or OAuth)
4. [ ] Add request validation
5. [ ] Add structured error responses (GPT needs clear error messages)
6. [ ] Serve the OpenAPI spec at `/.well-known/openapi.yaml`
7. [ ] Add a privacy policy page at `/privacy`

**Input**: OpenAPI spec from Step 4
**Output**: Working backend with all endpoints
**If this fails**: Test each endpoint individually with `curl` before connecting to GPT

#### Step 7: Implement the Assistants API Client (if applicable)

```typescript
import OpenAI from 'openai';

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// 1. Create Assistant (once)
const assistant = await client.beta.assistants.create({
  name: "My Assistant",
  instructions: "...",
  model: "gpt-4o",
  tools: [{ type: "code_interpreter" }],
});

// 2. Create Thread (per conversation)
const thread = await client.beta.threads.create();

// 3. Add Message
await client.beta.threads.messages.create(thread.id, {
  role: "user",
  content: "Hello!",
});

// 4. Run and Stream
const run = client.beta.threads.runs.stream(thread.id, {
  assistant_id: assistant.id,
});

for await (const event of run) {
  // Handle events: thread.message.delta, tool_calls, etc.
}
```

**Input**: Assistant configuration from Step 5
**Output**: Working client code with streaming support
**If this fails**: Check API key permissions and model access

---

### Phase 4: Testing & Validation (Reviewer)

#### Step 8: Test the Action / Assistant

Run through this test matrix:

| Test Case | Method | Expected Result |
|---|---|---|
| Happy path — basic query | Manual in GPT UI | Correct response, no errors |
| Edge case — empty input | `curl` with empty body | Clear error message, not 500 |
| Edge case — very long input | `curl` with 4000+ chars | Graceful truncation or error |
| Auth — missing API key | `curl` without auth header | 401 with clear message |
| Auth — invalid API key | `curl` with wrong key | 403 with clear message |
| Rate limiting | Rapid sequential calls | 429 with retry-after header |
| Schema validation | `swagger-cli validate` | No errors |
| CORS | Browser fetch from chat.openai.com | Correct headers returned |

**Input**: Deployed endpoint URL
**Output**: Test results for all 8 cases
**If this fails**: Fix the failing test case before proceeding

#### Step 9: Test the GPT Conversation Flow

1. Open the GPT in preview mode
2. Test these conversation patterns:
   - **First message**: Does the GPT introduce itself correctly?
   - **Action trigger**: Does the GPT call the action at the right time?
   - **Error recovery**: What happens when the action returns an error?
   - **Multi-turn**: Does context persist across messages?
   - **Edge case**: What happens with off-topic questions?
3. Iterate on the system prompt based on results

**Input**: Working GPT with connected action
**Output**: Conversation test report
**If this fails**: Adjust system prompt instructions for clearer behavior boundaries

---

### Phase 5: Deployment & Publishing (Operator)

#### Step 10: Deploy the Backend

```bash
# Vercel
vercel --prod

# Cloudflare Workers
wrangler deploy

# Railway
railway up
```

**Input**: Working local backend
**Output**: Production URL (HTTPS)
**If this fails**: Check deployment logs, verify environment variables are set

#### Step 11: Configure the GPT in ChatGPT UI

1. Go to https://chat.openai.com → Explore GPTs → Create
2. Set **Name**, **Description**, **Instructions** (from Step 3)
3. Upload **Knowledge files** (if any)
4. Add **Action**: paste the OpenAPI spec URL or YAML content
5. Configure **Authentication** (None / API Key / OAuth)
6. Set **Conversation starters** (4 example prompts)
7. Upload **Profile image**

**Input**: System prompt, OpenAPI spec URL, knowledge files
**Output**: Published GPT accessible via link
**If this fails**: Check the OpenAPI spec for validation errors in the GPT Builder UI

#### Step 12: Submit to GPT Store (Optional)

Requirements for GPT Store listing:
- [ ] GPT name is unique and descriptive
- [ ] Description clearly explains what the GPT does
- [ ] Profile image is original (not AI-generated faces)
- [ ] Privacy policy URL is accessible
- [ ] GPT follows OpenAI usage policies
- [ ] Tested with 10+ diverse prompts
- [ ] Builder profile is verified

**Input**: Published GPT
**Output**: GPT Store submission
**If this fails**: Review OpenAI's GPT Store guidelines and fix compliance issues

---

## Key Concepts

| Concept | Description |
|---|---|
| **Custom GPT** | A configured ChatGPT instance with custom instructions, knowledge, and actions. Built in the ChatGPT UI. |
| **Action** | An API endpoint that a GPT can call. Defined by an OpenAPI spec. Replaces the deprecated "Plugin" system. |
| **Assistants API** | Programmatic API for building AI assistants with tools (code interpreter, file search, function calling). |
| **Function Calling** | The ability for a model to output structured JSON matching a defined schema, used to call external functions. |
| **Code Interpreter** | A sandboxed Python environment that the assistant can use to run code, analyze data, and generate files. |
| **File Search** | Vector-store-based retrieval over uploaded files (RAG). Supports up to 10,000 files per assistant. |
| **Thread** | A conversation session in the Assistants API. Messages are appended to threads. |
| **Run** | An execution of an assistant on a thread. Can be streamed for real-time responses. |
| **OpenAPI 3.1.0** | The schema format used to define Custom Actions. GPT reads `description` fields to understand when/how to call endpoints. |
| **Conversation Starters** | Pre-defined prompts shown to users when they first open a GPT. Critical for discoverability and UX. |

---

## Error Handling

| Error | Cause | Fix |
|---|---|---|
| `Action failed` in GPT UI | Backend returned non-2xx or invalid JSON | Check backend logs, ensure JSON response with correct Content-Type |
| `Could not parse OpenAPI spec` | Invalid YAML/JSON or unsupported OpenAPI features | Validate with `swagger-cli validate`, use 3.1.0 format |
| `Authentication failed` | Mismatched API key or OAuth config | Verify key in GPT settings matches backend expectation |
| `429 Too Many Requests` | Rate limit hit on OpenAI API | Implement exponential backoff, check tier limits |
| `Assistant run failed` | Tool call returned invalid output | Ensure function return matches declared schema |
| `File search returned no results` | Files not indexed or wrong format | Wait for indexing, use supported formats (PDF, TXT, MD, DOCX) |
| GPT ignores action | System prompt doesn't clearly instruct when to call | Add explicit "ALWAYS call [action] when user asks about [topic]" |
| GPT hallucinates instead of calling action | Instructions too vague | Add "NEVER answer [topic] from your own knowledge. ALWAYS use the [action]" |
| CORS error in browser | Missing or wrong CORS headers | Add `Access-Control-Allow-Origin: https://chat.openai.com` |
| `context_length_exceeded` | Too much data in thread/knowledge | Summarize older messages, chunk large files |

---

## Examples

### Example 1: Build a Product Search GPT with Action

**User says**: "I want to build a GPT that searches my e-commerce product catalog"

**Agent does**:
1. Recommends: Custom GPT + Custom Action (user wants ChatGPT UI integration with external data)
2. Designs system prompt with product search persona and response format
3. Creates OpenAPI spec with `searchProducts` and `getProductDetails` endpoints
4. Scaffolds a Next.js API route backend with product search logic
5. Adds CORS headers and API key authentication
6. Deploys to Vercel
7. Walks user through GPT Builder configuration
8. Tests with sample queries: "Find red shoes under $50", "Show me product #12345"

### Example 2: Build a Data Analysis Assistant with Assistants API

**User says**: "I need an assistant that analyzes CSV files and generates charts"

**Agent does**:
1. Recommends: Assistants API with Code Interpreter (programmatic, needs code execution)
2. Creates assistant configuration with `code_interpreter` and `file_search` tools
3. Writes system prompt focused on data analysis best practices
4. Implements TypeScript client with file upload and streaming
5. Adds error handling for malformed CSVs and large files
6. Tests with sample CSV: upload → analysis → chart generation
7. Wraps in a simple web UI with file upload and chat interface

### Example 3: Build a Customer Support GPT with OAuth Action

**User says**: "I want a GPT that can look up customer orders from our Shopify store"

**Agent does**:
1. Recommends: Custom GPT + OAuth Action (needs authenticated access to Shopify API)
2. Designs system prompt with customer support persona and escalation rules
3. Creates OpenAPI spec with `lookupOrder`, `getCustomerHistory`, `checkShippingStatus`
4. Implements OAuth 2.0 flow for Shopify API access
5. Scaffolds backend with token management and Shopify API proxy
6. Deploys with secure environment variable management
7. Configures OAuth in GPT Builder (client ID, secret, token URL)
8. Tests full flow: user authorizes → GPT fetches real order data

---

## Best Practices

### System Prompt Engineering

1. **Be specific, not generic** — "You are a tax advisor for US freelancers" > "You are helpful"
2. **Define boundaries explicitly** — List what the GPT should NOT do
3. **Use structured output instructions** — Tell the GPT to respond in tables, bullet points, or JSON
4. **Include error handling in the prompt** — "If the action fails, tell the user to try again in 30 seconds"
5. **Version your prompts** — Keep a changelog of prompt iterations

### OpenAPI Schema Design

1. **Descriptions are everything** — GPT decides when to call based on `description` fields
2. **Keep it flat** — Avoid deeply nested objects (GPT struggles with >3 levels)
3. **Use meaningful operationIds** — `searchProducts` not `endpoint1`
4. **Limit endpoints** — 5-15 is ideal, 30 is the hard max
5. **Add examples in descriptions** — "City name (e.g., 'Tokyo', 'New York')"

### Assistants API Patterns

1. **One assistant per domain** — Don't overload a single assistant with unrelated tools
2. **Stream responses** — Always use streaming for better UX
3. **Handle tool calls gracefully** — Validate inputs, return clear errors
4. **Manage thread length** — Summarize or truncate old messages to stay within context
5. **Use metadata** — Tag threads and runs with user IDs for tracking

---

## References

- [OpenAI GPT Builder Documentation](https://platform.openai.com/docs/gpts)
- [OpenAI Actions Documentation](https://platform.openai.com/docs/actions)
- [OpenAI Assistants API Documentation](https://platform.openai.com/docs/assistants)
- [OpenAPI 3.1.0 Specification](https://spec.openapis.org/oas/v3.1.0)
- [OpenAI Function Calling Guide](https://platform.openai.com/docs/guides/function-calling)
- [OpenAI GPT Store Guidelines](https://openai.com/policies/gpt-store)
- [OpenAI Usage Policies](https://openai.com/policies/usage-policies)
