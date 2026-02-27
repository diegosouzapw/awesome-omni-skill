---
name: ai-briefing
description: Generate a comprehensive AI Intelligence Brief covering the last 14 days of AI ecosystem developments relevant to a healthcare technology CTO. Creates a timestamped markdown file in the news/ directory.
argument-hint: [optional focus area or date override]
---

# AI Intelligence Briefing Skill

You are generating a comprehensive **AI Intelligence Brief** for Ammar Darazanli, CTO of a healthcare technology company focused on AI-powered automation, voice AI systems, remote patient monitoring, and AI-assisted development workflows.

Your task is to research the last 14 days of AI ecosystem news and produce a **single timestamped markdown file** in the `news/` directory.

---

## Step 0: Setup & Context Loading

1. Create the `news/` directory if it does not exist.
2. Generate the filename using the current timestamp: `YYYY-MM-DD-HH-MM-SS.md` (use `date "+%Y-%m-%d-%H-%M-%S"`).
3. **Compute date variables** for use in all search queries throughout the brief:
   ```bash
   CURRENT_MONTH=$(date "+%B")         # e.g., "February"
   CURRENT_YEAR=$(date "+%Y")          # e.g., "2026"
   PREV_MONTH=$(date -v-14d "+%B")     # month from 14 days ago
   COVERAGE_START=$(date -v-14d "+%Y-%m-%d")
   COVERAGE_END=$(date "+%Y-%m-%d")
   ```
   These variables ensure all search queries reference the actual coverage period rather than hardcoded dates.
4. Calculate the coverage period: `$COVERAGE_START` through `$COVERAGE_END`.
5. **Scan `news/` for the most recent existing brief.** If one exists, read it in full. Extract:
   - All watchlist items and their scores
   - Categories that had thin coverage or were missing items
   - Items that need status updates
   - This informs your search strategy in Step 1.
6. If `$ARGUMENTS` contains a specific focus area, give it extra weight in the research and scoring. Otherwise, cover all categories equally.

## Step 1: Research Phase 1 — Broad Sweep (Parallel)

Use **WebSearch** extensively. Launch searches in parallel batches. Perform **at least 12 distinct searches** covering all 7 categories below.

### Date Handling in Search Queries

**Use `date` to compute the current month and year dynamically** — never hardcode month names or years. Before writing any search query, run:

```bash
CURRENT_MONTH=$(date "+%B")    # e.g., "February"
CURRENT_YEAR=$(date "+%Y")     # e.g., "2026"
PREV_MONTH=$(date -v-14d "+%B") # month from 14 days ago (may differ)
```

Then construct queries using these variables. If the coverage period spans two months, include both month names in relevant queries.

**Write specific queries anchored to the coverage period, not generic ones:**
- BAD: "Anthropic Claude announcements" *(too vague, returns old results)*
- BAD: "AI news March 2025" *(hardcoded stale date)*
- GOOD: "Anthropic Claude new model release features pricing {CURRENT_MONTH} {CURRENT_YEAR}"
- GOOD: "Claude Code enterprise adoption GitHub Agent HQ {CURRENT_YEAR}"
- GOOD: "healthcare voice AI startup funding Series A {CURRENT_YEAR}"
- GOOD: "Oracle Health ambient clinical AI launch {CURRENT_MONTH} {CURRENT_YEAR}"

Every search query that includes a date **must** use the computed current month/year or the coverage period boundaries. If a query doesn't need a date (e.g., searching for a specific product name), omit the date rather than guessing.

### 1.1 Major AI Labs
Search for announcements, model releases, and significant updates from:
- Anthropic (Claude models, Claude Code, API changes, enterprise features, partnerships)
- OpenAI (GPT models, ChatGPT, Codex, API changes, partnerships, security features)
- Google (Gemini models, Workspace AI, API changes, partnerships)
- Meta (LLaMA models, open-weight releases)
- xAI (Grok models, voice API, enterprise access)
- Mistral (model releases, edge deployment, pricing)
- Cohere (enterprise AI, RAG, embeddings)

### 1.2 AI Coding & Dev Tools
Search for updates from:
- GitHub Copilot (agent mode, features, pricing)
- Cursor (new versions, multi-agent, features)
- Windsurf (Cascade updates, Arena Mode, new models)
- Claude Code (enterprise adoption, GitHub integration, new capabilities)
- Xcode AI integrations (Claude Agent SDK, Codex)
- Devin (autonomous coding, pricing, capabilities)
- Replit Agent (new versions, autonomous workflows)
- v0 / Bolt.new / Lovable (AI app builders, vibe coding)

### 1.3 AI Automation Platforms
Search for updates from:
- n8n (AI agent features, HITL, new nodes)
- Make / Zapier AI (automation capabilities)
- LangChain / LangGraph (framework updates, production features)
- CrewAI (multi-agent, new versions, compliance)
- AutoGen (Microsoft, agent frameworks)
- OpenAI Agents SDK (Agent Builder, AgentKit, ChatKit)

### 1.4 Infrastructure & Models
Search for:
- Hugging Face (trending models, platform updates)
- Ollama (new versions, performance improvements)
- vLLM (inference optimizations)
- New open-weight models and fine-tuning breakthroughs
- NVIDIA AI inference updates
- GitHub trending AI repositories

### 1.5 Voice & Healthcare AI
Search for:
- Voice AI platforms (Speechmatics, Deepgram, AssemblyAI, ElevenLabs)
- Healthcare voice AI startups and funding rounds
- EHR integrations and ambient clinical AI (Oracle Health, Epic, Cerner)
- Remote patient monitoring and wearables (Validic, Philips, Masimo)
- Healthcare AI market developments and clinical AI agents

### 1.6 Standards & Regulation
Search for:
- MCP (Model Context Protocol) updates, new servers, security
- AI executive orders and federal policy
- HIPAA/AI guidance and enforcement
- FDA AI/ML framework updates, CDS guidance
- State AI regulation (Colorado, etc.)
- Healthcare compliance developments

### 1.7 Community & Research
Search for:
- GitHub trending repos (AI-tagged)
- Reddit community sentiment (r/LocalLLaMA, r/ClaudeAI, r/ChatGPT, r/MachineLearning)
- ArXiv notable papers (applied AI, agents, healthcare AI)
- Product Hunt AI launches

## Step 1.5: Research Phase 2 — Targeted Follow-ups

After Phase 1, review all results and identify:
- **Names, companies, or products mentioned** in search results that you haven't directly searched for yet
- **Categories with fewer than 2 substantive findings** — do additional searches
- **Items from the previous brief's watchlist** that need updated status
- **Funding rounds, partnerships, or launches** referenced in passing that deserve their own search

Launch **6-10 additional targeted searches** to fill these gaps. These should be specific and use the computed date variables where relevant:
- "Secai Voxira healthcare voice AI agent funding {CURRENT_YEAR}"
- "OpenAI Lockdown Mode ChatGPT enterprise security {CURRENT_MONTH} {CURRENT_YEAR}"
- "Apple Xcode Claude Agent SDK integration details {CURRENT_YEAR}"

**Do NOT proceed to scoring until every category has at least 2 findings backed by primary sources.**

## Step 2: Score Each Development

For each notable development found, assign a **Disruption Score (1-10)**:

| Score Range | Meaning |
|-------------|---------|
| 1-3 | Incremental improvement, nice to know |
| 4-6 | Meaningful capability change, worth evaluating within 30 days |
| 7-8 | Significant shift, should prototype or adopt within 2 weeks |
| 9-10 | Potential game-changer, immediate action recommended |

**Score adjustment rules:**
- +1 for sustained community momentum over 48hrs
- +1 for enterprise/healthcare adoption signals
- -1 for unresolved critical bugs or broken promises
- -1 for hype fade with no real usage evidence

## Step 2.5: Coverage Self-Check

Before writing the report, verify ALL of these:
- [ ] Every category (1.1-1.7) has at least 2 substantive findings
- [ ] Previous watchlist items all have updated status (if previous brief exists)
- [ ] At least 3 items are NEW (not carried from previous brief)
- [ ] You have at least 10 real, clickable URLs for the Top 10 Links section
- [ ] No item's "Impact on You" is generic filler — every bullet is specific and actionable

**If any check fails, do more targeted searches before proceeding.**

## Step 3: Generate the Report

Write the markdown file with the following structure. Every section is **required**.

```markdown
# AI INTELLIGENCE BRIEF — {Full Date}

**Coverage Period:** {14 days ago} - {today}
**Prepared For:** Ammar Kazanli, CTO — Healthcare Technology / AI Automation / Voice AI / Remote Patient Monitoring
**Generated:** {timestamp}

---

## TOP SCORER OF THE DAY

**{Item Name} — Score: X/10**

Why this score: {2-3 sentence justification citing specific signals — e.g., GitHub stars growth, enterprise announcements, benchmark results, community validation}

---

## MOVERS (Score changed this period)

| Item | Score | Change | Reason |
|------|-------|--------|--------|
{Items whose scores changed from previous brief. For first-time items, mark as NEW.}

---

## NEW ENTRIES

| Item | Score | What + Why It Matters |
|------|-------|-----------------------|
{All newly discovered items this period with one-line descriptions}

---

## TOP 10 LINKS I MUST VISIT TODAY

| # | Link | Why Visit | Score Context |
|---|------|-----------|---------------|
| 1 | [{descriptive title}]({URL}) | {One sentence on what you'll learn and why it matters today} | {Related item score}/10 |
| ... | ... | ... | ... |
| 10 | ... | ... | ... |

Selection criteria:
- Prioritize official announcements, release posts, and technical docs over news aggregator summaries
- Include at least 1 link from each: AI models, dev tools, healthcare AI, regulation
- Bias toward links with actionable information (APIs to try, docs to read, tools to evaluate)
- Include any link related to a score 8+ item
- Prefer primary sources (company blogs, GitHub repos, official docs) over secondary coverage

---

## WATCHLIST (All items scoring 5+, sorted by score)

| Rank | Item | Score | Trend | Status | Action By | Notes |
|------|------|-------|-------|--------|-----------|-------|
{All tracked items sorted by score descending. Trend: NEW/+N/-N/=. Status: Adopt / Prototype / Evaluate / Watch. Notes: one-line context.}

---

## DETAILED ANALYSIS BY CATEGORY

---

### 1. Major AI Labs

{For each lab with news this period:}

#### {Lab Name} — {Headline}

**Source:** {Markdown links to sources}

**What Changed:**
{Bullet list of specific changes}

**Impact on You:**
{1-3 bullets. Only include categories where impact is real and specific. Skip categories where the connection is forced.}

**Disruption Score: X/10** — {One-line action recommendation}

---

### 2. AI Coding & Dev Tools

{Same format. One sub-section per tool with significant news.}

---

### 3. AI Automation Platforms

{Same format}

---

### 4. Infrastructure & Open Models

{Same format}

---

### 5. Voice & Healthcare AI

{Same format}

---

### 6. Standards & Regulation

{Same format, with sub-sections for MCP, FDA, State regulation, HIPAA as relevant}

---

## RECOMMENDED ACTIONS

### Immediate (This Week)

{3-4 numbered actions with bold titles and one-line explanations}

### This Sprint (Next 2 Weeks)

{3-4 numbered actions}

### This Month

{3-4 numbered actions}

---

## MARKET CONTEXT

{3-5 macro observations. One bold thesis sentence each — no padding. Only include observations supported by 2+ items from this brief.}

---

## 7-DAY TREND DASHBOARD

This table tracks the **top-scoring items (7+ score)** across the last 7 days of the coverage period. Each column represents a day. The score shown is the item's disruption score on that day based on news activity. Use this to spot acceleration, sustained momentum, or fade-outs at a glance.

**How to build this table:**
1. For each item scoring 7+ in the current brief, look back through the last 7 days of the coverage period.
2. Assign a daily score based on whether significant news, releases, or developments occurred on that specific day. Use `—` if no news activity for that item on that day.
3. The "Trend" column summarizes the 7-day trajectory: `Surging` (3+ days of high activity), `Sustained` (steady presence), `Spiked` (one major day then quiet), `Building` (increasing over the week), `Cooling` (decreasing).
4. The "CTO Note" column provides a one-line actionable observation about what the trend means for decision-making.

| Item | {Day-7} | {Day-6} | {Day-5} | {Day-4} | {Day-3} | {Day-2} | {Day-1 (today)} | Trend | CTO Note |
|------|---------|---------|---------|---------|---------|---------|-----------------|-------|----------|
{Replace {Day-N} headers with actual dates (e.g., Feb 15, Feb 16, ..., Feb 21). One row per item scoring 7+. Daily scores reflect news activity on that specific day. Use — for quiet days. Trend column: Surging/Sustained/Spiked/Building/Cooling. CTO Note: one-line actionable insight.}

**Reading the dashboard:**
- **Surging items** (3+ active days) = high conviction signals, prioritize action
- **Spiked items** (single day) = may be announcement-driven, wait for follow-through before investing time
- **Sustained items** (steady across days) = durable trends worth strategic planning
- **Cooling items** = previous priorities losing momentum, consider deprioritizing

---

*Next brief: {tomorrow's date}*
*Archive items scoring below 5 after 14 days on watchlist*
*Flag any item hitting 8+ immediately — do not wait for daily brief*
```

## Important Guidelines

- **Bias toward signal over noise.** Don't report minor version bumps unless they unlock new capability.
- **Be honest about hype vs. substance.** Miss a trend for a day rather than chase vaporware.
- **Primary sources first.** Prefer official blogs, changelogs, GitHub repos, and documentation over news aggregator summaries.
- **Top 10 Links must be clickable.** Every URL must be a real URL discovered during research. Never fabricate URLs.
- **Disruption scores must be justified.** Cite specific evidence (benchmarks, GitHub activity, enterprise announcements).
- **No filler in "Impact on You."** Only include impact categories that are genuinely relevant to the item. If an item only affects AI-assisted development, don't force a healthcare or teaching angle.
- **Perform at least 18 web searches total** (12+ in Phase 1, 6+ in Phase 2). Cover all 7 sub-categories.
- **Track items for 14 days** on the watchlist, then archive unless still scoring 5+.
