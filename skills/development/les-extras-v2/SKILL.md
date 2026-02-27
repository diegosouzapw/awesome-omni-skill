---
name: Les EXTRAS V2
description: Expert knowledge for developing, maintaining, and refactoring the SocioPulse V2 monorepo stack (multi-brand B2B platform).
---

# SocioPulse V2 - Expert Developer Skill

This skill provides deep contextual knowledge and strict guidelines for working on the SocioPulse V2 project (formerly "Les Extras").

## 1. Project Identity & Branding (CRITICAL)

**Multi-Brand Platform:**

| Brand | Mode | Theme | Target |
|-------|------|-------|--------|
| **SocioPulse** | `SOCIAL` | Electric Teal (#14B8A6) / Deep Indigo (#4F46E5) | Éducateurs, travailleurs sociaux (MECS, IME, ITEP, Crèches, CCAS) |
| **MedicoPulse** | `MEDICAL` | Neon Rose (#F43F5E) / Electric Violet | Soignants (EHPAD, SSIAD, Cliniques, Hôpitaux, HAD) |

**Strict Branding Rules:**

- **Project Name:** SocioPulse (V2) / MedicoPulse.
- **Forbidden Terms:**
  - ❌ "Sanctuary", "Oracle", "SoulMirror" (Legacy astrological features → **DO NOT USE**).
  - ❌ "Eduat'heure" (Legacy marketplace name → Use **"Catalogue"**).
  - ❌ "Les Extras" (Legacy brand → Use **"SocioPulse"**).
- **Domains:** `socio-pulse.fr` / `medico-pulse.fr`
- **Packages:** Internal packages MUST be scoped as `@sociopulse/*`

## 2. Technical Architecture

### Monorepo Structure

```
├── app/                    # Next.js 14 App Router (Frontend)
│   ├── (auth)/            # Auth & onboarding routes (4 pages)
│   ├── (platform)/        # Protected routes (~65+ pages)
│   │   ├── dashboard/
│   │   │   ├── client/    # 18+ sub-routes (missions, bookings, vivier, finance, SOS...)
│   │   │   ├── talent/    # 16+ sub-routes (missions, applications, services, sociolive...)
│   │   │   ├── missions/[id]/ # Mission detail, contract, tracking
│   │   │   └── relief/    # SOS Relief form
│   │   ├── fil-pro/       # Professional social feed (Le Wall)
│   │   ├── services/      # Marketplace catalogue
│   │   ├── sos/           # SOS landing
│   │   ├── visio/         # LiveKit video rooms
│   │   └── ...            # bookings, checkout, contracts, finance, messages, etc.
│   ├── (admin)/           # Admin backoffice (6+ pages: users, contracts, moderation, profiles, missions)
│   └── t/[slug]/          # Public SEO talent profiles
├── apps/api/              # NestJS 10 Backend (18 feature modules + 7 common)
│   └── src/
│       ├── auth/          # JWT auth, login/register
│       ├── admin/         # Admin backoffice
│       ├── matching-engine/ # Mission matching
│       ├── video-booking/ # Catalogue + LiveKit
│       ├── wall-feed/     # Social wall
│       ├── contracts/     # E-signatures
│       ├── messages/      # Messaging
│       ├── payments/      # Stripe Connect
│       ├── mission-hub/   # Active mission tracking
│       ├── notifications/ # WebSocket (Socket.IO gateway)
│       ├── availability/  # Talent availability slots
│       ├── finance/       # Financial reports
│       ├── talents/       # Talent profiles
│       ├── quotes/        # Commercial proposals (devis)
│       ├── dashboard/     # Dashboard stats endpoints
│       ├── pulse/         # Virtual currency (Pulse Economy)
│       ├── growth/        # Gamification + referrals
│       ├── health/        # Healthcheck for Coolify
│       └── common/        # Guards, decorators, filters, prisma, uploads, mailer, geocoding
├── components/            # React components (~230+ files, 30+ folders)
│   ├── dashboard/         # DashboardResolver + brand-specific dashboards + 19 widgets
│   ├── wall/              # WallResolver + 20+ polymorphic cards
│   ├── landing/           # 23+ landing page sections
│   ├── profile/           # 12 public profile components
│   ├── feed/              # Social feed (9 components)
│   ├── pulse/             # 8 Pulse economy components
│   ├── finance/           # 7 wallet/invoice components
│   ├── chat/              # 6 messaging components
│   ├── visio/             # 6 LiveKit video components
│   ├── mission/           # 6 mission tracking components
│   ├── vitrine/           # 6 marketplace components
│   ├── onboarding/        # 5 wizard components
│   ├── growth/            # 5 gamification components
│   ├── planning/          # 4 calendar components
│   ├── sociolive/         # 4 workshop/coaching components
│   ├── ui/                # 14 Radix/shadcn primitives
│   ├── providers/         # SocketProvider, ThemeProvider
│   └── ...
├── lib/                   # Business logic & configuration (19+ files)
│   ├── auth.ts            # Auth module (login, register, token management)
│   ├── useAuth.ts         # React hook wrapping Zustand authStore
│   ├── brand.ts           # Multi-brand config (BrandConfig, isMedical, isSocial)
│   ├── domain-config.ts   # Feature flags, terminology, compliance rules, dashboard layouts
│   ├── config.ts          # API URL resolution (getApiUrl) — browser/SSR/Docker
│   ├── dashboard-config.ts # Widget registry (16 widgets), Framer Motion animations
│   ├── nav-config.ts      # Navigation items (360 lines, all roles)
│   ├── pricing-config.ts  # Economic model (plans, commissions, cancellation rules)
│   ├── sos-config.ts      # 50+ job definitions (1,071 lines)
│   ├── wall-config.ts     # Polymorphic feed modes by role (314 lines)
│   ├── vitrine-config.ts  # Marketplace homepage config (184 lines)
│   ├── hooks/             # useDashboardData, useClientStats, useTalentStats, usePulseWallet, useServices
│   ├── stores/authStore.ts # Zustand + persist (284 lines) — AuthUser type
│   ├── constants/jobTags.ts # Job tag constants
│   └── pricing/calculator.ts # Smart Rate Engine (323 lines)
├── packages/shared-types/ # Shared TypeScript DTOs
├── prisma/schema.prisma   # Database schema (1,762 lines, 32 models, 34 enums)
├── seeds/                 # 7 JSON talent category files
├── scripts/               # generate_seed_sociopulse.ts, health-check.ts, smoke-test.ts
└── tests/                 # Playwright E2E (smoke.spec.ts)
```

### Tech Stack Summary

| Layer | Technologies |
|-------|-------------|
| **Frontend** | Next.js 14, React 18, TypeScript 5, Tailwind CSS 3.4, Framer Motion 12, Radix UI, Zustand 5 |
| **Backend** | NestJS 10.3, Prisma ~5.22 (pinned — v7 breaks!), PostgreSQL 15, Socket.IO 4.8, LiveKit 2.x |
| **Payments** | Stripe Connect (marketplace) + Pulse Economy (1 Pulse = 5€ HT) |
| **Auth** | JWT (jose), Cookie-based (SameSite=lax, 7 days), Role guards |
| **Testing** | Playwright (E2E smoke tests) |
| **DevOps** | Docker multi-stage (Node 20 Alpine), npm workspaces, Coolify-compatible healthchecks |

## 3. Critical Configuration Files

| File | Purpose | Lines |
|------|---------|-------|
| `prisma/schema.prisma` | **Source of Truth** — 32 models, 34 enums | 1,762 |
| `lib/sos-config.ts` | Job definitions (50+ métiers both brands) | 1,071 |
| `app/globals.css` | Theme CSS vars (`:root` + `[data-theme="medical"]`) | 1,380 |
| `lib/dashboard-config.ts` | Widget registry (16 widgets), animations | 522 |
| `lib/nav-config.ts` | Navigation items for all roles | 360 |
| `lib/domain-config.ts` | Feature flags, terminology, compliance rules | 350 |
| `lib/pricing/calculator.ts` | Smart Rate Engine (shift breakdown) | 323 |
| `lib/wall-config.ts` | Polymorphic feed config by role | 314 |
| `lib/stores/authStore.ts` | Zustand + persist (AuthUser type) | 284 |
| `lib/pricing-config.ts` | Economic model (plans, commissions, surcharges) | 269 |
| `lib/brand.ts` | Multi-brand config (BrandConfig interface) | 206 |
| `lib/vitrine-config.ts` | Marketplace homepage config | 184 |
| `tailwind.config.ts` | Custom theme (polymorphic colors, aurora glows) | 159 |
| `middleware.ts` | Route protection, role redirects, admin subdomain | 154 |
| `lib/auth.ts` | Auth module (login, register, token management) | 144 |
| `apps/api/src/main.ts` | NestJS bootstrap (CORS, Swagger, validation) | 108 |

## 4. Economic Model (Hybrid)

### Pricing Plans (`lib/pricing-config.ts`)

```typescript
enum PricingPlan {
  BETA       // 2% technical fees only (launch period — currently active)
  STANDARD   // 15% commission (private sector)
  ENTERPRISE // 0% commission (SaaS subscription)
}

// Launch switch - Set to false to enable monetization
IS_LAUNCH_PERIOD: true
```

### Smart Rate Engine (Legal Surcharges — `lib/pricing/calculator.ts`)

```typescript
SURCHARGES: {
  NIGHT_PERCENT: 25,    // +25% (21h-06h)
  SUNDAY_PERCENT: 50,   // +50%
  HOLIDAY_PERCENT: 100, // +100%
}
// Cumulative: Sunday night = +75%, Holiday night = +125%
```

**Usage:**
```typescript
import { calculateShiftPrice, calculateFeePercentage, PricingPlan } from '@/lib/pricing';

// Calculate shift with surcharges
const pricing = calculateShiftPrice(startDate, endDate, baseRateCentimes);

// Get commission rate based on client plan
const feePercent = calculateFeePercentage(PricingPlan.STANDARD); // → 15 (or 2 if IS_LAUNCH_PERIOD)
```

### Pulse Economy (Virtual Currency)
- **1 Pulse = 5€ HT** — separate from EUR wallet (Stripe Connect)
- **FIFO consumption:** free Pulses burned before paid
- **3 packs:** Discovery (20P/100€), Establishment (100P/500€), Grand Compte (500P/2,500€)
- **Used for:** mission publishing, contract validation, SocioLive sessions, profile boost, matching shortlist
- **Talent rewards:** Mission excellence (5★), compliance validation, referrals, social shares
- **API module:** `apps/api/src/pulse/` with `PulseGuard`, `@PulseCost()` decorator, `pulse-pricing.config.ts`

## 5. Multi-Brand Polymorphism (CRITICAL PATTERN)

### Never Hardcode Brand Logic

```typescript
// ✅ CORRECT
import { isMedical, isSocial, currentBrand } from '@/lib/brand';
import { getTerm, isFeatureEnabled } from '@/lib/domain-config';

const title = getTerm('mission');     // "Vacation" (Medical) | "Mission" (Social)
const label = getTerm('talent');      // "Soignant" | "Intervenant"
if (isFeatureEnabled('enableWorkshops')) { /* Social only */ }

// ❌ WRONG
if (mode === 'MEDICAL') { ... }  // Hardcoded check
className="bg-teal-500"          // Brand-specific color
```

### Feature Flags (`lib/domain-config.ts`)

| Flag | Social | Medical | Description |
|------|--------|---------|-------------|
| `enableWorkshops` | ✅ | ❌ | SocioLive workshops/ateliers |
| `enableShiftView` | ❌ | ✅ | Calendar-dense shift view |
| `enablePortfolio` | ✅ | ❌ | Talent portfolio showcase |
| `enableJobTicker` | ❌ | ✅ | Fast urgent shifts ticker |
| `enableCriticalUrgency` | ❌ | ✅ | CRITICAL urgency level |
| `enableTeamVivier` | ✅ | ✅ | TalentPool team management |
| `enableProjects` | ✅ | ❌ | Project-based mission grouping |

### Theming via CSS Variables

```css
/* globals.css (1,380 lines) */
/* Social: Electric Teal (default) */
:root {
  --primary-h: 174; --primary-s: 84%; --primary-l: 45%;
  --radius-sm: 0.5rem; /* Soft, friendly */
  --radius-full: 9999px; /* Pills allowed */
}
/* Medical: Neon Rose — sharp/clinical overrides */
[data-theme="medical"] {
  --primary-h: 340; --primary-s: 85%; --primary-l: 55%;
  --radius-sm: 0.25rem; /* Sharp, clinical */
  --radius-full: 0.5rem; /* NO pills! */
}
```

```tsx
// ✅ Use CSS variables
className="bg-[hsl(var(--primary))]"
className="text-primary-600"        // Tailwind via CSS vars (50-900 scale)
className="shadow-theme-glow"       // Brand-aware glow
className="rounded-[var(--radius-sm)]"  // Brand-aware radius

// ❌ NEVER direct colors
className="bg-teal-500"   // Breaks on Medical!
className="rounded-full"  // Breaks on Medical! (no pills)

// Static cross-brand colors (always safe):
className="bg-brand-500"  // Indigo — shared accent (#4F46E5)
className="bg-live-500"   // Teal — SocioLive (#14B8A6)
className="bg-alert-500"  // Rose — alerts/urgency (#F43F5E)
```

### Dashboard Resolution

`DashboardResolver` dynamically loads brand+role dashboards:

| Brand | Client Layout | Talent Layout |
|-------|--------------|---------------|
| Social | `project-hub` (card-based) | `portfolio-feed` (skills + feed) |
| Medical | `shift-planner` (calendar-dense) | `job-ticker` (fast list) |

Widget system in `lib/dashboard-config.ts`:
- **8 Client widgets:** SOS launcher, upcoming missions, talent spotlight, team stability, active projects, SocioLive discovery, finance summary, reviews
- **8 Talent widgets:** profile hero, mission feed, SOS ticker, upcoming bookings, savoir-être, sociolive, finance, reviews
- Each widget has `featureFlag` for brand filtering

## 6. API Security Pattern (NestJS)

```typescript
import { JwtAuthGuard, RolesGuard } from '@/common/guards';
import { Roles } from '@/common/decorators/roles.decorator';
import { CurrentUser, CurrentUserPayload } from '@/common/decorators/current-user.decorator';

@UseGuards(JwtAuthGuard, RolesGuard)  // Order matters!
@Roles('CLIENT')
@Post('missions')
async create(@CurrentUser() user: CurrentUserPayload) {
  // user.id, user.email, user.role, user.status
}
```

**Guards Available:**
| Guard | Purpose |
|-------|---------|
| `JwtAuthGuard` | Validates Bearer token |
| `RolesGuard` | Enforces `@Roles()` decorator |
| `MissionAccessGuard` | Mission-scoped data access |
| `SeedInterceptionGuard` | Prevents real actions on seed/demo data (`isSeed: true`) |
| `PulseGuard` | Checks Pulse wallet balance before debit actions |

**API Architecture:**
- **Global prefix:** `/api/v1`
- **Swagger:** `http://localhost:4000/docs`
- **Rate limiting:** 3 tiers (10/1s, 50/10s, 200/60s) via ThrottlerModule
- **CORS:** Multi-domain (localhost, sociopulse.fr, medicopulse.fr, sslip.io, FRONTEND_URL env)
- **Validation:** class-validator with whitelist + forbidNonWhitelisted
- **Exception filter:** Global `AllExceptionsFilter`
- **Raw body:** enabled for Stripe webhook signature verification

## 7. Development Workflows

### Essential Commands

```bash
# Development (run both in parallel)
npm run dev              # Frontend :3000
npm run api:dev          # Backend :4000

# Database (ALWAYS run db:generate after schema changes!)
npm run db:generate      # ⚠️ REQUIRED after schema.prisma edits
npm run db:push          # Push to dev DB
npm run db:migrate       # Create migration
npm run db:seed          # Seed test data (tsx prisma/seed.ts)
npm run db:studio        # Prisma Studio GUI
npm run db:seed:sociopulse # Generate + import SocioPulse seed data

# Testing
npm run test:smoke       # Playwright smoke tests

# Production Build
docker build --build-arg NEXT_PUBLIC_APP_MODE=SOCIAL -t sociopulse .
docker build --build-arg NEXT_PUBLIC_APP_MODE=MEDICAL -t medicopulse .
docker build -f Dockerfile.api -t sociopulse-api .
```

### Frontend API Calls

```typescript
import { getApiUrl } from '@/lib/config';
import { auth } from '@/lib/auth';

const res = await fetch(`${getApiUrl()}/endpoint`, {
  headers: { Authorization: `Bearer ${auth.getToken()}` }
});
// getApiUrl() handles: browser (hostname detection), SSR (Docker INTERNAL_API_URL), production fallback
```

### Auth Flow
- JWT in `accessToken` cookie (SameSite=lax, 7 days via js-cookie)
- `lib/auth.ts` → `{ login, register, setToken, getToken, logout, isAuthenticated, fetchMe }`
- `lib/useAuth.ts` → React hook: `{ user, isLoading, isAuthenticated, error, refetch, logout }` (50ms hydration delay)
- `lib/stores/authStore.ts` → Zustand + persist middleware, `AuthUser` type with full profile/establishment
- `middleware.ts` → Route protection, role-based redirects, admin subdomain (`dash.sociopulse.com`)
- Register supports cross-brand enrollment (`enableCrossBrand` param)

## 8. Key Models Quick Reference (32 models, 34 enums)

### Core Entities

| Model | Purpose | Key Fields |
|-------|---------|------------|
| `User` | Account (CLIENT/TALENT/ADMIN) | `role`, `status`, `clientType`, `allowedBrands[]`, `walletBalance`, `stripeAccountId`, `isSeed` |
| `Profile` | Talent professional profile | `jobId`, `complianceStatus`, `slug`, `profileScore`, `hourlyRate`, `languages[]`, `postureTags[]` |
| `Establishment` | Client organization | `pricingPlan`, `paymentMode`, `creditLimit`, `siret`, `isSocialActive`, `isMedicalActive` |
| `Service` | Catalogue offering | `type` (WORKSHOP/COACHING_VIDEO), `deliveryType`, `pricingOptions`, `ageGroups` |
| `ReliefMission` | SOS urgent mission | `jobId`, `urgencyLevel`, `missionType`, `specialtiesTags[]`, `requiresCar/Night/Diploma` |
| `Booking` | Reservation (polymorphic: Service OR ReliefMission) | `appliedPlan`, `commissionRate`, all surcharge snapshots, timesheet |
| `Contract` | E-signature | `type` (MISSION_SOS/SERVICE_BOOKING/FRAMEWORK), dual signatures |
| `Quote` | Commercial proposal (devis) | `reference`, `lines[]`, `status` (DRAFT→SENT→ACCEPTED), revisions chain |

### Supporting Models
`TalentPool`, `TalentPoolMember`, `AvailabilitySlot`, `MissionApplication` (with rate negotiation), `Post` (OFFER/NEED/SOCIAL), `Review`, `Transaction`, `Notification`, `Message`, `UserDocument`, `AdminNote`, `ExternalNews`, `AuditLog`, `Dispute` (6 categories + resolution), `Resource` (document hub), `AnalyticsEvent` (pre-launch metrics)

### Mission Hub Models
`MissionMessage` (TEXT/SYSTEM), `MissionInstruction` (checklist with acknowledgment), `MissionReport`, `MissionTimelineEvent` (BRIEFING_READ, MISSION_STARTED, CHAT_MESSAGE, REPORT_SUBMITTED)

### Pulse Economy Models
`PulseWallet` (balanceFree + balancePaid, FIFO, monthly stats, alert threshold), `PulseTransaction` (16 types, idempotency key, FIFO tracking)

### Key Enums
`UserRole` (CLIENT/TALENT/ADMIN), `UserStatus` (6 states inc. PENDING_INVITE/INVITED), `BookingStatus` (12 states), `ContractStatus` (9 states), `MissionUrgency` (LOW→CRITICAL), `PricingPlan` (BETA/STANDARD/ENTERPRISE), `PaymentMode` (PREPAID/POSTPAID), `DisputeCategory` (6 types), `QuoteStatus` (7 states)

### Booking Status Flow

```
PENDING → AWAITING_CONTRACT → CONFIRMED → PAID → IN_PROGRESS → COMPLETED
                                    ↓
                    CANCELLED / REJECTED / EXPIRED / NO_SHOW / DISPUTED / REFUNDED
```

## 9. Common Pitfalls

| Issue | Cause | Fix |
|-------|-------|-----|
| "Property X does not exist" | Schema changed | Run `npm run db:generate` |
| Wrong terminology | Hardcoded strings | Use `getTerm('key')` from domain-config |
| Colors wrong on one brand | Direct Tailwind | Use `var(--primary)` CSS vars |
| Border radius pills on Medical | `rounded-full` | Use `rounded-[var(--radius-*)]` |
| API 404 in production | Wrong URL | Use `getApiUrl()` helper |
| Guards not working | Wrong decorator order | `@UseGuards(JwtAuthGuard, RolesGuard)` before `@Roles()` |
| Commission wrong | Using old function | Use `calculateFeePercentage(plan)` |
| Prisma v7 breaks Docker | Upgrading past 5.x | Keep `~5.22.0` pinned in both package.json |
| Hydration mismatch | Auth state not ready | `useAuth()` hook has 50ms hydration delay |
| Seed data interactions | Users clicking demo data | `SeedInterceptionGuard` prevents actions on `isSeed: true` |
| Rate limiting errors | Exceeding API limits | 3 tiers: 10/1s, 50/10s, 200/60s |
| Stripe webhook fails | No raw body | `rawBody: true` set in main.ts bootstrap |

## 10. Key Features Overview

| Feature | Route(s) | API Module | Components |
|---------|----------|------------|------------|
| **SOS Renfort** | `/sos`, `/dashboard/*/sos`, `/dashboard/relief` | `matching-engine` | `SOSMissionCard`, `UrgencyBadge`, `FlashRequestForm` |
| **Catalogue** | `/services/*` | `video-booking` | `SocioLiveCard`, `SlotManager`, `BookingFlow` |
| **Fil Pro (Wall)** | `/fil-pro` | `wall-feed` | `WallResolver`, `WallFeedClient`, 20+ cards |
| **Mission Hub** | `/dashboard/missions/[id]/*` | `mission-hub` | `MissionChatLive`, `MissionMonitor` |
| **Vivier** | `/vivier`, `/dashboard/client/vivier` | — | `TalentCRMCard`, `TalentImportModal` |
| **Finance** | `/finance`, `/dashboard/*/finance` | `finance`, `payments` | `WalletCard`, `InvoiceTable`, `BudgetOverview` |
| **Visio** | `/visio/[roomId]`, `/live-session/[id]` | `video-booking` | `VideoRoom`, `ControlBar`, `PreJoinScreen` |
| **Pulse** | `/dashboard/client/finance/pulses` | `pulse` | `PulseWalletWidget`, `PulsePackGrid`, `PulseTransactionHistory` |
| **Growth** | (embedded in dashboards) | `growth` | `GamificationWidget`, `ReferralCard`, `TagSelector` |
| **Marketplace** | `/` (platform root) | — | `VitrineResolver`, `ClientVitrine`, `TalentVitrine` |
| **Public Profile** | `/t/[slug]`, `/talent/[id]` | `talents` | `PublicTalentProfile`, `QRBusinessCard` |
| **Landing** | `/` (unauthenticated) | — | 23+ sections (hero, bento grids, testimonials, trust) |

## 11. Dashboard Resolution

`DashboardResolver` dynamically loads brand+role specific dashboards:

```
├── SocialClientDashboard   (project-hub layout — card-based)
│   └── Widgets: SOS launcher, upcoming missions, projects, SocioLive, finance, reviews
├── MedicalClientDashboard  (shift-planner layout — calendar-dense)
│   └── Widgets: SOS launcher, upcoming missions, talent spotlight, team stability, finance, reviews
├── SocialTalentDashboard   (portfolio-feed layout — skills showcase)
│   └── Widgets: profile hero, mission feed, bookings, savoir-être, sociolive, finance
└── MedicalTalentDashboard  (job-ticker layout — fast list)
    └── Widgets: profile hero, SOS ticker, bookings, mission feed, finance, reviews
```

## 12. Docker & Deployment

### Frontend Dockerfile (4-stage multi-stage)
- **Base:** `node:20-alpine` with `libc6-compat`, `openssl`
- **Build args:** `NEXT_PUBLIC_APP_MODE` (SOCIAL/MEDICAL), `NEXT_PUBLIC_API_URL`
- **Runner:** Non-root `nextjs` user, standalone output, port 3000
- **Healthcheck:** curl-based for Coolify

### API Dockerfile (4-stage multi-stage)
- **Builder:** `@nestjs/cli` global, builds shared-types first, then NestJS
- **Runner:** Non-root `nestjs` user, copies `dist/`, `prisma/`, `seeds/`, `scripts/`, port 4000
- **Pinned:** `prisma@5` and `tsx` installed globally (v7 has breaking changes)

---

## Quick Links

- **API Docs**: `http://localhost:4000/docs`
- **DB GUI**: `npm run db:studio`
- **Technical Audit**: `TECHNICAL_AUDIT_2026-02-04.md`
- **Copilot Instructions**: `.github/copilot-instructions.md`

---
*Version 2.1.0 — Updated 2026-02-07*
