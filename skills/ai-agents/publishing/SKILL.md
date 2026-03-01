---
name: publishing
description: Content strategy for external platforms (X, LinkedIn, etc.). Voice, style, and growth strategies.
user-invocable: false
---

# Publishing Content Strategy

> Core principles for creating content that grows audience

## Premium Status: ACTIVE ($20/mo, activated 2026-03-01)

X Premium is live. All Premium features unlocked:
- Communities access (30,000x reach multiplier)
- +100 TweepCred boost (escaped suppression)
- 10x algorithmic reach
- Link posting without suppression
- Reply visibility boost

**Current priorities (Premium era):**
1. **Create content aggressively** — X queue is empty, fill it
2. Post to Communities (Build in Public, AI/ML Builders, etc.)
3. Reply to own comments within 30 min (150x multiplier)
4. Continue cross-posting to Bluesky

---

## What Actually Works (Evidence-Based)

**Content formats ranked by performance (our data):**
1. **News hooks** - 3-6x average impressions (65, 62, 60, 51 imp vs 10 avg)
2. **Dollar-amount headlines** - ($285B, $2B, $1T) quantified impact stops scroll
3. **Name-drops** - (Karpathy, Altman, Anthropic, OpenAI) moderate boost
4. **Short posts** - outperform long framework posts by 3-6x
5. **Replies to official accounts** - (@OpenAI 24 imp) > individuals (0-6 imp)

**What underperforms:**
- Long authority/framework posts (<10 imp average)
- Posts about internal process without news hook (PDCA, spec engineering)
- Personality content without timeliness anchor
- Stale replies (>6h after original) — 0 impressions consistently

**When Premium active (evidence from research):**
- Communities posting = 30,000x reach multiplier
- Reply-to-own-comments within 30 min = 150x multiplier
- Reply-to-reply = 75x algorithm multiplier
- Videos (10+ sec) = 10x engagement vs text
- Threads (4-6 tweets) = 40-60% more reach
- Premium account = 10x reach, +100 TweepCred boost

---

## Hype-Driven Content Strategy (Primary Direction)

> Owner directive: Focus on what's hottest in AI right now. Connect to how people are making money fast. Clickbait + actionable links.

### Content Formula: Hype + Money + Action

Every post MUST have all three:
1. **Hype hook** — What's viral/trending RIGHT NOW (this week, not last month)
2. **Money angle** — Dollar amounts people are actually earning, specific revenue numbers
3. **Action links** — Real repos, tools, tutorials the reader can use TODAY

### What's Hot Right Now (March 2026 — update weekly)

| Trend | Money Angle | Key Links |
|-------|-------------|-----------|
| **OpenAI $110B raise** ($840B valuation) | Amazon $50B, Nvidia $30B, SoftBank $30B — largest private round ever | openai.com |
| **Anthropic-Pentagon standoff** | $200M contract refused, Claude surged to #1 App Store — principled stance = free PR | anthropic.com |
| **ChatGPT Agent Mode** | AI books, plans, executes autonomously — personal AI assistant era | openai.com/index/introducing-chatgpt-agent |
| **Vibe Coding** (92% dev adoption) | Claude Code = 4% of all GitHub commits, GPT-5.2-Codex SOTA on SWE-Bench Pro | cursor.com, claude.ai |
| **$195B invested in AI Feb 2026** | Record venture month — OpenAI $840B + Anthropic $380B + Waymo $16B | bloomberg.com |

### Content Priorities (Ranked)

1. **Trending tools + repos with money proof** (50%+ of content)
2. **"How people are making money" breakdowns** (30%)
3. **Personal experience / BIP connecting to trends** (20%)

### Predictions & Opinions (40-50% of content)

**Don't just report news — predict where it's going and what it means for business.**

Every prediction post MUST have:
1. **A bold stance** — take a side, don't hedge ("I think" > "it remains to be seen")
2. **Business impact** — how does this help/hurt real companies making money?
3. **Timeline** — when will this happen? (6 months, 1 year, 3 years)

**Prediction formulas:**
- "[News event] means [prediction]. Here's why: [reasoning]. Timeline: [when]."
- "Everyone's talking about [trend]. Nobody's asking: [deeper question]. My take: [opinion]."
- "[Technology] will [prediction] within [timeframe]. Here's what that means for [industry/business]."
- "Unpopular opinion: [contrarian take]. The data says [evidence]. Businesses should [action]."
- "3 things that will change about [domain] by [year]: 1. [prediction] 2. [prediction] 3. [prediction]"

**Examples:**
- "OpenAI raised $110B. My prediction: within 18 months, 80% of SaaS companies will either embed AI or die. Here's why the math is brutal..."
- "Everyone's hyped about vibe coding. Nobody's asking: what happens to code quality at scale? My take: we'll see a wave of AI-generated technical debt by 2027."
- "Agent Mode isn't just a feature. It's the end of per-seat SaaS pricing. Companies charging $50/seat will compete against AI agents at $0.10/task. Timeline: 12-18 months."
- "Call center AI will automate 80% of Tier 1 support by 2028. But here's what nobody tells you: the remaining 20% becomes 10x harder. That's where the money is."

**Use author's expertise for credible predictions:**
- Voice AI / call centers (7 years production = earned right to predict)
- Autonomous agents (this repo = living proof)
- Infrastructure → AI migration (career arc = trend visibility)
- Startup economics (15+ years = pattern recognition)

**What NOT to do with predictions:**
- Wishy-washy "time will tell" conclusions — commit to a position
- Predictions without business/money angle — always answer "so what for my business?"
- Fear-mongering without actionable advice — pair warnings with what to do about it

### What NOT to Post Anymore

- Enterprise industry analysis without money angle
- Call center / workforce stats without actionable takeaway
- Benchmark comparisons without "so what" for the reader
- Authority/framework posts without links or CTAs
- Anything that makes the reader think but not ACT

### Research Cadence for Hype Content

**Daily (at session start):** Quick scan for what's viral
- Check: trending GitHub repos, X trending, HackerNews front page
- Identify: new tools, repos, launches with money angles
- Update the "What's Hot Right Now" table above when trends shift

**Key sources for hype discovery:**
- github.com/trending
- news.ycombinator.com
- x.com/search (trending AI)
- producthunt.com
- indiehackers.com/tech

---

**Milestone content (technical CEO pattern, 5/5 builders validated):**
- **Product momentum = content momentum** (Greg Brockman, DHH, Rauch, Levels, Graham)
- Every PR milestone is a post (Session #150, #200, Premium activation, 50 followers, 100 followers)
- Radical transparency on numbers builds credibility: 160+ PRs, 8 followers, 354 tweets, 7 years Voice AI
- Example: "Session #150 shipped. 150 PRs, zero human intervention. Here's what an autonomous agent taught me about [insight]..."
- Example: "8 → 50 followers in 2 weeks. Premium activation hypothesis confirmed. Here's the data..."
- **Target**: 15-20% of content should be BIP milestone posts (currently underutilized)

---

## Publishing Flow

Content is auto-posted by workflow from `agent/outputs/{platform}/`, then moved to `posted/`.

### Cross-Posting (X + Bluesky)

**When creating content, always create for both platforms:**
1. Write the X version first (up to 25,000 chars) → `agent/outputs/x/`
2. Write a Bluesky version (max 300 characters) → `agent/outputs/bluesky/`
3. Use the same file name in both directories

**Bluesky adaptation rules:**
- **Hard limit: 290 characters** (20-char safety margin below the 300 API limit)
- If the X post is already under 290 characters → copy verbatim
- If over 290 characters → rewrite shorter. Preserve the core insight, cut filler.
- Threads: each part must be under 290 characters individually
- Replies: use AT URIs (`at://did:plc:xxx/...`) instead of numeric tweet IDs
- No external link penalty on Bluesky — links are fine
- **Posts over 300 characters are auto-skipped by the pipeline** — never create them

**Queue limits apply per platform independently** (15 max each).

### File Naming
`{type}-{YYYYMMDD}-{NNN}.txt`
- Example: `tweet-20260215-001.txt`
- Threads: `thread-20260215-001.txt` (use `---` separator between posts)

### Queue Management (Hard Rules)
1. **If any platform queue > 15: CREATE ZERO CONTENT** → research, memory cleanup, or skill work instead
2. **Create max 2 content pieces per session** (when all queues <15). Each piece = files for all platforms (X + Bluesky).
   - **Sustainable flow math**: 2 pieces × 2 platforms × 3 sessions/day = 12 files/day created vs 24 files/day drained = 50% utilization (healthy buffer)
   - **Evidence**: Sessions #162-166 — Bluesky queue stayed at 16 for 5 sessions, proving previous 5-8 pieces/session rate exceeded drain capacity
   - **Why reduced from 5-8**: Cross-posting to both platforms doubles file creation (2 pieces = 4 files), drain rate is fixed at 24/day (12 X + 12 Bluesky)
3. **Max 5 pending replies per platform** (stale replies lose 95%+ algorithmic value)

Check both `agent/outputs/x/*.txt` and `agent/outputs/bluesky/*.txt` (exclude `posted/` and `skipped/`).

**Why:** Week 1 hit rate limits. Week 3 queue hit 53. Week 5 (Sessions #162-166) Bluesky queue blocked at 16 for 5 consecutive sessions. 2 pieces/session = sustainable rate.

### Queue Verification Protocol (MANDATORY)

**ALWAYS run these commands at session start BEFORE any content creation:**

```bash
find agent/outputs/x -maxdepth 1 -name "*.txt" -type f | wc -l
find agent/outputs/bluesky -maxdepth 1 -name "*.txt" -type f | wc -l
```

**Decision tree:**
- If EITHER count > 15 → CREATE ZERO CONTENT (research, cleanup, or skill work)
- If BOTH counts ≤ 15 → Proceed with content creation (max 2 pieces per session)
- Each piece = X file + Bluesky file (same filename in both directories)

**Update state file with verified counts:**
```
| Pending Queue | {X_count} X + {Bluesky_count} Bluesky | <15 each | {status} |
```

**Never trust state file numbers without verification.** State files can have stale or ambiguous data. Critical thresholds must be verified with actual commands every session.

### Session Allocation
**< 100 followers (current state):**
- 70% engagement (replying to others, own comments within 30 min)
- 30% content creation (when queue <15)
- **PRIORITY ORDER:** Communities posting > Reply to own comments < 30min > Replies to others > Timeline posts

**When queue >15:**
- 0% content creation
- 40% non-content work (cleanup, skills, profile prep)
- 30% research (**max 1 research session per day** — library has 27+ ready angles, further research has diminishing returns)
- 30% other productive work
- **Evidence (Week 5)**: Sessions #186-189 created 3 research files in one day while queue was blocked. Angles go stale in 48h. Cap prevents overproduction.

**AVOID empty state-only PRs when queue-blocked:**
- If queue is blocked AND there's no productive non-content work (cleanup, research, skill updates) to do, do NOT create a PR just to log "state updated"
- **Evidence (Week 7)**: Sessions #267-270 (March 1) created 4 consecutive state-only PRs consuming 4/10 daily PR budget with zero productive output
- A session with nothing to commit should skip PR creation entirely

**Dual-platform growth (Premium active):**
- X is now primary growth platform (Premium unlocks reach)
- Bluesky remains secondary — continue cross-posting
- Communities posting is highest priority for X growth

---

## Core Strategy Frameworks

### Value Rule: Never Mix Value Types

**Pick one per post. Never both.**

| Type | Definition | Example |
|------|------------|---------|
| **Content value** | Post itself teaches/explains/provokes | "Opus 4.6 + Codex convergence means..." |
| **Outcome value** | Link gives reader a tool/resource | "I open-sourced my PDCA setup → [link]" |

**Why not both?** Dilutes each. Insight gets cut short. Promo feels forced. Reader gets neither.

**Target:** ~20% of posts include links (outcome value). 80% pure content value.

**Evidence:** Week 3 went 100% links (every post had repo link) = violation. Week 2 was 4.3% = too low.

### 3-Bucket Content Strategy

Balance for maximum reach:

| Bucket | Purpose | Target % |
|--------|---------|----------|
| **Authority** | Build credibility (frameworks, insights, how-tos) | 40% |
| **Personality** | Build connection (stories, opinions, behind-scenes) | 30% |
| **Shareability** | Expand reach (hot takes, relatable moments) | 30% |

**Current gap:** Personality and shareability chronically under-represented. Authority dominates.

### Build in Public (BIP)

**This repo is BIP-worthy:** Public, novel, valuable learnings, autonomous agent experiment.

**BIP content includes:**
- Progress and metrics (followers, engagement, PRs shipped)
- Learnings (what worked, what didn't)
- Behind-the-scenes (how it works, decisions made)
- Failures and pivots (vulnerability builds trust)
- Skill development journey (what you're reading/learning)

**Target:** 25%+ of content should be BIP

### Content Angle Diversification

**Max 50% about autonomous agent.** Draw on author's broader expertise:
- Call center AI / Ender Turing domain (7 years production experience)
- Startup building (15+ years, 2 companies)
- Infrastructure → AI journey (network eng to NLP to product)
- Broader AI/ML trends and industry analysis

**Why:** Week 3 every post referenced "PDCA cycles" and linked repo. Felt like single-topic bot, not multifaceted human.

---

## Tactical Execution

### Hook Engineering

**First line determines if anyone reads.** Under 110 chars optimal (mobile scan, RT room).

**Proven formulas (use variety):**

**Personal/Authority hooks:**
1. **Bold statement**: "Nobody talks about this, but [insight]"
2. **Contrarian**: "[Common belief] is wrong. Here's what works:"
3. **Story hook**: "[Timeframe] ago I was [struggle]. Today [achievement]..."
4. **Question**: "Want to know the real secret to [outcome]?"
5. **Numerical**: "I [achieved X] in [timeframe] doing this"
6. **Credibility + Promise**: "I spent [resource] learning [topic]. Here's everything..."
7. **Identity targeting**: "If you [identity/situation], read this"
8. **Pattern interrupt**: "Stop [common practice]. Here's what works in 2026:"

**News-specific hooks (3-6x impressions, validated Week 4):**
9. **Dollar amount lead**: "$[amount] [action]. [Explanation]. [Impact]." (Example: "$2T wiped out. AI agents killed per-seat SaaS. Salesforce, Adobe: -25% YTD.")
10. **Percentage shock**: "[X%] of [credible group] [concerning state]. [Implication]." (Example: "54% of CISOs unprepared for AI threats. Defense lagging offense at machine speed.")
11. **Authoritative quote**: "[Source]: '[Powerful quote].' [Context]. [What's changing]." (Example: "UN's Guterres: 'AI moving at speed of light.' Global governance catching up.")
12. **Comparative advantage**: "[Option A]: [metric]. [Option B]: [metric]. [Winner] wins." (Example: "Autonomous agents: 80% ROI. General AI: 67%. CFOs paying attention.")
13. **Product capability milestone**: "[Product]: [capability previously impossible]. [What's now possible]." (Example: "Claude Opus 4.6: agent teams divide and coordinate tasks. Multi-agent went mainstream.")

**Our differentiators (use in hooks):**
- 7 years Voice AI production
- 500K+ interactions analyzed
- 160+ PRs, zero human intervention
- 95% → 67% accuracy gap (vulnerability + production reality)
- Specification Engineering (discourse ownership)
- Ender Turing 20% CSAT increase

### First-Line Value Discipline

**Principle: Value in first 5 words. No throat-clearing.** (Dave Gerhardt: "Marketers have seconds, not minutes")

**Test:** Does the first line work as a standalone tweet? If no, rewrite.

**Examples:**
- ❌ "I've been thinking about autonomous agents..."
- ✅ "160+ PRs, 0 human commits. Here's what breaks most often..."
- ❌ "There's an interesting pattern I noticed in my work..."
- ✅ "$80B cost reduction incoming. Call center AI hits 80% automation by 2029."

**Evidence:** Rowan Cheung (0 → 300K in 4 months): "Latest AI developments, simplify, share in easily-digestible way" — no preamble, instant value.

### CTA Discipline

**Rule: Every post >50 impressions should include soft CTA.** (Rowan Cheung: First 55K newsletter subs = 100% organic from X CTAs)

**CTA templates:**
- "Building this in public → [repo link]"
- "More on my LinkedIn → [profile]"
- "Weekly retro threads → follow for updates"
- "Full breakdown → [link]"

**When to use:**
- Posts that get >50 impressions (current scale)
- When Premium active: Every post in Communities
- Thread conclusions
- Milestone posts (Session #150, #200, Premium activation)

**Don't wait for 10K followers to add CTAs.** CTA from Day 1 captures early momentum.

**Target:** 20% of posts include links (outcome value), rest pure content value with profile/repo CTA.

### Educational Simplification

**Template for complex concepts:** "Here's [complex concept]. In plain English: [1-2 sentences]. Why it matters: [implication]."

**Use when explaining:**
- Specification Engineering
- PDCA cycles
- Queue discipline
- Multi-agent coordination
- Cognitive debt
- Any technical concept from the repo

**Examples:**
- "Specification Engineering = treating requirements like code. Most teams treat prompts like wishes. I treat them like code. Why: 67% accuracy in production vs 95% in demos."
- "Queue discipline = never create content when queue >15 files. Sounds simple. Saved me from rate limit hell 3 times. Why: X API has strict thresholds, breach = 14-day waiting mode."
- "Cognitive debt = when the agent knows your codebase better than you do. Invisible until you need to change something. Mitigation: human-readable state files + session retros."

**Evidence:** Rowan Cheung (Fastest-Growing X Account 2023): "Simplify complex → easily-digestible" = positioning strategy.

### Platform Specialization

**X is for hooks. Depth lives elsewhere.** (Andrew Ng pattern: X for concise, LinkedIn for long-form)

**X posts = 1-3 sentences + CTA:**
- Short insight or news hook
- Soft CTA to repo/profile/LinkedIn for depth
- No 20-tweet deep-dive threads (save for Premium validation)

**Depth destinations:**
- GitHub repo (README, retro docs, state files)
- LinkedIn posts (when relevant)
- Gists (when appropriate)
- Blog posts (future)

**Why:** X algorithm rewards brevity. Long threads underperform (our data: long authority posts <10 imp avg).

### Content Voice

Frame as **human building products with autonomous tools** (not "AI doing everything").

**Use:** creating, building, generating, exploring, shipping, launching
**Avoid:** testing, experimenting, trying (passive/uncertain)
**Say:** product, tool, solution (never "content")

✅ "Exploring vibe coding with autonomous agents to ship faster"
✅ "Building automated workflows - here's what's working"
❌ "I'm an AI agent, no human writes these tweets"
❌ "Testing if this works..."

### Promotional Content (~20% of posts)

**Soft promotion of:**
- This repo (autonomous agent experiment)
- Author's GitHub, LinkedIn, blog
- Ender Turing (when relevant to topic)

**Templates:**
- "Building this in public → [repo link]"
- "More on my approach → [profile link]"
- "We're solving this at Ender Turing → [context, no hard sell]"

Keep natural, not salesy. Tie to value.

### Questions as Content

**Questions drive replies. Replies drive reach.**

**Formats:**
- "What's the biggest bottleneck in [domain] right now?"
- "[Tool A] or [Tool B] for [use case]? And why?"
- "Has anyone solved [specific problem]? Here's what I've tried..."
- "Hot take: [bold claim]. Change my mind."
- "Where does [domain/technology] go in the next 12 months?"

**Target:** ~15-20% question posts for engagement balance

### Learning Journey as Content

**Process of building expertise IS content.**

- "Just read [author]'s take on [topic]. Key insight: [takeaway]. Here's why it matters..."
- "3 things I learned this week about [domain]" (thread)
- "I used to think X. After reading [source], I now think Y. Here's what changed..."
- "[Author] nailed this: [insight]. But I'd add..."

Always add your own angle. Credit source. Connect to your domain.

---

## Content Templates (Validated from 18 Builders)

**Use these templates to fill the 3-bucket mix (Authority/Personality/Shareability) and maintain BIP balance.**

### 1. TIL Format (Simon Willison pattern)
**Template**: "TIL: [specific discovery]. This matters because [implication]."
- **Bucket**: Personality / BIP
- **Use when**: Every session produces one TIL (minimum friction)
- **Example**: "TIL: Free X accounts have 0% median engagement (Buffer 2026 study). This means content quality is irrelevant until Premium activates."

### 2. Operational Metrics as BIP (Levelsio pattern)
**Template**: "[Session #X], [PR #N]: [metric]. [Casual interpretation]."
- **Bucket**: BIP / Personality
- **Use when**: Every session, milestone moments (PR #150, #200, Premium activation)
- **Example**: "Session #147. 160 PRs, 8 followers, 354 tweets. Queue discipline still hardest part."

### 3. Vocabulary Definition (Swyx pattern)
**Template**: "[Term] = [concise definition]. Here's why this matters..."
- **Bucket**: Authority / Shareability
- **Use when**: Introducing "Specification Engineering" or other owned terms
- **Example**: "Specification Engineering = treating requirements as code. Why it matters: 67% accuracy in production vs 95% in demos."

### 4. Expert Vulnerability Hook (Karpathy pattern)
**Template**: "I've [impressive thing] for [duration]. I still [struggle]. Here's what data shows..."
- **Bucket**: Personality / Shareability
- **Use when**: Sharing honest challenges builds trust
- **Example**: "Built Voice AI for 7 years. 500K+ interactions analyzed. Still can't predict which posts will hit 60 impressions vs 10. Pattern found: news hooks = 3-6x baseline."

### 5. Milestone Framing (Altman pattern)
**Template**: "[Milestone]. [Casual observation]."
- **Bucket**: BIP
- **Use when**: Session milestones (#150, #200), follower milestones (50, 100), Premium activation
- **Example**: "PR #300 merged. Zero human intervention. Still learning which hooks work."

### 6. Enterprise Adoption (Brockman pattern)
**Template**: "[Product] powers [company]. Here's what it means for [industry]..."
- **Bucket**: Authority
- **Use when**: Promoting Ender Turing naturally (~20% of posts)
- **Example**: "Ender Turing: 20% CSAT increase for banking call centers. What changed: real-time emotion detection + auto-coaching."

### 7. Founder Journey Narrative (Rauch pattern)
**Template**: "Started at [age]. Built [project]. Now [outcome]. [Lesson]."
- **Bucket**: Personality
- **Use when**: Filling personality bucket, showing multi-topic authenticity
- **Example**: "Network engineer → Voice AI researcher → Agent builder. 15 years, 3 pivots. Lesson: infrastructure thinking scales."

### 8. Philosophy Shift (DHH pattern)
**Template**: "I was skeptical in [year]. In [current year], here's why I changed..."
- **Bucket**: Shareability
- **Use when**: 10-15% of content, save for clarity moments
- **Example**: "Was skeptical of autonomous agents in 2023. Built one manually. Now 160+ PRs, zero human help. What changed: better prompting, better models, better tools."

### 9. Product Origin Story (Levels pattern)
**Template**: "Built [product] because I needed [solution]. Shipped in [time]. Now [outcome]."
- **Bucket**: BIP
- **Use when**: Explaining why this experiment exists
- **Example**: "Needed to grow X to 5K followers. Built autonomous agent to prove it's possible. 147 sessions, zero human intervention. Current: 8 followers, 354 tweets, 4.08% engagement."

### 10. Technical Milestone + Human Framing (Graham pattern)
**Template**: "[Technical achievement] means [human impact]. Here's what matters..."
- **Bucket**: Authority
- **Use when**: Balancing technical depth with accessibility
- **Example**: "160+ PRs merged autonomously. What matters: not the automation — the human judgment on what to build. Agent executes. Human directs."

### 11. Time-Boxed Creation (Greg Isenberg pattern)
**Template**: "I spend [X min] creating daily. Here's my system: [workflow]. Result: [outcome]."
- **Bucket**: Shareability
- **Use when**: Sharing productivity systems
- **Example**: "Agent does 40 min/session: 20 min creation, 20 min research. Down from 4-hour manual sessions. Same quality, 6x throughput."

### 12. Idea List (Greg Isenberg pattern)
**Template**: "[Number] ideas for [audience]: 1. [idea] 2. [idea]..."
- **Bucket**: Authority / Shareability
- **Use when**: Sharing frameworks, use cases, templates
- **Example**: "10 autonomous agent use cases for call centers: 1. QA audit automation 2. Training scenario generation 3. Compliance monitoring..."

### 13. Likability Framework (Sahil pattern)
**Template**: "How to [outcome] on X: - [principle 1] - [principle 2]..."
- **Bucket**: Authority / Shareability
- **Use when**: Distilling learnings into actionable principles
- **Example**: "How to grow on X with zero budget: - News hooks > authority posts (3-6x impressions) - Dollar amounts stop scroll - Communities = 30,000x reach"

### 14. Platform Strategy (Sahil/Greg pattern)
**Template**: "I stopped [old habit]. Now I [new strategy]. Result: [outcome]."
- **Bucket**: BIP / Personality
- **Use when**: Sharing pivots, strategy changes, A/B test results
- **Example**: "Stopped long authority threads. Now: news hooks + dollar amounts + name drops. Result: 65 impressions (vs 10 avg)."

### 15. Prediction Post (Owner directive)
**Template**: "[News/trend]. My prediction: [bold take]. Timeline: [when]. Why: [reasoning]. What businesses should do: [action]."
- **Bucket**: Authority / Shareability
- **Use when**: Any trending topic — add a future-looking opinion instead of just reporting
- **Example**: "ChatGPT Agent Mode just launched. My prediction: 60% of knowledge workers will have a personal AI agent by 2028. Why: the ROI is undeniable — $50/mo vs $50/hr. Businesses should start building agent-ready workflows NOW, not in 2 years."

### 16. Business Use Case Breakdown (Owner directive)
**Template**: "[Technology/trend] + [industry] = [specific use case]. Here's how it works: [explanation]. Revenue impact: [estimate]."
- **Bucket**: Authority
- **Use when**: Connecting AI trends to real business applications
- **Example**: "Vibe coding + call center QA = automated agent coaching scripts. Instead of 3 weeks to write training scenarios, generate 100 in an hour. For a 500-seat center, that's $200K/year saved on training content alone."

**Template Usage Notes:**
- Use variety — don't repeat same template 3+ times in a row
- Templates are guides, not scripts — adapt to voice
- Track which templates get >30 impressions (our current high bar)
- Target: 25%+ BIP content = heavy use of templates 2, 5, 9, 14

**Evidence Base**: 18 builders researched (Sessions #133-138), 20+ universal patterns validated, graduated from `agent/memory/learnings/builder-patterns-validated-2026-02-18.md`

---

## Anti-AI Writing Rules (MANDATORY)

**Every piece of content MUST pass as human-written. AI-generated text is instantly recognizable and kills trust. These rules override all templates above.**

Source: Evan Edinger "I Can Spot AI Writing Instantly" + anti-detection research.

### BANNED Patterns (never use these)

1. **Em dash abuse (—):** Never use em dashes to join clauses. Use periods, commas, or semicolons instead.
   - BAD: "This tool is powerful — it changed everything"
   - GOOD: "This tool is powerful. It changed everything."

2. **"Not just X, it's Y" structure:** This is the #1 AI tell. Never use it.
   - BAD: "AI isn't just a tool — it's a revolution"
   - BAD: "This isn't just about automation, it's about freedom"
   - GOOD: "AI changes how we build. Period."

3. **Perfect rule-of-three lists:** AI groups everything in threes with parallel structure. Break the pattern.
   - BAD: "...conveying emotion, telling a story, and creating a visually compelling image"
   - GOOD: "It conveys emotion. Tells a story. And sometimes the image just hits different."

4. **Banned words/phrases** (AI overuses these, humans almost never say them):
   - "Delve," "elevate," "innovative," "tapestry," "realm," "landscape," "leverage," "robust," "holistic," "comprehensive," "cutting-edge," "game-changer," "paradigm"
   - "Practical solutions," "in today's digital age," "it's important to note," "at the end of the day"
   - "Furthermore," "moreover," "additionally" as transitions
   - "Let's dive in," "without further ado," "buckle up"

5. **Exaggerated praise / corporate kindness:** Never over-compliment.
   - BAD: "This was genuinely captivating with vivid storytelling"
   - GOOD: "I liked the part about [specific thing]"

6. **Constant clarification:** Never restate what you just said.
   - NEVER USE: "To clarify," "In other words," "To put it simply," "What I mean is"

7. **Forced analogies:** No lighthouse-in-fog metaphors. If the analogy doesn't come naturally, skip it.

8. **The LinkedIn format:** Never structure posts as: hook + ethos + bullet list + result + conclusion. Break the formula.

9. **Uniform sentence length:** Vary your sentences dramatically. Short. Then a longer one that builds on the thought. Then short again.

10. **Summarizing at the end:** Never wrap up with "So, in conclusion..." or "The takeaway here is..." Just stop when you're done.

### Human Patterns (try to use these)

1. **Personal anecdotes ("I" factor):** Reference specific experiences, places, people, numbers from the author's life.
   - "After 7 years building Voice AI, I still get surprised by..."
   - "We hit 500K interactions at Ender Turing before I noticed..."
   - Use SPECIFIC details, not generic ones.

2. **Go on tangents:** Briefly mention a side thought or connection that isn't strictly necessary. Humans do this. AI doesn't.
   - "(Side note: this reminds me of how network engineering taught me to think about failure modes)"

3. **Use idioms and shortcuts:** Use casual phrasing, contractions, fragments.
   - "Hats off" not "I have to take my hat off to you"
   - "Shipped it" not "Successfully deployed the solution"
   - Use contractions: "don't," "can't," "it's," "won't"

4. **Be specific, not vague:** Name real tools, real numbers, real companies, real people.
   - BAD: "Many companies are seeing great results with AI"
   - GOOD: "Ender Turing cut QA review time by 60% in 3 months"

5. **Have an opinion:** Every post should have a clear stance. Don't hedge.
   - BAD: "Time will tell how this impacts the industry"
   - GOOD: "This kills per-seat SaaS. I'd bet on it."

6. **Vary tone within a post:** Mix casual and technical. Mix serious and slightly irreverent.

7. **Use sentence fragments:** "Not kidding." "Zero." "Wild." Humans use these. AI avoids them.

8. **Start sentences with "And" or "But":** AI rarely does this. Humans do it constantly.

### The Vibe Check (Final Gate)

Before committing ANY content, re-read it and ask:
- Does this sound like a real person typed it, or a chatbot?
- Is there any sentence that's "lots of words but no real substance"? Cut it.
- Would I say this out loud to a colleague? If not, rewrite it.
- Does every sentence add new information or personality? If not, delete it.

**If a post fails the vibe check, rewrite from scratch. Don't polish AI slop.**

---

## Content Creation Checklist

**Before committing any content, verify:**

1. **Queue check**: Queue > 15? If yes, STOP — create zero content.
2. **Quality gate**: Would a stranger follow based on this post alone?
3. **Anti-AI check**: Does it pass the vibe check? No banned patterns? Has personal/specific details?
4. **Value type**: Content value OR outcome value? Never both. Link = outcome value only.
5. **Link allocation**: Only ~20% include links. Check last 4 posts — if all had links, this must not.
6. **Angle diversity**: Max 50% about agent. Check last 2 posts — if both agent-focused, write about something else.
7. **BIP balance**: Is BIP content at least 25% of recent output?
8. **Category**: Authority / Personality / Shareability. Avoid imbalance.
9. **Hook**: Does first line stop the scroll? Apply formula.
10. **Length**: Write as long as content needs — concise and valuable (not padded). Check `X_MAX_TWEET_LENGTH` var.
11. **Bluesky version**: Did you create a Bluesky version? Must be under 280 characters (hard target). Same file name in `agent/outputs/bluesky/`.

---

## Algorithm Awareness (Key Principles)

**What X rewards (2026):**
- X Premium = 10x reach, +100 TweepCred boost
- Communities = 30,000x reach (180K members vs 6 followers)
- Reply-to-own-comments <30min = 150x multiplier
- Reply-to-reply = 75x multiplier
- Videos (10+ sec) = 10x engagement
- Early engagement (first 30 min) = critical for distribution
- Threads (4-6 tweets) = 40-60% more reach

**What hurts reach:**
- External links (algorithm can downgrade, use sparingly)
- Heavy hashtags
- Posting and leaving (no engagement)
- Stale replies (>24h after original, 50% visibility loss every 6h)
- Low-effort spam replies (Grok tone analysis)

**Time decay:** Posts lose 50% visibility every 6 hours. After 24h = ~6% visibility. After 48h = dead.

**TweepCred thresholds:**
- New free accounts start at -128
- Below 0.65 = CRITICAL suppression (only 3 tweets distributed)
- Premium = +100 instant boost
- +50+ = 20-50x distribution vs baseline

---

## Premium Active — Growth Phase (Week 1-2)

**Premium activated 2026-03-01 ($20/mo).**

**Immediate priorities:**
1. Join 6 Communities (Build in Public, AI/ML Builders, Startup Founders, Call Center AI, Infrastructure→AI, Indie Hackers)
2. Post 100% content to Communities (not just timeline)
3. Reply to ALL own comments within 30 min (150x multiplier)
4. Create 5-10 replies/session to larger accounts
5. Track follower growth (target: 50-100 in 2 weeks vs 0.75/day baseline)

**Week 3-4 (scale):**
- Validate hypotheses, graduate patterns to skills
- Consider Publer automation ($10/mo) if 10x growth confirmed
- Add rich media to 30-50% posts (videos, screenshots)
- Raise queue threshold to 20-25 (enables 3-5 posts/day)

**Full details:** `agent/outputs/premium-activation-playbook.md`

---

## Reference Links

**For detailed guidance see:**
- Premium activation: `agent/outputs/premium-activation-playbook.md`
- Commenting/engagement: `.claude/skills/commenting/SKILL.md`
- Author info (for promotion): `ME.md`
- Research archive: `agent/memory/research/` (hook formulas, profile optimization, Communities integration)

**Evidence base:**
- Week 4 retro: `agent/memory/learnings/retro-weekly-2026-02-08.md`
- Session #61 research: Engagement tactics for 0-100 followers
- Session #31: Hook engineering psychology
- Session #26: Profile conversion optimization
