---
name: prd
description: "Generate a Product Requirements Document (PRD) for a new MÄÄK feature. Use when planning a feature, starting a new project, or when asked to create a PRD. Triggers on: create a prd, write prd for, plan this feature, requirements for, spec out."
---

# PRD Generator for MÄÄK

Create detailed Product Requirements Documents for the MÄÄK personality-based dating app. PRDs should be clear, actionable, and suitable for autonomous AI implementation via the Ralph loop.

---

## About MÄÄK

MÄÄK is a Swedish personality-based dating platform with:

**Tech Stack:**
- **Frontend**: React 18 + TypeScript + Vite
- **Backend**: Supabase (PostgreSQL + Edge Functions)
- **UI**: shadcn/ui + Tailwind CSS + Framer Motion
- **Auth**: Phone-based authentication (Twilio)
- **State**: TanStack Query + React Context
- **i18n**: i18next (Swedish primary)
- **PWA**: Service worker enabled

**Core Features:**
- Phone-based authentication
- Personality test & matching algorithm
- Daily match batches with countdown timers
- Real-time chat with icebreakers
- Achievement system with confetti
- AI assistant mascot
- GDPR consent management

**Key Pages:** Index, PhoneAuth, Onboarding, Profile, Matches, Chat

**Database Tables:** profiles, matches, messages, achievements, user_achievements, daily_match_batches, daily_questions, icebreakers, personality_results

---

## The Job

1. Receive a feature description from the user
2. Ask 3-5 essential clarifying questions (with lettered options)
3. Generate a structured PRD based on answers
4. Save to `PRD.md`
5. Create empty `progress.txt`

**Important:** Do NOT start implementing. Just create the PRD.

---

## Step 1: Clarifying Questions

Ask only critical questions where the initial prompt is ambiguous. Focus on MÄÄK-specific concerns:

- **User Journey Phase:** Which phase does this affect? (Onboarding, Matching, Chat, Profile)
- **Scope:** Full feature or MVP?
- **i18n:** Swedish only or both Swedish/English?
- **Realtime:** Does it need Supabase realtime subscriptions?
- **GDPR:** Does it involve new personal data collection?

### Format Questions Like This:

```
1. Which user journey phase does this feature affect?
   A. Onboarding (personality test, profile setup)
   B. Matching (daily matches, match discovery)
   C. Chat (messaging, icebreakers)
   D. Profile (settings, achievements, subscription)

2. What is the scope?
   A. Minimal viable version (one happy path)
   B. Full-featured implementation
   C. Backend/Edge Function only
   D. UI component only

3. Does this feature require real-time updates?
   A. Yes, needs Supabase realtime subscription
   B. No, polling or on-demand fetch is fine
   C. Unsure

4. Does this feature collect new personal data?
   A. Yes, needs GDPR consent update
   B. No, uses existing data only
   C. Unsure
```

This lets users respond with "1B, 2A, 3A, 4B" for quick iteration.

---

## Step 2: Story Sizing (THE NUMBER ONE RULE)

**Each story must be completable in ONE context window (~10 min of AI work).**

Ralph spawns a fresh instance per iteration with no memory of previous work. If a story is too big, the AI runs out of context before finishing and produces broken code.

### Right-sized stories for MÄÄK:
- Add a column to profiles table + migration
- Create a new shadcn/ui component with Framer Motion
- Add an Edge Function endpoint
- Update a hook to fetch new data
- Add a filter to the matches page
- Add Swedish translations to i18n

### Too big (MUST split):
| Too Big | Split Into |
|---------|-----------|
| "Add premium subscription" | Schema, Edge Function, useSubscription hook, paywall UI, settings toggle |
| "Improve matching algorithm" | Score calculation, weight adjustments, edge function update, match display |
| "Add video profiles" | Storage bucket, upload UI, playback component, profile integration |
| "Build admin dashboard" | Schema, auth middleware, data tables, analytics charts |

**Rule of thumb:** If you cannot describe the change in 2-3 sentences, it is too big.

---

## Step 3: Story Ordering (Dependencies First)

Stories execute in priority order. Earlier stories must NOT depend on later ones.

**Correct order for MÄÄK features:**
1. Supabase migrations (schema changes)
2. Edge Functions (backend logic)
3. Types regeneration (`supabase gen types`)
4. Hooks (data fetching with TanStack Query)
5. UI components (shadcn/ui + Tailwind)
6. Page integration
7. i18n translations

**Wrong order:**
```
US-001: UI component (depends on hook that doesn't exist!)
US-002: Hook that fetches data
US-003: Migration for the table
```

---

## Step 4: Acceptance Criteria (Must Be Verifiable)

Each criterion must be something Ralph can CHECK, not something vague.

### Good criteria for MÄÄK:
- "Add `premium_until` column to profiles table with type `timestamptz`"
- "Edge Function returns 200 with match data"
- "Component renders with Framer Motion fade-in"
- "Swedish translation added for 'matches.empty' key"
- "`npm run build` passes"
- "`npm run lint` passes"

### Bad criteria (vague):
- "Works correctly"
- "Good UX"
- "Matches display nicely"
- "Handles edge cases"

### Always include as final criteria:
```
- [ ] `npm run build` passes
- [ ] `npm run lint` passes
```

### For stories that change UI, also include:
```
- [ ] Verify changes work in browser at localhost:8080
```

### For stories with Edge Functions:
```
- [ ] Test Edge Function locally or via Supabase dashboard
```

---

## PRD Structure

Generate the PRD with these sections:

### 1. Introduction
Brief description of the feature and the problem it solves for MÄÄK users.

### 2. Goals
Specific, measurable objectives (bullet list).

### 3. User Stories
Each story needs:
- **ID:** Sequential (US-001, US-002, etc.)
- **Title:** Short descriptive name
- **Description:** "As a [MÄÄK user/admin], I want [feature] so that [benefit]"
- **Acceptance Criteria:** Verifiable checklist

**Format:**
```markdown
### US-001: [Title]
**Description:** As a [user type], I want [feature] so that [benefit].

**Acceptance Criteria:**
- [ ] Specific verifiable criterion
- [ ] Another criterion
- [ ] `npm run build` passes
- [ ] `npm run lint` passes
- [ ] [UI stories] Verify changes work in browser
```

### 4. Non-Goals
What this feature will NOT include. Critical for scope.

### 5. Technical Considerations
- Existing hooks to extend (useMatches, useAchievements, etc.)
- shadcn/ui components to reuse
- Supabase RLS policies needed
- i18n keys to add

---

## Example PRD for MÄÄK

```markdown
# PRD: Match Expiry Warning

## Introduction

Add visual warnings when a match is about to expire, helping users prioritize conversations before the 24-hour window closes. Currently users don't know a match is expiring until it's gone.

## Goals

- Show time remaining on each match card
- Visual urgency indicator when < 2 hours remain
- Push notification 1 hour before expiry (if enabled)
- Reduce expired matches by 20%

## User Stories

### US-001: Add expires_at display to match cards
**Description:** As a MÄÄK user, I want to see how much time I have left with each match so I can prioritize my conversations.

**Acceptance Criteria:**
- [ ] MatchCountdown component shows hours/minutes remaining
- [ ] Uses date-fns for time formatting
- [ ] Updates every minute via interval
- [ ] `npm run build` passes
- [ ] `npm run lint` passes
- [ ] Verify changes work in browser

### US-002: Add urgency styling for expiring matches
**Description:** As a MÄÄK user, I want expiring matches to stand out visually so I don't miss them.

**Acceptance Criteria:**
- [ ] Match card border turns amber when < 4 hours remain
- [ ] Match card border turns red when < 2 hours remain
- [ ] Framer Motion pulse animation on urgent matches
- [ ] `npm run build` passes
- [ ] `npm run lint` passes
- [ ] Verify changes work in browser

### US-003: Add Swedish translations for expiry warnings
**Description:** As a Swedish user, I want expiry warnings in my language.

**Acceptance Criteria:**
- [ ] Add `matches.expires_in` key: "Försvinner om {time}"
- [ ] Add `matches.expiring_soon` key: "Snart borta!"
- [ ] `npm run build` passes
- [ ] `npm run lint` passes

## Non-Goals

- No automatic match extension
- No premium feature to extend matches
- No notification sound effects

## Technical Considerations

- Extend existing MatchCountdown component in `src/components/matches/`
- Reuse Tailwind border utilities with amber-500/red-500
- Use Framer Motion `animate` prop for pulse effect
- i18n keys go in `src/i18n/locales/sv.json`
```

---

## Output

Save to `PRD.md` in the project root.

Also create `progress.txt`:
```markdown
# Progress Log

## Project Context
- Tech: React 18 + TypeScript + Vite + Supabase + shadcn/ui
- Run dev: `npm run dev` (port 8080)
- Build: `npm run build`
- Lint: `npm run lint`

## Learnings
(Patterns discovered during implementation)

---
```

---

## Checklist Before Saving

- [ ] Asked clarifying questions with lettered options
- [ ] Incorporated user's answers
- [ ] User stories use US-001 format
- [ ] Each story completable in ONE iteration (small enough)
- [ ] Stories ordered by dependency (migration → Edge Function → types → hook → UI → i18n)
- [ ] All criteria are verifiable (not vague)
- [ ] Every story has `npm run build` and `npm run lint` checks
- [ ] UI stories have "Verify changes work in browser"
- [ ] Non-goals section defines clear boundaries
- [ ] Technical considerations reference existing MÄÄK patterns
- [ ] Saved PRD.md and progress.txt
