---
name: account-qualification
description: Qualifies and tiers accounts based on signals, fit, and potential. Use this skill when building target lists, prioritizing accounts, identifying high-potential prospects, or defining ideal customer profile criteria.
---

# Account Qualification

This skill helps you systematically qualify and tier accounts to focus your sales efforts on the highest-potential opportunities.

## Objective

Identify, score, and prioritize accounts based on fit signals, buying indicators, and strategic value to maximize sales efficiency and win rates.

## Account Tiering System

### Tier 1: High Priority Accounts (HPA)
- Strong signal matches
- Excellent ICP fit
- Active buying indicators
- High strategic value

**Action**: Immediate, personalized outreach with research investment

### Tier 2: Qualified Accounts
- Good signal matches
- Solid ICP fit
- Some buying indicators
- Good potential value

**Action**: Prioritized outreach with moderate personalization

### Tier 3: Developing Accounts
- Partial signal matches
- Reasonable ICP fit
- Limited buying indicators
- Moderate potential

**Action**: Nurture sequences and monitoring

### Tier 4: Low Priority / Disqualified
- Few or no signal matches
- Poor ICP fit
- No buying indicators
- Limited potential

**Action**: Deprioritize or remove from active pursuit

## Signal Categories

### Buying Intent Signals

**Active Signals (High Value):**
- Searching for solutions like yours
- Requesting demos or trials from competitors
- Attending relevant webinars or events
- Engaging with your content repeatedly
- Asking questions in industry forums

**Passive Signals (Medium Value):**
- Following competitors on social media
- Downloading industry reports
- Job postings indicating relevant needs
- Technology stack changes

### Organizational Signals

**Growth Indicators:**
- Funding announcements
- Hiring sprees (especially in relevant departments)
- New office openings
- Revenue milestones
- Market expansion

**Change Indicators:**
- New leadership (especially in relevant roles)
- Mergers or acquisitions
- Strategic pivots announced
- Technology platform changes
- Vendor relationship changes

### Fit Signals

**Company Characteristics:**
- Industry alignment
- Company size (employees, revenue)
- Geographic match
- Technology stack compatibility
- Business model fit

**Timing Indicators:**
- Fiscal year timing
- Budget cycles
- Contract renewal windows
- Regulatory deadlines
- Seasonal patterns

## Qualification Criteria Framework

### Must-Have Criteria (Disqualifying if absent)
Define absolute requirements:
- Minimum company size
- Required industry or vertical
- Geographic constraints
- Technology prerequisites
- Budget authority level

### Fit Scoring Criteria

| Criteria | Weight | Score 1-5 |
|----------|--------|-----------|
| Industry match | 20% | |
| Company size | 15% | |
| Geographic fit | 10% | |
| Tech stack | 15% | |
| Budget potential | 20% | |
| Timing signals | 10% | |
| Engagement level | 10% | |

### Signal Scoring

| Signal Type | Points |
|-------------|--------|
| Active buying intent | +10 |
| Recent funding | +8 |
| Leadership change | +7 |
| Job postings (relevant) | +5 |
| Technology change | +5 |
| Content engagement | +3 |
| Company growth | +3 |
| Passive interest | +1 |

## Account Research Checklist

### Company-Level Research
- [ ] Company website and about page
- [ ] Recent press releases
- [ ] Leadership team profiles
- [ ] Job postings (growth areas)
- [ ] Technology stack (BuiltWith, Wappalyzer)
- [ ] Funding history (Crunchbase, PitchBook)
- [ ] Financial performance (if public)
- [ ] Industry news and mentions

### Signal Validation
- [ ] Verify signals are current (within 90 days)
- [ ] Confirm decision-maker presence
- [ ] Check for competing vendor relationships
- [ ] Assess timing and urgency
- [ ] Identify potential blockers

## Ideal Customer Profile (ICP) Template

### Firmographics
- **Industry**: [Primary and secondary verticals]
- **Company Size**: [Employee range]
- **Revenue**: [Revenue range]
- **Geography**: [Target regions]
- **Growth Stage**: [Startup, Growth, Enterprise]

### Technographics
- **Current Stack**: [Technologies they likely use]
- **Complementary Tools**: [Solutions that work with yours]
- **Competing Solutions**: [Alternatives they might have]

### Behavioral Indicators
- **Buying Triggers**: [Events that create urgency]
- **Content Interests**: [Topics they care about]
- **Decision Process**: [How they typically buy]

### Negative Indicators
- **Disqualifiers**: [Immediate deal-breakers]
- **Warning Signs**: [Red flags to watch for]
- **Poor Fit Patterns**: [Characteristics of bad deals]

## Account Scoring Output

### Account Scorecard Template

```
## Account: [Company Name]

### Tier Assignment: [Tier 1/2/3/4]

### Fit Score: [X/100]
- Industry: [Score] - [Notes]
- Size: [Score] - [Notes]
- Geography: [Score] - [Notes]
- Tech Stack: [Score] - [Notes]
- Budget: [Score] - [Notes]

### Signal Score: [X points]
- [Signal 1]: +X points
- [Signal 2]: +X points
- [Signal 3]: +X points

### Qualification Status
- [ ] Meets must-have criteria
- [ ] Decision-maker identified
- [ ] Timing validated
- [ ] No blockers identified

### Recommended Action
[Specific next step based on tier and signals]

### Key Stakeholders to Target
1. [Name, Title] - [Why relevant]
2. [Name, Title] - [Why relevant]
```

## Output Format

When qualifying an account, produce:

1. **Tier Assignment**: Which tier and why
2. **Fit Score**: Detailed scoring breakdown
3. **Signal Summary**: Active signals identified
4. **Qualification Status**: Go/No-Go recommendation
5. **Recommended Action**: Specific next steps
6. **Stakeholder Map**: Key people to engage

## Available Tools

When enabled, these MCP tools enhance qualification capabilities:

| Tool | What It Does | How to Use |
|------|-------------|------------|
| **Perplexity** | Verify signals and find news | "Use Perplexity to check if [company] recently raised funding" |
| **Exa** | Search for company signals | "Search Exa for [company] new VP of Sales announcement" |
| **Apify** | Extract job postings data | "Use Apify to find [company]'s open sales positions" |

**Note:** Tools must be enabled in `.mcp.json` and API keys configured. See README for setup instructions.

### Tool Usage Examples

```
"Use Perplexity to verify if Acme Corp announced a new CRO in the last 6 months"
"Search Exa for Acme Corp Series B funding announcement"
"Use Apify to scrape job postings from Acme Corp careers page"
```

## Cross-References

- Use `company-intelligence` for deeper research on qualified accounts
- Feed qualified accounts to `prospect-research` for stakeholder profiles
- Inform `sales-orchestrator` on account prioritization
- Guide `multithread-outreach` strategy based on stakeholder map
