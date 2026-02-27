---
name: cold-email
description: Generate cold emails for B2B personas. Use when asked to write cold outreach, sales emails, or prospect messaging. Supports 19 persona archetypes (Founder-CEO, CTO, VP Engineering, CIO, CPO, Product Directors, VP CX, Head of Support, Support Ops, DevRel, Head of Docs, Technical Writer, Head of Community, VP Growth, Head of AI, etc.). Can generate first-touch and follow-up emails. When a LinkedIn profile URL is provided, uses Crustdata MCP to enrich prospect data (name, title, company, career history, recent posts) for deep personalization.
---

# Cold Email Generation

Generate high-quality cold emails tailored to specific B2B personas, using evidence-backed messaging strategies.

## Workflow

1. **Parse the request**
   - Identify the target persona (see Persona Quick Reference below)
   - Extract company context (name, industry, size, any signals like funding, hiring, product launches)
   - Determine email type: first-touch or follow-up (default: first-touch)
   - **If user provides a LinkedIn profile URL**, proceed to step 1b

1b. **Enrich prospect via Crustdata MCP** (when LinkedIn URL provided)
   - Call `mcp__crustdata__enrich_person_by_linkedin` with the LinkedIn URL
   - Extract and use the following for personalization:
     - **Name and title**: Use first name in greeting, title to identify persona
     - **Current company**: Company name, domain, description, funding stage
     - **Career history**: Past employers and roles (useful for "you've scaled teams before" angles)
     - **Education**: Schools and degrees (use sparingly, only if highly relevant)
     - **Skills/languages**: Can inform communication style
   - Call `mcp__crustdata__get_person_linkedin_posts` to find recent posts for personalization hooks
   - Call `mcp__crustdata__enrich_company_by_domain` with their company domain for deeper company intel (headcount, revenue, funding, founders)
   - **Personalization priorities from Crustdata data:**
     1. Recent LinkedIn posts (best hook if they posted about relevant topic)
     2. Current role + company context (product-specific hooks)
     3. Career trajectory (e.g., "Since joining from [previous company]...")
     4. Company growth signals (funding, headcount growth)

2. **Research the company first** (CRITICAL for CS/CX leaders)
   - If Crustdata already provided company data, use it; otherwise use web search
   - Find the company's core products and platform names
   - Identify what their CS/support teams actually manage day-to-day
   - Look for product-specific terminology (e.g., "HealthRules Payer", "RingEX", "Qualtrics XM")
   - Find recent news, integrations, or platform updates
   - This research powers the subject line and hook

3. **Load persona-specific guidance**
   - Read `references/personas.md` for the matching persona archetype
   - Note their pain points, buying behavior, and anti-patterns

4. **Match product capability to persona pain**
   - Read `references/product-intel.md` for Inkeep product context
   - Identify which product pillar (Ask AI, Copilots, Workflows, Build Your Own) solves their problem
   - Select relevant proof point (e.g., "48% ticket reduction" for support, "18% activation" for product)
   - Never lead with product features, lead with outcome, then connect to capability

5. **Select content CTA** (optional but recommended)
   - Read `references/blog-mapping.md` to find relevant articles for this persona
   - Match buying stage: Awareness (cold), Consideration (exploring), Decision (evaluating)
   - For multi-step sequences: Select 2 articles with different angles for emails 2 and 3

6. **Add social proof** (when relevant)
   - Read `references/customer-proof.md` to find industry-matched customers
   - Use 1-2 customer names that match prospect by industry and size

7. **Draft the email**
   - Follow the Email Structure below
   - Apply persona-specific messaging angle
   - Weave in product benefit naturally (not as a pitch)
   - Keep it short (under 100 words for first-touch, under 150 for follow-ups)

8. **Output the email**
   - Provide subject line + body
   - If multiple variants requested, provide 2-3 options

---

## Crustdata MCP Integration

When a LinkedIn profile URL is provided, use Crustdata MCP tools to gather rich prospect data for personalization.

### Available Crustdata Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `mcp__crustdata__enrich_person_by_linkedin` | Get person details: name, title, company, career history, education, skills | Always, when LinkedIn URL provided |
| `mcp__crustdata__get_person_linkedin_posts` | Get last 5 LinkedIn posts with engagement metrics | For personalization hooks based on recent activity |
| `mcp__crustdata__enrich_company_by_domain` | Get company details: revenue, headcount, funding, founders | When company context needed beyond basic info |
| `mcp__crustdata__get_company_linkedin_posts` | Get last 5 company posts | For company news/announcements to reference |

### Data Fields from Person Enrichment

```
person:
  - name, title, headline, location
  - email (if available), twitter_handle
  - summary, profile_picture_url
  - connections count
  - skills, languages

current_employment:
  - company_name, title, location, start_date
  - company_details: linkedin_id, website_domain, logo_url, description

past_employment:
  - company_name, title, start_date, end_date

career_summary:
  - all_titles, all_employers, all_schools, all_degrees
```

### Personalization Strategy by Data Type

| Data Type | How to Use | Example |
|-----------|------------|---------|
| **Recent LinkedIn post** | Reference specific topic they posted about | "Your recent post on AI in support resonated..." |
| **Current title + tenure** | Tailor persona messaging | New in role = quick wins; 2+ years = strategic initiatives |
| **Company description** | Extract product names for subject line | "Qualtrics XM QBR prep" not "faster QBR prep" |
| **Past employers** | Build credibility through shared context | "Since scaling support at [previous co]..." |
| **Company headcount/funding** | Identify growth stage for pain points | Series B = scaling chaos; Enterprise = tool consolidation |
| **Skills/languages** | Inform communication style | Technical skills = can go deeper on architecture |

### Example: LinkedIn URL to Personalized Email

**Input:** `https://www.linkedin.com/in/johndoe`

**Crustdata returns:**
- Name: John Doe
- Title: VP of Customer Success
- Company: Acme Corp (Series B, 200 employees, data analytics platform)
- Recent post: "Just shipped our new dashboard..."
- Past: Director of CS at DataCo

**Personalized hook:**
> "Your team is proving ROI across Acme's analytics deployments while keeping pace with the dashboard updates you just shipped. Turning usage signals, support themes, and stakeholder feedback into renewal-ready briefs still takes manual stitching."

**vs. Generic hook:**
> "I saw you lead Customer Success at Acme Corp." (wastes characters, no insight)

### When Crustdata Data is Limited

If enrichment returns sparse data:
1. Fall back to web search for company research
2. Use title alone to identify persona archetype
3. Focus on company-specific hooks rather than personal hooks
4. Check if `enrich_realtime: true` option provides fresher data (costs more credits)

---

## Persona Quick Reference

| Persona | Key Pain Point | CTA Style |
|---------|---------------|-----------|
| **Founder-CEO** | Growth slowdown, CAC efficiency | Business outcomes, ROI data |
| **CTO / Founder-CTO** | AI adoption, security, tech debt | Technical depth, architecture |
| **VP of Engineering** | Developer productivity (32% coding time) | DORA metrics, team efficiency |
| **CIO / VP IT** | AI strategy, vendor consolidation, security | TCO, compliance, enterprise integration |
| **CPO / VP Product** | Stakeholder conflicts, AI integration | User engagement, feature adoption |
| **Director/Head of Product** | Proving product ROI, alignment | Cross-functional case studies |
| **Senior PM / GPM** | Feature impact measurement | Peer testimonials, frameworks |
| **Technical / Platform PM** | Quantifying infrastructure value | DevEx metrics, architecture |
| **VP of CX/CS** | Proving ROI, NRR protection | Dollar-denominated outcomes |
| **Director of CX/Support** | Organizational silos (73%) | CSAT, FRT improvements |
| **Head of Support** | Knowledge gaps (51%), team capacity | Ticket deflection, agent productivity |
| **Support Ops / CX Ops** | Tool sprawl (81%), integration | API depth, TCO, automation ROI |
| **CSM / Onboarding Manager** | Time-to-Value, burnout | Time savings, automation |
| **Support Team Lead** | Agent productivity, FCR | Quick wins, templates |
| **Head of DevRel** | Proving DevRel ROI, content efficiency | Developer activation metrics |
| **Senior Developer Advocate** | Wearing many hats, content volume | Time savings, peer usage |
| **Junior Developer Advocate** | Career path, credibility, tool overload | Free resources, templates, peer usage |
| **Head of Technical Writing** | Docs going stale (30% SME time) | Freshness, support ticket reduction |
| **Technical Writer (IC)** | Review bottlenecks, SME coordination | Templates, peer testimonials, free trial |
| **Head of Community** | Proving ROI (58%), resources | Engagement, retention impact |
| **VP/Head of Growth** | Lead quality (61%), rising CAC | Activation, conversion data |
| **Head of AI** | POC abandonment (42%), data quality | POC-to-production, governance |
| **Content Creator (Recruiting)** | Budget constraints (8%), video cost | Efficiency, cost vs agency |

---

## Email Structure (First-Touch)

```
Subject: [2-3 words, internal-camo style, no punctuation]

[1 sentence: Personalized observation or trigger]

[1-2 sentences: Problem statement with loss framing or unconsidered need]

[1 sentence: Social proof:"We helped [similar company] [specific outcome] in [timeframe]"]

[1 sentence: Interest-based CTA with optional content offer]
```

**Characteristics:**
- Under 100 words total (guideline, not hard rule)
- 3rd-5th grade reading level
- Plain text, no links in first-touch email
- 2-3 paragraphs, 1 sentence each
- Start with "you/your" not "I/we"
- Specific numbers, named companies, exact timelines
- Use hyphens only for compound words (edge-case, Tier-1, 50-80%). Never use dashes to connect separate thoughts or clauses.
- Never use em dashes. Use commas or periods instead.

**Note:** Follow-up emails (2 and 3) should include blog article links as CTAs. See Follow-Up Email Progression below.

---

## VP/Head of Customer Success Email Structure (Proven Pattern)

For CS leaders (VP CS, Head of CS, SVP CS, Chief Customer Officer), use this research-first structure that has been tested across 30+ companies:

### Subject Line Formula
```
[Company Product Name] QBR prep
```

Examples:
- `Qualtrics XM QBR prep`
- `Mural rollout QBR prep`
- `RingEX and RingCX QBR prep`
- `LeanIX onboarding, faster first value`
- `Vanta Trust Center QBR prep`

The subject line MUST reference their specific product/platform. Generic subjects like "faster QBR prep" or "CS efficiency" fail.

### Email Structure for CS Leaders

```
Hi [First Name],

[1-2 sentences: Company-specific CS challenge with loss framing. Reference their actual products/platform and the manual work involved.]

We build a CSM AI Agent that connects to the systems you already use (CRM, support, call notes, product usage) and can answer in seconds:
• "Which accounts are trending at risk, and why?"
• "What should we cover in the next QBR for <customer>?"
• "Generate a renewal or QBR summary with outcomes, adoption, and open risks."

Open to a quick 15-minute chat next week?

Best,
[Your Name]
[Company]
```

### Hook Patterns for CS Leaders

**Good hooks** (product-specific, loss framing):
- "With [Company] supporting [customer type] across [Product A] and [Product B], I imagine your team spends a lot of time pulling context together before QBRs and renewals."
- "For [product type] customers, the renewal story is usually scattered across [signal 1], [signal 2], and [signal 3]."
- "Turning [platform] activity, [metric], and [data source] into an exec-ready renewal story still takes hours of manual stitching."

**Bad hooks** (generic, title-focused):
- "I saw you lead Customer Success at [Company]." (Wastes characters on their title)
- "Your team probably spends time on manual work." (Not specific to their product)
- "I noticed you're hiring CSMs." (Generic trigger)

### Bullet Customization

Customize the three bullets based on company research:

| Company Type | Risk Signals | Health Snapshot | QBR Content |
|--------------|--------------|-----------------|-------------|
| **Observability/IT** | coverage gaps, noisy alerts, stalled workflows | deployment health, topology gaps, integrations | outcomes, adoption, open risks |
| **Security/Compliance** | coverage gaps, rising vulns, unresolved incidents | risk posture, remediation status | MDR outcomes, risk trendline |
| **Healthcare/Payer** | implementation delays, stakeholder changes | project health, system performance | outcomes delivered, open action items |
| **Data/Analytics** | low activation, stale metadata, failing integrations | data trust health, lineage gaps | adoption trends, coverage |
| **Payments/Fintech** | program performance, fraud/dispute signals | issuer processing health | program outcomes, compliance |

---

## Senior Support Leader Email Structure (VP/SVP/EVP)

For senior support leaders at large technical B2B companies, use this executive micro email format (~80 words, peer-to-peer tone).

### Key Principles

- **Under ~80 words** (executive micro emails)
- **Peer-to-peer tone**, not salesy or marketing copy
- **Plain, credible language**, avoid buzzwords
- **Focus on leverage, capacity, and knowledge reuse**, not features
- **Assume they already have good tooling** and possibly an AI assistant
- **Personalize based on real company context**

### The Three Inkeep Surfaces (Must Connect All Three)

Always connect all three surfaces to the problem in one sentence:

1. **Customer-facing AI assistant** with source-cited answers
2. **Agent Copilot** that analyzes requests and drafts replies with linked sources
3. **Content writer** that turns resolved tickets into KB/docs updates

Example one-liner:
> "Inkeep gives you one shared knowledge layer that powers a cited customer AI assistant, an agent Copilot inside case workflows, and automatic capture of resolved cases into docs."

### Two Problem Framing Angles

**Angle A: "Answers exist but aren't surfaced"**
Use when the company has strong docs/KB/community but cases still open:
> "A lot of support questions are already documented, but the right answer isn't surfaced in the moment, so cases still open and agents re-search."

**Angle B: "Resolution stays in the case"**
Use when the issue is knowledge not propagating after resolution:
> "When a complex case closes, the resolution often stays in the case thread, while agents, customers, and AI tools keep using older guidance."

Choose based on company context. Angle A works better for companies with mature self-serve (KB, community, docs). Angle B works better for companies where escalations are the bottleneck.

### Email Structure for Senior Support Leaders

```
Hi [First Name],

[1-2 sentences: Company-specific hook with real data (integrations count, endpoints, products). State the Support problem concretely.]

[1 sentence: The Inkeep solution connecting all three surfaces to the problem.]

Open to a [10-15] min compare?

Best,
[Your Name]
[Company]
```

### Subject Line Patterns (Company-Specific Required)

**Good (tailored to company):**
- `Snowflake Copilot: keeping answers current`
- `Support leverage across 650+ integrations` (New Relic)
- `Clean Room setup: Snowflake + BigQuery` (LiveRamp)
- `SKAN + OneLink edge cases` (AppsFlyer)
- `When Qlik + Talend case learnings don't travel`

**Bad (generic, applies to any company):**
- `Docs exist. Finding them is the work`
- `When edge-case answers don't stick`
- `faster QBR prep`

### Company Research Hooks

Find 2-4 specific hooks from public sources:

| Hook Type | Examples |
|-----------|----------|
| **Ecosystem scale** | "650+ integrations", "10,000+ partners", "270k news sources" |
| **Deployment models** | "AWS, Azure, and GCP", "cloud + on-prem", "hybrid data estates" |
| **Key workflows** | "SKAN + OneLink setup", "Clean Room connections", "RampID identity resolution" |
| **Their AI assistant** | "Snowflake Copilot", "Alteryx Copilot", "Qlik Answers", "Breeze" |
| **KB/community presence** | "MyAlteryx portal", "Knowledge Center", "large Community" |
| **Compliance/security** | "SOC 2 Type II", "HIPAA-ready", "ransomware resilience" |
| **Global footprint** | "140+ countries", "50,000+ endpoints", "follow-the-sun" |

### Examples (Senior Support Leaders)

**VP, Customer Support & CX Operations at Alteryx**

**Subject:** Keeping Copilot answers current

Hi [First Name],

You already run MyAlteryx for cases, the Knowledge Center, and a large Community. When a hard issue is solved, the final resolution can stay in the case thread, while agents, customers, and Alteryx Copilot keep pulling older guidance.

Inkeep closes that gap by syncing resolved cases into an always-current knowledge layer that powers cited customer answers and an agent Copilot inside your tools.

Open to a 12-min compare?

Best,
Matthew

---

**GVP, Global Technical Support at New Relic**

**Subject:** Support leverage across 650+ integrations

Hi [First Name],

With 650+ integrations and first-class OpenTelemetry, Global Tech Support ends up debugging ingest, attribute mapping, and NRQL edge cases daily. When a tricky case is resolved, the steps often stay in the ticket, while customers and New Relic AI keep pulling older docs.

Inkeep gives you one shared knowledge layer: a cited customer AI assistant, an in-workflow Copilot for agents, and automatic capture of resolved cases into the KB.

Open to 12 min?

Best,
Matthew

---

**Head of Technical Support, NA & Latam at AppsFlyer**

**Subject:** Support across 10,000+ partners

Hi [First Name],

AppsFlyer supports 10,000+ integrated partners plus SKAN and OneLink setup, so NA and Latam teams field a lot of "we followed the doc, still stuck" questions.

Often the answer already exists across docs and past cases, but it is not surfaced fast enough, and real gaps take time to close.

Inkeep surfaces source-cited answers in customer chat and an in-workflow agent Copilot, then drafts KB updates when something is truly new.

Open to a quick compare?

Best,
Matthew

---

## T2/T3 Technical Support Emails

For technical support leaders dealing with escalations, use the "context hunt" angle.

### The T2/T3 Problem

The slow part of T2/T3 is NOT debugging. It's gathering context BEFORE debugging starts:
- Environment/version
- Configs
- Logs and traces
- Similar past cases
- Customer account state

This is the "6-system scavenger hunt" across: Datadog/Splunk, Jira, Slack, Confluence, CRM, product admin tools.

### T2/T3 Email Structure

```
Hi [First Name],

[1 sentence: Company-specific T2/T3 challenge with concrete systems/workflows.]

[1 sentence: The context gathering problem, stated plainly.]

Inkeep gathers that context, suggests the reply with linked sources, and turns new learnings into updated docs, so the next engineer or customer does not have to re-collect it.

Open to a [10-12] min compare?

Best,
[Your Name]
```

### T2/T3 Follow-Up Sequence

**Email 2 (bump):**
```
Hi [First Name],

Quick bump.

On T2/T3, the slow part is often gathering context before debugging even starts: env/version, configs, logs, traces, and similar past cases.

Inkeep gathers that context, suggests the reply with linked sources, and turns new learnings into updated docs, so the next engineer or customer does not have to re-collect it.

Open to a quick 10-12 min compare?

Best,
[Your Name]
```

**Email 3 (measurement angle):**
```
Hi [First Name],

How do you measure the "context gathering" step today?

One simple metric is time from ticket opened to the first helpful reply that includes the key facts and next steps, not just "we're looking."

Inkeep reduces that time by pulling the facts from your systems and drafting the response with sources.

Worth a short chat?

Best,
[Your Name]
```

**Email 4 (breakup):**
```
Hi [First Name],

Last note from me.

If your engineers already get env details, logs, and past-case matches pulled into every escalation automatically, I'll bow out.

If not, that context work repeats on every T2/T3 ticket.

Inkeep gathers the context, suggests the reply with linked sources, and turns new learnings into updated docs, so the same question is easier next time.

Who owns this workflow?

Best,
[Your Name]
```

---

## Customer Success Leader Renewal Emails (Bespoke CSM AI Agents)

For CS leaders (VP CS, Head of CS, SVP CS, CCO) focused on renewals and QBRs, use this "bespoke CSM AI Agent" template.

### Key Differences from Support Leader Emails

- Focus on **renewals, QBRs, and proving customer value**
- Emphasize **"bespoke CSM AI Agents tailored to your playbooks"**
- Connect to **CRM, ticketing, call notes, and product telemetry**
- Three bullets: **risk identification, health snapshot, exec-ready QBR/renewal brief**

### Email Structure

```
Hi [First Name],

[1-2 sentences: Company AI/product initiatives and what it means for CS. Research their specific products and positioning.]

Inkeep builds bespoke CSM AI Agents tailored to your playbooks. They connect to your CRM, ticketing, call notes, and product telemetry to:
• Identify accounts trending at risk and why ([company-specific risk signals])
• Snapshot current health with recommended next steps
• Draft an exec-ready QBR or renewal brief tied to outcomes ([company-specific outcomes])

Open to a quick 15-minute chat next week?

[Your Name]
```

### Subject Line Patterns

**Formula options:**
- `[Company] renewals: [outcome metric], auto-summarized`
- `Before renewals: auto-brief on [outcome] + [outcome]`
- `[Company] CS: renewal risk + value story in 1 brief`
- `CS agent for [Company] renewals`

**Good examples:**
- `Smartly renewals: ROAS + creative velocity, auto-summarized`
- `BMC renewals: MTTR + automation wins, auto-briefed`
- `Before renewals: auto-brief on patch + exposure outcomes` (Tanium)
- `Nooks ROI brief: connects, meetings, pipeline`
- `Atlan: instant account intel for renewals`
- `Aqua renewals: measurable runtime impact, faster`

**Bad (too generic):**
- `QBR-ready renewal brief for CSMs`
- `CS agent for renewals`

### Risk Signals by Industry

| Industry | Risk Signals |
|----------|--------------|
| **Security/DSPM** | coverage gaps, rising vulns, unresolved incidents, stakeholder churn, slow remediation |
| **Data/Observability** | adoption stalls, low marketplace engagement, data gaps, alert fatigue, slow time-to-resolution |
| **IT Ops/AEM** | MTTR rising, automation adoption stalls, noisy events, patch backlog, recurring incidents |
| **Sales Tech** | usage drop, connect rate slide, spam/number-quality issues, rollout stalls |
| **Data Governance** | adoption stalls, access bottlenecks, open support themes, low marketplace engagement |
| **Marketing Tech** | spend drop, pipeline impact, tracking/integration issues, stalled adoption |

### Outcomes by Industry

| Industry | Outcomes to Reference |
|----------|----------------------|
| **Security/DSPM** | risk reduction, faster remediation, tool consolidation, exposures reduced, DDoS readiness |
| **Data/Observability** | data downtime avoided, faster root cause, reliability outcomes, fewer disruptions |
| **IT Ops/AEM** | MTTR reduction, higher availability, automation coverage, faster troubleshooting |
| **Sales Tech** | meetings and pipeline impact, connects, ROI, conversion rates |
| **Data Governance** | trusted data, governed AI, cycle time reduction, fewer manual handoffs |
| **Marketing Tech** | ROAS, creative velocity, performance optimization |

### Examples

**SVP, Customer Success Organization at NETSCOUT**

**Subject:** nGenius + Arbor renewals: exec-ready account brief

Hi Tracy,

NETSCOUT's Visibility Without Borders platform unifies performance, security, and availability, and your VaaS offering sets a high bar for proving outcomes like faster troubleshooting and fewer disruptions.

For CS, the hard part is turning scattered signals (deployment coverage, incident trends, support themes, stakeholder changes) into a clear renewal story before an account goes quiet.

Inkeep builds bespoke CSM AI Agents tailored to your playbooks. They connect to your CRM, ticketing, call notes, and product signals to:
• Identify accounts trending at risk and why
• Generate a current health snapshot plus recommended next steps
• Draft an exec-ready QBR or renewal brief tied to outcomes (availability, MTTR, DDoS readiness)

Open to a quick 15-minute chat next week?

Matt

---

**VP of Customer Success at Varonis**

**Subject:** Varonis renewals: auto-brief on exposures reduced + response outcomes

Hi Linor,

Varonis is leaning hard into automated DSPM that goes beyond visibility to remediate risk, plus MDDR for 24/7 data-centric response. For CS, the challenge is packaging scattered signals (exposures reduced, policy enforcement, detections, adoption gaps) into a crisp renewal story before an account drifts.

Inkeep builds bespoke CSM AI Agents tailored to your playbooks. They connect to your CRM, support, and Varonis telemetry to:
• Identify accounts trending at risk and why
• Snapshot health with recommended next steps
• Draft an exec-ready QBR or renewal brief tied to risk reduction outcomes

Open to a quick 15-minute chat next week?

Matt

---

**Global Head of Customer Success at Monte Carlo**

**Subject:** Renewal brief: data downtime avoided + ROI (auto-drafted)

Hi Pamela,

Monte Carlo is pushing end-to-end data + AI observability, including agents that speed up monitoring and troubleshooting. For CS, that usually means proving impact (less data downtime, faster root cause, broader coverage) before exec reviews and renewals.

Inkeep builds bespoke CSM AI Agents tailored to your playbooks. They connect to your CRM, support, call notes, and Monte Carlo usage and alert signals to:
• Flag renewals trending at risk and why (coverage gaps, alert fatigue, slow time-to-resolution, stakeholder churn)
• Generate a current health snapshot plus recommended next steps
• Draft an exec-ready QBR or renewal brief tied to reliability outcomes

Open to a quick 15-minute chat next week?

Matt

---

## Follow-Up Email Progression

When user requests follow-up emails, follow this arc:

| Position | Type | Purpose | Length |
|----------|------|---------|--------|
| Email 1 | Anchor/Pain + Social Proof | Personalized problem + customer proof + interest CTA | Under 80 words |
| Email 2 | Value + Blog CTA | New insight or stat with relevant blog link | Under 100 words |
| Email 3 | Reframe + Blog CTA | Different angle with relevant blog link | Under 75 words |
| Email 4 | Re-Angle/Pivot | Fresh thread, different problem angle | Under 100 words |
| Email 5 | Value-Add | Useful resource, no ask | Under 75 words |
| Email 6 | Objection Preempt | Address likely reason for silence | Under 100 words |
| Email 7 | Breakup | Gracious close, loss aversion | Under 75 words |

**Blog CTA Guidelines (Emails 2 and 3):**
- Select articles from `references/blog-mapping.md` that match the persona
- Email 2: Use article that adds new insight or reinforces problem framing
- Email 3: Use article with different angle (re-frame the problem)
- Include full URL on its own line for easy clicking
- Frame the article: "This covers [specific insight]:" then link
- Keep the ask soft: "Worth 5 minutes if [pain point] is on your radar"

**Follow-up notes:**
- Email 2 has highest leverage (+49% reply lift)
- Follow-ups can be longer (4+ sentences get 15x more meetings)
- Avoid "I never heard back" (-14% meetings)
- "Hope all is well" works only when personalized to specific event
- Blog links add value without being salesy
- Never use meta-language like "Different angle:", "One stat that stood out:", "Bumping this", or "Here's another way to think about it:". Instead, just open with the new angle or stat directly. Let the content speak for itself.
- Use social proof only once per sequence (typically in Email 1). Repeating customer names across emails signals templated outreach.

---

## Proven 4-Email Sequence (Converted to Demo)

This exact sequence sent to the Head of Global Support at SnapLogic resulted in a demo call. Use this as a template.

### Sequence Strategy

| Email | Job | Angle | CTA Style |
|-------|-----|-------|-----------|
| 1 | Research + Value | Company-specific problem + full Inkeep solution | Strong: "12-min compare" |
| 2 | Pain amplification | Why repeat questions persist (knowledge trapped) | Soft question: "Does that sound familiar?" |
| 3 | Solution angle 1 | Agent speed + assist-first adoption path | Medium: "if exploring this year" |
| 4 | Solution angle 2 | Reactive → proactive + unified platform | Soft: "if relevant now or later" |

### Key Success Factors

1. **Deep company research in Email 1**: Referenced "800+ Snaps", "Groundplexes", "Ultra Tasks"
2. **Progressive story arc**: Knowledge trapped → Agent speed → Proactive support
3. **Each email has ONE job**: No repetition of the same pitch
4. **Decreasing CTA intensity**: Strong → question → soft → softer
5. **All under 80 words**: Respects exec time
6. **No meta-language**: Opens directly with the new angle

### Email 1: Research + Full Value Prop

```
Hi [First Name],

With [company-specific data: products, integrations, scale], Support sees a long tail of [specific issue types].

Fix is usually in a prior case or doc, but agents still search, then pull [specific data sources], and the KB update comes later.

Inkeep delivers cited customer answers, drafts agent replies with linked sources, and turns solved cases into docs, with real-time [product-specific] lookups and actions.

Open to a 12-min compare?

Best,
[Your Name]
```

### Email 2: Pain Amplification (Engagement Question)

```
Hi [First Name],

One reason repeat questions do not go away is that support knowledge gets trapped.

Answers live in closed tickets, macros, or internal notes instead of the help center, so the same issues keep resurfacing.

Does that sound familiar at all?

Best,
[Your Name]
```

**Why this works**: The question "Does that sound familiar at all?" invites engagement without requiring commitment. It's a pattern interrupt that feels conversational, not salesy.

### Email 3: Agent Assist Angle

```
Hi [First Name],

Once teams start fixing their knowledge flow, the next bottleneck is agent speed.

Inkeep integrates with tools like [their ticketing system] to analyze incoming tickets and surface relevant answers from docs and past tickets while agents are responding.

Teams often use this first as agent assist before moving to automated replies.

Open to a short conversation if this is something you are exploring this year.

Best,
[Your Name]
```

**Why this works**: Shows a progressive adoption path (assist first, then automate). Reduces perceived risk. Names their actual tool (Zendesk, Salesforce, etc.).

### Email 4: Proactive Support Angle

```
Hi [First Name],

The last step many teams take is shifting from reactive to proactive support.

With Inkeep, teams can automatically respond to common questions and proactively surface answers in docs or product flows before users submit tickets.

All of this runs on the same AI agent foundation, so teams do not need separate tools for each workflow.

If this is relevant now or later, happy to connect.

Best,
[Your Name]
```

**Why this works**: Introduces the full vision (proactive) while emphasizing "same foundation" (no tool sprawl). The softest CTA leaves the door open without pressure.

### Adapting This Sequence

When customizing for a new prospect:

**Email 1 customizations:**
- Replace product data (800+ Snaps → their equivalent)
- Replace issue types (connector auth → their common escalations)
- Replace data sources (run logs, task status → their monitoring tools)

**Email 3 customizations:**
- Replace ticketing system (Zendesk → Salesforce, ServiceNow, etc.)
- Keep the "assist first, then automate" framing

**Email 4:**
- Usually works as-is since it's about the broader vision

---

## CTA Patterns by Level

| Level | CTA Approach |
|-------|-------------|
| **Executive (VP+)** | "Worth a quick 15-minute chat?" / "Mind if I send a 2-min Loom?" |
| **Director/Manager** | "See how [similar company] achieved X" / "Happy to share our benchmark" |
| **IC/Individual Contributor** | "Try free" / "Here's a template you can use today" |
| **Technical roles** | "Technical deep-dive available" / "See our API docs" |

**Interest-based CTAs outperform meeting requests 2x** (30% vs 15% response rate).

---

## Anti-Patterns (What Kills Replies)

**Never do:**
- "Quick chat" / "Quick call" (trivializes their time)
- "Just following up" (no new value)
- Generic "I hope this finds you well"
- "We're a leading provider of..." (template smell)
- ROI claims without context (-15% success rate)
- Pitching your product first (-57% reply rate)
- Multiple CTAs (one ask per email)
- Wall of text (no paragraphs)
- Over 125 words on first touch
- Meta-language that signals templated outreach ("Different angle:", "One stat that stood out:", "Bumping this", "Circling back", "Following up on my last email", "Here's another way to think about it:")
- Sales-speak that reveals you're analyzing across prospects ("One pattern we see:", "What we're hearing from teams like yours", "A trend we've noticed")
- Em dashes anywhere in the email (use commas or periods instead)
- Opening with their title ("I saw you lead Customer Success at..." wastes characters and doesn't catch attention)
- Generic subject lines that apply to any company ("faster QBR prep" instead of "Qualtrics XM QBR prep")

**Template smell checklist:**
- Starts with "I/My/We/Our"
- Contains buzzwords ("innovative", "cutting-edge", "all-in-one")
- Includes rounded numbers ("save 40%") instead of specific ("save 37%")
- Has generic social proof ("leading companies")
- Asks for meeting before establishing value
- Reads above 8th grade level
- Subject line could apply to any company (not product-specific)
- Opens with recipient's job title

**Clarity anti-patterns (vague phrases to avoid):**

| Vague Phrase | Problem | Clearer Version |
|--------------|---------|-----------------|
| "reusable guidance" | Guidance for who? What type? | "answers that agents, customers, and AI all pull from" |
| "the fix lives in the ticket" | Abstract, forces reader to translate | "the resolution stays in the case thread" or "the steps stay in ticket notes" |
| "decision path" | Jargon | "troubleshooting steps" or "resolution logic" |
| "cost-to-serve repeats" | Abstract business-speak | "the same senior triage repeats across regions" |
| "so it sticks" | Unclear what "it" is | "so the same question is easier next time" |
| "that" without clear referent | Forces reader to guess | Name the thing explicitly |
| "so your team starts on the real problem" | What is the real problem? | "so your team can start debugging instead of hunting for context" |
| "the thinking work" | Too abstract | "the same investigation" or "the same troubleshooting" |

**Self-check:** If a sentence uses "that", "it", or "this" without a clear referent in the same sentence, rewrite it.

---

## Examples

### Good (VP of CX)

**Subject:** Support deflection

Noticed [Company] is scaling fast. Congrats on the Series B.

Most CX teams at this stage see ticket volume outpace headcount 3:1. The ones avoiding burnout are deflecting 40-60% with AI that actually understands technical docs.

Fingerprint cut tickets 48% while increasing activation 18%. Worth a quick look at how?

---

### Good (Head of DevRel)

**Subject:** Docs activation

Saw your talk at [Conference] on developer onboarding friction.

Most DevRel teams spend 50%+ on content creation but struggle to prove impact on activation. The gap is usually between "docs exist" and "developers find answers."

Solana scaled developer support without adding headcount. Happy to share their approach if useful.

---

### Good (Head of CS at Qualtrics)

**Subject:** Qualtrics XM QBR prep

Hi Charlie,

Your team is accountable for proving ROI and driving usage and adoption across large XM deployments, which usually means a lot of manual work to prep exec readouts from utilization, surveys, users, support history, and open action items.

We build a CSM AI Agent that connects to the systems you already use (CRM, support, call notes, product usage) and can answer in seconds:
• "Which enterprise accounts are trending at risk, and why?"
• "What should we cover in the next XM QBR for <customer> based on adoption and outcomes?"
• "Generate a renewal brief with results, usage trends, and open issues."

Open to a quick 15-minute chat next week?

Best,
Matt Plotkin
Inkeep

---

### Good (Head of CS at Meltwater)

**Subject:** social listening QBR prep

Hi Ana,

With teams using Meltwater for media intelligence plus social listening and reporting, your CSMs spend a lot of time pulling the full account picture together before QBRs, renewals, and escalations.

We build a CSM AI Agent that connects to the systems you already use (CRM, support, call notes, product usage) and can answer in seconds:
• "Which accounts are trending at risk, and why?"
• "What should we cover in the next QBR for <customer> based on usage and outcomes?"
• "Generate a renewal or QBR summary with results, adoption, and open issues."

Open to a quick 15-minute chat next week?

Best,
Matt Plotkin
Inkeep

---

### Good (VP of CS at Arctic Wolf)

**Subject:** Concierge Security QBR prep

Hi Kyle,

With Arctic Wolf's Concierge Security Team, customers get 24x7 monitoring plus ongoing risk posture reviews and remediation guidance. Turning that MDR plus Managed Risk work into a clean exec story for QBRs and renewals still takes a lot of manual stitching.

We build a CSM AI Agent that connects to the systems you already use (CRM, ticketing, call notes, platform telemetry) and can answer in seconds:
• "Which accounts look renewal risk, and why (coverage gaps, open risks, recent incidents)?"
• "For <customer>, what's the current risk posture and what remediation is blocked?"
• "Draft an exec-ready QBR/renewal brief with MDR outcomes, Managed Risk trendline, open items, and next-quarter plan."

Open to a quick 15-minute chat next week?

Best,
Matt Plotkin
Inkeep

---

### Good Follow-Up with Blog CTA (Email 2)

**Subject:** RE: Docs activation

58% of SaaS companies are seeing NRR decline. Usage behavior accounts for 80% of outcomes, yet most AI investment goes to sales instead of CX.

This covers why that's backwards:
https://inkeep.com/blog/why-customer-success-needs-ai-agents-before-sales-does-in-20

Worth 5 minutes if retention is on your radar.

---

### Avoid

**Subject:** Exciting opportunity to revolutionize your customer experience!

Hi [Name],

I hope this email finds you well! My name is [Rep] and I'm reaching out from [Company]. We're a leading provider of AI-powered customer support solutions that help companies like yours achieve up to 50% improvement in customer satisfaction scores.

I'd love to schedule a quick 30-minute call to discuss how we can help [Company] transform their customer experience journey. Would you have time next Tuesday or Wednesday?

Best regards,
[Rep]

*Problems: Opens with "I", uses "leading provider", vague ROI claim, asks for 30-min meeting, no personalization, no social proof, over 100 words.*

---

### Avoid (Generic CS Email)

**Subject:** faster QBR prep

Hi Kyle,

I saw you lead Customer Success at Arctic Wolf. Your team probably spends a lot of time on manual prep work before renewals and QBRs.

We help CS teams automate their workflows and save time.

Would you be open to a quick chat?

*Problems: Generic subject line (could apply to any company), opens with their title (wastes characters), no product-specific language, no specific pain points, no social proof, vague value prop.*

---

## Output Format

When generating an email, output:

```
**Subject:** [subject line]

[email body]

---
**Notes:** [Optional: brief explanation of choices made]
```

If generating multiple variants, label them Variant A, B, C.

If generating a follow-up sequence, label by email number and type.

---

---

## CSM AI Agent Demo Transcript Guidelines

When writing demo transcripts or marketing content showing the CSM AI Agent in action, follow these guidelines:

### Tool Access Messaging

**Bad (too robotic):**
```
Searching CRM... Searching Totango...
```

**Better (demo-friendly):**
```
Checking account health across Totango and your CRM...
```

**Best (enterprise copilot vibe):**
```
Reviewing customer health signals from Totango and CRM data...
```

### Example Demo Transcript

```
CSM: "Which of my accounts are at risk?"

Inkeep (CSM Copilot): Reviewing customer health signals from Totango and account data in your CRM...

Three accounts currently have health scores below 30:
• Acme Inc
• Example.io
• RocketLabs

CSM: "Add a reminder on my calendar to reach out to our main point of contact tomorrow."

Inkeep (CSM Copilot): Locating the primary champion in your CRM and syncing with Google Calendar...

A reminder has been scheduled for tomorrow at 2:00 PM, with a draft outreach email and the relevant contact details included.

CSM: "What renewals do I have coming up?"

Inkeep (CSM Copilot): Checking upcoming renewal dates in your CRM...

You have renewals coming up with:
• Stackforge
• Acme LLC
• Umbrella Co.

CSM: "Could you create QBR materials for the upcoming renewal?"

Inkeep (CSM Copilot): Gathering historical context from CRM records, Gong conversations, Totango health data, and product analytics, and assembling materials in Notion...

I've generated a QBR document you can use for the renewal discussion, including key outcomes, usage trends, risks, and recommendations.
```

### Demo Transcript Principles

- Show which tools are being accessed (signals real integrations)
- Collapse multiple steps into one readable action
- Sound proactive, not just reactive
- Add lightweight follow-ups ("Want next steps?") like real copilots
- Use clear business language ("primary champion", "risk factors", "expansion")
- Output feels exec-ready

---

## Source Reports

For deeper research beyond the skill references, consult these reports:

| Report | Path | Use For |
|--------|------|---------|
| **B2B Persona Messaging Playbook** | `~/reports/b2b-persona-messaging-playbook/REPORT.md` | Full persona research: 19 archetypes, pain points, buying behavior, anti-patterns, compensation data |
| **Blog-to-Persona Mapping** | `~/reports/blog-persona-mapping/REPORT.md` | Article CTAs by persona and buying stage, case study mappings |
| **Customer Social Proof** | `~/reports/customer-social-proof/REPORT.md` | Customer logos by industry, size, and persona for social proof |

## Skill References

| Reference | File | Use For |
|-----------|------|---------|
| **Personas** | `references/personas.md` | Pain points, metrics, buying behavior, anti-patterns |
| **Blog Mapping** | `references/blog-mapping.md` | Article CTAs by persona, case studies |
| **Customer Proof** | `references/customer-proof.md` | Social proof by industry and size |
| **Product Intel** | `references/product-intel.md` | Inkeep product capabilities, proof points, positioning |
| **Best Practices** | `references/best-practices.md` | Cold email effectiveness data (85M+ emails) |
