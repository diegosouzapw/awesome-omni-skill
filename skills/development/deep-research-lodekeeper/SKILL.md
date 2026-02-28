---
name: deep-research
description: Multi-agent deep research pipeline for complex questions (EIP analysis, architecture decisions, cross-client comparisons, protocol design). Use when single-shot answers are insufficient and you need decomposition, parallel investigation, adversarial critique, and a formal output document.
---

# Deep Research Skill

Multi-agent deep research pipeline for complex topics. Produces formalized research documents (specs, analyses, proposals) through iterative investigation, synthesis, and adversarial critique.

**When to use:** Complex questions requiring genuine research — EIP analysis, implementation strategies, novel ideas, cross-client comparisons, protocol design, or any topic where a single-shot answer isn't good enough.

**Expected duration:** 30-90 minutes depending on complexity.

---

## Prerequisites

- **Oracle CLI:** `oracle` (GPT-5 Pro access for deep reasoning)
- **Oracle Bridge:** See `skills/oracle-bridge/SKILL.md` — required for browser mode on this server
- **Sub-agents:** Available via `sessions_spawn` (explorer, specialist, adversary roles)
- **Web search:** For prior art, papers, existing implementations
- **File access:** For reading specs, code, EIPs locally

## Related Skills

- `skills/oracle-bridge/SKILL.md` — starts and troubleshoots Oracle browser mode (default path for GPT-5.2-pro reasoning).
- `skills/web-scraping/SKILL.md` — use when `web_search`/`web_fetch` are insufficient or blocked and you need robust page acquisition before synthesis.
- `skills/dev-workflow/SKILL.md` — use when research is feeding directly into a Lodestar implementation plan/PR.

Check Oracle is available:
```bash
source ~/.nvm/nvm.sh && nvm use 22 && oracle --version
```

### ⚠️ Always Save Oracle Output to File

Oracle runs take minutes+. Context can compact mid-run, losing stdout. **Every Oracle call MUST pipe to a file via `| tee <path>`**. No exceptions. This ensures output survives compaction and can be read back later.

### Oracle Engine Priority

Oracle has two engines. **Always use browser mode first** (uses ChatGPT Pro subscription, no per-query cost).

| Engine | Command | Cost | Reliability |
|--------|---------|------|-------------|
| **Browser (default)** | `ORACLE_REUSE_TAB=1 oracle --engine browser --remote-chrome localhost:9222` | Free (Pro sub) | Requires bridge running + valid session token |
| **API (fallback)** | `oracle --engine api` | ~$0.09/query | Always works if API key set |

**⚠️ CRITICAL:** Do NOT silently fall back to API mode. If browser mode fails (expired token, bridge down):
1. **Stop** — do not continue research
2. **Alert user:** "ChatGPT session token expired. Need fresh `__Secure-next-auth.session-token` from chatgpt.com, or explicit approval to use API mode."
3. Only switch to API if user explicitly approves

### Starting the Oracle Bridge

Before any Oracle browser-mode call, ensure the bridge is running:

```bash
# Check if bridge is already running
curl -s http://localhost:9222/json/version && echo "Bridge running" || echo "Bridge not running"

# Start bridge (if not running)
source ~/camoufox-env/bin/activate
python3 ~/.openclaw/workspace/research/oracle-bridge-v3.py \
  --cookies ~/.oracle/chatgpt-cookies.json &
sleep 15  # wait for browser + CF bypass + login

# Verify
curl -s http://localhost:9222/json/version | grep -q Chrome && echo "Ready"
```

For full bridge documentation, see `skills/oracle-bridge/SKILL.md`.

---

## Research Type Classification

Different research questions need different tools. **Classify each sub-question during Phase 1** and route accordingly:

### Type A: Web Literature / Ecosystem Survey
*"What tools exist for X?", "Compare approaches to Y", "Find prior art on Z"*

**Best tool:** `o3-deep-research` or `o4-mini-deep-research` (OpenAI API)
- Purpose-built for multi-source web browsing + synthesis with citations
- Automatically searches, reads, reconciles, and produces documented reports
- Far superior to manual web_search + sub-agent for broad surveys

```bash
source ~/.nvm/nvm.sh && nvm use 22
oracle --engine api \
  -p "Research [topic]. Browse multiple sources, compare approaches, and produce a cited report covering: [specific questions]" \
  --model o4-mini-deep-research --wait \
  2>&1 | tee ~/research/<topic>/findings/web-survey.md
```

**Cost:** `o4-mini-deep-research` ~$1.10/$4.40 per 1M tokens (cheaper). `o3-deep-research` ~$10/$40 per 1M tokens (most powerful).

**⚠️ MANDATORY: Ask Nico before using ANY deep research API model.** Explain why `web_search` + sub-agents weren't sufficient. Only proceed after explicit "yes".

**Fallback (free):** Sub-agent + `web_search` + `web_fetch`; if sources are blocked/partial, switch to `skills/web-scraping/SKILL.md` for robust acquisition before synthesis.
```
sessions_spawn task:"Research [sub-question]. Start with web_search/web_fetch for prior art and papers. If key pages are blocked or JS-rendered, use skills/web-scraping/SKILL.md tiered scraper. Write findings to ~/research/<topic>/findings/web-research.md"
```

### Type B: Codebase / Spec Analysis
*"How does Lodestar handle X?", "What does the spec say about Y?", "Find the bug in Z"*

**Best tool:** Codex CLI (`xhigh` reasoning) or Claude CLI + sub-agents
- Needs local file access (repos, specs, code)
- Can run tests, grep codebases, read large files
- Deep research API models can't do this

```bash
# Codex for focused code investigation
codex exec --full-auto "Analyze [question] in ~/lodestar/packages/... Write findings to ~/research/<topic>/findings/code-analysis.md"

# Or Claude CLI for broader reasoning
claude "Read [files] and analyze [question]. Write to ~/research/<topic>/findings/code-analysis.md"
```

**Or via sub-agent:**
```
sessions_spawn task:"Analyze [sub-question] by reading:
- Relevant consensus specs: ~/consensus-specs/specs/...
- Lodestar implementation: ~/lodestar/packages/...
- Other client implementations (search GitHub)
Write findings to ~/research/<topic>/findings/spec-analysis.md"
```

### Type C: Deep Reasoning / Novel Analysis
*"What are the tradeoffs of X?", "Design an approach for Y", "What's the best architecture for Z?"*

**Best tool:** GPT-5.2 Pro (via Oracle browser mode)
- Strongest reasoning for novel analysis and synthesis
- Best when you already have the materials and need deep thinking
- Also excellent for adversarial critique

```bash
ORACLE_REUSE_TAB=1 oracle --engine browser \
  --remote-chrome localhost:9222 \
  -p "[Your reasoning prompt]" \
  --file ~/research/<topic>/plan.md \
  --model gpt-5.2-pro --wait \
  2>&1 | tee ~/research/<topic>/findings/analysis.md
```

### Type D: Cross-Client Comparison
*"How do other clients implement X?"*

**Best tool:** Sub-agent (surveyor) — can search GitHub, read code
```
sessions_spawn task:"Survey how Prysm, Lighthouse, Teku, and Nimbus handle [topic].
Compare approaches, identify patterns. Write to ~/research/<topic>/findings/cross-client.md"
```

---

## Workflow

### Phase 0: Scoping (5-10 min) — MANDATORY

Before any research begins, return to the human with:

1. **Problem statement** — your understanding of what's being asked
2. **Decomposition** — 3-5 sub-questions, each **classified by type** (A/B/C/D)
3. **Tool routing** — which model/agent handles each sub-question and why
4. **Assumptions** — anything you'd need to assume if not clarified
5. **Cost estimate** — if using API models (deep research, GPT-5.2 Pro API), estimate token cost
6. **Estimated time** — rough estimate based on complexity
7. **Clarifying questions** — anything ambiguous or underspecified

**Wait for approval before proceeding.** Especially important when API-cost models are proposed.

### Phase 1: Decomposition (5 min)

Once approved, finalize the research plan:

1. Break the topic into 3-5 independent sub-questions
2. **Classify each sub-question** by research type (A/B/C/D — see above)
3. Assign each to the best agent/tool based on classification
4. Create the research workspace:
   ```bash
   mkdir -p ~/research/<topic-slug>/{findings,drafts}
   ```
5. Write the research plan to `~/research/<topic-slug>/plan.md`

### Phase 2: Parallel Investigation (15-30 min)

Launch all sub-questions simultaneously. Use the routing from Phase 1.

**Example mixed investigation:**

```
# Type A: Web survey (sub-agent with web_search, or deep research API if approved)
sessions_spawn task:"Research [web question]. Write to ~/research/<topic>/findings/web-survey.md"

# Type B: Code analysis (Codex or sub-agent)
sessions_spawn task:"Analyze [code question] in ~/lodestar/... Write to ~/research/<topic>/findings/code-analysis.md"

# Type C: Deep reasoning (Oracle browser mode)
ORACLE_REUSE_TAB=1 oracle --engine browser --remote-chrome localhost:9222 \
  -p "[reasoning question]" --model gpt-5.2-pro --wait \
  2>&1 | tee ~/research/<topic>/findings/oracle-analysis.md

# Type D: Cross-client survey (sub-agent)
sessions_spawn task:"Survey other clients on [topic]. Write to ~/research/<topic>/findings/cross-client.md"
```

**Wait for all agents to complete before proceeding.**

### Phase 3: Synthesis (10-15 min)

1. Read all findings from `~/research/<topic>/findings/`
2. Identify:
   - Common themes across sources
   - Contradictions or disagreements
   - Gaps in coverage
   - Surprising or novel findings
3. Write a draft document to `~/research/<topic>/drafts/v1.md` using the output template (see below)

### Phase 4: Adversarial Critique (10-15 min)

Send the draft through adversarial review. Use **two different perspectives**:

**Adversary #1 — GPT-5.2 Pro (via Oracle):**
```bash
ORACLE_REUSE_TAB=1 oracle --engine browser \
  --remote-chrome localhost:9222 \
  -p "You are a rigorous adversarial reviewer. Find weaknesses, gaps, and flawed reasoning.

For each section:
1. Challenge the key claims — are they well-supported?
2. Identify missing perspectives or counterarguments
3. Point out logical gaps or unsupported leaps
4. Suggest what additional evidence would strengthen weak points
5. Rate confidence: HIGH / MEDIUM / LOW for each major conclusion

Be constructive but ruthless." \
  --file ~/research/<topic>/drafts/v1.md \
  --model gpt-5.2-pro --wait \
  2>&1 | tee ~/research/<topic>/drafts/critique.md
```

**Adversary #2 — Claude Sonnet (different model family):**
```
sessions_spawn task:"Review this research document as a devil's advocate.
Read ~/research/<topic>/drafts/v1.md
Challenge every assumption. Find what's missing. Identify risks.
Write critique to ~/research/<topic>/drafts/critique-2.md"
model:"anthropic/claude-sonnet-4-5" thinking:"high"
```

### Phase 5: Revision (5-10 min)

1. Read both critiques
2. Address valid criticisms — strengthen weak arguments, add missing perspectives
3. Mark unresolvable disagreements as "Open Questions"
4. Write final document to `~/research/<topic>/output.md`
5. If critiques revealed fundamental gaps, loop back to Phase 2 for targeted investigation

### Phase 6: Delivery

1. Present the final document to the human
2. Highlight:
   - Key findings / recommendations
   - Confidence levels for major conclusions
   - Open questions that need human judgment
   - Suggested next steps
3. Save to `~/research/<topic>/output.md` (and any supplementary materials)

---

## ChatGPT Deep Research (Manual Mode)

ChatGPT's built-in **Deep Research** feature is the most powerful option for web-based research with citations. It's included in the Pro subscription (free to use) but **cannot be automated** via Oracle — it requires manual interaction in the browser.

**When to suggest it:** For major web literature reviews where the human is available to trigger it manually.

**How it works:**
1. Open chatgpt.com → select "Deep Research" from the model/agent dropdown
2. Enter the research question
3. Review and optionally edit the research plan
4. Wait 5-30 min for results (it browses, reads, synthesizes automatically)
5. Get a documented report with citations

**When our skill is better:** When research involves local code/specs, needs code execution, or requires custom agent coordination. ChatGPT Deep Research can't read our repos or run tests.

**Hybrid approach:** For mixed research, suggest the human triggers Deep Research for the web survey portion, then feed those results into our Phase 3 synthesis alongside our code/spec findings.

---

## Output Template

```markdown
# Research: [Topic Title]

**Date:** YYYY-MM-DD
**Requested by:** [who]
**Duration:** [time spent]
**Confidence:** HIGH / MEDIUM / LOW
**Models used:** [list models/tools used for each phase]

## Executive Summary
[2-3 paragraph summary of findings and recommendations]

## Problem Statement
[Clear definition of what was researched and why]

## Prior Art / Related Work
[What exists, who's done what, relevant papers/EIPs/implementations]

## Analysis
### [Sub-topic 1]
[Findings, evidence, reasoning]

### [Sub-topic 2]
[Findings, evidence, reasoning]

### [Sub-topic N]
[Findings, evidence, reasoning]

## Cross-Client Comparison (if applicable)
| Aspect | Lodestar | Lighthouse | Prysm | Teku |
|--------|----------|------------|-------|------|
| ...    | ...      | ...        | ...   | ...  |

## Proposed Approach
[Recommended solution/direction with justification]

### Alternatives Considered
[Other approaches and why they were rejected]

### Tradeoffs
[Explicit tradeoffs of the proposed approach]

## Implementation Sketch (if applicable)
[High-level design, key interfaces, data flow]

## Risk Assessment
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| ...  | ...       | ...    | ...        |

## Open Questions
[Things that couldn't be resolved and need human judgment or further research]

## Sources
[Links, references, citations]
```

---

## Model Selection Guide

| Role | Best Model | Fallback | Why |
|------|-----------|----------|-----|
| **Scoping** | Opus (me) | — | Needs judgment about what matters |
| **Web survey** | `o4-mini-deep-research` (API) | Sub-agent + web_search (free) | Purpose-built for web research with citations |
| **Deep web research** | `o3-deep-research` (API) | GPT-5.2 Pro (browser) | Most powerful web research model |
| **Code/spec analysis** | Codex CLI (xhigh) | Claude CLI / sub-agent | Best for long-horizon code investigation |
| **Deep reasoning** | GPT-5.2 Pro (Oracle browser) | GPT-5.2 Pro (API, with approval) | Strongest reasoning for novel analysis |
| **Cross-client survey** | Sub-agent (surveyor) | — | Needs GitHub access, code reading |
| **Adversary #1** | GPT-5.2 Pro (Oracle browser) | — | Strongest adversarial reasoning |
| **Adversary #2** | Claude Sonnet (thinking:high) | — | Different model family = different blind spots |
| **Synthesis** | Opus (me) | — | Quality control, coherent narrative |
| **Manual deep research** | ChatGPT Deep Research (browser) | — | Most powerful but requires human to trigger |

### Cost Reference

| Model | Input | Output | Notes |
|-------|-------|--------|-------|
| GPT-5.2 Pro (browser) | Free | Free | Pro subscription, via Oracle bridge |
| GPT-5.2 Pro (API) | ~$0.03/query | ~$0.09/query | Needs user approval |
| `o4-mini-deep-research` | $1.10/1M | $4.40/1M | Cheaper deep research |
| `o3-deep-research` | $10/1M | $40/1M | Most powerful deep research |
| Sub-agents (Claude) | Session cost | Session cost | Included in OpenClaw |

**Rules:**
1. **Always try free options first:** `web_search` + `web_fetch` + sub-agents. Only escalate to API models if manual search is genuinely insufficient.
2. **`o3-deep-research` and `o4-mini-deep-research` require explicit approval from Nico before every use.** These are expensive — ask first, explain why manual search wasn't enough, and get a "yes" before running.
3. **GPT-5.2 Pro browser mode is free** (Pro subscription) — use it freely for reasoning tasks.
4. **GPT-5.2 Pro API mode** (~$0.09/query) — acceptable for occasional use but prefer browser mode.

---

## Self-Healing

If something fails during research:

1. **Oracle browser mode fails (token expired):** Alert user immediately. Do NOT silently fall back to API. Only use `--engine api` with explicit user approval.
2. **Oracle bridge won't start:** Kill stale processes (`pkill -f "chromium.*headless"`), check `~/.oracle/chatgpt-cookies.json` exists, reinstall browser if needed (`python3 -m rebrowser_playwright install chromium`). See `skills/oracle-bridge/SKILL.md` for full troubleshooting.
3. **Oracle completely unavailable (no bridge, no API key):** Fall back to sub-agents with thinking:high for deep reasoning.
4. **Deep research API model fails:** Fall back to sub-agent + web_search approach (free, just slower).
5. **Web search returns nothing:** Try alternative search queries, check specific repos/forums directly.
6. **Sub-agent times out:** Retry with a narrower scope or split the task.
7. **Source contradictions:** Document both perspectives, flag for human judgment.
8. **Scope creep:** If a sub-question opens up a rabbit hole, note it in "Open Questions" rather than derailing the main research.

**After each research run, update this skill:**
- If a tool/approach consistently fails, document the failure and alternative
- If a new tool or source proves valuable, add it to the workflow
- If the output template needs adjustment based on feedback, update it

---

## Iteration

Research is rarely one-shot. The skill supports iterative deepening:

### "Go Deeper" Loop
When the human says "go deeper on X":
1. Extract the specific area from the previous output
2. Re-enter at Phase 1 with a narrowed scope focused on X
3. Use previous findings as context for the new investigation
4. Produce an updated document that integrates both rounds

### Follow-up Research
When new information emerges after initial research:
1. Read the previous output from `~/research/<topic>/output.md`
2. Identify what's changed or what new information is available
3. Run targeted Phase 2 investigation on the delta
4. Revise the document (don't start from scratch)

### Research Chains
Some topics naturally lead to follow-up questions:
1. After delivering output, explicitly note "This research suggests the following follow-up investigations: ..."
2. The human can trigger any of these as new research tasks
3. Link related research documents together via references

---

## Oracle Quick Reference

```bash
source ~/.nvm/nvm.sh && nvm use 22

# --- BROWSER MODE (default — uses ChatGPT Pro subscription, free) ---

# 1. Ensure bridge is running (see skills/oracle-bridge/SKILL.md)
curl -s http://localhost:9222/json/version | grep -q Chrome || {
  echo "Start bridge first!"
  echo "source ~/camoufox-env/bin/activate"
  echo "python3 ~/.openclaw/workspace/research/oracle-bridge-v3.py --cookies ~/.oracle/chatgpt-cookies.json &"
}

# 2. Run queries
ORACLE_REUSE_TAB=1 oracle --engine browser \
  --remote-chrome localhost:9222 \
  -p "Your prompt" --file path/to/context.md \
  --model gpt-5.2-pro --wait

# --- API MODE (fallback — costs per query, needs user approval) ---

# Standard reasoning
oracle --engine api -p "Your prompt" --file context.md --model gpt-5.2-pro

# Deep research (web survey with citations — needs approval for API cost)
oracle --engine api -p "Research [topic] comprehensively" --model o4-mini-deep-research
oracle --engine api -p "Research [topic] comprehensively" --model o3-deep-research

# Dry run (preview without spending tokens)
oracle --dry-run summary -p "Your prompt" --file context.md
```

**Browser mode:** Requires oracle-bridge running + valid session token at `~/.oracle/chatgpt-cookies.json`.
**API mode:** Requires `OPENAI_API_KEY` (set in `~/.bashrc`). Only use as explicit fallback with user approval.

See `skills/oracle-bridge/SKILL.md` for full bridge setup, troubleshooting, and token refresh.

---

## Notes

- **Always create `~/research/<topic-slug>/`** for each research task — keeps outputs organized and referenceable
- **Save intermediate findings** — if a session crashes, you don't lose work
- **Time-box phases** — if Phase 2 is taking >30 min, wrap up what you have and move to synthesis
- **Human in the loop** — Phase 0 (scoping) is mandatory. Don't skip it, even for "obvious" topics
- **Classify before routing** — the Research Type Classification section is the key improvement. Use it.
- **Quality > Speed** — this skill is designed for depth, not quick answers. Take the time needed.
- **ChatGPT Deep Research** — suggest it for major web surveys when the human is available. It's the most powerful option and free with Pro sub.
