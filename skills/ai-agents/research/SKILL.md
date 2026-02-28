---
name: research
description: "Researches any topic by dispatching 6-10 parallel sub-agents across community discussions and official sources. Use when user wants to research something, asks 'what's new with X', needs recommendations, wants community opinions, or mentions analysing trends, news, or sentiment about any topic."
license: MIT
argument-hint: "[topic]"
---

<instructions>

# research: Deep Research Any Topic

Research ANY topic across Reddit, X, community forums, official docs, academic papers, and industry publications. Dispatches 6-10 parallel sub-agents to cover every angle — community AND official sources — then synthesizes a two-sided report.

Use cases:
- **Prompting**: "photorealistic people in Nano Banana Pro" — learn techniques from community tips AND official guides
- **Recommendations**: "best Claude Code skills" — get specific names from community + official feature comparisons
- **News**: "what's happening with OpenAI" — community reactions + official announcements
- **General**: any topic — understand what the community says AND what officials report

<example>
Input: `/research best Claude Code skills`
Output: Two-sided report with:
- Community: "Most mentioned: /commit (5x, r/ClaudeAI + HN), remotion skill (4x, X + Dev.to)..."
- Official: "Anthropic docs highlight /pr, /review as recommended workflows..."
- Cross-reference: 3 aligned, 1 divergent, 1 gap
- Stats: 12 Reddit threads (847 upvotes) + 8 X posts + 6 community + 4 official sources
Then waits for user's vision to write a tailored prompt.
</example>

<example>
Edge case — Web-only mode (no API keys):
SCRIPT_DATA returns "Mode: web-only". Sub-agents do ALL discovery work.
Display omits Reddit/X stats lines. Report still delivers full two-sided analysis.
</example>

<example>
Edge case — Graceful degradation:
C4 returns empty or errors. Log "C4: dev communities — no results".
Continue with remaining agents. Note gap in stats footer: "1 agent failed".
Meanwhile, C2 says "Tool A recommended (12 upvotes on HN)" but O1 says
"Tool B is the documented approach". Cross-reference marks this as
"Divergent" with both views attributed.
</example>

## Parse User Intent

**User input:** $ARGUMENTS

Extract four variables from user input before proceeding:

| Variable | Extract | Example |
|----------|---------|---------|
| `TOPIC` | What they want to learn about | "web app mockups", "Claude Code skills" |
| `TARGET_TOOL` | Where they'll use prompts (or "unknown") | "Nano Banana Pro", "Midjourney" |
| `QUERY_TYPE` | PROMPTING \| RECOMMENDATIONS \| NEWS \| GENERAL | Auto-detect from phrasing |
| `DEPTH` | `--quick` (6 agents) \| default (8) \| `--deep` (10) | Flag in user input |

See `references/intent_parsing.md` for query type definitions, detection patterns, and variable storage rules.

Do not ask about target tool before research. If unspecified, ask after showing results.

<context>
State: TOPIC, TARGET_TOOL, QUERY_TYPE, DEPTH are now defined.
Next: Determine API key availability to select script mode.
</context>

## Setup Check

The Python script works in three modes based on available API keys:

1. **Full Mode** (both keys): Reddit + X with real engagement metrics
2. **Partial Mode** (one key): Reddit-only or X-only
3. **Web-Only Mode** (no keys): Script provides no data, sub-agents do all the work

API keys are optional. The skill always dispatches sub-agents regardless. Determine mode quickly.

## MCP Tool Detection

Discover available search MCP tools before dispatching agents. Run `ToolSearch` for EACH query below and record every tool found as `AVAILABLE_MCP_TOOLS`:

| Query | Known tool names |
|-------|-----------------|
| `"searxng"` | `searxng_web_search`, `web_url_read` |
| `"brave search"` | `brave_web_search`, `brave_news_search`, `brave_local_search`, `brave_image_search`, `brave_video_search` |
| `"exa"` | `exa_search`, `exa_find_similar`, `exa_get_contents` |
| `"tavily"` | `tavily_search`, `tavily_extract` |
| `"firecrawl"` | `firecrawl_search`, `firecrawl_scrape`, `firecrawl_crawl`, `firecrawl_map` |
| `"perplexity"` | `perplexity_ask` |
| `"jina"` | `jina_search`, `jina_read_url` |
| `"kagi"` | `kagi_search`, `kagi_summarize` |
| `"serper"` | `serper_google_search`, `serper_news_search` |
| `"google search"` | `google_web_search` |

Run all ToolSearch calls in a single parallel message. Any tool that appears in the results is loaded and available.

Construct the `MCP_TOOLS` instruction block embedded into every sub-agent prompt:

- **MCP tools found**: List discovered tools by name. Instruct agents to use them as PRIMARY search tools, falling back to harness tools only on errors or empty results.
- **No MCP tools found**: Instruct agents to use harness-provided tools in this order:
  1. `WebSearch` / `WebFetch` (Claude Code built-in)
  2. `antigravity_search` / Antigravity search tools (if available)
  3. `codex_search` / Codex search tools (if available)

<context>
State: MCP_TOOLS instruction block is defined. Script runs synchronously.
Output: SCRIPT_DATA (Reddit/X results with engagement metrics, or empty if web-only mode).
</context>

## Phase 1: Run Python Script (Reddit + X)

Run the research script synchronously — it provides Reddit/X data with real engagement metrics that sub-agents cannot replicate.

```bash
RESEARCH_SCRIPT="$([ -f .claude/skills/research/scripts/research.py ] && echo .claude/skills/research/scripts/research.py || echo ~/.claude/skills/research/scripts/research.py)" && python3 "$RESEARCH_SCRIPT" "$TOPIC" --emit=compact 2>&1
```

The `$DEPTH` flag maps to: `--quick` -> pass `--quick`; default -> no flag; `--deep` -> pass `--deep`.

Store the output as `SCRIPT_DATA`. Check mode from output:
- **"Mode: both"** / **"Mode: reddit-only"** / **"Mode: x-only"**: Script found data
- **"Mode: web-only"**: No API keys, sub-agents provide all data

Do not stop or warn if web-only. Proceed to Phase 2.

<context>
State: SCRIPT_DATA collected. All agents launched in ONE message for parallelism.
Dependencies: MCP_TOOLS block embedded in every sub-agent prompt.
</context>

## Phase 2: Sub-Agent Dispatch

Dispatch the exact agent count below. Each agent targets its specific focus area -- do not reduce the count or use generic agents.

### Step 1: Read Prompt Templates

Use the Read tool to read `references/subagent_prompts.md` BEFORE constructing any prompts. That file contains the Community Agent Template and Official Agent Template you must fill for each agent.

### Step 2: Agent Roles

Agent count by depth: `--quick` = 6 (3C + 3O) | default = 8 (4C + 4O) | `--deep` = 10 (5C + 5O)

**Community agents** (exclude reddit.com and x.com from all searches):

| Agent | Focus | When Active |
|-------|-------|-------------|
| C1 | HN + tech forums (Lobsters, Stack Overflow) | Always |
| C2 | Broader community (forums, Discourse, Product Hunt) | Always |
| C3 | Niche communities + comparisons (review sites, specialised forums) | Always |
| C4 | Developer/creator communities (Dev.to, GitHub Discussions, blogs) | default + deep |
| C5 | International perspectives + user reviews (G2, Capterra, global forums) | deep only |

**Official agents:**

| Agent | Focus | When Active |
|-------|-------|-------------|
| O1 | Official docs + changelogs + release notes | Always |
| O2 | Industry publications + analysis (Ars Technica, Wired, InfoQ) | Always |
| O3 | Academic papers + research (arXiv, Google Scholar) | Always |
| O4 | Government/institutional + standards bodies | default + deep |
| O5 | Expert blogs + thought leadership | deep only |

### Step 3: Dispatch

For each agent, fill the matching template from `references/subagent_prompts.md` with: `{TOPIC}`, `{QUERY_TYPE}`, `{FOCUS}`, `{QUERIES}` (3-5 queries from `references/subagent_prompts.md` Query Generation tables), `{DATE_FROM}` (60 days ago), `{MCP_TOOLS}`.

Every Task call uses `subagent_type: "general-purpose"` and a description matching the agent role (e.g. `"C1: HN + tech forums"`, `"O2: industry pubs"`).

**Dispatch ALL agents in a SINGLE message with parallel Task calls. Do not dispatch sequentially.**

## Phase 3: Collect Results

After dispatching, collect results from all agents:

1. Call `TaskOutput` for each dispatched agent
2. Organize into: `COMMUNITY_FINDINGS` (C1-C5) and `OFFICIAL_FINDINGS` (O1-O5)
3. **Graceful failure**: If an agent fails or returns empty, log which agent failed, continue with remaining results, note the gap in the final report. Do not retry.

<context>
State: SCRIPT_DATA + COMMUNITY_FINDINGS + OFFICIAL_FINDINGS all collected.
Mode: Synthesis — ground in actual research, not pre-existing knowledge.
</context>

## Phase 4: Judge Synthesis

Synthesize all findings (SCRIPT_DATA + COMMUNITY_FINDINGS + OFFICIAL_FINDINGS) into a coherent two-sided report.

### Weighting

| Source | Weight | Why |
|--------|--------|-----|
| Reddit (from script) | HIGHEST | Real upvotes + comments = proven engagement |
| X (from script) | HIGHEST | Real likes + reposts = proven engagement |
| HN / Lobsters | HIGH | Voting system = community curation |
| Official docs | HIGH | Authoritative, primary source |
| Academic papers | HIGH | Peer-reviewed |
| Industry publications | MEDIUM | Expert but potentially biased |
| Expert blogs | MEDIUM-LOW | Individual perspective |
| Misc community | MEDIUM-LOW | Volume varies |

### Cross-Reference Analysis

Identify 3-5 topics where community and official sources can be compared:
- **Aligned**: Both sides agree
- **Divergent**: Community says one thing, officials say another
- **Gap**: One side has information the other lacks

## Internalize the Research

Ground your synthesis in actual research content, not pre-existing knowledge. Read all agent outputs carefully, paying attention to exact names, specific insights, and real engagement numbers.

### If QUERY_TYPE = RECOMMENDATIONS

Extract specific names from all sources (script + community + official agents). Count mentions across sources, note which sources recommend each, list by popularity.

<example>
BAD: "Skills are powerful. Keep them under 500 lines."
GOOD: "Most mentioned: /commit (5 mentions, r/ClaudeAI + HN), remotion skill (4x, X + Dev.to), git-worktree (3x, GitHub Discussions). Official docs highlight /pr as recommended."
</example>

### For All Query Types

From the actual research output, identify:
- **PROMPT FORMAT** — Does research recommend JSON, structured params, natural language, keywords?
- Top 3-5 patterns/techniques that appeared across multiple sources
- Specific keywords, structures, or approaches mentioned by the sources
- Common pitfalls mentioned by the sources

If research says "use JSON prompts" or "structured prompts", deliver prompts in that format later.

Self-check: Re-read your synthesis before displaying. If it does not match what the research actually says, rewrite it.

## Display Two-Sided Report

Refer to `references/output_format.md` for the full template. Do not output any "Sources:" lists.

### 1. What the Community Says

Synthesize SCRIPT_DATA (Reddit/X) + COMMUNITY_FINDINGS (C1-C5):

<example>
If RECOMMENDATIONS:

Most Mentioned:
1. [Specific name] - mentioned {n}x (r/sub, HN, @handle, blog.com)
2. [Specific name] - mentioned {n}x (sources)
3. [Specific name] - mentioned {n}x (sources)

Notable mentions: [other specific things with 1-2 mentions]

If PROMPTING/NEWS/GENERAL:

What the community is saying:

[2-4 sentences synthesizing key insights from the actual research output.]

Key patterns:
1. [Pattern from research]
2. [Pattern from research]
3. [Pattern from research]
</example>

### 2. What the Official Sources Say

Synthesize OFFICIAL_FINDINGS (O1-O5):
- Key findings with authority attribution
- Recent official changes with dates
- Gaps in official coverage

### 3. Where They Agree and Disagree

Cross-reference table (3-5 rows):

| Topic | Community View | Official Position | Status |
|-------|---------------|-------------------|--------|
| [aspect] | [what community says] | [what officials say] | Aligned / Divergent / Gap |

### 4. Stats Footer

Display real numbers from the research:
```
All agents reported back!
|- Reddit: {n} threads | {upvotes} upvotes | {comments} comments
|- X: {n} posts | {likes} likes | {reposts} reposts
|- Community web: {n} sources (HN, forums, blogs)
|- Official web: {n} sources (docs, papers, reports)
|- Agents dispatched: {total} ({C}C + {O}O)
|- Cross-reference: {agree} aligned, {disagree} divergent, {gaps} gaps
```

If web-only mode, omit Reddit/X lines and add the API key hint from `references/output_format.md`.

### 5. Invitation

```
Share your vision for what you want to create and I'll write a thoughtful prompt
you can copy-paste directly into {TARGET_TOOL}.
```

Use real numbers from the research output. Patterns should be actual insights, not generic advice.

If TARGET_TOOL is still unknown after showing results, ask now:
```
What tool will you use these prompts with?

Options:
1. [Most relevant tool based on research]
2. Nano Banana Pro (image generation)
3. ChatGPT / Claude (text/code)
4. Other (tell me)
```

After displaying the report and invitation, wait for the user to respond.

## Prompt Generation

When the user shares their vision, write ONE tailored prompt using expertise from BOTH community and official sources.

Match the format the research recommends (JSON, structured params, natural language, keywords).

See `references/prompt_generation.md` for the full prompt writing protocol, quality checklist, and output footer templates.

## Context Memory

After research completes, retain topic expertise for follow-up questions. See `references/context_memory.md` for full context retention protocol.

</instructions>
