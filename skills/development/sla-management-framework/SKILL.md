---
name: sla-management-framework
description: "SLAs for rev ops: response times, escalation, enforcement. Use when defining or managing sales and marketing SLAs."
---

# SLA Management — Framework

Service Level Agreements for revenue operations: response times, escalation processes, and enforcement mechanisms.

---

## Why SLAs Matter

**78% of deals go to the vendor that responds first.** Response speed is the single biggest controllable factor in conversion.

The reality:
- Average B2B lead response time: **42-47 hours**
- 63% of businesses **never respond** to inbound leads
- Only 17% respond instantly

The impact:
- Sub-5-minute response = **21x more likely** to qualify
- Sub-1-minute response = **391% higher conversion** (Velocify)
- Within 1 hour = **53% conversion** vs 17% after 24 hours

---

## SLA Types

### Response SLAs

Time between a lead action and the first sales contact attempt.

| Lead Type | Recommended SLA | Rationale |
|---|---|---|
| **Demo Request** | < 5 minutes | High-intent; buyers are actively evaluating |
| **Pricing Inquiry** | < 5 minutes | Clear buying signal |
| **Inbound Call** | Immediate | Live prospect on the line |
| **Free Trial Signup** | < 10 minutes | Active evaluation mode |
| **Content Download** | 2-24 hours | Allow time to consume content |
| **Webinar Registration** | Post-event or 24h | Contact relative to attendance |
| **Newsletter Signup** | 24-48 hours | Low-intent, nurture appropriate |

### Follow-Up SLAs

Cadence and timing for subsequent contact attempts.

| Attempt | Timing | Channel | Purpose |
|---|---|---|---|
| 1st | Within SLA | Call + Email | Initial connection |
| 2nd | +1 day | Email with social proof | Value reinforcement |
| 3rd | +3 days | Call | Persistence |
| 4th-7th | Over 2 weeks | Multi-channel mix | Sustained outreach |
| 8th+ | Weekly or nurture | Email cadence | Long-term nurture |

**Benchmark:** 8-12 total attempts is optimal. 7+ touches typically required before buying decision.

### Handoff SLAs (Marketing to Sales)

| Stage | SLA | Owner |
|---|---|---|
| MQL to Sales Assignment | < 15 minutes | Automation |
| Sales Review/Accept | < 2 hours | SDR/AE |
| First Contact Attempt | < 5 minutes after acceptance | SDR/AE |
| SQL Disposition/Feedback | Within 7 days | SDR/AE |

### Meeting SLAs

| Action | SLA | Owner |
|---|---|---|
| Calendar invite sent | Within 1 hour of booking | SDR |
| Pre-meeting research complete | 24 hours before meeting | AE |
| Meeting reminder sent | 24 hours + 1 hour before | Automation |
| Post-meeting notes logged | Within 2 hours | AE |

---

## SLA by Lead Tier

### Enterprise Leads (1,000+ employees / $1B+ revenue)

| SLA Type | Target |
|---|---|
| Response time (all high-intent) | < 5 minutes |
| Support availability | 24/7 |
| Dedicated account management | Required |
| Escalation response | 1-4 hours |

### Mid-Market Leads (100-999 employees / $50M-$1B revenue)

| SLA Type | Target |
|---|---|
| Response time (demos) | < 5 minutes |
| Response time (other) | < 1 hour |
| Support availability | Business hours + extended |
| Account management | Pooled or dedicated |
| Escalation response | 4-8 hours |

### SMB Leads (<100 employees / <$50M revenue)

| SLA Type | Target |
|---|---|
| Response time (demos) | < 15 minutes |
| Response time (other) | < 4 hours |
| Support availability | Business hours |
| Account management | Self-serve + CSM support |
| Escalation response | 24 hours |

---

## SLA by Lead Source

| Lead Source | Intent Level | Response SLA | Follow-Up Cadence |
|---|---|---|---|
| Demo Request | High | < 5 min | 6 touches in 48h |
| Pricing Inquiry | High | < 5 min | 6 touches in 48h |
| Free Trial | High | < 10 min | Daily for 7 days |
| G2/Capterra Intent | High | < 15 min | 5 touches in 72h |
| Webinar Attendee | Medium | < 2 hours post-event | 4 touches in week |
| Content Download | Medium | 2-24 hours | 3 touches in week |
| Newsletter Signup | Low | 24-48 hours | Nurture sequence |
| Outbound Reply | Medium-High | < 30 min | 5 touches in 72h |

---

## Escalation Matrix

### Escalation Levels

| Level | Timeframe | Escalated To | Action Required |
|---|---|---|---|
| **L1** | SLA at 50% | Assigned Rep | Reminder notification |
| **L2** | SLA at 75% | Rep + Manager | Warning + priority flag |
| **L3** | SLA Breached | Manager | Immediate reassignment or assist |
| **L4** | 2x SLA | Director/VP | Review + root cause analysis |
| **L5** | Pattern/Repeat | Executive | Process/resource intervention |

### Escalation Triggers by SLA Type

**Response SLA Breach:**
```
50% elapsed  → Slack/email reminder to rep
75% elapsed  → Manager cc'd, lead flagged urgent
100% elapsed → Auto-reassign to backup rep or round-robin
200% elapsed → VP notified, incident logged
```

**Follow-Up SLA Breach:**
```
Missed attempt     → Task auto-created for next business hour
2 missed attempts  → Manager alerted
3+ missed attempts → Lead recycled with disposition required
```

**Meeting SLA Breach:**
```
No-prep 12h before → Manager notified
No-show without reschedule → Auto-escalate + SDR re-engagement
```

### Escalation Path by Severity

| Severity | Criteria | Escalation Path |
|---|---|---|
| **Warning** | 50-74% SLA elapsed | SDR/AE (self-correct) |
| **At-Risk** | 75-99% SLA elapsed | SDR Manager (intervene) |
| **Breach** | 100% SLA elapsed | Sales Director (reassign) |
| **Critical** | 150%+ SLA elapsed | VP Sales (process review) |

---

## HubSpot SLA Configuration

### Workflow: Response SLA Tracking

```
Trigger: Contact Property "Lead Source" is any of [Demo Request, Pricing Inquiry]

Actions:
1. Set property "SLA Start Time" = Current timestamp
2. Set property "SLA Deadline" = SLA Start Time + 5 minutes
3. Set property "SLA Status" = "Active"

Branch: If (Current time - SLA Start Time) > 2.5 minutes
  → Send internal notification "SLA Warning - 50% elapsed"

Branch: If (Current time - SLA Start Time) > 4 minutes
  → Send internal notification "SLA At Risk - 75% elapsed"
  → Set property "SLA Status" = "At Risk"
  → Add to list "SLA Escalation Review"
```

### Workflow: SLA Breach Response

```
Trigger: Contact Property "SLA Status" = "Breached"

Actions:
1. Reassign to backup owner (round-robin)
2. Send notification to original owner
3. Send notification to manager
4. Log activity "SLA Breach - Auto-reassigned"
5. Add to list "SLA Breach Review"
6. Create task for manager "Review SLA breach - [Contact Name]"
```

### Workflow: Speed-to-Lead Automation

```
Trigger: Form Submission (Demo Request form)

Actions:
1. Assign owner based on territory/round-robin
2. Create task "Call lead" due in 5 minutes
3. Send internal SMS notification via Twilio
4. Start SLA timer (set SLA Start Time)
5. Delay 5 minutes
6. Branch: If no activity logged
   → Escalate to manager
   → Reassign via round-robin
```

### SLA Dashboard Metrics

| Metric | Calculation | Target |
|---|---|---|
| % Leads contacted within SLA | Contacts within SLA / Total contacts | >95% |
| Average response time by rep | Avg (First contact time - Lead create time) | < SLA target |
| SLA breach count by period | Count of breached leads per week/month | < 5% |
| Escalation frequency by level | Count per level per period | Declining trend |
| Response time trend | Weekly average response time | Improving |

---

## Consequences for SLA Breach

### Immediate Consequences

| Consequence | Implementation |
|---|---|
| **Lead Reassignment** | Auto-transfer to available rep or manager |
| **Notification Cascade** | Rep, manager, director notified |
| **Priority Override** | Lead flagged as urgent in queue |
| **Activity Logging** | Breach recorded in contact timeline |

### Performance Management Consequences

| Breach Frequency | Consequence |
|---|---|
| 1 breach/week | Verbal coaching |
| 2-3 breaches/week | Written warning + training |
| 4+ breaches/week | Performance improvement plan |
| Chronic pattern | Role reassessment |

### Business Consequences

- **Revenue Loss:** 78% of deals go to first responder
- **Pipeline Leakage:** Slower response = lower conversion
- **Customer Experience:** Negative first impression
- **Competitive Disadvantage:** Competitors win on speed

### Compensation Implications

| Performance Level | Impact |
|---|---|
| SLA compliance >95% | Full commission eligible |
| SLA compliance 80-95% | Commission reduction warning |
| SLA compliance <80% | Commission reduction (5-10%) |
| Chronic non-compliance | Accelerator disqualification |

---

## Common SLA Mistakes

| Mistake | Impact | Fix |
|---|---|---|
| Vague targets | "Respond quickly" is unmeasurable | Define specific timeframes |
| Unrealistic targets | 1-minute SLA without resources | Audit capacity, then set targets |
| One-size-fits-all | Same SLA for demos and content | Tier by source and intent |
| No escalation process | Breaches with no correction | Automated escalation matrix |
| Insufficient monitoring | Monthly review instead of real-time | Live dashboards, automated alerts |
| Poor communication | Marketing doesn't know disposition | Shared CRM, mandatory disposition |
| Static SLAs | Set once, never revisited | Quarterly review based on data |
| No consequences | SLAs exist on paper only | Tie to comp, recognition, career |
| Missing feedback loop | No lead quality reports | Required disposition within 7 days |
| Manual tracking | Human error, delays | Automated triggers and workflows |

---

## Implementation Checklist

### Phase 1: Baseline (Week 1)

- [ ] Audit current response times by lead source
- [ ] Document current (informal) SLAs
- [ ] Identify biggest gaps vs. benchmarks
- [ ] Gather input from sales and marketing

### Phase 2: Design (Week 2)

- [ ] Define SLA targets by tier and source
- [ ] Design escalation matrix with clear owners
- [ ] Define consequence framework
- [ ] Get leadership alignment on enforcement

### Phase 3: Automation (Week 3-4)

- [ ] Configure HubSpot/CRM workflows for tracking
- [ ] Set up automated escalation triggers
- [ ] Build real-time dashboard for monitoring
- [ ] Test with sample leads

### Phase 4: Rollout (Week 5-6)

- [ ] Train team on SLA importance and process
- [ ] Communicate expectations clearly
- [ ] Monitor closely during first 2 weeks
- [ ] Adjust based on early feedback

### Phase 5: Optimization (Ongoing)

- [ ] Weekly SLA performance review
- [ ] Monthly analysis of breach patterns
- [ ] Quarterly SLA target adjustments
- [ ] Continuous feedback loop improvement

---

## Cross-References

- **Lead Routing:** See `lead-management/lead-routing-framework.md`
- **Escalation Playbooks:** See `sla-management-escalation.md`
- **HubSpot Automation:** See `sla-management-automation.md`
- **Territory Planning:** See `territory-planning-framework.md`

---

> **Built by [ColdIQ](https://www.coldiq.com) & [Ivan Falco](https://www.linkedin.com/in/ivanfalco/en/).** For questions on implementation or anything not covered here, reach out to Ivan directly on [LinkedIn](https://www.linkedin.com/in/ivanfalco/en/).
