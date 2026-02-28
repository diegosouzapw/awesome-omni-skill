---
name: salesforce
description: Query and manage Salesforce CRM data for Jewish Federation of San Diego (JFSD). Use for donor queries, gift transactions, pledges, campaigns, soft credits, retention analysis, LYBUNT reports, designations, and any Salesforce data questions. Triggers on: donor lookup, gift history, pledge status, campaign totals, retention metrics, FY comparisons, SOQL queries, Salesforce reporting.
---

# Salesforce Skill — Jewish Federation of San Diego

Query and manage the JFSD Salesforce org via MCP tools.

## Organization Context

**Jewish Federation of San Diego** — nonprofit federation serving the San Diego Jewish community.

**Key Functions:**
- Annual Campaign fundraising
- Donor management and stewardship
- Community programs and missions
- Grants and designated giving
- Events and trips to Israel

## MCP Tools

| Tool | Use For |
|------|---------|
| `sf_query` | Run SOQL queries |
| `sf_describe` | Get object/field metadata |
| `sf_get_record` | Fetch single record by ID |
| `sf_search` | Search across objects (SOSL) |
| `sf_list_objects` | List available objects |

## Fiscal Year

**July 1 – June 30** (Custom Fiscal Year)

| Label | Period |
|-------|--------|
| FY26 | Jul 1, 2025 – Jun 30, 2026 |
| FY25 | Jul 1, 2024 – Jun 30, 2025 |
| FY24 | Jul 1, 2023 – Jun 30, 2024 |

Use `THIS_FISCAL_YEAR` and `LAST_FISCAL_YEAR` in SOQL.

---

## Data Model Overview

### Core Objects

```
Account (Donor)
    ├── GiftCommitment (pledges)
    │       └── GiftTransaction (pledge payments)
    ├── GiftTransaction (direct gifts)
    │       └── GiftTransactionDesignation (fund allocations)
    ├── GiftSoftCredit (influence credits)
    └── Household__c → Account (household rollups)

Campaign
    ├── Child Campaigns (hierarchy)
    ├── GiftTransaction (gifts)
    └── GiftCommitment (pledges)

GiftDesignation (funds/purposes)
    └── GiftTransactionDesignation / GiftDefaultDesignation
```

### Account (Donors)

Person Accounts for individuals. Key field groups:

**Identification:**
- `Name`, `PersonEmail`, `Phone`
- `Household__c` — reference to household account
- `OwnerId` — assigned fundraiser

**Giving History (Date-Based):**
| Field | Description |
|-------|-------------|
| `Total_Gifts_All_Time__c` | Lifetime giving |
| `Giving_this_Fiscal_Year__c` | Current FY total |
| `Total_Gifts_Last_Year__c` | Last FY total |
| `First_Gift_Date__c` / `First_Gift_Amount__c` | First gift |
| `Last_Gift_Date__c` / `Last_Gift_Amount__c` | Most recent gift |
| `Largest_Gift__c` | Largest single gift |

**Annual Campaign Credit (Campaign-Based):**

Donor Credit = Commitments + Direct Gifts + Soft Credits

| Component | FY24 | FY25 | FY26 |
|-----------|------|------|------|
| Commitments | `Gift_Commitments_FY24__c` | `Gift_Commitments_FY25__c` | `Gift_Commitments_FY26__c` |
| Direct Gifts | `Total_Gifts_FY24_Non_Pledge__c` | `Total_Gifts_FY25_Non_Pledge__c` | `Total_Gifts_FY26_Non_Pledge__c` |
| Soft Credits | `Soft_Credits_FY24__c` | `Soft_Credits_FY25__c` | `Soft_Credits_FY26__c` |

**Household Rollups (HH_*):**
Same structure with `HH_` prefix — aggregates all household members.

**Soft Credits (Date-Based):**
- `Soft_Credit_All_Time__c`
- `Soft_Credit_This_Fiscal_Year__c`
- `Soft_Credits_Last_Fiscal_Year__c`

**Recognition:**
- `HH_Total_Recognition_Amount_Annual_2026__c` etc.

---

## Campaign Structure

### Annual Campaign Hierarchy

```
Annual - FY26 (Parent)
├── Ambassador - FY26
├── Chaplaincy - FY26
├── DRM - FY26
├── Monthly - FY26
├── Fall Direct Mail - FY26
├── FED360 - FY26 - Donations
├── Corporate Event Sponsorships - FY26
└── Unsolicited Annual Gifts - FY26
```

### Campaign Types
- `Fundraising` — Annual, Designated, Emergency, Endowment
- `Appeal` — Sub-campaigns (DRM, Ambassador, etc.)
- `Event` — FED360, Cornerstone, etc.
- `Mission` — Israel trips, community missions
- `Sponsorship` — Event sponsorships
- `Group` — Cohort programs (Giborim, WLI)

### Major Campaign Categories (FY26)

| Category | Campaign Name |
|----------|---------------|
| Annual | `Annual - FY26` |
| Designated | `Designated - FY26` |
| Emergency | `Emergency - FY26` |
| Endowment | `Endowment - FY26` |
| Events | `Events - FY26` |
| Programs | `Programs - FY26` |
| Tributes | `Tributes` |
| Rady Match | `Rady Match for Israel's Recovery` |

### Campaign Naming Convention

⚠️ **FY26 changed format:**
- FY24: `Annual 2024`
- FY25: `Annual 2025`  
- FY26: `Annual - FY26`

Filter pattern for Annual Campaign:
```sql
Campaign.Name = 'Annual - FY26' OR Campaign.Parent.Name = 'Annual - FY26'
```

---

## Gift Designations

Common designations (funds):
- `Annual Campaign` — unrestricted annual
- `Emergency Relief` — disaster response
- `Beit Melachah` — specific project
- `Chaplaincy Fund` — chaplaincy program
- `Community Mission` — missions/trips
- `Day Schools` — education
- `Israel Unfiltered` — young adult Israel trips
- `Legacy of Light` — Holocaust education
- `Credit Card Fees` — processing fees

---

## Common Query Patterns

### Retention: FY25 donors not yet in FY26
```sql
SELECT Id, Name, 
       Gift_Commitments_FY25__c, Total_Gifts_FY25_Non_Pledge__c, Soft_Credits_FY25__c
FROM Account
WHERE (Gift_Commitments_FY25__c > 0 OR Total_Gifts_FY25_Non_Pledge__c > 0 OR Soft_Credits_FY25__c > 0)
  AND (Gift_Commitments_FY26__c = 0 OR Gift_Commitments_FY26__c = null)
  AND (Total_Gifts_FY26_Non_Pledge__c = 0 OR Total_Gifts_FY26_Non_Pledge__c = null)
  AND (Soft_Credits_FY26__c = 0 OR Soft_Credits_FY26__c = null)
```

### LYBUNT (Last Year But Unfortunately Not This)
```sql
SELECT Id, Name, Total_Gifts_Last_Year__c
FROM Account
WHERE Total_Gifts_Last_Year__c > 0
  AND (Giving_this_Fiscal_Year__c = 0 OR Giving_this_Fiscal_Year__c = null)
ORDER BY Total_Gifts_Last_Year__c DESC
```

### Top donors this FY
```sql
SELECT Id, Name, Giving_this_Fiscal_Year__c
FROM Account
WHERE Giving_this_Fiscal_Year__c > 0
ORDER BY Giving_this_Fiscal_Year__c DESC
LIMIT 20
```

### New donors (first gift this FY)
```sql
SELECT Id, Name, First_Gift_Date__c, First_Gift_Amount__c
FROM Account
WHERE First_Gift_Date__c >= 2025-07-01
ORDER BY First_Gift_Date__c DESC
```

### Recent gifts
```sql
SELECT Id, Donor.Name, CurrentAmount, TransactionDate, Campaign.Name
FROM GiftTransaction
WHERE Status = 'Paid' AND TransactionDate = LAST_N_DAYS:30
ORDER BY TransactionDate DESC
```

### Campaign totals
```sql
SELECT Id, Name, Total_Gift_Transactions__c, Gift_Commitments_All_Time__c
FROM Campaign
WHERE Name LIKE 'Annual - FY26%'
```

### Household giving
```sql
SELECT Id, Name, HH_Gift_Commitments_FY26__c, HH_Total_Gifts_FY26_Non_Pledge__c, HH_Soft_Credit_FY26__c
FROM Account
WHERE Household__c = '[HOUSEHOLD_ID]'
```

---

## Filter Patterns

**Paid gifts only:**
```sql
Status = 'Paid' AND CurrentAmount > 0
```

**Direct gifts (not pledge payments):**
```sql
GiftCommitmentId = NULL
```

**Current fiscal year:**
```sql
TransactionDate = THIS_FISCAL_YEAR
```

**Active pledges:**
```sql
Status = 'Active'
```

**Excluding write-offs:**
```sql
Status = 'Active' OR Status = 'Closed'
```

---

## Detailed References

For complete field lists and technical details:
- [references/schema.md](references/schema.md) — Full field documentation
- [references/dlrs.md](references/dlrs.md) — Rollup summary configurations
- [references/campaigns.md](references/campaigns.md) — Campaign hierarchy

---

## Tips

1. **Individual vs Household:** Use `HH_*` fields for household totals, base fields for individual
2. **Annual Campaign vs Total Giving:** FY-specific fields use campaign filters; `Giving_this_Fiscal_Year__c` uses dates
3. **Soft Credits:** Always check `GiftTransaction.Status = 'Paid'` when summing
4. **Campaign hierarchy:** Use `Campaign.Parent.Name` to include child campaigns
5. **Person Accounts:** Filter with `IsPersonAccount = true` for individuals
