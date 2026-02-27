---
name: openai-docs
description: Use when the user asks how to build with OpenAI products or APIs and needs up-to-date official documentation with citations (for example: Codex, Responses API, Chat Completions, Apps SDK, Agents SDK, Realtime, model capabilities or limits); prioritize OpenAI docs MCP tools and restrict any fallback browsing to official OpenAI domains.
---

# OpenAI Docs

Use official OpenAI developer documentation as the source of truth for API behavior, model limits, pricing references, and SDK usage. Prefer the MCP docs tools; use web search only as a fallback and only on OpenAI domains.

## Goals
- Answer with accurate, up-to-date guidance.
- Cite the exact doc pages used.
- Call out version constraints and deprecations.

## Workflow
1. Identify the product area (OpenAI API, Codex, ChatGPT Apps SDK, Agents SDK, Realtime, etc.).
2. Clarify the specific API surface or task (endpoint, SDK, tool, model, limit).
3. Search the OpenAI docs MCP server.
4. Fetch the exact page/section needed.
5. Answer concisely with citations and code snippets only when doc-backed.

## MCP Tooling (Preferred)
- Use `mcp__openaiDeveloperDocs__search_openai_docs` for discovery.
- Use `mcp__openaiDeveloperDocs__fetch_openai_doc` for precise sections.
- Use `mcp__openaiDeveloperDocs__list_openai_docs` only when discovery is too broad.

## Fallback (Only If MCP Fails)
- Use `web.run` with domain restrictions:
  - `developers.openai.com`
  - `platform.openai.com`
- Do not use third-party sources for OpenAI docs questions.

## Quality Rules
- Keep quotes short; prefer paraphrase with citations.
- If multiple docs disagree, note the discrepancy and cite both.
- If docs are silent, say so and propose next steps.
- Never invent API parameters or model limits.

## Version and Deprecation Checks
- Confirm model or SDK versions mentioned in the docs.
- Call out deprecations or replacements explicitly.
- If the user's stack version is unknown, say you used latest docs.

## Typical Queries
- "How do I call the Responses API with tools?"
- "What are the current limits for model X?"
- "How do I set up a ChatGPT app with an MCP server?"
- "How do I stream outputs from the API?"

## Search Query Tips
- Include the exact endpoint or SDK method name.
- Add the feature keyword (streaming, tools, files, images, audio).
- If the user mentions an error, include the error string verbatim.

## Common Pitfalls
- Mixing legacy and current endpoints in the same example.
- Using model names that are deprecated or renamed.
- Omitting required auth scopes or project IDs.
- Copying SDK snippets without noting the language/version.

## When Asked About Pricing or Limits
- Treat the docs as time-sensitive and always cite the source.
- If pricing tables are not in the docs tool output, say so and point to the official page.

## Citation Rules
- Cite every non-trivial factual claim from the docs.
- Place citations near the sentences they support.
- Avoid large block quotes; keep excerpts short.

## If the User Wants a Link
- Provide the official OpenAI doc page URL.
- Do not share unofficial mirrors.

## Answer Structure (Default)
1. Direct answer (short).
2. Minimal code or config example.
3. Caveats/version notes.
4. Links/citations to the specific sections used.

## If the MCP Server Is Missing
1. Attempt install: `codex mcp add openaiDeveloperDocs --url https://developers.openai.com/mcp`.
2. If that fails, ask the user to install it and restart Codex.
3. Retry MCP search after restart.

## Product Snapshot Checklist
When relevant, confirm:
- API family (Responses vs Chat Completions vs legacy endpoints).
- Supported modalities (text, image, audio).
- Tooling support (function calling, tools, streaming).
- Auth requirements (API keys, project scoping).

## Output Expectations
- Precise, doc-backed instructions.
- Citations for every non-trivial factual claim.
- Clear assumptions when user context is missing.
- Note when guidance is based on latest docs only.

## Definition of Done
- Answer references the correct OpenAI docs pages.
- The user can implement the guidance without guesswork.
