---
name: courier-notification-skills
description: Use when building notifications across email, SMS, push, in-app, Slack, Teams, or WhatsApp. Covers transactional messages (password reset, OTP, orders, billing), growth notifications (onboarding, engagement, referral), multi-channel routing, compliance (GDPR, TCPA, CAN-SPAM), and reliability patterns.
---

# Courier Notification Skills

Guidance for building deliverable, compliant, and engaging notifications across all channels.

## How to Use This Skill

1. **Identify the task** — What channel, notification type, or cross-cutting concern is the user working on?
2. **Read only what's needed** — Use the routing tables below to find the 1-2 files relevant to the task. Do NOT read all files.
3. **Check for live docs** — For current API signatures and SDK methods, fetch `https://www.courier.com/docs/llms.txt`
4. **Synthesize before coding** — Plan the complete implementation (channels, routing, compliance, error handling) before writing code.
5. **Apply the rules** — Each resource file starts with a "Quick Reference" section containing hard rules. Treat these as constraints, not suggestions.
6. **Check universal rules** — Before generating any notification code, verify it doesn't violate the Universal Rules below.

## Universal Rules (Never Violate)

- NEVER send promotional content in transactional notifications (CAN-SPAM violation)
- NEVER batch or delay OTP, password reset, or security alert notifications
- NEVER send SMS without TCPA-compliant consent records
- NEVER skip idempotency keys for transactional sends
- NEVER send during quiet hours (10pm-8am local) unless critical/security
- NEVER expose full email/phone in security change notifications (mask them)
- ALWAYS include "I didn't request this" links in security-related emails
- ALWAYS use E.164 format for phone numbers
- ALWAYS configure SPF + DKIM + DMARC before sending production email
- ALWAYS respect user opt-out preferences immediately
- ALWAYS use `method: "single"` unless the notification is critical enough to warrant all channels

## Official Courier Documentation

When you need current API signatures, SDK methods, or features not covered in these resources:
1. Fetch `https://www.courier.com/docs/llms.txt` — returns a structured index of all Courier documentation
2. Use it to find specific endpoint details, SDK method signatures, and configuration options
3. Prefer the patterns in THIS skill for best practices; use llms.txt for API specifics

## Architecture Overview

```
[User Action / System Event]
            │
            ▼
    ┌───────────────┐
    │ Notification  │
    │   Trigger     │
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   Routing     │──── User Preferences
    │   Decision    │──── Channel Availability
    └───────┬───────┘──── Urgency Level
            │
            ▼
    ┌───────────────────────────────────────┐
    │           Channel Selection           │
    ├───────┬───────┬───────┬───────┬──────┤
    │ Email │  SMS  │ Push  │ Inbox │ Chat │
    └───┬───┴───┬───┴───┬───┴───┬───┴───┬──┘
        │       │       │       │       │
        ▼       ▼       ▼       ▼       ▼
    [Delivery] [Delivery] [Delivery] [Delivery] [Delivery]
        │       │       │       │       │
        └───────┴───────┴───────┴───────┘
                        │
                        ▼
                ┌───────────────┐
                │   Webhooks    │
                │   & Events    │
                └───────────────┘
```

## Quick Reference

### By Channel

| Need to... | See |
|------------|-----|
| Send emails, fix deliverability, set up SPF/DKIM/DMARC | [Email](./resources/channels/email.md) |
| Send SMS, handle TCPA/10DLC compliance | [SMS](./resources/channels/sms.md) |
| Send push notifications, handle iOS/Android differences | [Push](./resources/channels/push.md) |
| Build in-app notification center | [Inbox](./resources/channels/inbox.md) |
| Send Slack messages with Block Kit | [Slack](./resources/channels/slack.md) |
| Send Microsoft Teams messages | [MS Teams](./resources/channels/ms-teams.md) |
| Send WhatsApp messages with templates | [WhatsApp](./resources/channels/whatsapp.md) |

### By Transactional Type

| Need to... | See |
|------------|-----|
| Build password reset, OTP, verification, security alerts | [Authentication](./resources/transactional/authentication.md) |
| Build order confirmations, shipping, delivery updates | [Orders](./resources/transactional/orders.md) |
| Build receipts, invoices, dunning, subscription notices | [Billing](./resources/transactional/billing.md) |
| Build booking confirmations, reminders, rescheduling | [Appointments](./resources/transactional/appointments.md) |
| Build welcome messages, profile updates, settings changes | [Account](./resources/transactional/account.md) |
| Understand transactional notification principles | [Transactional Overview](./resources/transactional/index.md) |

### By Growth Type

| Need to... | See |
|------------|-----|
| Build activation flows, setup guidance, first value | [Onboarding](./resources/growth/onboarding.md) |
| Build feature announcements, discovery, education | [Adoption](./resources/growth/adoption.md) |
| Build activity notifications, retention, habit loops | [Engagement](./resources/growth/engagement.md) |
| Build winback, inactivity, cart abandonment | [Re-engagement](./resources/growth/reengagement.md) |
| Build referral invites, rewards, viral loops | [Referral](./resources/growth/referral.md) |
| Build promotions, sales, upgrade campaigns | [Campaigns](./resources/growth/campaigns.md) |
| Understand growth notification principles | [Growth Overview](./resources/growth/index.md) |

### Cross-Cutting Guides

| Need to... | See |
|------------|-----|
| Route across multiple channels, set up fallbacks | [Multi-Channel](./resources/guides/multi-channel.md) |
| Manage user notification preferences | [Preferences](./resources/guides/preferences.md) |
| Ensure GDPR, TCPA, CAN-SPAM compliance | [Compliance](./resources/guides/compliance.md) |
| Handle retries, idempotency, error recovery | [Reliability](./resources/guides/reliability.md) |
| Combine notifications, build digests | [Batching](./resources/guides/batching.md) |
| Control frequency, prevent fatigue | [Throttling](./resources/guides/throttling.md) |
| Plan notifications for your app type | [Catalog](./resources/guides/catalog.md) |
| Reusable code patterns (consent, quiet hours, masking, retry) | [Patterns](./resources/guides/patterns.md) |

## Minimal File Sets by Task

For common tasks, you only need to read these specific files:

| Task | Files to Read |
|------|---------------|
| OTP/2FA implementation | [authentication.md](./resources/transactional/authentication.md), [sms.md](./resources/channels/sms.md) |
| Password reset | [authentication.md](./resources/transactional/authentication.md), [email.md](./resources/channels/email.md) |
| Order notifications | [orders.md](./resources/transactional/orders.md), [multi-channel.md](./resources/guides/multi-channel.md) |
| Email setup & deliverability | [email.md](./resources/channels/email.md), [compliance.md](./resources/guides/compliance.md) |
| SMS setup & compliance | [sms.md](./resources/channels/sms.md) (includes 10DLC, TCPA) |
| Push notification setup | [push.md](./resources/channels/push.md) |
| In-app inbox setup | [inbox.md](./resources/channels/inbox.md) |
| Onboarding sequence | [onboarding.md](./resources/growth/onboarding.md), [multi-channel.md](./resources/guides/multi-channel.md) |
| Security alerts | [authentication.md](./resources/transactional/authentication.md), [multi-channel.md](./resources/guides/multi-channel.md) |
| Digest/batching | [batching.md](./resources/guides/batching.md), [preferences.md](./resources/guides/preferences.md) |
| Payment/billing notifications | [billing.md](./resources/transactional/billing.md), [reliability.md](./resources/guides/reliability.md) |
| Appointment reminders | [appointments.md](./resources/transactional/appointments.md), [sms.md](./resources/channels/sms.md) |
| WhatsApp templates | [whatsapp.md](./resources/channels/whatsapp.md) |
| Slack/Teams integration | [slack.md](./resources/channels/slack.md) or [ms-teams.md](./resources/channels/ms-teams.md) |

## Decision Guide

**What are you building?**

- **A specific notification** (OTP, order confirm, password reset, etc.)
  → Use the [Minimal File Sets](#minimal-file-sets-by-task) table above to find exactly which 1-2 files to read.

- **A new notification channel** (email, SMS, push, Slack, etc.)
  → See [By Channel](#by-channel) for the channel-specific guide.

- **Notification infrastructure** (routing, preferences, reliability, batching)
  → See [Cross-Cutting Guides](#cross-cutting-guides) for the relevant guide.

- **Planning which notifications to build** for a new app
  → Start with [Catalog](./resources/guides/catalog.md), then [Email](./resources/channels/email.md), then [Multi-Channel](./resources/guides/multi-channel.md).

- **Growth / lifecycle notifications** (onboarding, engagement, referral)
  → Read [Growth Overview](./resources/growth/index.md) for consent requirements first, then the specific type.

- **Compliance concerns** (GDPR, TCPA, CAN-SPAM)
  → Read [Compliance](./resources/guides/compliance.md), then channel-specific rules in [SMS](./resources/channels/sms.md) or [Email](./resources/channels/email.md).

- **Debugging delivery issues**
  → Email going to spam? [Email](./resources/channels/email.md). SMS not arriving? [SMS](./resources/channels/sms.md). General failures? [Reliability](./resources/guides/reliability.md).

- **Reusable code patterns** (consent check, quiet hours, idempotency, fallback)
  → See [Patterns](./resources/guides/patterns.md) for copy-paste implementations.
