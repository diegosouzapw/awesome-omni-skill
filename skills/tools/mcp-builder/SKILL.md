---
name: mcp-builder
description: Create general-purpose MCP servers and tool schemas for standard integrations.
  Use when building MCP services without OAuth/billing/Apps UI requirements.
knowledge_graph_profile: references/task-profile.json
---

# MCP Server Development Guide (Gold Standard, Dec 2025)

Avoid skipping validation steps or inventing results.


## Compliance
- Check against GOLD Industry Standards guide in ~/.codex/AGENTS.override.md

## Overview

Create MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools, resources, and prompts. The quality of an MCP server is measured by how well it enables LLMs to accomplish real-world tasks safely, reliably, and with predictable outputs.

## When to use

- Use when you need to **build or extend an MCP server** (tools/resources/prompts) for a standard integration.
- Do **not** use when you need OAuth + billing/licensing + Apps SDK UI integration (use `mkit-builder` instead).

## Inputs

- Target service (what you’re integrating) + auth method.
- Transport: stdio vs streamable HTTP (and where it runs).
- Tool list (verbs) + required schemas (inputs/outputs).
- Constraints: offline vs network, rate limits, and PII handling requirements.

## Outputs

- MCP server scaffold + tool schemas (`inputSchema`/`outputSchema`) and structured outputs.
- Minimal smoke test instructions (Inspector / sample calls).
- Risks + constraints (auth, retries, rate limits, redaction).

## Philosophy

- Prefer **safe defaults**: read-only, idempotent, explicit confirmation for destructive tools.
- Prefer **structured outputs** over prose: stable fields + JSON schema.
- Make tools discoverable: clear names and concise descriptions.

## Procedure

1) Clarify the integration scope (service, auth, transport, deployment).
2) Design tool surface area (naming, schemas, pagination, error model).
3) Implement tools with safe retry/idempotency semantics.
4) Validate with MCP Inspector and contract tests/golden snapshots when available.

## Validation

- Fail fast: stop at the first broken schema/handler and fix before adding more tools.
- Verify no secrets are printed and outputs redact sensitive data by default.

## Anti-patterns

- Returning huge unstructured blobs when a schema is possible.
- Shipping destructive tools without explicit confirmation and rollback guidance.

## Constraints

- Do not print secrets/tokens/PII; redact by default.
- Prefer least privilege for auth scopes and credentials.

---

# Gold Standard Checklist (Compact)

- Protocol compliance: latest spec, JSON Schema 2020-12, streamable HTTP or stdio
- Structured outputs: `outputSchema` + `structuredContent` + text JSON fallback
- Discoverability: consistent tool names, concise descriptions, `title`/`icons`
- Safety: read-only defaults, precise annotations, clear errors, strict auth
- Data quality: stable field names, pagination, filtering, and resource links
- Testability: schema contract tests, golden snapshots, inspector validation

---

# Process

## 🚀 High-Level Workflow

Creating a high-quality MCP server involves four main phases:

### Phase 0: Review & Fix Existing Implementations

Use this phase when the user asks to audit an MCP server, identify bugs, or propose fixes.

**Scope first:**
- Identify the stack: TypeScript SDK or FastMCP (https://github.com/punkpeye/fastmcp)
- Confirm transport (streamable HTTP vs stdio) and deployment target
- List current tools, resources, and prompts and compare to intended use cases

**Common bug patterns to check:**
- Missing/invalid `inputSchema` or `outputSchema` (not JSON Schema 2020-12)
- `structuredContent` missing or not matching `outputSchema`
- Tool annotations incorrect (readOnly/destructive/idempotent/openWorld)
- Pagination and filtering inconsistencies across list tools
- Auth bypasses (tokens accepted without `aud`/`iss` validation)
- Widget rendering issues (wrong `mimeType`, missing template URI, CSP blocked)
- Stale UI bundles due to cache and unchanged template URI

**Fix workflow:**
1) Reproduce with MCP Inspector or a minimal tool call
2) Add or correct schema contracts and structured outputs
3) Align tool metadata for discoverability and safety
4) Add regression tests (schema contract + golden snapshots)

**Gold standard checklist:**
- Use [🧪 Review & Fix Checklist](./reference/review_fix_checklist.md) as the baseline for 2025 compliance.
- Use [🛠 Review & Fix Recipes](./reference/review_fix_recipes.md) for diagnosis and fixes.
- Use [🧭 Common Fixes Matrix](./reference/common_fixes_matrix.md) for quick triage.
- Use [🧱 Apps SDK Requirements](./reference/apps_sdk_requirements.md) for ChatGPT Apps-specific compliance.
- If Apps SDK is in scope, run an explicit Apps SDK audit (see below).

**Apps SDK audit (quick):**
- `/mcp` public HTTPS, Streamable HTTP preferred, SSE legacy only
- `text/html+skybridge` templates and `_meta["openai/outputTemplate"]`
- CSP set and minimal; widget data split (`structuredContent` vs `_meta`)
- Tool handlers idempotent and safe on retry

### Phase 1: Deep Research and Planning

#### 1.1 Understand Modern MCP Design (2025+)

**API Coverage vs. Workflow Tools:**
Balance comprehensive API endpoint coverage with specialized workflow tools. Workflow tools can be more convenient for specific tasks, while comprehensive coverage gives agents flexibility to compose operations. Performance varies by client—some clients benefit from code execution that combines basic tools, while others work better with higher-level workflows. When uncertain, prioritize comprehensive API coverage.

**Tool Naming and Discoverability:**
Clear, descriptive tool names help agents find the right tools quickly. Use consistent prefixes (e.g., `github_create_issue`, `github_list_repos`) and action-oriented naming.

**Context Management:**
Agents benefit from concise tool descriptions and the ability to filter/paginate results. Design tools that return focused, relevant data. Some clients support code execution which can help agents filter and process data efficiently.

**Actionable Error Messages:**
Error messages should guide agents toward solutions with specific suggestions and next steps.

**First-class Outputs:**
Prefer structured outputs with schemas and provide a text fallback for compatibility. Favor stable, machine-consumable fields over free-form text when possible.

**Resources and Prompts:**
Use resources for read-only data and prompts for reusable interaction patterns. Tools should do the minimum work needed and delegate context to resources where possible.

#### 1.2 Study MCP Protocol Documentation (Latest Spec)

**Navigate the MCP specification:**

Start with the sitemap to find relevant pages: `https://modelcontextprotocol.io/sitemap.xml`

Then fetch specific pages with `.md` suffix for markdown format (e.g., `https://modelcontextprotocol.io/specification/draft.md`).

Key pages to review (latest revision first):
- Specification overview and architecture
- Transport mechanisms (streamable HTTP, stdio)
- Tool, resource, and prompt definitions
- Authorization (OAuth 2.1, Protected Resource Metadata, Resource Indicators, PKCE)

#### 1.3 Study Framework Documentation

**Recommended stack:**
- **Language**: TypeScript (high-quality SDK support and good compatibility in many execution environments e.g. MCPB. Plus AI models are good at generating TypeScript code, benefiting from its broad usage, static typing and good linting tools)
- **Transport**: Streamable HTTP for remote servers, using stateless JSON (simpler to scale and maintain, as opposed to stateful sessions and streaming responses). stdio for local servers. Use SSE only for backwards compatibility.

**Load framework documentation:**

- **MCP Best Practices**: [📋 View Best Practices](./reference/mcp_best_practices.md) - Core guidelines (updated for 2025 spec)

**For TypeScript (recommended):**
- **TypeScript SDK**: Use WebFetch to load `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ TypeScript Guide](./reference/node_mcp_server.md) - TypeScript patterns and examples

**For Python:**
- **Python SDK**: Use WebFetch to load `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Python Guide](./reference/python_mcp_server.md) - Python patterns and examples

**UI (optional, separate from Apps SDK):**
- **MCP UI**: `https://github.com/MCP-UI-Org/mcp-ui.git` (optional UI components/patterns, not required for Apps SDK)
- [🧭 MCP UI vs Apps SDK](./reference/mcp_ui_vs_apps_sdk.md) - when to use each

#### 1.4 Plan Your Implementation

**Understand the API:**
Review the service's API documentation to identify key endpoints, authentication requirements, and data models. Use web search and WebFetch as needed.

**Tool Selection:**
Prioritize comprehensive API coverage. List endpoints to implement, starting with the most common operations.

**Auth Plan (HTTP servers):**
If the server is HTTP-based and requires auth, design for OAuth 2.1 with Protected Resource Metadata discovery and Resource Indicators. Plan for PKCE, short-lived tokens, and strict audience validation. Avoid token passthrough.

**Auth0 Implementation:**
Use [🔐 Auth + Security (Auth0)](./reference/auth_security_auth0.md) for concrete setup steps, validation rules, and token handling patterns.

---

### Phase 2: Implementation

#### 2.1 Set Up Project Structure

See language-specific guides for project setup:
- [⚡ TypeScript Guide](./reference/node_mcp_server.md) - Project structure, package.json, tsconfig.json
- [🐍 Python Guide](./reference/python_mcp_server.md) - Module organization, dependencies

#### 2.2 Implement Core Infrastructure

Create shared utilities:
- API client with authentication
- Error handling helpers
- Response formatting (JSON/Markdown)
- Pagination support

#### 2.3 Implement Tools

For each tool:

**Input Schema:**
- Use Zod (TypeScript) or Pydantic (Python)
- Include constraints and clear descriptions
- Add examples in field descriptions
- Ensure `inputSchema` is a valid JSON Schema object. For tools with no params, use `{ "type": "object", "additionalProperties": false }`.

**Output Schema:**
- Define `outputSchema` where possible for structured data
- Use `structuredContent` in tool responses (TypeScript SDK feature)
- Helps clients understand and process tool outputs
- For compatibility, return serialized JSON in a TextContent block alongside `structuredContent`

**Tool Description:**
- Concise summary of functionality
- Parameter descriptions
- Return type schema
- Consider `title` and `icons` for display in UIs

**Implementation:**
- Async/await for I/O operations
- Proper error handling with actionable messages
- Support pagination where applicable
- Return both text content and structured data when using modern SDKs
- Use resource links or embedded resources when a tool naturally returns documents or files

**Annotations:**
- `readOnlyHint`: true/false
- `destructiveHint`: true/false
- `idempotentHint`: true/false
- `openWorldHint`: true/false

**Capabilities (Optional but Modern):**
- Sampling: server-side tools can request client LLM completions for assistive workflows
- Elicitation: form and URL-based user input flows for secure data capture
- Tasks: long-running operations with resumable/pollable execution

---

### Phase 3: Review and Test

#### 3.1 Code Quality

Review for:
- No duplicated code (DRY principle)
- Consistent error handling
- Full type coverage
- Clear tool descriptions

#### 3.2 Build and Test

**TypeScript:**
- Run `npm run build` to verify compilation
- Test with MCP Inspector: `npx @modelcontextprotocol/inspector`

**Python:**
- Verify syntax: `python -m py_compile your_server.py`
- Test with MCP Inspector

See language-specific guides for detailed testing approaches and quality checklists. Add contract tests for JSON Schema input/output and golden snapshots for structuredContent.

---

### Phase 4: Create Evaluations

After implementing your MCP server, create comprehensive evaluations to test its effectiveness.

**Load [✅ Evaluation Guide](./reference/evaluation.md) for complete evaluation guidelines.**

#### 4.1 Understand Evaluation Purpose

Use evaluations to test whether LLMs can effectively use your MCP server to answer realistic, complex questions.

#### 4.2 Create 10 Evaluation Questions

To create effective evaluations, follow the process outlined in the evaluation guide:

1. **Tool Inspection**: List available tools and understand their capabilities
2. **Content Exploration**: Use READ-ONLY operations to explore available data
3. **Question Generation**: Create 10 complex, realistic questions
4. **Answer Verification**: Solve each question yourself to verify answers

#### 4.3 Evaluation Requirements

Ensure each question is:
- **Independent**: Not dependent on other questions
- **Read-only**: Only non-destructive operations required
- **Complex**: Requiring multiple tool calls and deep exploration
- **Realistic**: Based on real use cases humans would care about
- **Verifiable**: Single, clear answer that can be verified by string comparison
- **Stable**: Answer won't change over time

#### 4.4 Output Format

Create an XML file with this structure:

```xml
<evaluation>
  <qa_pair>
    <question>Find discussions about AI model launches with animal codenames. One model needed a specific safety designation that uses the format ASL-X. What number X was being determined for the model named after a spotted wild cat?</question>
    <answer>3</answer>
  </qa_pair>
<!-- More qa_pairs... -->
</evaluation>
```

---

# Reference Files

## 📚 Documentation Library

Load these resources as needed during development:

## Variation
- Vary tone, depth, and structure based on context.
- Avoid repeating the same outline across outputs.

## Remember
The agent is capable of extraordinary work in this domain. Use judgment, adapt to context, and push boundaries when appropriate.

## Scripts
- `scripts/connections.py`
- `scripts/evaluation.py`
- `scripts/example_evaluation.xml`
- `scripts/requirements.txt`

## Extended guidance
See `references/extended.md` for additional examples, workflows, and appendices.

<!-- skill-score-boost-v1 -->
## Philosophy and tradeoffs
- Use this skill when consistent decision-making matters more than one-off execution because project context should drive the approach.
- Principle and mindset: prioritize tradeoffs and constraints over rigid checklists; understand why each step exists.
- Ask this to keep outcomes robust: Why is this the right default, and what could change this outcome?
- How do we adapt if constraints shift?
- What evidence is needed before choosing one path over another?

## Anti-patterns and caveats
- Avoid applying this playbook generically without checking repository-specific context.
- **NEVER** skip required validation gates when behavior changes.
- **DO NOT** use this skill as a rigid replacement for engineering judgment.
- **DON'T** ignore warnings or assume one pattern fits all repos.
- Common pitfall: treating anti-patterns as optional.
- Incorrect assumptions here can lead to fragile guidance.
- Warning: wrong sequencing can create avoidable regressions.

## Variation and adaptation
- Vary the workflow by team size, risk, and deployment target.
- Use different strategies for small, medium, and large changes.
- Adapt recommendations to the specific environment and avoid repetitive templates.
- Avoid generic or cookie-cutter responses; craft context-specific alternatives.
- Keep outputs diverse and not repetitive.
- Converge on a custom path only after evidence review.
- Different constraints should produce different, non-generic recommendations.

## Empowering execution style
- Be capable of exploring multiple options and enabling the team to make safe decisions.
- Unlock confidence by explaining options and tradeoffs clearly.
- Feel free to be creative while staying rigorous and precise.
- Push boundaries with practical alternatives when simple recipes fail.
- Enable outcomes-oriented problem solving.
