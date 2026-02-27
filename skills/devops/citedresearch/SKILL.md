---
name: CitedResearch
description: Research output with proper source citations. USE WHEN conducting research, creating sector analyses, or generating investment notes that need verifiable sources.
triggers:
  - research
  - cite
  - sources
  - sector analysis
  - investment research
  - market research
  - due diligence
---

# CitedResearch - Source-Cited Research Output

Generate research outputs with proper inline citations and source tables for PE investment research.

## Citation Format

**Hybrid approach:**
1. **Inline citations** on key statistics and claims: `[value](url)`
2. **Sources table** at bottom with reliability ratings

### Inline Citation Rules

```markdown
# Good - key stats linked
Market size is [$209B](https://source.com/report) with [7.9% CAGR](https://source.com/growth).

# Bad - clutters reading
According to [Grand View Research](https://...), the market [size](https://...) is [$209B](https://...) and growing at [7.9%](https://...) [CAGR](https://...).
```

**When to inline cite:**
- Market size figures
- Growth rates / CAGR
- Company counts / statistics
- Percentages from studies
- Geographic concentration data
- Succession/demographic figures

**When NOT to inline cite:**
- General statements of fact
- Logical conclusions
- Internal analysis
- Qualitative assessments

### Sources Table Format

```markdown
## Sources

| Source | Type | Reliability | Key Data |
|--------|------|-------------|----------|
| [KfW Research](url) | Government | ⭐⭐⭐⭐⭐ | Succession stats |
| [PitchBook](url) | Financial Data | ⭐⭐⭐⭐⭐ | PE activity |
| [Industry Report](url) | Market Research | ⭐⭐⭐⭐ | Market sizing |
```

**Reliability Ratings:**
- ⭐⭐⭐⭐⭐ - Government, major financial data (KfW, Destatis, PitchBook)
- ⭐⭐⭐⭐ - Industry reports, advisory firms (McKinsey, Rödl, IMAP)
- ⭐⭐⭐ - Trade publications, news sources
- ⭐⭐ - Company websites, press releases
- ⭐ - Blogs, unverified sources

---

## Research Note Templates

### Sector Analysis (SECT-XXX)

```markdown
---
id: SECT-XXX
type: sector
created: {{date}}
tags: [sector, {{industry}}, {{geography}}]
thesis_fit: high|medium|low
---

# SECT-XXX - {{Sector Name}}

## Executive Summary
[2-3 sentences with inline citations on key stats]

## Market Overview
### Market Size & Growth
| Metric | Value | Source |
[All values should be inline linked]

### Geographic Concentration
[Regional data with citations]

## Succession Dynamics
[Statistics with KfW/government sources]

## 4M Framework Assessment
[Internal analysis - no citations needed]

## Sources
[Full sources table with reliability]
```

### Company Research (COMP-XXX)

```markdown
---
id: COMP-XXX
type: company
tags: [company, {{sector}}, {{status}}]
---

# COMP-XXX - {{Company Name}}

## Overview
[Basic info with website link]

## Financial Summary
| Metric | Value | Source |
[Revenue, EBITDA, margins - cite if public]

## Thesis Fit
[Internal analysis]

## Sources
[Sources table]
```

---

## Usage

### With Helias/PAI

When I conduct research, I will automatically:
1. Collect sources during web searches
2. Apply inline citations to key statistics
3. Generate sources table with reliability ratings
4. Follow the appropriate template

### Example Command

```
"Research [topic] for the investment thesis"
→ Produces fully cited sector note

"Create a sector analysis of [industry] in [geography]"
→ SECT-XXX note with 4M framework and citations
```

---

## Integration with Existing Skills

### With Fabric

Fabric patterns can pre-process content, then CitedResearch formats the output:

```bash
# Extract insights, then format with citations
fabric -p extract_article_wisdom < article.txt | \
  bun run CitedResearch/tools/FormatCited.ts --template sector
```

### With ObsidianSync

Research notes are automatically routed:
- Sector analyses → `03-Resources/Sectors/`
- Company research → `03-Resources/Companies/`
- Market reports → `03-Resources/Research/`

---

## Source Collection Guidelines

### During Research

1. **Track all sources** - save URL + key data point
2. **Verify reliability** - cross-reference claims
3. **Prefer primary** - government > advisory > news
4. **Note access date** - markets change

### Source Types by Priority

| Priority | Type | Examples |
|----------|------|----------|
| 1 | Government | KfW, Destatis, EU Commission |
| 2 | Financial data | PitchBook, S&P Capital IQ |
| 3 | Industry reports | IMAP, McKinsey, Bain |
| 4 | Advisory | Rödl, EY, Deloitte |
| 5 | Trade associations | VDMA, ZVEI, BDI |
| 6 | News | FT, Handelsblatt, Reuters |

---

## Quick Reference

**Inline cite:**
- Numbers, percentages, counts
- Market sizes, growth rates
- Geographic concentrations
- Survey results

**Don't inline cite:**
- Your analysis
- Logical conclusions
- General knowledge
- Internal assessments

**Sources table always includes:**
- Clickable link
- Source type
- Reliability rating
- What data it provides

---

## Source Freshness Requirements

**All cited statistics must meet freshness thresholds:**

| Data Type | Max Age | DACH Primary Sources |
|-----------|---------|----------------------|
| Market sizing | 12 months | Statista DE, Roland Berger, ZVEI |
| Succession stats | 6 months | KfW Nachfolge-Monitor, IfM Bonn |
| PE deal activity | 3 months | PitchBook, IMAP DACH, Finance Magazin |
| Company financials | Current FY | Bundesanzeiger, Unternehmensregister |
| Industry trends | 6 months | VDMA, ZVEI, BDI sector reports |
| Credit ratings | 1 month | Creditreform, Bürgel, CRIF |
| Insolvency data | Real-time | Bundesanzeiger, Creditreform |

**Reject sources that exceed max age for data type.**

---

## DACH-First Source Priority

When researching DACH markets, always check German sources first before international alternatives.

### Tier 1 - German Government & Official (⭐⭐⭐⭐⭐)

| Source | Data Type | URL |
|--------|-----------|-----|
| KfW Research | Succession, SME financing | kfw.de/forschung |
| Destatis | Federal statistics | destatis.de |
| Bundesanzeiger | Company filings | bundesanzeiger.de |
| IfM Bonn | Mittelstand research | ifm-bonn.org |
| Unternehmensregister | Company registry | unternehmensregister.de |

### Tier 2 - DACH M&A Intelligence (⭐⭐⭐⭐⭐)

| Source | Data Type | Access |
|--------|-----------|--------|
| PitchBook | PE deals, valuations | IESE login |
| S&P Capital IQ | Financials, comps | IESE login |
| IMAP DACH | M&A market reports | imap.com/de |
| Finance Magazin | German PE news | finance-magazin.de |
| Mergermarket | Deal intelligence | mergermarket.com |

### Tier 2 - Credit & Risk Intelligence (⭐⭐⭐⭐⭐)

| Source | Data Type | Access |
|--------|-----------|--------|
| Creditreform | Credit ratings, insolvency, payment behavior | creditreform.de (paid) |
| Bürgel | Credit risk, company data | buergel.de (paid) |
| CRIF | B2B credit data | crif.de (paid) |
| Bundesanzeiger | Insolvency filings | bundesanzeiger.de (free) |

#### Creditreform Data Available

**Company Intelligence:**
- Credit rating (1-600 scale, 600 = best)
- Payment behavior index
- Insolvency probability
- Management/ownership changes
- Company financials (when filed)

**Due Diligence Use Cases:**
- Pre-acquisition credit check
- Supplier/customer risk assessment
- Management background verification
- Financial health monitoring

**API Integration:** Creditreform offers API access for programmatic queries. Contact sales for enterprise pricing.

### Tier 3 - Industry Associations (⭐⭐⭐⭐)

| Source | Sector Coverage | URL |
|--------|-----------------|-----|
| VDMA | Machinery, automation | vdma.org |
| ZVEI | Electronics, electrical | zvei.org |
| BDI | General industry | bdi.eu |
| BITKOM | Digital, IT | bitkom.org |
| VDA | Automotive | vda.de |

### Tier 4 - Business Media (⭐⭐⭐)

| Source | Focus | URL |
|--------|-------|-----|
| Handelsblatt | German business daily | handelsblatt.com |
| Manager Magazin | Executive/strategy | manager-magazin.de |
| WirtschaftsWoche | Economic analysis | wiwo.de |
| Private Equity Magazin | German PE specific | private-equity-magazin.de |
| Finance Magazin | M&A news | finance-magazin.de |

---

## Research Verification Protocol

**Before citing ANY statistic:**

1. **Date Check** - Verify publication date is within max age for data type
2. **Source Tier** - Prefer Tier 1-2 over Tier 3-4
3. **DACH Priority** - Use German source if available before international
4. **Cross-Reference** - Major claims need 2+ independent sources
5. **Access Date** - Always note when source was accessed

### Sources Table Enhanced Format

```markdown
## Sources

| Source | Type | Reliability | Published | Accessed | Key Data |
|--------|------|-------------|-----------|----------|----------|
| [KfW Nachfolge-Monitor](url) | Government | ⭐⭐⭐⭐⭐ | 2025-09 | 2026-01-06 | Succession stats |
| [PitchBook PE Report](url) | Financial | ⭐⭐⭐⭐⭐ | 2025-Q4 | 2026-01-06 | Deal activity |
```

### Freshness Indicators in Notes

Add freshness status to sources:
- ✅ Fresh (within max age)
- ⚠️ Aging (within 2x max age)
- ❌ Stale (exceeds 2x max age - do not use)

---

## Available Tools

### ValidateSources.ts
Validate source freshness against data type rules.

```bash
bun run ValidateSources.ts https://kfw.de/report --type succession_stats
```

### PENewsFeed.ts
Aggregate DACH PE/M&A news from 5 RSS sources with relevance scoring.

```bash
# Fetch latest high-relevance PE news
bun run PENewsFeed.ts --since 24h --min-relevance 5

# Filter for succession-related news
bun run PENewsFeed.ts --filter nachfolge

# Output JSON for programmatic use
bun run PENewsFeed.ts --json
```

**Sources:** Finance Magazin, Private Equity Magazin, Handelsblatt, Manager Magazin, WirtschaftsWoche

**Relevance Scoring:**
- 🔥 10+ points: PE-specific terms (buyout, nachfolge, beteiligung)
- 📊 5+ points: M&A terms (deal, transaktion, investor)

### HandelsregisterLookup.ts
German company registry lookup helper - generates URLs and data templates.

```bash
# Search by company name
bun run HandelsregisterLookup.ts "Bosch GmbH"

# Filter by city
bun run HandelsregisterLookup.ts "Siemens" --city München

# Get blank data template
bun run HandelsregisterLookup.ts --template
```

**Data Sources:**
- unternehmensregister.de - Official portal
- handelsregister.de - Direct registry
- bundesanzeiger.de - Public filings

**Available Data:**
- HRB number, registry court, legal form
- Registered capital, company purpose
- Management, shareholders (when filed)
- Annual financial statements
