---
name: squall-deep-research
description: "Deep research via Codex web search and optionally Gemini deep research. Use when asked to 'deep research', 'squall deep research', 'research deeply', or when a question needs web-sourced evidence. Single-agent, not a swarm. (project)"
one_liner: "Codex searches the web, Gemini goes deep — real sources for real questions."
activation_triggers:
  - "squall deep research"
  - "deep research"
  - "research deeply"
  - "web research"
  - When user needs web-sourced evidence for a question
  - When user wants deep investigation with real sources
related_skills:
  - "[squall-research](../squall-research/SKILL.md) (for multi-agent swarm research)"
  - "[squall-review](../squall-review/SKILL.md) (for code review, not research)"
---

# Squall Deep Research

> **TL;DR - QUICK REFERENCE**
>
> | Concept | Translation |
> |---------|-------------|
> | Codex web search | `clink` with model `codex`, timeout 600s — proven path, 20+ web searches per query |
> | Gemini deep research | `clink` with model `gemini` — standard search works, deep research needs `GEMINI_API_KEY` |
> | Prompt engineering | Frame as research question with "search the web for...", "cite sources", "include URLs" |
> | Output persistence | Write results to `.squall/research/<topic>.md` so they survive context compaction |
>
> **Entry criteria:**
> - User needs web-sourced evidence, specifications, or current information
> - Question goes beyond what's in the codebase or Claude's training data
> - User says "deep research", "web research", or "research deeply"

Single-agent deep research using Squall's CLI model dispatch. Codex performs multi-step
web searches and synthesizes sourced reports. Gemini adds depth when available. This is
NOT a multi-agent swarm — for parallel team research, use `/squall-research` instead.

## Workflow

1. **Check memory** — call `memory` category "patterns" to see if prior research exists on this topic. Call `memory` category "tactics" for prompt engineering insights that improve Codex/Gemini research quality.
2. **Frame the research question** — Convert the user's request into a specific, searchable research prompt
3. **Dispatch via Squall `clink`** — Send to Codex (primary) and optionally Gemini (secondary)
4. **Persist results** — Write to `.squall/research/<topic>.md`
5. **Synthesize and memorize** — Summarize key findings for the user with source attribution. Call `memorize` with category "pattern" for insights worth remembering across sessions.

```
┌─────────────────────────────────────────────────────────────┐
│  DEEP RESEARCH WORKFLOW                                     │
│                                                             │
│  Check  ──► Frame  ──► Craft  ──► clink(codex) ──► Persist │
│  memory     question    prompt        │                │    │
│                           └──► clink(gemini) ──────────┘    │
│                                (optional)       ▼           │
│                                          .squall/research/  │
│                                          <topic>.md         │
│                                                ▼            │
│                                         Memorize findings   │
└─────────────────────────────────────────────────────────────┘
```

## Key Principles

### Principle 1: Codex Web Search is the Primary Path

Codex CLI performs real web searches — not just LLM recall. A single query triggers 20+
searches, opens pages, extracts data, and synthesizes. Proven at ~93s for detailed topics
producing 82K input tokens of sourced material. This is the reliable, working-today path.

### Principle 2: Prompt Engineering Drives Quality

Research quality depends entirely on how you frame the prompt. Codex responds to specific
triggers that activate its search behavior. Vague prompts get vague answers. Specific,
data-oriented prompts with explicit search instructions get sourced reports.

### Principle 3: Persist Everything

MCP tool results are ephemeral — they vanish during context compaction. Always write
research output to `.squall/research/`. The file path survives in compaction summaries,
so Claude can `Read` the file to recover full details later.

## Dispatching Research

### Primary: Codex Web Search

Use Squall's `clink` tool:

```
Tool: clink
Arguments:
  model: "codex"
  prompt: "<research prompt — see Prompt Engineering below>"
  timeout: 600
```

Codex needs time for multi-step web searches. The 600s timeout prevents premature cutoff.
Typical research queries complete in 60-120s.

### Optional: Gemini

```
Tool: clink
Arguments:
  model: "gemini"
  prompt: "<research prompt>"
  timeout: 600
```

Gemini CLI has standard search capabilities via OAuth. For deeper investigation, the
Gemini deep research extension requires `GEMINI_API_KEY` (paid tier) — see Limitations.

### Combining Both

Run Codex and Gemini as parallel `clink` calls with different angles:

- **Codex**: Breadth — cast a wide net with web search, gather many sources
- **Gemini**: Depth — analyze, synthesize, find patterns across findings

Example: researching "Rust async runtimes"
- Codex prompt: "Search the web for comprehensive comparisons of Rust async runtimes in 2025-2026. Include benchmarks, adoption numbers, and ecosystem compatibility. Cite all sources with URLs."
- Gemini prompt: "Analyze the tradeoffs between tokio, async-std, and smol for building MCP servers. Focus on cancellation safety, memory overhead, and real-world production experience."

## Prompt Engineering for Research

### What triggers Codex's search behavior

| Prompt pattern | Effect |
|----------------|--------|
| "Search the web for..." | Activates web search mode |
| "Cite sources" / "Include URLs" | Produces attributed results |
| "Find recent..." / "What's the latest..." | Targets current information |
| Specific data points (numbers, specs, versions) | Gets precise answers |
| "Compare X and Y with evidence" | Multi-source synthesis |

### Template: Research Prompt

```
Search the web for [specific topic]. I need:

1. [Specific data point or question]
2. [Another specific question]
3. [Comparison or analysis needed]

Requirements:
- Cite all sources with URLs
- Include specific numbers, dates, and version information
- Distinguish between confirmed facts and claims
- Note any contradictions between sources
```

### Template: Persist Results

After receiving `clink` output, write to disk:

```
File: .squall/research/<topic-slug>.md

# <Topic>

> Deep research via Squall/Codex — <date>

## Key Findings
<synthesized summary>

## Sources
<URLs and attributions from Codex output>

## Raw Output
<full Codex response>
```

Then memorize reusable insights:

```
Tool: memorize
Arguments:
  category: "pattern"
  content: "<key finding worth remembering across sessions>"
  tags: ["research", "<topic-tag>"]
```

## Reference Tables

| Scenario | Model(s) | Timeout | Notes |
|----------|----------|---------|-------|
| Quick fact check | Codex | 300s | Single specific question |
| Detailed research | Codex | 600s | Multi-faceted topic, many sources |
| Dual-perspective | Codex + Gemini | 600s each | Parallel calls, different angles |
| API/library research | Codex | 600s | "Search the web for X documentation" |

| Model | Strength | Typical Time | Token Usage |
|-------|----------|-------------|-------------|
| Codex | Web search breadth, 20+ searches, sourced reports | 60-120s | ~82K input |
| Gemini | Analysis depth, synthesis, standard search | 30-90s | Varies |

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| Set timeout below 300s for Codex | Use 600s — Codex needs time for web searches |
| Use vague prompts ("tell me about X") | Be specific: "Search the web for X. Include URLs, dates, version numbers." |
| Use this for code review | Use `/squall-review` — different tool, different purpose |
| Expect Gemini deep research without API key | Use Codex as primary; Gemini is supplementary |
| Leave results only in context | Always persist to `.squall/research/<topic>.md` |
| Use this for parallel multi-vector research | Use `/squall-research` for swarm-based investigation |
| Frame as a code generation task | Frame as a research question — "search", "find", "compare" |
| Start research without checking memory | Call `memory` first — prior research or patterns may already exist |
| Finish without memorizing findings | Call `memorize` for insights worth preserving across sessions |

## Examples

### Example 1: Single-Topic Research

User asks: "What are the current rate limits for the xAI Grok API?"

```
Tool: clink
Arguments:
  model: "codex"
  prompt: "Search the web for xAI Grok API rate limits as of 2026. I need:
    1. Rate limits per tier (free, paid, enterprise)
    2. Token limits per request
    3. Any recent changes to rate limits
    Cite all sources with URLs."
  timeout: 600
```

Then persist the output to `.squall/research/xai-grok-rate-limits.md`.

### Example 2: Dual-Model Comparative Research

User asks: "Research how other MCP servers handle streaming responses"

Run in parallel:

```
Tool: clink (call 1)
Arguments:
  model: "codex"
  prompt: "Search the web for MCP (Model Context Protocol) servers that implement
    streaming responses. Find GitHub repos, blog posts, and documentation.
    Include implementation approaches and any limitations. Cite all sources."
  timeout: 600

Tool: clink (call 2)
Arguments:
  model: "gemini"
  prompt: "Analyze the MCP specification's support for streaming responses.
    What are the protocol-level mechanisms? How do existing implementations
    handle partial results? What are the tradeoffs?"
  timeout: 600
```

Combine outputs into `.squall/research/mcp-streaming-responses.md`.

## Limitations

- **Gemini deep research** requires `GEMINI_API_KEY` (paid tier) — not currently configured
- **Codex deep research model** (`o4-mini-deep-research`) requires `OPENAI_API_KEY` — not available
- **Codex normal web search** is the reliable path today — uses existing ChatGPT auth
- **Results quality** depends heavily on prompt engineering — be specific, ask for sources
- **No structured output** — research results are free-form text, not JSON

## Related Skills

- [squall-research](../squall-research/SKILL.md) - Multi-agent swarm research with parallel team members investigating different vectors
- [squall-review](../squall-review/SKILL.md) - Code review via Squall's review tool, not for research

<!-- SENTINEL:SESSION_LEARNINGS_START - Do not remove this line -->
## Session Learnings Archive

Insights captured during skill execution:

(Add learnings here as they occur, using this format:)

### YYYY-MM-DD: Brief Title
**Origin**: What triggered this learning
**Core insight**: The key learning
**Harvested** → [Link to where it was integrated] OR "Pending harvest"

---
<!-- SENTINEL:SESSION_LEARNINGS_END -->
