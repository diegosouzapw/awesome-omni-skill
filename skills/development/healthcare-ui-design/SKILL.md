---
name: healthcare-ui-design
description: Clinical-grade UI/UX patterns for healthcare applications across web (Bootstrap 5/Tabler + PHP) and Android (Jetpack Compose + Material 3). Covers patient records, vital signs, medication safety, care plans, scheduling, telemedicine, dashboards, patient portals, and clinical communication. Enforces HIPAA compliance, WCAG 2.2 AA accessibility, medical safety workflows, and role-based interfaces for clinicians, nurses, patients, and administrators. Use when building or reviewing EMR/EHR systems, hospital management, clinic apps, telemedicine platforms, patient portals, health dashboards, or any healthcare-related interface.
---

## Required Plugins

**Superpowers plugin:** MUST be active for all healthcare UI work — design decisions, component generation, accessibility audits, and compliance checks.

**Frontend Design plugin:** Required for all web and Android UI generation.

**Companion skills:** Load `webapp-gui-design` for web implementations, `jetpack-compose-ui` for Android implementations. Always load `vibe-security-skill` — healthcare demands zero-compromise security.

# Healthcare UI Design Standards

## Design Philosophy

Five non-negotiable principles for healthcare interfaces:

1. **Clinical Safety First** — UI decisions can impact patient outcomes. Never auto-select medications, never hide allergies, always require confirmation for critical actions. A confusing dosage field can be fatal.
2. **Calm Under Pressure** — Users are stressed, sick, or overwhelmed. Use clean layouts, soft palettes, generous whitespace, and predictable navigation. Reassure, never overwhelm.
3. **Role-Specific Experiences** — Clinicians need rapid-scan dashboards. Nurses need task-oriented workflows. Patients need plain language and large touch targets. Admin needs analytics. Never build one-size-fits-all.
4. **Compliance by Design** — HIPAA, WCAG 2.2 AA, ADA are embedded in every component, not bolted on. Every screen logs access, enforces timeouts, and protects PHI.
5. **Context-Aware Density** — Show the right amount of data for the situation. Triage screens are dense. Patient portals are spacious. Emergency views strip to essentials.

## Tech Stack Integration

| Platform | Stack | Pattern |
|----------|-------|---------|
| **Web** | Bootstrap 5 / Tabler + PHP | Clone seeder-page, use `webapp-gui-design` patterns, extend with healthcare components |
| **Android** | Jetpack Compose + Material 3 | Follow `jetpack-compose-ui` standards, extend with healthcare composables |
| **API** | REST (PHP) | Follow `api-error-handling` + `api-pagination` skills, add HIPAA audit headers |
| **Auth** | Dual Auth + RBAC | Use `dual-auth-rbac` skill with healthcare role extensions |

## Quick Reference

| Domain | Reference File | When to Use |
|--------|----------------|-------------|
| **Colors, Typography, Spacing** | [`references/design-tokens.md`](references/design-tokens.md) | Starting any healthcare screen, theming, token setup |
| **Patient Lookup & Records** | [`references/patient-records-ui.md`](references/patient-records-ui.md) | Patient lists, profiles, medical history, timeline views |
| **Vitals, Meds, Care Plans** | [`references/clinical-workflows-ui.md`](references/clinical-workflows-ui.md) | Vital sign entry, medication admin, care plan builders |
| **Scheduling & Telemedicine** | [`references/scheduling-telemedicine-ui.md`](references/scheduling-telemedicine-ui.md) | Appointment booking, provider search, video/chat consults |
| **Dashboards & Analytics** | [`references/dashboards-analytics-ui.md`](references/dashboards-analytics-ui.md) | Admin dashboards, KPIs, bed occupancy, risk scores |
| **Patient Portals** | [`references/patient-portal-ui.md`](references/patient-portal-ui.md) | Patient-facing self-service, lab results, payments |
| **Communication & Outreach** | [`references/communication-outreach-ui.md`](references/communication-outreach-ui.md) | Secure messaging, campaigns, health bots, notifications |
| **HIPAA, WCAG, Security** | [`references/compliance-accessibility.md`](references/compliance-accessibility.md) | Compliance audits, accessibility checks, security UI |
| **Web (Bootstrap/Tabler)** | [`references/web-implementation.md`](references/web-implementation.md) | PHP/Bootstrap-specific healthcare components |
| **Android (Compose)** | [`references/android-implementation.md`](references/android-implementation.md) | Kotlin/Compose-specific healthcare composables |

## Core Healthcare Color System

Use semantic clinical colors across both platforms:

| Token | Hex | Usage |
|-------|-----|-------|
| `clinical-primary` | `#2563EB` | Primary actions, headers, navigation |
| `clinical-secondary` | `#0F766E` | Secondary actions, supporting elements |
| `clinical-surface` | `#F8FAFC` | Page backgrounds, card surfaces |
| `clinical-critical` | `#DC2626` | Critical alerts, abnormal vitals, allergies |
| `clinical-warning` | `#D97706` | Warnings, approaching-threshold vitals |
| `clinical-success` | `#059669` | Normal ranges, completed tasks, confirmations |
| `clinical-info` | `#0284C7` | Informational, educational content |
| `clinical-muted` | `#64748B` | Secondary text, metadata, timestamps |

See [`references/design-tokens.md`](references/design-tokens.md) for complete token system.

## Essential Patterns

### Patient Card (Universal Component)

Every healthcare screen that displays a patient must show this minimum context:

```
┌─────────────────────────────────────────────┐
│ [Avatar] John Doe, M, 45y    MRN: 1234567  │
│ Allergies: Penicillin ⚠️  Blood: O+         │
│ Primary: Dr. Smith │ Last Visit: 2026-02-20 │
│ ┌─────────┐ ┌──────────┐ ┌───────────────┐ │
│ │ Vitals  │ │ Records  │ │ Medications   │ │
│ └─────────┘ └──────────┘ └───────────────┘ │
└─────────────────────────────────────────────┘
```

**Rules:** Allergy banner is ALWAYS visible (red background if severe). Blood type visible on clinical views. MRN searchable and copyable.

### Clinical Alert Hierarchy

Four-tier alert system — consistent across web and mobile:

| Level | Color | Icon | Behavior |
|-------|-------|------|----------|
| **Critical** | Red bg `#DC2626` | Shield-exclamation | Blocks workflow, requires acknowledgment |
| **Warning** | Amber bg `#D97706` | Triangle-exclamation | Prominent banner, dismissible after read |
| **Info** | Blue bg `#0284C7` | Info-circle | Inline notification, auto-dismiss 10s |
| **Success** | Green bg `#059669` | Check-circle | Toast/snackbar, auto-dismiss 5s |

### Vital Signs Display Pattern

```
┌──────────────┬──────────────┬──────────────┐
│ ❤️ HR: 72    │ 🫁 SpO2: 98% │ 🌡️ Temp: 37.1│
│ bpm [Normal] │    [Normal]  │  °C [Normal] │
├──────────────┼──────────────┼──────────────┤
│ BP: 120/80   │ RR: 16       │ Pain: 3/10   │
│ mmHg [Normal]│ /min [Normal]│    [Mild]    │
└──────────────┴──────────────┴──────────────┘
```

**Rules:** Color-code each value (green/amber/red) based on clinical thresholds. Show units ALWAYS. Show trend arrows (↑↓→) when historical data exists. Never rely on color alone — include text labels.

### Medication Safety — 5 Rights Check

Before any medication administration, enforce UI verification:

1. **Right Patient** — Display patient name + MRN + photo, require confirmation
2. **Right Medication** — Show drug name + form + strength, barcode scan option
3. **Right Dose** — Display calculated dose with weight-based verification
4. **Right Time** — Show scheduled time vs current time, flag if outside window
5. **Right Route** — Display route (oral/IV/IM/SC), require selection confirmation

Block administration if any check fails. Log all overrides with reason.

## Screen Layout Principles

### Web (Tabler/Bootstrap)

```
┌─────────────────────────────────────────────┐
│ Top Bar: Facility │ Ward │ Shift │ User │ 🔔│
├──────┬──────────────────────────────────────┤
│ Nav  │ Context Bar: Patient/Ward/Date       │
│ Side │ ┌──────────────────────────────────┐ │
│ bar  │ │ Main Content Area               │ │
│      │ │ Cards / Tables / Forms          │ │
│      │ │                                 │ │
│      │ └──────────────────────────────────┘ │
│      │ ┌────────────┐ ┌────────────────┐   │
│      │ │ Quick Panel│ │ Activity Feed  │   │
│      │ └────────────┘ └────────────────┘   │
└──────┴──────────────────────────────────────┘
```

### Android (Compose)

```
┌─────────────────────────┐
│ TopAppBar: Screen Title │
│ Patient Context Strip   │
├─────────────────────────┤
│                         │
│ Scrollable Content      │
│ (LazyColumn/Grid)       │
│                         │
│ Cards, Forms, Lists     │
│                         │
├─────────────────────────┤
│ BottomNav: Home│Patients│
│ Schedule│Chat│Profile   │
└─────────────────────────┘
```

## Healthcare-Specific DO's and DON'Ts

### DO

- Display allergies on EVERY screen that shows patient context
- Use confirmation dialogs for all medication and order actions
- Show data source attribution (manual entry vs wearable vs lab)
- Log every patient record access in audit trail
- Differentiate human-entered vs device-captured vitals visually
- Show measurement units on ALL numeric values
- Support offline mode for bedside data entry (sync when connected)
- Use `aria-live` regions for real-time vital sign updates
- Provide undo/cancel within 10s for non-critical actions
- Enforce session timeout (15min inactive) with save-state recovery

### DON'T

- Never auto-select medications from search results
- Never hide the allergy banner for any reason
- Never display PHI in URLs, page titles, or browser notifications
- Never use color alone to convey clinical status — always pair with text/icons
- Never skip confirmation for orders, prescriptions, or discharge actions
- Never store PHI in localStorage, sessionStorage, or unencrypted cookies
- Never show full SSN/ID — mask to last 4 digits
- Never allow copy-paste of patient data without audit logging
- Never use placeholder text as the only label on medical forms
- Never auto-dismiss critical alerts

## Integration with Existing Skills

```
healthcare-ui-design (this skill)
    ├── webapp-gui-design ──→ Web layout, Tabler components, DataTables
    ├── jetpack-compose-ui ──→ Android Compose, Material 3, state management
    ├── vibe-security-skill ──→ HIPAA security, encryption, XSS prevention
    ├── dual-auth-rbac ──→ Clinical role-based access control
    ├── mobile-rbac ──→ Android permission gates for clinical modules
    ├── api-error-handling ──→ Standardized clinical API error responses
    ├── api-pagination ──→ Patient list pagination (cursor-based)
    ├── image-compression ──→ Medical image upload optimization
    ├── photo-management ──→ Patient photo capture and storage
    └── report-print-pdf ──→ Clinical report PDF generation
```

## Workflow: Building a Healthcare Screen

1. **Identify role** — Who uses this screen? (Clinician / Nurse / Patient / Admin)
2. **Select layout** — Read platform-specific reference (`web-implementation.md` or `android-implementation.md`)
3. **Apply tokens** — Use `design-tokens.md` for colors, typography, spacing
4. **Build components** — Use domain reference (patient-records, clinical-workflows, etc.)
5. **Add compliance** — Apply `compliance-accessibility.md` checklist
6. **Validate** — Run WCAG checker, test with screen reader, verify HIPAA audit logging
