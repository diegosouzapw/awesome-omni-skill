---
name: GHL Reporting & Analytics
description: Guide for GoHighLevel dashboards and analytics — dashboard widgets, custom reports, attribution tracking, call tracking, ad reporting, conversion analytics, and KPI scorecards.
---

# GHL Reporting & Analytics

## Overview

GoHighLevel provides built-in reporting and analytics across CRM, marketing, sales, and communication activities. Use these tools to track performance, identify bottlenecks, and make data-driven decisions.

---

## Dashboard Overview

### Default Dashboard Widgets

| Widget             | Metrics Tracked                                 |
| ------------------ | ----------------------------------------------- |
| **Opportunities**  | Deal count, value, stage distribution, win rate |
| **Pipeline Value** | Total pipeline value, month-over-month change   |
| **Appointments**   | Booked, confirmed, showed, no-show, cancelled   |
| **Conversations**  | Message volume, response time, unread count     |
| **Tasks**          | Open, completed, overdue tasks                  |
| **Contact Growth** | New contacts over time, source breakdown        |
| **Revenue**        | Invoices, payments, subscriptions               |

### Custom Dashboard Setup

1. Navigate to **Dashboard** in your sub-account
2. Click **Customize** to add/remove/rearrange widgets
3. Available widget types:
   - Contact stats (new, total, by source)
   - Opportunity stats (by pipeline, stage, value)
   - Appointment stats (by calendar, status)
   - Conversation metrics
   - Custom field aggregations
   - Revenue tracking

---

## Key Reports

### Contact Reports

| Report                  | What It Shows                                               |
| ----------------------- | ----------------------------------------------------------- |
| **Source Report**       | Where contacts came from (Facebook, Google, referral, etc.) |
| **Tag Distribution**    | Contact count by tag                                        |
| **Growth Over Time**    | New contacts per day/week/month                             |
| **DND Report**          | Contacts with Do Not Disturb enabled                        |
| **Custom Field Report** | Breakdown by any custom field value                         |

### Pipeline & Sales Reports

| Report                    | What It Shows                                     |
| ------------------------- | ------------------------------------------------- |
| **Pipeline Overview**     | Opportunities per stage, values, conversion rates |
| **Velocity Report**       | Average time in each pipeline stage               |
| **Win/Loss Analysis**     | Won vs. lost deals, reasons for loss              |
| **Sales Rep Performance** | Deals per rep, value per rep, close rate          |
| **Revenue Forecast**      | Projected revenue based on pipeline               |
| **Opportunity Source**    | Which lead sources produce the most deals         |

### Communication Reports

| Report                  | What It Shows                                        |
| ----------------------- | ---------------------------------------------------- |
| **Email Performance**   | Open rate, click rate, bounce rate, unsubscribe rate |
| **SMS Performance**     | Delivery rate, response rate, opt-out rate           |
| **Response Time**       | Average time to first response                       |
| **Conversation Volume** | Messages sent/received over time                     |
| **Channel Breakdown**   | Volume by channel (email, SMS, call, chat)           |

### Appointment Reports

| Report                   | What It Shows                      |
| ------------------------ | ---------------------------------- |
| **Booking Rate**         | Appointments booked vs. page views |
| **Show Rate**            | Appointments showed vs. booked     |
| **No-Show Rate**         | Percentage of no-shows             |
| **Calendar Utilization** | Booked vs. available slots         |
| **Rep Distribution**     | Appointments per team member       |

---

## Attribution Tracking

### UTM Parameters

Track the source of every lead using UTM parameters in your marketing URLs:

```
https://yourfunnel.com/opt-in
  ?utm_source=facebook
  &utm_medium=paid
  &utm_campaign=spring_promo
  &utm_content=video_ad_v2
  &utm_term=dental_implants
```

### GHL Attribution Fields

| Field          | Description                              |
| -------------- | ---------------------------------------- |
| `utm_source`   | Traffic source (facebook, google, email) |
| `utm_medium`   | Marketing medium (paid, organic, cpc)    |
| `utm_campaign` | Campaign name                            |
| `utm_content`  | Ad/content variation                     |
| `utm_term`     | Keyword (for paid search)                |
| `source`       | GHL-specific source field                |
| `referrer`     | URL the contact came from                |

### Attribution Best Practices

1. **Use UTMs on every link** — Marketing emails, ads, social posts
2. **Be consistent** — Use lowercase, hyphens, and standard naming
3. **Map to custom fields** — Store UTM data in dedicated contact fields
4. **Report by source** — Identify which channels drive the most revenue
5. **Calculate ROI** — Compare ad spend against attributed revenue

### UTM Naming Convention

```
Source:    facebook, google, tiktok, email, organic
Medium:   paid, cpc, social, email, referral
Campaign: [year]-[month]-[name] (e.g., 2025-02-spring-promo)
Content:  [ad-type]-[variation] (e.g., video-ad-v2)
```

---

## Call Tracking

### Setup

1. Purchase tracking phone numbers (LC Phone)
2. Assign numbers to specific campaigns or channels
3. Forward calls to your main business line
4. GHL records call source, duration, and outcome

### Call Metrics

| Metric                 | Description                               |
| ---------------------- | ----------------------------------------- |
| **Call Volume**        | Total inbound/outbound calls              |
| **Call Duration**      | Average call length                       |
| **Missed Calls**       | Calls not answered                        |
| **First-Time Callers** | New callers vs. returning                 |
| **Source Tracking**    | Which campaign/channel generated the call |
| **Call Outcome**       | Booked, not interested, follow-up needed  |

---

## Ad Reporting

### Connected Ad Platforms

- Facebook / Instagram Ads
- Google Ads
- TikTok Ads

### Ad Metrics Available in GHL

| Metric          | Description                    |
| --------------- | ------------------------------ |
| **Impressions** | Times the ad was shown         |
| **Clicks**      | Clicks on the ad               |
| **CTR**         | Click-through rate             |
| **CPC**         | Cost per click                 |
| **Leads**       | Form submissions / conversions |
| **CPL**         | Cost per lead                  |
| **ROAS**        | Return on ad spend             |

### Integration Setup

1. Go to **Settings → Integrations**
2. Connect your Facebook, Google, or TikTok ad accounts
3. View ad performance alongside CRM data
4. Map ad leads to pipeline opportunities for full-funnel tracking

---

## KPI Scorecard

### Essential Metrics to Track

| Category       | KPI                       | Target            |
| -------------- | ------------------------- | ----------------- |
| **Lead Gen**   | New leads per week        | Industry-specific |
| **Lead Gen**   | Cost per lead (CPL)       | Decreasing trend  |
| **Speed**      | First response time       | < 5 minutes       |
| **Sales**      | Lead-to-appointment rate  | > 20%             |
| **Sales**      | Appointment show rate     | > 80%             |
| **Sales**      | Close rate                | > 20%             |
| **Revenue**    | Average deal value        | Increasing trend  |
| **Revenue**    | Monthly recurring revenue | Growing           |
| **Retention**  | Client churn rate         | < 5% monthly      |
| **Reputation** | Average review rating     | ≥ 4.5 stars       |
| **Reputation** | Reviews per month         | > 10              |
| **Email**      | Open rate                 | > 25%             |
| **SMS**        | Response rate             | > 15%             |

### Building a Weekly Scorecard

Create a simple reporting workflow:

```
Trigger: Recurring schedule (every Monday)
→ Action: Compile key metrics
→ Action: Send internal email to team
   Subject: "Weekly Performance Scorecard — Week of {{date}}"
   Body:
   - New Leads: [count]
   - Appointments Booked: [count]
   - Show Rate: [%]
   - Deals Closed: [count]
   - Revenue: [$]
   - Reviews Received: [count]
```

---

## Reporting for Agency Clients

### Client-Facing Reports

1. **What to include:**
   - Lead volume and sources
   - Appointment metrics
   - Pipeline progress
   - Communication stats
   - ROI calculations

2. **What NOT to include:**
   - Internal team performance breakdowns
   - Cost data (unless agreed upon)
   - Technical metrics they won't understand

3. **Delivery format:**
   - Weekly summary email (automated)
   - Monthly PDF report
   - Live dashboard access (via white-labeled GHL)

### Report Automation

Use GHL workflows to automate report generation and delivery:

- Schedule weekly email summaries
- Track and send key metric updates automatically
- Alert clients when milestones are hit (e.g., "50 leads this month!")

---

## Key Resources

- **Dashboard**: Main dashboard in sub-account
- **Reporting**: Reporting tab for detailed analytics
- **Ad Manager**: Settings → Integrations → Ad accounts
- **Help Docs**: [GoHighLevel Reporting Guide](https://help.gohighlevel.com)
