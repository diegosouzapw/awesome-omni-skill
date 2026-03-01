---
name: Onboarding Check-in Drafter
description: |
  Draft onboarding check-in emails at 7, 14, and 30 days after deal close.
  Use when an onboarding milestone triggers or user asks "draft onboarding check-in",
  "send new customer welcome", or needs to proactively engage new accounts.
  Returns stage-appropriate check-in with setup assistance, adoption tips, or expansion conversation.
metadata:
  author: sixty-ai
  version: "2"
  category: sales-ai
  skill_type: atomic
  is_active: true
  context_profile: account
  agent_affinity:
    - pipeline
    - meetings
  triggers:
    - pattern: "draft onboarding check-in"
      intent: "onboarding_checkin_email"
      confidence: 0.90
      examples:
        - "write onboarding check-in email"
        - "prepare customer welcome check-in"
        - "draft new account check-in"
    - pattern: "new customer check-in"
      intent: "new_customer_outreach"
      confidence: 0.85
      examples:
        - "send welcome email to new customer"
        - "onboarding follow-up"
        - "new account outreach"
    - pattern: "7 day check-in"
      intent: "early_onboarding"
      confidence: 0.80
      examples:
        - "day 7 onboarding email"
        - "first week check-in"
        - "welcome email after onboarding"
  keywords:
    - "onboarding"
    - "check-in"
    - "welcome"
    - "new customer"
    - "new account"
    - "setup"
    - "adoption"
    - "day 7"
    - "day 14"
    - "day 30"
  required_context:
    - deal_id
    - contact_id
    - company_name
  inputs:
    - name: deal_id
      type: string
      description: "The deal identifier for the newly closed account"
      required: true
    - name: contact_id
      type: string
      description: "Primary onboarding contact"
      required: false
    - name: days_since_close
      type: number
      description: "Number of days since deal close (7, 14, or 30)"
      required: true
    - name: product_purchased
      type: string
      description: "Product or service purchased"
      required: false
  outputs:
    - name: email_draft
      type: object
      description: "Onboarding check-in email with stage-appropriate tone and content"
    - name: suggested_resources
      type: array
      description: "Helpful resources to include based on onboarding stage"
    - name: next_checkin_date
      type: string
      description: "Recommended date for next check-in milestone"
  priority: high
  requires_capabilities:
    - crm
    - email
---

## Available Context & Tools
@_platform-references/org-variables.md
@_platform-references/capabilities.md

# Onboarding Check-in Drafter

## Goal
Generate stage-appropriate onboarding check-in emails at 7, 14, and 30 days after deal close. Each stage has a different tone and purpose: Day 7 focuses on setup assistance and quick wins, Day 14 checks progress and provides adoption tips, and Day 30 reviews success and opens expansion conversations. The tone evolves from tactical to strategic as the customer matures.

## Why Proactive Onboarding Check-ins Matter

Structured onboarding check-ins are the single strongest predictor of customer retention and expansion:

- **Customers who receive a proactive check-in at day 7 have 41% higher feature adoption rates** at day 30 compared to those who receive no outreach (Gainsight Onboarding Benchmarks, 2024).
- **Day 14 check-ins reduce time-to-value by 23%** by surfacing blockers before they compound (ChurnZero retention study).
- **30-day check-ins uncover expansion opportunities in 38% of accounts** when framed as success reviews vs. product pitches (SaaStr Customer Success Survey).
- **67% of customers churn within the first 90 days if they don't achieve an early win** (ProfitWell retention data) — proactive check-ins identify and create those wins.
- **Customers who engage with onboarding resources shared in check-in emails have 2.4x higher lifetime value** (Intercom Product Engagement Study).

The key insight: proactive onboarding is not about being helpful — it is about engineering success before the customer encounters friction.

## Required Capabilities
- **CRM**: To fetch deal data, contact details, onboarding activity, usage metrics
- **Email**: To draft and send onboarding check-in communications

## Inputs
- `deal_id`: The deal identifier for the newly closed account (required)
- `contact_id`: Primary onboarding contact (optional — will infer from deal if not provided)
- `days_since_close`: Number of days since deal closed — must be 7, 14, or 30 (required)
- `product_purchased`: Product or service purchased (optional — will infer from deal if not provided)

## Data Gathering (via execute_action)

Gather onboarding context before drafting:

1. **Deal record**: `execute_action("get_deal", { id: deal_id })` — close date, value, products/services purchased
2. **Contact details**: `execute_action("get_contact", { id: contact_id })` — primary stakeholder for onboarding
3. **Activity history**: `execute_action("get_deal_activities", { deal_id, limit: 20 })` — onboarding meetings, emails, support tickets
4. **Open tasks**: `execute_action("list_tasks", { deal_id })` — pending onboarding deliverables, setup steps
5. **Recent meetings**: Check for kickoff calls, training sessions, implementation check-ins
6. **Usage data** (if available): Product usage metrics, feature adoption, user activation

If data calls fail, note missing information and draft a more generic check-in with a discovery-first CTA.

## Three-Stage Onboarding Cadence

Onboarding check-ins follow a 7-14-30 day cadence, each with a distinct purpose and tone:

### Day 7: Welcome, Setup Assistance, Quick Wins
**Purpose**: Ensure successful initial setup and identify early blockers.

**Tone**: Warm, hands-on, tactical. "We're here to help you get started."

**Key Content**:
- Acknowledge the partnership and express enthusiasm
- Offer specific setup assistance (integrations, user onboarding, configuration)
- Share 1-2 quick win resources (getting started guide, video tutorial, best practices doc)
- Propose a brief check-in call to ensure smooth launch
- Low-pressure, high-support

**CTA**: "Would a quick 15-minute call next week help? Happy to walk through any setup questions."

### Day 14: Progress Check, Adoption Metrics, Tips
**Purpose**: Assess adoption progress, surface blockers, provide optimization tips.

**Tone**: Helpful, progress-oriented, collaborative. "Let's make sure you're getting the most out of this."

**Key Content**:
- Acknowledge progress since launch (usage data, features activated, users onboarded)
- Surface any adoption gaps or underutilized features
- Share advanced tips or use case examples relevant to their industry/role
- Offer training resources or team enablement sessions
- Check if they have questions or blockers

**CTA**: "I noticed your team is using [Feature A] heavily — would a 20-minute deep dive on [Feature B] be helpful for your use case?"

### Day 30: Success Review, Value Realization, Expansion Conversation
**Purpose**: Celebrate early wins, quantify value delivered, open expansion discussion.

**Tone**: Strategic, partnership-focused, growth-oriented. "Let's review what's working and plan your next phase."

**Key Content**:
- Recap value delivered in the first 30 days (metrics, wins, time saved)
- Acknowledge team's adoption efforts and successes
- Surface any remaining gaps or optimization opportunities
- Introduce expansion possibilities based on usage patterns or goals
- Propose a success review call (mini-QBR)

**CTA**: "Let's schedule a 30-minute success review to discuss what's working, address any gaps, and explore how we can support your team's next growth phase. Would [date/time options] work?"

## Email Structure by Stage

### Day 7 Email Structure (5 sections)

#### Section 1: Welcome + Partnership Acknowledgment (2-3 sentences)
**Rules**:
- Lead with enthusiasm and gratitude for the partnership
- Reference the specific product/service purchased
- Acknowledge the close date or kickoff milestone
- Use the contact's name and personalize based on sales process

**Good example**: "Sarah, welcome to ${company_name}! We're excited to partner with Acme on your data integration platform. Now that we've completed the kickoff and initial setup, I wanted to check in and make sure your team has everything needed to get started smoothly."

**Bad example**: "Hi Sarah, welcome aboard. Let me know if you need help."

#### Section 2: Setup Assistance Offer (2-3 sentences)
**Rules**:
- Acknowledge that setup can be complex
- Offer specific help areas (integrations, user onboarding, configuration)
- Make it easy to ask for help ("no question is too small")

**Good example**: "Getting your API integrations and user permissions configured can be tricky in the first week. If you run into any setup questions — authentication, data sync, or anything else — I'm here to help. No question is too small."

#### Section 3: Quick Win Resources (2-3 bullet points)
**Rules**:
- Share 1-3 resources that help them achieve an early win
- Make resources specific to their use case or industry
- Include links (getting started guide, video tutorial, template)
- Frame as "here's what helps most customers like you get value fast"

**Good example**:
```
Resources to help you get started:
- [Getting Started Guide]: 5-minute walkthrough of core features
- [API Integration Template]: Pre-built config for your tech stack
- [Quick Wins Checklist]: 3 high-impact actions you can complete this week
```

#### Section 4: Support Availability (1-2 sentences)
**Rules**:
- Reassure them that help is available (support channels, your availability)
- Make it clear they are not bothering you by asking questions
- Offer a brief onboarding call if helpful

**Good example**: "I'm available if you'd like a quick 15-minute call to walk through any setup questions next week — just let me know what works for your schedule."

#### Section 5: CTA (1 sentence)
**Rules**:
- Offer a low-friction next step (optional call, resource access, support availability)
- No hard deadlines or required actions
- Frame as "I'm here to help" not "you must do this"

**Good example**: "Would a quick call next Tuesday or Wednesday morning be helpful, or are you all set for now?"

### Day 14 Email Structure (5 sections)

#### Section 1: Progress Acknowledgment (2-3 sentences)
**Rules**:
- Reference specific adoption progress (features activated, users onboarded, usage metrics)
- Celebrate early wins ("great to see your team already...")
- Acknowledge their effort and engagement

**Good example**: "Sarah, great to see your team is already processing 10K+ API calls per day — that's strong adoption for the first two weeks. I noticed you've onboarded 8 users and activated the core data sync module."

#### Section 2: Adoption Gaps or Optimization Tips (2-3 sentences)
**Rules**:
- Surface underutilized features that could help them
- Frame as helpful tips, not criticism ("you're missing out on...")
- Use "I noticed..." language to show you're paying attention

**Good example**: "I noticed your team hasn't activated the automated reconciliation module yet — that's typically the #1 time-saver for data engineering teams like yours. Would a quick walkthrough be helpful?"

#### Section 3: Relevant Resources or Training (2-3 bullet points)
**Rules**:
- Share resources specific to their adoption stage
- Include advanced tips, use case examples, or training sessions
- Make resources actionable and concise

**Good example**:
```
Resources to optimize your setup:
- [Advanced Data Sync Guide]: Tips for high-volume API usage
- [Industry Use Case]: How fintech teams like yours use reconciliation automation
- [Training Session Invite]: 30-minute team enablement on advanced features (optional)
```

#### Section 4: Blocker Check (1-2 sentences)
**Rules**:
- Explicitly ask if they are encountering any blockers
- Make it safe to admit struggles ("onboarding always has bumps")
- Offer to help resolve issues

**Good example**: "Onboarding always has a few bumps — if you're running into any issues with setup, performance, or feature access, let's troubleshoot together."

#### Section 5: CTA (1-2 sentences)
**Rules**:
- Offer a training deep-dive, optimization call, or team enablement session
- Frame as optional but valuable
- Provide specific time options

**Good example**: "Would a 20-minute deep dive on the reconciliation module be helpful for your team? I have availability Tuesday at 10am or Thursday at 2pm if either works."

### Day 30 Email Structure (5 sections)

#### Section 1: Success Recap (2-3 sentences)
**Rules**:
- Lead with value delivered in the first 30 days
- Include quantified metrics if available (usage, time saved, efficiency gains)
- Frame as a partnership win ("together, we've...")

**Good example**: "Sarah, we've hit the 30-day mark since your team launched ${company_name}. In that time, your team has processed 320K+ API calls, reduced manual reconciliation time by an estimated 18 hours/week, and onboarded 12 users with strong adoption across core features."

#### Section 2: Team Acknowledgment (1-2 sentences)
**Rules**:
- Celebrate the customer team's effort and success
- Acknowledge specific wins or milestones they achieved
- Use "your team" language (give them credit)

**Good example**: "Your engineering team's quick adoption of the advanced modules has been impressive — most teams take 60+ days to activate those features."

#### Section 3: Remaining Gaps or Optimization (2-3 sentences)
**Rules**:
- Surface any underutilized features or optimization opportunities
- Frame as "what's next" not "what you're missing"
- Be honest about areas where they could get more value

**Good example**: "I noticed you're not yet using the audit log export feature — given your SOC 2 compliance focus, that could save your team significant manual work during audits. Worth exploring?"

#### Section 4: Expansion Opportunity (if applicable)
**Rules**:
- Only include if there is genuine evidence of expansion fit (usage patterns, team growth, feature requests)
- Frame as solving THEIR problem, not upselling YOUR product
- Make it conversational, not pushy

**Good example**: "Your team has grown from 8 to 12 users and you're approaching the API call limits on your current plan. If you're planning to continue scaling, our Growth plan might be worth discussing — it includes unlimited API calls and multi-team permissions."

**Skip this section if**: No clear expansion fit exists.

#### Section 5: Success Review CTA (1-2 sentences)
**Rules**:
- Propose a 30-minute success review call (mini-QBR)
- Frame as mutual benefit ("let's review what's working and plan next steps")
- Provide specific time options

**Good example**: "Let's schedule a 30-minute success review next week to discuss what's working, address any remaining gaps, and plan your next phase of growth. Would Tuesday March 4 at 2pm or Thursday March 6 at 10am work?"

## Tone Calibration by Stage

**Day 7 Tone**: Warm, supportive, hands-on
- Language: "We're here to help", "Getting started", "Let us know if you need anything"
- Emotion: Enthusiasm, accessibility, service-oriented

**Day 14 Tone**: Helpful, progress-focused, collaborative
- Language: "Great progress", "Let's optimize", "Here's what helps most teams"
- Emotion: Encouragement, partnership, expertise

**Day 30 Tone**: Strategic, partnership-focused, growth-oriented
- Language: "Value delivered", "What's next", "Let's plan your next phase"
- Emotion: Confidence, mutual success, forward-looking

## Output Contract

Return a SkillResult with:

### `data.email_draft`
Object:
- `subject`: string (recommended subject line)
- `subject_variants`: array of 3 subject line options
- `body`: string (full email text)
- `body_html`: string | null (HTML formatted version)
- `to`: string (primary contact email)
- `cc`: string[] | null (customer success team CC list)
- `sections`: array of section objects:
  - `type`: "welcome" | "setup_assistance" | "resources" | "support" | "progress" | "gaps" | "success_recap" | "expansion" | "cta"
  - `content`: string
  - `links`: string[] | null (resource URLs)
- `tone`: "warm" | "helpful" | "strategic"
- `onboarding_stage`: "day_7" | "day_14" | "day_30"
- `word_count`: number

### `data.suggested_resources`
Array of 3-5 objects:
- `title`: string (resource name)
- `url`: string (resource link)
- `description`: string (why this resource helps)
- `priority`: "high" | "medium"

### `data.next_checkin_date`
String (ISO date): Recommended date for the next check-in milestone.

### `data.adoption_snapshot` (Day 14 and Day 30 only)
Object:
- `usage_level`: "high" | "medium" | "low"
- `features_activated`: string[] (list of features in use)
- `features_underutilized`: string[] (features purchased but not used)
- `user_count`: number
- `last_activity_date`: string (ISO date)

### `data.expansion_opportunity` (Day 30 only, if applicable)
Object | null:
- `recommended_tier`: string (name of higher plan tier)
- `key_benefits`: string[] (specific features/limits that address their needs)
- `evidence`: string[] (usage patterns or growth signals that support the recommendation)

## Quality Checklist

Before returning results, validate:

- [ ] Email tone matches onboarding stage (warm for Day 7, helpful for Day 14, strategic for Day 30)
- [ ] Email includes at least ONE specific adoption metric or progress indicator (not generic "hope things are going well")
- [ ] Resources shared are relevant to the customer's use case or industry (not generic product docs)
- [ ] CTA is low-friction and optional (not required or pushy)
- [ ] Expansion suggestion (Day 30) is backed by actual usage data OR omitted entirely
- [ ] Subject line is under 50 characters
- [ ] Email is 120-200 words (concise but substantive)
- [ ] No pressure language or hard deadlines ("you must", "time running out")
- [ ] Day 7 email focuses on setup assistance, Day 14 on optimization, Day 30 on value recap
- [ ] Resources include working links (or placeholders for actual URLs)

## Error Handling

### Insufficient deal data
If deal data is minimal (no close date, no product details), draft a generic welcome check-in and flag: "Deal details unavailable — draft focuses on generic support offer."

### No usage data available
If usage metrics are not available, omit specific adoption stats and focus on engagement offers: "I wanted to check in and see how your team's onboarding is going. Are there any areas where we can provide additional support?" Flag: "Usage data unavailable — draft focuses on qualitative check-in."

### Wrong onboarding stage
If `days_since_close` is not 7, 14, or 30, choose the closest milestone and flag: "Onboarding stage {days_since_close} does not match standard cadence — using closest milestone (Day X)."

### Contact changed during onboarding
If the primary contact is different from the deal owner, acknowledge the handoff: "I know you've recently taken over the ${company_name} onboarding from [previous contact]. I wanted to introduce myself and make sure your team has everything needed to succeed." Flag: "Contact changed — requires relationship re-establishment."

### No recent activity (disengaged account)
If there is no activity in the past 7 days (no logins, no support tickets, no emails), escalate the urgency: "I noticed we haven't heard from your team since the kickoff. Is everything going smoothly, or are there blockers we can help resolve?" Flag: "Account disengaged — potential onboarding risk."

## Examples

### Good Day 7 Email
```
Subject: Acme x ${company_name} — Welcome & Setup Support

Hi Sarah,

Welcome to ${company_name}! We're excited to partner with Acme on your data
integration platform. Now that we've completed the kickoff and initial setup,
I wanted to check in and make sure your team has everything needed to get
started smoothly.

Getting your API integrations and user permissions configured can be tricky in
the first week. If you run into any setup questions — authentication, data sync,
or anything else — I'm here to help. No question is too small.

Resources to help you get started:
- [Getting Started Guide]: 5-minute walkthrough of core features
- [API Integration Template]: Pre-built config for your tech stack
- [Quick Wins Checklist]: 3 high-impact actions you can complete this week

I'm available if you'd like a quick 15-minute call to walk through any setup
questions next week — just let me know what works for your schedule.

Best,
[Rep]
```

### Good Day 14 Email
```
Subject: Acme x ${company_name} — 2-Week Progress Check-In

Hi Sarah,

Great to see your team is already processing 10K+ API calls per day — that's
strong adoption for the first two weeks. I noticed you've onboarded 8 users
and activated the core data sync module.

I noticed your team hasn't activated the automated reconciliation module yet —
that's typically the #1 time-saver for data engineering teams like yours.
Would a quick walkthrough be helpful?

Resources to optimize your setup:
- [Advanced Data Sync Guide]: Tips for high-volume API usage
- [Industry Use Case]: How fintech teams like yours use reconciliation automation

Onboarding always has a few bumps — if you're running into any issues with
setup, performance, or feature access, let's troubleshoot together.

Would a 20-minute deep dive on the reconciliation module be helpful for your
team? I have availability Tuesday at 10am or Thursday at 2pm if either works.

Best,
[Rep]
```

### Good Day 30 Email
```
Subject: Acme x ${company_name} — 30-Day Success Review

Hi Sarah,

We've hit the 30-day mark since your team launched ${company_name}. In that time,
your team has processed 320K+ API calls, reduced manual reconciliation time by
an estimated 18 hours/week, and onboarded 12 users with strong adoption across
core features.

Your engineering team's quick adoption of the advanced modules has been
impressive — most teams take 60+ days to activate those features.

I noticed you're not yet using the audit log export feature — given your SOC 2
compliance focus, that could save your team significant manual work during
audits. Worth exploring?

Your team has grown from 8 to 12 users and you're approaching the API call
limits on your current plan. If you're planning to continue scaling, our
Growth plan might be worth discussing — it includes unlimited API calls and
multi-team permissions.

Let's schedule a 30-minute success review next week to discuss what's working,
address any remaining gaps, and plan your next phase of growth. Would Tuesday
March 4 at 2pm or Thursday March 6 at 10am work?

Best,
[Rep]
```
