---
name: Rankscale GEO Analytics
description: Fetch and interpret Rankscale GEO (Generative Engine Optimization) analytics. Pulls brand visibility score, citation rate, sentiment, and top AI search terms.
version: 1.0.11
metadata:
  openclaw:
    requires:
      env:
        - RANKSCALE_API_KEY
      bins:
        - node
      config:
        - .env
    primaryEnv: RANKSCALE_API_KEY
    always: false
    skillKey: rankscale
    emoji: 📊
    homepage: https://rankscale.ai
    os: []
    install:
      - kind: node
        bins: [node]
---






# Rankscale GEO Analytics

**The best generative engine optimization and AI rank tracking for ChatGPT, Perplexity, Gemini, Claude, DeepSeek, Mistral, and more.**

---

## Overview

RS-Skill connects OpenClaw to the Rankscale API, bringing your brand's AI search performance directly into your assistant. Get visibility scores, reputation analysis, content gap reports, citation intelligence, and more — all in one place.

Ask your assistant questions like:
- _"How is my brand performing across AI engines?"_
- _"Where are my content gaps this week?"_
- _"Find PR opportunities for my brand."_
- _"What's our reputation score and what's dragging it down?"_
- _"Which AI engines am I losing ground on?"_

---

## Quick Start (3 Steps)

**Step 1 — Create a Rankscale Account** *(PRO account required — trial not sufficient)*
Sign up at [rankscale.ai](https://rankscale.ai/dashboard/signup) and set up your brand profile.

> **⚠️ Requirements:** A **Rankscale PRO account** (or higher) is required. Trial accounts do not have REST API access. You must be on PRO before requesting API activation.

**Step 2 — Request REST API Access & Get Your API Key**
1. Email `support@rankscale.ai` with subject: _"Please activate REST API access for my account"_
2. Once activated (usually within 24 hours), log into your Rankscale dashboard
3. Go to **Settings → Integrations → API Keys**
4. Generate a new API key (format: `rk_<hash>_<brandId>`)
5. Copy it immediately — it's only shown once

The Brand ID is embedded in your API key suffix and will be extracted automatically.

**Step 3 — Set Environment Variables**
Add to your OpenClaw Gateway config:
```
RANKSCALE_API_KEY=rk_...
```

The Brand ID is optional (auto-extracted from your API key):
```
RANKSCALE_BRAND_ID=...  # Optional — only if querying a different brand
```

Then run: `node rankscale-skill.js --discover-brands` to verify your setup.

> **Tip:** Your API key format is `rk_<hash>_<brandId>`. The Brand ID is automatically extracted, so you usually only need to set `RANKSCALE_API_KEY`.

---

## Features

| Feature | Description | CLI Flag |
|---------|-------------|----------|
| **GEO Overview** | Full visibility report + search terms breakdown | _(default)_ |
| **Engine Strength Profile** | Visibility spread across all tracked AI engines | `--engine-profile` |
| **Content Gap Analysis** | Topics and queries where your brand has low visibility | `--gap-analysis` |
| **Reputation Score** | Sentiment-based 0–100 brand health score | `--reputation` |
| **Engine Gainers & Losers** | Top movers vs prior period, per engine | `--engine-movers` |
| **Sentiment Shift Alert** | Trend detection + risk flags for sentiment changes | `--sentiment-alerts` |
| **Citation Intelligence** | Authority ranking, gap analysis, engine preferences, PR targets | `--citations` |

---

## Real Example Prompts

These prompts work out-of-the-box with RS-Skill connected to OpenClaw:

**1. Daily brand health check**
> _"Give me my Rankscale GEO overview for this week."_

**2. Engine-specific strategy**
> _"Which AI engines am I weakest on? Show me my engine strength profile."_

**3. Content planning**
> _"What topics should I be writing about? Run a content gap analysis."_

**4. PR campaign prep**
> _"Find citation gaps vs my competitors and suggest PR targets."_
```
node rankscale-skill.js --citations full
```

**5. Reputation monitoring**
> _"Is our brand reputation improving or declining? Any risk areas?"_

---

## Usage Examples

### GEO Overview (default)

```
node rankscale-skill.js
```

Sample output (healthy brand):
```
=======================================================
                 RANKSCALE GEO REPORT
             Brand: AcmeCorp | 2026-02-19
=======================================================
  GEO SCORE:      72 / 100   [+5 vs last week]
  CITATION RATE: 55.5%     [Industry avg: 45%]
  SENTIMENT:     Pos 61% | Neu 29% | Neg 10%
-------------------------------------------------------
  TOP AI SEARCH TERMS
  1. "best crm software"                  (500 mentions)
  2. "crm comparison"                     (300 mentions)
  3. "crm pricing"                        (200 mentions)
-------------------------------------------------------
  GEO INSIGHTS  [1 action]
  [INFO] Strong positive momentum detected.
  Action: Maintain current content cadence.
  Double down on formats producing citations.
  Consider expanding to adjacent topics.
-------------------------------------------------------
  Full report: https://rankscale.ai/dashboard/brands/[your-brand]
=======================================================
```

---

### Engine Strength Profile

```
node rankscale-skill.js --engine-profile
```

Sample output (live API — ROA-40 test run, 2026-02-26):
```
-------------------------------------------------------
                ENGINE STRENGTH PROFILE
-------------------------------------------------------
  Engine       Visibility            Score
  Average      ──────────────────     69.3
-------------------------------------------------------
  mistral_larg ██████████████████████ 83.2 ✦
  deepseek_cha █████████████████████  79.5 ✦
  chatgpt_gui  ████████████████████   77.5 ✦
  perplexity_s ████████████████████   73.9
  google_ai_ov ███████████████████      73
  google_ai_mo ███████████████████    70.8
  google_gemin ██████████████████     66.2
  openai_gpt-5 ████████████████       60.1 ▼
  anthropic_cl ███████████████        57.7 ▼
  perplexity_g █████████████          50.7 ▼
-------------------------------------------------------
  ✦ Top-3 engines  ▼ Bottom-3 engines
```

---

### Content Gap Analysis

```
node rankscale-skill.js --gap-analysis
```

Sample output (from live test data):
```
-------------------------------------------------------
                 CONTENT GAP ANALYSIS
-------------------------------------------------------
  ENGINE GAPS (vs avg 44.5):
  ▼ grok           score:   15  gap:-29.5
  ▼ gemini         score:   20  gap:-24.5

  LOW-VISIBILITY TERMS (<50%) — 3 found:
  email campaigns        ░                      5%
  sales pipeline         ░░░░                  18%
  marketing automation   ░░░░░░░░              42%

  RECOMMENDATIONS:
  1. Create content targeting top 3 gap terms:
     • "email campaigns"
     • "sales pipeline"
     • "marketing automation"
  2. Optimise for grok: score 15 vs avg 44.5
-------------------------------------------------------
```

---

### Reputation Score

```
node rankscale-skill.js --reputation
```

Sample output (live test data, brand with sentiment keywords):
```
-------------------------------------------------------
              REPUTATION SCORE & SUMMARY
-------------------------------------------------------
  Score:  ██████████████████░░░░░░░░░░░░ 61/100
  Status: Good   Trend: ↑ improving

  Sentiment breakdown:
    Positive: 56.2%  Negative: 15.7%  Neutral: 28.1%

  Top positive signals:
    easy to use, great support, powerful

  Risk areas:
    expensive, slow

  Summary: Brand health is good (61/100) and improving.
           Monitor: expensive, slow.
-------------------------------------------------------
```

---

### Citation Intelligence

```
node rankscale-skill.js --citations
node rankscale-skill.js --citations full
node rankscale-skill.js --citations gaps
```

Sample output:
```
🔗 Citation Intelligence
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Top Sources:
  1. techcrunch.com      42 citations   🟢 High authority
  2. forbes.com          38 citations   🟢 High authority
  3. g2.com              21 citations   🟡 Mid authority

Gap vs Competitors:
  Competitor A leads +31 citations on Perplexity
  Competitor B leads +18 citations on Gemini

PR Targets:
  → wired.com (competitor coverage, not yours)
  → venturebeat.com (high Gemini indexing)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## All Commands

```
node rankscale-skill.js [flag]

--help                  Show help
--discover-brands       List your tracked brands
--engine-profile        Engine strength analysis
--gap-analysis          Content gap finder
--reputation            Brand reputation score
--engine-movers         Top gainers and losers by engine
--sentiment-alerts      Sentiment trends + risk detection
--citations             Citation intelligence overview
--citations authority   Top citation sources ranked
--citations gaps        Gaps vs competitors
--citations engines     Per-engine citation breakdown
--citations correlation Citation↔Visibility correlation
--citations full        All citation sections + PR targets
```

For a full command reference, see → [`references/COMMANDS.md`](references/COMMANDS.md)
For detailed feature guides, see → [`references/FEATURES.md`](references/FEATURES.md)
For real usage scenarios, see → [`references/EXAMPLES.md`](references/EXAMPLES.md)
For troubleshooting help, see → [`references/TROUBLESHOOTING.md`](references/TROUBLESHOOTING.md)

---

## What's Next?

Enhancements planned for future versions:

- **Competitor comparison view** — Side-by-side delta scores vs tracked competitors
- **Detection rate metric** — "Content Gap Investigation" rule for low-detection brands  
- **Engine-specific optimization rules** — Tailored advice when engine disparity > 30 pts
- **Scheduled reports** — Auto-run weekly summary via OpenClaw cron
- **Multi-brand switching** — Quick brand toggle without changing env vars
- **Export to PDF/CSV** — Shareable reports for team or client delivery

---

## Support

Questions? We are happy to support.

📧 `support@rankscale.ai`
🌐 [rankscale.ai](https://rankscale.ai/dashboard/signup)

See also: [`references/onboarding.md`](references/onboarding.md) for setup and first-run guidance.
