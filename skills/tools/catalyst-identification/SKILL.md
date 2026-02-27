---
name: catalyst-identification
description: Identify upcoming and recent catalysts that could move the stock
allowed-tools: get_news_sentiment get_company_overview
metadata:
  domain: news
  complexity: intermediate
---

## Catalyst Identification Workflow

OBJECTIVE: Find events that could significantly move the stock price.

### Step 1: Search for Upcoming Events
- Use `get_news_sentiment` with keywords like "earnings", "conference", "announcement"
- Check for scheduled events in the news

### Step 2: Identify Recent Catalysts
- Look at news that coincided with significant price moves
- What drove the most recent rally or selloff?

### Step 3: Categorize Catalysts
- EARNINGS: Quarterly reports (high impact)
- PRODUCT: Launches, updates, delays
- MANAGEMENT: CEO changes, departures
- REGULATORY: FDA approvals, investigations
- MACRO: Industry-wide events affecting sector
- M&A: Acquisition rumors, deals

### Step 4: Assess Impact Potential
For each catalyst, rate:
- HIGH IMPACT: Could move stock 5%+
- MEDIUM IMPACT: Could move stock 2-5%
- LOW IMPACT: Minor or already priced in

### Output Format
UPCOMING CATALYSTS:
1. [Event] - [Date if known] - [Expected Impact]
2. [Event] - [Date if known] - [Expected Impact]

RECENT CATALYSTS:
1. [Event] - [What happened] - [Actual Impact]

CATALYST WATCH: [Key event to monitor]
