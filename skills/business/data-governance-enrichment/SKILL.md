---
name: data-governance-enrichment
description: "Enrich CRM data: tools, waterfall approach, automation, quality control. Use when designing or improving data enrichment in rev ops."
---

# Data Governance — Enrichment Strategies

How to enrich CRM data: tools, waterfall approach, automation, and quality control.

---

## Why Enrichment Matters

**B2B data decays at 2.1% per month (22.5% annually).** Without enrichment:
- Job titles are 50-65% outdated within a year
- Email addresses decay 37% annually
- Phone numbers decay 43% annually

Enrichment keeps your CRM current and your outreach effective.

---

## Enrichment Hierarchy

Not all enrichment sources are equal. Prioritize by accuracy:

| Source | Accuracy | Coverage | Cost | Best For |
|---|---|---|---|---|
| **Manual research** | 95%+ | Low | High (time) | High-value accounts, executives |
| **LinkedIn Sales Navigator** | 90%+ | High | $$$ | Job titles, company info |
| **Clay** | 85-90% | High | $$ | Bulk enrichment, waterfall approach |
| **ZoomInfo** | 80-85% | Very High | $$$$ | Enterprise, comprehensive data |
| **Apollo** | 75-80% | High | $ | Cost-effective bulk enrichment |
| **Clearbit** | 80-85% | Medium | $$$ | Tech stack, firmographics |
| **Lusha** | 75-85% | Medium | $$ | Phone numbers, emails |
| **Hunter** | 70-75% | Medium | $ | Email finding only |

---

## Waterfall Enrichment Strategy

Don't rely on one source. Use a waterfall approach where each step fills gaps from the previous:

```
Step 1: Check existing CRM data (free)
  ↓ If missing fields
Step 2: Clay enrichment (uses multiple sources)
  ↓ If still missing
Step 3: ZoomInfo/Apollo lookup
  ↓ If still missing
Step 4: LinkedIn Sales Navigator
  ↓ If still missing
Step 5: Manual research
  ↓ If still missing
Step 6: Flag for human verification
```

### Clay Waterfall Example

```
Clay Column Configuration:

1. Find Email (Waterfall)
   → Apollo → ZoomInfo → Lusha → Hunter
   → Takes first valid result

2. Find Phone (Waterfall)
   → ZoomInfo → Apollo → Lusha
   → Takes first valid result

3. Enrich Company (Waterfall)
   → Clearbit → ZoomInfo → Apollo
   → Takes first valid result

4. Find LinkedIn URL
   → Apollo → ZoomInfo
   → Validate with LinkedIn profile lookup
```

---

## Enrichment Triggers

### Real-Time Enrichment

| Trigger | Fields to Enrich | Priority |
|---|---|---|
| **New lead created** | Email verification, phone, job title, company size, industry | High — do immediately |
| **Lead becomes MQL** | Full firmographics, tech stack, LinkedIn URL | High — before sales touches |
| **Deal created** | Decision-makers, org chart, recent news | Medium |
| **Form submission** | Verify and complete missing fields | High |
| **Website visit (known)** | Refresh stale data if >90 days old | Medium |

### Batch Enrichment

| Trigger | Scope | Frequency |
|---|---|---|
| **Scheduled refresh** | Active contacts/companies | Weekly/Monthly |
| **Before campaign** | Email verification, phone validation | Before each campaign |
| **Quarterly refresh** | Job title verification, company size update | Quarterly |
| **Annual audit** | Full database enrichment | Annually |

---

## Enrichment by Field Type

### Contact-Level Enrichment

| Field | Enrichment Source | Frequency |
|---|---|---|
| **Email** | ZoomInfo, Apollo, Hunter | On creation + verify before campaigns |
| **Phone** | ZoomInfo, Lusha, Apollo | On creation + verify before call campaigns |
| **Job Title** | LinkedIn, ZoomInfo, Apollo | On creation + every 6 months |
| **LinkedIn URL** | Apollo, ZoomInfo | On creation only |
| **Personal Bio** | LinkedIn Sales Navigator | For high-value contacts |

### Company-Level Enrichment

| Field | Enrichment Source | Frequency |
|---|---|---|
| **Industry** | Clearbit, ZoomInfo | On creation + annually |
| **Employee Count** | ZoomInfo, Clearbit, Apollo | On creation + annually |
| **Revenue** | ZoomInfo, Clearbit | On creation + annually |
| **Tech Stack** | BuiltWith, Clearbit, Wappalyzer | On creation + every 6 months |
| **Funding Info** | Crunchbase, PitchBook | On creation + quarterly |
| **News/Signals** | Bombora, G2, News APIs | Real-time or weekly |

---

## HubSpot Enrichment Configuration

### Workflow: Auto-Enrich New Leads

```
Trigger: Contact created with Email filled

Actions:
1. Delay 5 minutes (allow HubSpot native enrichment)

2. Branch: If Company Name blank
   → Use Clay webhook to find company from email domain
   → Update Company Name, Industry, Employee Count

3. Branch: If Phone blank
   → Use Clay webhook to find phone
   → Update Phone property

4. Branch: If Job Title blank
   → Use Clay webhook to find title
   → Update Job Title and Seniority Level

5. Set "Enrichment Status" = "Completed"
6. Set "Enrichment Date" = Today
```

### Workflow: Pre-Campaign Enrichment

```
Trigger: Added to list "Campaign - [Name] - Outreach"

Actions:
1. Branch: If "Email Verified" != "Valid"
   → Use NeverBounce webhook to verify
   → Update "Email Verified" status

2. Branch: If "Last Enriched" > 90 days ago
   → Trigger full enrichment workflow
   → Update all stale fields

3. Branch: If "Email Verified" = "Invalid"
   → Remove from list
   → Add to "Bad Emails" list

4. Set "Ready for Campaign" = Yes
```

### Workflow: Scheduled Refresh

```
Trigger: Scheduled daily at 2:00 AM

Enrollment: Contacts where:
- Last Enriched > 180 days ago
- Lifecycle Stage = [MQL, SQL, Opportunity]
- Limit: 500 contacts per day (cost management)

Actions:
1. Use Clay webhook for full enrichment
2. Update enriched fields
3. Set "Last Enriched" = Today
4. Log activity "Enrichment refresh completed"
```

---

## Enrichment Quality Control

### Validation After Enrichment

| Field | Validation Check | Action if Failed |
|---|---|---|
| **Email** | Valid format, verified deliverable | Flag, don't use in campaigns |
| **Phone** | Valid format, 7-15 digits | Flag for manual review |
| **Job Title** | In approved list OR reasonable | Flag if nonsensical |
| **Company Size** | Within expected range for industry | Flag if outlier |
| **Industry** | From standardized list | Map or flag |
| **Revenue** | Positive number, reasonable for size | Flag if outlier |

### Quality Metrics to Track

| Metric | Formula | Target |
|---|---|---|
| **Enrichment coverage** | Records enriched / Total records | >80% |
| **Field fill rate post-enrichment** | Filled fields / Target fields | >90% |
| **Email verification rate** | Valid emails / Total emails | >85% |
| **Enrichment accuracy** | Spot-check sample for correctness | >90% |
| **Cost per enriched record** | Total enrichment cost / Records enriched | Track trend |

---

## Cost Management

### Credit Allocation Strategy

| Priority | Segment | Enrichment Level | Budget % |
|---|---|---|---|
| **1** | MQLs and SQLs | Full enrichment | 40% |
| **2** | Target accounts (ABM) | Full enrichment | 30% |
| **3** | New leads | Essential fields only | 20% |
| **4** | Database refresh | Verification only | 10% |

### Credit Optimization Tips

```
1. Don't re-enrich unnecessarily
   - Check existing data first
   - Set minimum time between enrichments (90 days)

2. Prioritize high-value records
   - Enrich MQLs before raw leads
   - Enrich target accounts first

3. Use appropriate sources
   - Free sources first (LinkedIn public data)
   - Cheap sources for low-value records
   - Premium sources for high-value only

4. Batch for efficiency
   - Batch requests where possible
   - Off-peak pricing if available
```

### Cost Tracking Dashboard

| Metric | Calculation | Benchmark |
|---|---|---|
| **Cost per enriched contact** | Enrichment spend / Contacts enriched | $0.10-0.50 |
| **Cost per verified email** | Verification spend / Emails verified | $0.01-0.05 |
| **Enrichment ROI** | (Pipeline from enriched contacts - Cost) / Cost | >5x |
| **Budget pacing** | Month-to-date spend / Monthly budget | <100% |

---

## Multi-Source Integration

### API Integration Pattern

```
For each enrichment request:

1. Check cache (Clay, CRM)
   IF found AND fresh (< 90 days)
   → Return cached data

2. Waterfall through providers
   FOR each provider in priority order:
     → Make API request
     → IF valid data returned
        → Save to cache
        → Return data
        → Exit loop
     → ELSE continue to next provider

3. If no provider returns data
   → Flag for manual research
   → Return partial data with flag
```

### Provider Selection Matrix

| Use Case | Primary | Secondary | Tertiary |
|---|---|---|---|
| **Email discovery** | Apollo | ZoomInfo | Hunter |
| **Phone discovery** | ZoomInfo | Lusha | Apollo |
| **Job title verification** | LinkedIn | ZoomInfo | Apollo |
| **Company firmographics** | Clearbit | ZoomInfo | Apollo |
| **Tech stack** | BuiltWith | Clearbit | Wappalyzer |
| **Intent signals** | Bombora | G2 | ZoomInfo |

---

## Common Enrichment Mistakes

| Mistake | Impact | Fix |
|---|---|---|
| Single source reliance | Low coverage, single point of failure | Waterfall approach |
| No validation | Bad data enters CRM | Validate after enrichment |
| Over-enriching | Wasted budget | Prioritize by value, set limits |
| Under-enriching | Stale data, missed opportunities | Regular refresh schedule |
| Ignoring cost | Budget overrun | Daily/weekly cost monitoring |
| No tracking | Can't measure ROI | Enrichment source tracking |

---

## Implementation Checklist

### Phase 1: Setup (Week 1)

- [ ] Audit current data gaps
- [ ] Select enrichment providers
- [ ] Set up API integrations
- [ ] Configure Clay tables (if using)
- [ ] Create enrichment tracking properties

### Phase 2: Automation (Week 2)

- [ ] Build real-time enrichment workflows
- [ ] Build batch enrichment workflows
- [ ] Set up validation rules
- [ ] Configure error handling
- [ ] Test with sample records

### Phase 3: Launch (Week 3)

- [ ] Start with low-priority records
- [ ] Monitor accuracy and costs
- [ ] Adjust waterfall order based on results
- [ ] Roll out to all segments

### Phase 4: Optimization (Ongoing)

- [ ] Weekly cost review
- [ ] Monthly accuracy audit
- [ ] Quarterly provider evaluation
- [ ] Annual strategy review

---

## Cross-References

- **Data Governance Framework:** See `data-governance-framework.md`
- **Validation Rules:** See `data-governance-validation.md`
- **Quality Metrics:** See `data-governance-framework.md#data-quality-kpis-and-targets`

---

> **Built by [ColdIQ](https://www.coldiq.com) & [Ivan Falco](https://www.linkedin.com/in/ivanfalco/en/).** For questions on implementation or anything not covered here, reach out to Ivan directly on [LinkedIn](https://www.linkedin.com/in/ivanfalco/en/).
