---
name: analytics-measurement
description: Analytics and conversion tracking for lead generation websites. Use when implementing GTM, GA4, or any measurement. Works with astro-architecture for GTM setup. No CRO mérés nélkül vakrepülés.
---

# Analytics & Measurement Skill

Measurement rules for lead generation sites. GTM + GA4 focused.

## Core Rules (Non-Negotiable)

1. **Every conversion MUST have a measurable event** — Form submit, phone click, WhatsApp = tracked
2. **GTM is the only tag container** — No inline GA4, no direct scripts
3. **One GTM container per site** — No multiple containers
4. **Consent before tracking** — GDPR: no GA4 until cookie accepted
5. **Event naming convention MUST be followed** — See naming rules below
6. **Every event MUST have context** — Location, element, value where applicable
7. **Test before deploy** — GTM Preview + GA4 DebugView MUST pass
8. **Production and staging MUST be separated** — Different GA4 properties or filtered

## Event Types

| Type | Purpose | Use as KPI? |
|------|---------|-------------|
| **Conversion** | Business outcome (lead, sale) | ✅ Yes |
| **Micro** | Supports analysis (scroll, click) | ❌ No |

**Conversion events:** form_submit, phone_click, whatsapp_click, calculator_complete
**Micro events:** cta_click, scroll_depth, video_play, calculator_start

Only conversion events may be used as primary KPIs. Micro events support optimization, not reporting.

## Environment Policy

- Production and staging MUST use separate GA4 properties (or filtered views)
- Staging data MUST NOT pollute production analytics
- Debug mode MUST be disabled in production
- Test events MUST be filtered out before reporting

## Forbidden (STOP)

STOP and fix if any of these occur:

### Implementation
- ❌ GA4/Ads script outside GTM
- ❌ Multiple GTM containers
- ❌ Tracking before consent (GDPR violation)
- ❌ Hardcoded Measurement ID in code
- ❌ Staging data in production property

### Events
- ❌ Double-firing events (same action = multiple events)
- ❌ Auto-events without explicit need (enhanced measurement spam)
- ❌ Event without naming convention
- ❌ Conversion without event
- ❌ Micro event used as KPI

### Privacy (Non-Negotiable)
- ❌ PII in event parameters (email, phone, name, address)
- ❌ Tracking without consent where required
- ❌ IP not anonymized (GA4 does this by default)
- ❌ User ID without consent

### Quality
- ❌ Deploy without GTM Preview test
- ❌ Event names with spaces or capitals
- ❌ Parameters without documentation

## Event Naming Convention

**Format:** `[action]_[object]` (lowercase_snake_case)

- **action**: click, submit, view, start, complete
- **object**: form, cta, video, phone, whatsapp

**Parameter rules:** lowercase_snake_case, max 40 chars, no PII

See [events.md](references/events.md) for full event list with parameters.

## Required Events

| Event | Trigger | Page Type |
|-------|---------|-----------|
| `page_view` | Page load | All |
| `scroll_depth` | 25%, 50%, 75%, 90% | All |
| `cta_click` | CTA button click | Landing |
| `phone_click` | tel: link click | Landing |
| `whatsapp_click` | WhatsApp link | Landing |
| `form_start` | First form interaction | Landing |
| `form_submit` | Successful submission | Landing |
| `calculator_start` | First step | Calculator |
| `calculator_complete` | Result shown | Calculator |
| `video_play` | Play clicked | Video |

See [events.md](references/events.md) for parameters and implementation.
See [gtm-setup.md](references/gtm-setup.md) for GTM container configuration.

## Conversions (Mark in GA4)

| Event | Conversion? | Send to Ads? |
|-------|-------------|--------------|
| `form_submit` | ✅ Yes | ✅ Yes |
| `phone_click` | ✅ Yes | ✅ Yes |
| `whatsapp_click` | ✅ Yes | ✅ Yes |
| `calculator_complete` | ✅ Yes | ✅ Yes |
| `cta_click` | ❌ No | ❌ No |
| `scroll_depth` | ❌ No | ❌ No |

## Server-Side Tracking

For ad blocker bypass and enhanced conversions, use server-side GTM via Cloudflare:

```
Browser → Web GTM → Cloudflare Worker → sGTM → GA4 + Google Ads
```

**Requirements:**
- First-party subdomain (e.g., `data.yourdomain.com`)
- Cloudflare Worker as proxy
- Server-Side GTM container
- Enhanced conversions: SHA256 hashed PII only

See [server-side.md](references/server-side.md) for full implementation.

## References

### Required

- [events.md](references/events.md) — Full event documentation
- [gtm-setup.md](references/gtm-setup.md) — GTM configuration guide

### Conditional

- [server-side.md](references/server-side.md) — Cloudflare + sGTM + Ads setup
- [debugging.md](references/debugging.md) — Testing and troubleshooting

## Definition of Done

### Core (Zero Tolerance)
- [ ] GTM is only tag container
- [ ] Consent blocks tracking until accepted
- [ ] Staging and production separated
- [ ] 0 double-firing events
- [ ] 0 PII in parameters
- [ ] All events follow naming convention

### Conversions (MUST fire correctly)
- [ ] `form_submit`, `phone_click`, `whatsapp_click` fire once per action
- [ ] All marked as conversions in GA4
- [ ] Sending to Google Ads (if enabled)

### Testing
- [ ] All events tested in GTM Preview
- [ ] All events visible in GA4 DebugView
- [ ] GTM container exported as backup

### Server-Side (If Enabled)
- [ ] Cloudflare Worker + sGTM deployed
- [ ] Enhanced conversions SHA256 hashed
