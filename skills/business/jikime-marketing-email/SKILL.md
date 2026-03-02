---
name: jikime-marketing-email
description: Email marketing and automation specialist for creating email sequences, drip campaigns, lifecycle emails, and nurture flows.
version: 1.0.0
tags: ["marketing", "email", "automation", "drip-campaign", "lifecycle"]
triggers:
  keywords: ["email sequence", "drip campaign", "nurture", "welcome email", "onboarding email", "email automation", "lifecycle email", "이메일 마케팅", "이메일 시퀀스", "자동화 메일"]
  phases: ["plan", "run"]
  agents: ["frontend", "backend", "manager-strategy"]
  languages: []
# Progressive Disclosure Configuration
progressive_disclosure:
  enabled: true
  level1_tokens: ~100
  level2_tokens: ~8000
user-invocable: true
context: fork
agent: general-purpose
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - Task
  - TodoWrite
---

# Email Sequence Design

## Quick Reference (30 seconds)

Email Marketing Specialist - Create email sequences that nurture relationships, drive action, and convert interest into customers.

Core Principles:
- **One Email, One Job**: Single purpose, single CTA per email
- **Value Before Ask**: Lead with usefulness, earn the right to sell
- **Relevance Over Volume**: Fewer, better emails always win

Key Deliverables:
- Welcome sequences (3-7 emails)
- Lead nurture sequences (5-10 emails)
- Re-engagement sequences (3-5 emails)
- Onboarding sequences (5-10 emails)

---

## Sequence Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                    EMAIL SEQUENCE TYPES                          │
├─────────────────────────────────────────────────────────────────┤
│  WELCOME         NURTURE          RE-ENGAGE        ONBOARDING   │
│  ───────         ───────          ─────────        ──────────   │
│  3-7 emails      5-10 emails      3-5 emails       5-10 emails  │
│  Post-signup     Pre-sale         30-60d inactive  Product user │
│  Build trust     Educate          Win back         Activate     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Welcome Sequence Template

| Email | Timing | Subject Pattern | Purpose |
|-------|--------|-----------------|---------|
| 1 | Immediate | Welcome to [Product] | Deliver promise, set expectations |
| 2 | Day 1-2 | Get your first [result] in 10 min | Enable quick win |
| 3 | Day 3-4 | Why we built [Product] | Connect emotionally |
| 4 | Day 5-6 | How [Customer] achieved [Result] | Social proof |
| 5 | Day 7-8 | "I don't have time" — sound familiar? | Handle objection |
| 6 | Day 9-11 | Have you tried [Feature]? | Feature discovery |
| 7 | Day 12-14 | Ready to [upgrade/commit]? | Convert |

---

## Lead Nurture Sequence Template

| Email | Timing | Focus | CTA Type |
|-------|--------|-------|----------|
| 1 | Immediate | Deliver lead magnet + introduce | Light (content) |
| 2 | Day 2-3 | Expand on topic | Content link |
| 3 | Day 4-5 | Problem deep-dive | Self-reflection |
| 4 | Day 6-8 | Solution framework | Educational |
| 5 | Day 9-11 | Case study | Soft CTA |
| 6 | Day 12-14 | Differentiation | Comparison |
| 7 | Day 15-18 | Objection handler | FAQ/resources |
| 8 | Day 19-21 | Direct offer | Clear pitch |

---

## Re-Engagement Sequence Template

| Email | Timing | Subject Pattern | Approach |
|-------|--------|-----------------|----------|
| 1 | Day 30-60 inactive | Is everything okay, [Name]? | Genuine concern |
| 2 | +2-3 days | Remember when you [achieved X]? | Value reminder |
| 3 | +5-7 days | We miss you — here's something special | Incentive offer |
| 4 | +10-14 days | Should we stop emailing you? | Final chance |

---

## Lifecycle Email Audit Checklist

### Onboarding Emails
```
□ New users series (5-7 emails)
□ New customers series (3-5 emails)
□ Key onboarding step reminders
□ New user invite sequence
```

### Retention Emails
```
□ Upgrade to paid sequence
□ Upgrade to higher plan triggers
□ Ask for review (post-milestone)
□ Proactive support outreach
□ Product usage reports
□ NPS survey
□ Referral program emails
```

### Billing Emails
```
□ Switch to annual campaign
□ Failed payment recovery sequence (3-4 emails)
□ Cancellation survey
□ Upcoming renewal reminders
```

### Win-Back Emails
```
□ Expired trial sequence (3-4 emails)
□ Cancelled customer sequence (2-3 emails over 90 days)
```

---

## Email Copy Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  EMAIL ANATOMY                                                   │
├─────────────────────────────────────────────────────────────────┤
│  1. HOOK          First line grabs attention                    │
│  2. CONTEXT       Why this matters to them                      │
│  3. VALUE         The useful content                            │
│  4. CTA           What to do next                               │
│  5. SIGN-OFF      Human, warm close                             │
└─────────────────────────────────────────────────────────────────┘
```

### Copy Guidelines

| Element | Best Practice |
|---------|---------------|
| **Subject** | 40-60 chars, clear > clever, benefit-driven |
| **Preview** | 90-140 chars, extends subject, don't repeat |
| **Paragraphs** | 1-3 sentences, lots of white space |
| **Tone** | Conversational, first-person, active voice |
| **Length** | 50-125 words (transactional), 150-300 (educational) |

### Subject Line Patterns

| Pattern | Example |
|---------|---------|
| Question | Still struggling with X? |
| How-to | How to [achieve outcome] in [timeframe] |
| Number | 3 ways to [benefit] |
| Direct | [First name], your [thing] is ready |
| Story tease | The mistake I made with [topic] |

---

## Billing & Recovery Emails

### Failed Payment Recovery Sequence

| Email | Timing | Subject | Tone |
|-------|--------|---------|------|
| 1 | Day 0 | Quick update about your account | Friendly notice |
| 2 | Day 3 | Reminder: Update your payment | Reminder |
| 3 | Day 7 | Urgent: Service may be interrupted | Urgent |
| 4 | Day 10-14 | Final notice: What you'll lose | Final chance |

**Key principles:**
- Assume accident (card expired)
- Clear, direct, no guilt
- Single CTA to update payment
- Explain consequences

---

## Personalization & Segmentation

### Merge Fields
- First name (fallback: "there" or "friend")
- Company name (B2B)
- Usage data (activity, plan, etc.)

### Segmentation Strategies

| Segment By | Examples |
|------------|----------|
| **Behavior** | Openers, clickers, active, inactive |
| **Stage** | Trial, paid, new, long-term |
| **Profile** | Industry, role, company size |

---

## Metrics & Benchmarks

| Metric | Benchmark | Notes |
|--------|-----------|-------|
| **Open rate** | 20-40% | Subject line effectiveness |
| **Click rate** | 2-5% | CTA and content quality |
| **Unsubscribe** | < 0.5% | List health indicator |
| **Conversion** | Varies | Sequence-specific goal |

---

## Output Format

### Sequence Overview
```
Sequence Name: [Name]
Trigger: [What starts the sequence]
Goal: [Primary conversion goal]
Length: [Number of emails]
Timing: [Delay between emails]
Exit Conditions: [When they leave]
```

### Per Email
```
Email [#]: [Name/Purpose]
Send: [Timing]
Subject: [Subject line]
Preview: [Preview text]
Body: [Full copy]
CTA: [Button text] → [Link destination]
Segment/Conditions: [If applicable]
```

---

## Works Well With

Skills:
- `jikime-marketing-onboarding` - Coordinate email with in-app onboarding
- `jikime-marketing-copywriting` - Write landing pages emails link to
- `jikime-marketing-ab-test` - Test email elements
- `jikime-marketing-psychology` - Apply psychological triggers

---

Version: 1.0.0
Last Updated: 2026-01-25
Attribution: Enhanced from marketingskills by Corey Haines (MIT License)
