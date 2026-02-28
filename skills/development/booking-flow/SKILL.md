---
name: booking-flow
description: End-to-end booking lifecycle in the Ever Club Members App — booking request creation, booking flow, booking lifecycle, session creation, conflict detection, bay assignment, approval flow, auto-approve, Trackman sync, Trackman booking modifications, booking request statuses, guest pass holds, usage tracking, member cancel, staff approval, prepayment, calendar sync, and reconciliation.
---

# Booking Flow

How a booking moves through its lifecycle from member request to completion.

## Lifecycle Stages

```
Request → Guest Pass Hold → Staff Approval → Session Creation → Invoice Draft → Trackman Link → Check-in → Completion / Auto Check-In
```

### 1. Request (status: `pending`)

Member submits via `POST /api/booking-requests`. The route:

1. Validate time bounds (no cross-midnight: `endHours >= 24` blocked), resource availability, and participant membership status.
2. Run `findConflictingBookings()` — check owner bookings, participant bookings, and invites on the same date for time overlap.
3. Run `checkUnifiedAvailability()` — check closures, availability blocks, and existing sessions on the target resource.
4. Sanitize and deduplicate participants:
   - Resolve email → userId by looking up users table.
   - Resolve userId → email for directory-selected members (userId set but email missing).
   - Reject participants with `membershipStatus = 'inactive' | 'cancelled'`.
   - Deduplicate by email and userId sets; owner email always added first.
5. Determine initial status:
   - **Conference rooms** → `confirmed` (auto-approve, skip staff review).
   - **Golf simulators** → `pending` (require staff approval).
6. Insert into `booking_requests` table within a raw pg transaction.
7. If guests present, call `createGuestPassHold()` to reserve guest passes (non-blocking — hold failure does not block booking).
8. COMMIT transaction, then for conference rooms call `ensureSessionForBooking()` and create a draft invoice via the booking invoice service (same flow as simulators since v8.16.0).
9. Send HTTP response, then asynchronously:
    - Publish `booking_created` event via `bookingEvents.publish()`.
    - Notify staff via `notifyAllStaff()` with push notification.
    - Broadcast availability update via WebSocket.
    - Track first booking for onboarding (`users.first_booking_at`).

### 2. Guest Pass Hold

Guest passes use a hold-then-convert pattern for atomicity:

- **At request time**: `createGuestPassHold()` reserves passes in `guest_pass_holds` table (does NOT decrement `passes_used` yet).
- **At approval time**: Inside the session creation transaction, holds are converted to actual usage via `SELECT ... FOR UPDATE` on `guest_passes`, then the hold row is deleted.
- **At cancellation**: `releaseGuestPassHold()` deletes the hold without decrementing.

This ensures passes are never double-spent even if approval and cancellation race.

### 3. Approval (status: `approved`)

Staff approves via `PUT /api/booking-requests/:id` with `status: 'approved'`. The route:

1. Validate `trackman_booking_id` format if provided (must be numeric, not UUID). Check for duplicates on other bookings.
2. Enter `db.transaction()`:
   a. Verify bay is assigned (`resource_id` required before approval).
   b. Run conflict detection within the transaction: booking time overlaps (approved/confirmed/attended), closure conflicts, availability block conflicts.
   c. Create Google Calendar event: `getCalendarNameForBayAsync()` → `getCalendarIdByName()` → `createCalendarEventOnCalendar()`.
   d. Determine final status: conference rooms → `'attended'`, simulators → `'approved'`.
   e. If `pending_trackman_sync` flag is set without a `trackman_booking_id`, append `[PENDING_TRACKMAN_SYNC]` marker to `staff_notes`.
   f. Build participant list from `request_participants` JSON column:
      - Start with owner as first participant (resolve `userId` from email if needed).
      - For each request participant: resolve email ↔ userId, detect "guests" who are actually members, deduplicate.
      - Guests with a matching email in the `users` table are converted to `member` type.
   g. Call `createSessionWithUsageTracking()` with `bookingId` (enables hold-to-usage conversion).
   h. Link session: `UPDATE booking_requests SET session_id`.
   i. Resolve `ownerUserId` from `users` table if not already known.
3. Post-transaction (after COMMIT):
   - **Fee calculation** (MUST be post-commit): Call `recalculateSessionFees(sessionId, 'approval')` → returns `{ totalCents, overageCents, guestCents }`. This CANNOT run inside the transaction because `recalculateSessionFees` uses the global `db` pool, not the `tx` handle — Postgres Read Committed isolation prevents it from seeing uncommitted participant rows, causing $0 fees or deadlock. (Fixed v8.26.7, Bug 22)
   - If `totalCents > 0`, create a prepayment intent via `createPrepaymentIntent()`.
   - If simulator booking with fees > 0, create draft Stripe invoice via `createDraftInvoiceForBooking()`. Stores `stripe_invoice_id` on `booking_requests`. Non-blocking — invoice failure does not block approval.
   - Link and notify participants via `linkAndNotifyParticipants()`.
   - Notify member: push notification, WebSocket update, email.
   - Publish `booking_approved` event.
   - Broadcast availability and billing updates.

### 4. Session Creation

`createSessionWithUsageTracking()` is the central orchestrator. It accepts an optional external transaction and either joins it or creates its own. Steps:

**Pre-transaction validation** (runs outside transaction to avoid long locks):

1. **Tier validation**: `getMemberTier(ownerEmail)` → ownerTier. Then `enforceSocialTierRules(ownerTier, participants)`:
   - Check if any participant has `type: 'guest'`.
   - If owner is Social tier and guests are present → return `{ allowed: false, errorType: 'social_tier_blocked' }`.
2. **Resolve identities**: For each participant with a `userId`, call `resolveUserIdToEmail(userId)` → build `userIdToEmail` map. This is needed because `usage_ledger.member_id` stores emails.
3. **Calculate billing**: `calculateFullSessionBilling(sessionDate, durationMinutes, billingParticipants, ownerEmail, declaredPlayerCount, { resourceType })` → returns `billingBreakdown[]` with per-participant `minutesAllocated`, `overageFee`, `guestFee`, plus totals and `guestPassesUsed`.

**Database writes** (all-or-nothing within transaction):

4. **Find or create session**: `findOverlappingSession(resourceId, sessionDate, startTime, endTime)` uses Postgres `tsrange` with `[)` bounds and the overlap operator `&&`. If found, call `linkParticipants(existingSession.id, participants, tx)`. If not found, call `createSession({ resourceId, sessionDate, startTime, endTime, trackmanBookingId, createdBy }, participants, source, tx)`.
5. **Record usage ledger**: For each billing entry:
   - **Guest**: `recordUsage(sessionId, { memberId: ownerEmail, minutesCharged: 0, guestFee, ... })` — fee assigned to host, zero minutes to avoid double-counting in host's daily usage.
   - **Member/Owner**: `recordUsage(sessionId, { memberId: email, minutesCharged, overageFee, ... })`.
6. **Deduct guest passes** (if `guestPassesUsed > 0`):
   - **Path 1 — Booking request flow** (has `bookingId`): `SELECT guest_pass_holds FOR UPDATE` → verify hold exists. `SELECT guest_passes FOR UPDATE` → verify `passes_used + passesToConvert <= passes_total`. `UPDATE guest_passes SET passes_used += passesToConvert`. `DELETE FROM guest_pass_holds`.
   - **Path 2 — Staff/Trackman flow** (no `bookingId`): `SELECT guest_passes FOR UPDATE` → verify available ≥ needed. `UPDATE guest_passes SET passes_used += passesNeeded`.
   - If either path fails validation → throw error → entire transaction rolls back.

### 5. Trackman Link

Trackman webhooks and CSV imports link external bookings to app bookings:

- **Webhook auto-approve**: `tryAutoApproveBooking()` matches pending bookings by email + date + time (±10 minute tolerance). On match, sets `status='approved'`, links `trackman_booking_id`, and calls `ensureSessionForBooking()`. After session creation and fee calculation, creates a draft Stripe invoice via `createDraftInvoiceForBooking()` (non-blocking, simulator bookings only).
- **Webhook create**: `createBookingForMember()` tries to match existing `[PENDING_TRACKMAN_SYNC]` bookings first, then creates new booking records. Uses `was_auto_linked=true` flag.
- **Duration updates**: If Trackman reports different duration, update `booking_requests` and `booking_sessions` times, then call `recalculateSessionFees()` for delta billing, then sync the draft invoice via `syncBookingInvoice()` to update line items.
- **Bay changes**: If Trackman reports a different `resource_id`, update both `booking_requests.resource_id` and `booking_sessions.resource_id`, then broadcast availability for old and new bays.

See `references/trackman-sync.md` for full details.

### 6. Check-in (status: `attended` or `no_show`)

Staff marks booking as attended or no-show via the BookingStatusDropdown. The dropdown allows toggling between statuses after initial selection. Session must exist before check-in. If no session exists yet, the check-in flow calls `ensureSessionForBooking()` to create one.

### 7. Cancellation

Two cancellation paths:

**Member cancel** (`PUT /api/booking-requests/:id/member-cancel`):

1. Validate ownership — three accepted conditions:
   - Session email matches booking's `user_email`.
   - Admin/staff with `acting_as_email` matching booking email.
   - Session user has linked email matching booking email (checked via `trackman_email`, `linked_emails` JSONB, or `manually_linked_emails` JSONB).
2. Verify booking is not already cancelled.
3. Update `booking_requests.status = 'cancelled'`.
4. Release guest pass holds via `releaseGuestPassHold(bookingId)`.
5. Cancel all pending Stripe payment intents linked to the booking (`status IN ('pending', 'requires_payment_method', 'requires_action', 'requires_confirmation')`).
5a. Handle Stripe invoice cleanup via `voidBookingInvoice(bookingId)` (non-blocking). This handles all invoice states: draft (deletes), open (voids), paid (auto-refunds via `stripe.refunds.create`, notifies staff on failure), void/uncollectible (skips).
6. Delete Google Calendar event via `deleteCalendarEvent(calendarEventId)`.
7. Publish `booking_cancelled` event with `cleanupNotifications: true` (deletes related notifications).
8. Broadcast availability update.

**Trackman cancel** (`cancelBookingByTrackmanId()`):

1. Find booking by `trackman_booking_id`.
2. If already cancelled, return early.
3. Detect if `wasPendingCancellation` (status was `cancellation_pending`).
4. Delegate to `BookingStateService`:
   - If `wasPendingCancellation` → `BookingStateService.completePendingCancellation()`.
   - Otherwise → `BookingStateService.cancelBooking()` with `source: 'trackman_webhook'` and staff notes `'[Cancelled via Trackman webhook]'`.
5. `BookingStateService` handles all side effects atomically via a side-effects manifest:
   - Refund Stripe payment intents (paid → refund, pending → cancel).
   - Refund balance payments (credit balance restoration).
   - Void booking invoice via `voidBookingInvoice(bookingId)`.
   - Delete Google Calendar event.
   - Release guest pass holds.
   - Notify staff and member (push + WebSocket).
   - Publish `booking_cancelled` event.
   - Broadcast availability update.

### 8. Auto Check-In (status: `attended`)

The auto-complete scheduler (`server/schedulers/bookingAutoCompleteScheduler.ts`) runs every 2 hours. It marks approved/confirmed bookings as `attended` (auto checked-in) when 24 hours have passed since the booking's end time. This assumes most members attended their bookings and avoids noisy false no-show notifications. Staff can still manually mark a booking as `no_show` via the BookingStatusDropdown if needed.

- Uses Pacific timezone via `getTodayPacific()` / `formatTimePacific()`
- Notifies staff when 2+ bookings are auto checked-in
- Manual trigger available via `runManualBookingAutoComplete()`

### 9. Booking Modifications (Trackman Webhooks)

Booking modifications are handled exclusively through Trackman webhook `booking.updated` events via `handleBookingModification()` in `server/routes/trackman/webhook-handlers.ts`. When a booking is modified in Trackman (bay change, time change), the webhook updates the local record, recalculates fees, syncs the invoice, and notifies the member. The previous staff-initiated reschedule feature (start/confirm/cancel routes) was removed in v8.41.0.

## Conflict Detection Detail

`findConflictingBookings(memberEmail, date, startTime, endTime, excludeBookingId?)` checks two sources:

1. **Owner conflicts**: Query `booking_requests` where `LOWER(user_email)` matches and `status IN OCCUPIED_STATUSES` on the same date. Apply `timePeriodsOverlap()` in application code.
2. **Participant conflicts**: Query `booking_participants` → `booking_sessions` → `booking_requests` where `bp.user_id` matches the member's UUID and `bp.invite_status = 'accepted'`. Also apply `timePeriodsOverlap()`. (Note: All participants are now auto-accepted — `invite_status` defaults to `'accepted'` at the database level. The invite system and `inviteExpiryScheduler` have been removed as of v7.92.0.)

`timePeriodsOverlap()` handles cross-midnight by adding 1440 minutes to any end time < start time. However, cross-midnight bookings cannot be created through normal flows (club closes at 10 PM).

`hasTimeOverlap(start1, end1, start2, end2)` in `bookingValidation.ts` handles overnight wrap-around closures (where `start2 > end2`, e.g., 22:00–06:00). When detected, it splits the range into two sub-ranges: `[start2, 1440)` (night portion) and `[0, end2)` (morning portion), checking overlap against each. This ensures facility closures spanning midnight correctly block bookings in both the late-night and early-morning windows.

**Audit note (Feb 2026):** The SQL-level overlap checks in `checkBookingConflict`, `checkAvailabilityBlockConflict`, and `checkSessionConflict`/`checkSessionConflictWithLock` use the standard `start < endTime AND end > startTime` pattern without overnight handling. This was audited and confirmed correct — bookings, availability blocks, and sessions are all single-day constructs that cannot cross midnight. Only closure overlap checks need the midnight-spanning logic.

`checkUnifiedAvailability(resourceId, date, startTime, endTime, excludeSessionId?)` runs three layered checks:

1. `checkClosureConflict()` — facility-wide closures from `facility_closures` table.
2. `checkAvailabilityBlockConflict()` — per-resource event blocks from `availability_blocks` table (e.g., tournaments, private events).
3. `checkSessionConflict()` — existing sessions on `booking_sessions` with time overlap (`start_time < endTime AND end_time > startTime`).

For pessimistic locking during concurrent session creation, `checkSessionConflictWithLock()` uses `FOR UPDATE NOWAIT` to immediately fail if another transaction holds the lock on a conflicting session row.

## Key Decision Trees

### Approval vs Auto-Approve

```
Is resource a conference room?
├── Yes → Auto-confirm (status='confirmed'), create session + draft invoice (same invoice flow as simulators since v8.16.0)
│   └── Invoice auto-finalized and charged to member's Stripe credit balance
└── No (golf simulator) → Pending staff approval (status='pending')
    └── Trackman webhook arrives?
        ├── Yes, matching pending booking → Auto-approve, create session
        └── No match → Save to trackman_unmatched_bookings
```

### Conference Room vs Simulator at Approval

```
Resource type?
├── conference_room → Final status = 'attended' (skip 'approved' stage)
└── simulator → Final status = 'approved'
```

## Key Invariants

1. **Session before roster**: A `booking_sessions` row must exist before any `booking_participants` can be linked. The session is the anchor for the participant roster and usage ledger.

2. **Conflict detection scope**: `findConflictingBookings()` checks OCCUPIED_STATUSES = `['pending', 'pending_approval', 'approved', 'confirmed', 'checked_in', 'attended']`. It checks both owner bookings and participant bookings on the same date. The `attended` status was added in v8.6.0 to prevent double-booking against checked-in sessions. The auto-complete scheduler moves stale `approved`/`confirmed` bookings to `attended` after 24h, removing them from conflict detection.

3. **Availability guard layers**: `checkUnifiedAvailability()` runs three checks in order:
   - Facility closures (`facility_closures` table)
   - Availability blocks (`availability_blocks` table)
   - Existing sessions (`booking_sessions` table with time overlap)

3a. **Overnight closure detection**: `hasTimeOverlap()` supports wrap-around time ranges where `startMinutes > endMinutes` (e.g., 22:00 to 06:00). The range is split into `[start, 1440)` and `[0, end)` for overlap checks. This is critical for closures that span midnight.

4. **Guest pass atomicity**: Guest pass deduction happens INSIDE the session creation transaction. If session creation fails, guest passes are not deducted. Two paths: hold-then-convert (booking request flow) vs direct deduction (staff/Trackman flow).

5. **Social tier restriction**: Social tier members cannot bring guests. Enforced by `enforceSocialTierRules()` before session creation. This is a hard block, not a warning.

6. **Overlapping session reuse**: `findOverlappingSession()` uses Postgres `tsrange` overlap operator (`&&`) to find existing sessions within the time window. If a session already exists (e.g., from Trackman import), participants are linked to it rather than creating a duplicate.

7. **Time tolerance matching**: Trackman webhook matching uses ±10 minute tolerance (`ABS(EXTRACT(EPOCH FROM (start_time::time - $3::time))) <= 600`). This accounts for Trackman sessions starting slightly earlier/later than booked times.

8. **Row-level locking**: `checkSessionConflictWithLock()` uses `FOR UPDATE NOWAIT` on `booking_sessions` for pessimistic locking during session creation. Guest pass deduction uses `FOR UPDATE` on `guest_passes` for atomic read-modify-write.

9. **Usage ledger stores emails**: `usage_ledger.member_id` stores email addresses (not UUIDs) for historical consistency. The `resolveUserIdToEmail()` step converts UUIDs to emails before ledger writes.

10. **Post-commit notifications**: Booking creation sends the HTTP response BEFORE executing post-commit operations (staff notifications, event publishing, availability broadcast). This ensures the client gets a success response even if notifications fail.

12. **Fee calculation is post-commit**: `recalculateSessionFees()` uses the global `db` pool, NOT transaction handles. It MUST run after `db.transaction()` commits, never inside it. Inside an uncommitted transaction, the global pool cannot see the new session/participant rows (Postgres Read Committed isolation), causing $0 fees or deadlock. (v8.26.7, Bug 22)

13. **Cancellation status guard includes 'confirmed'**: The `wasApproved` check in `completeCancellation` includes both `'approved'` AND `'confirmed'` statuses to prevent members from cancelling an in-progress booking without Trackman cleanup. (v8.26.7, Bug 14)

14. **Optimistic locking on status transitions**: `devConfirmBooking` uses `WHERE status IN ('pending', 'pending_approval')` on the UPDATE to prevent TOCTOU races where a concurrent cancellation could be overwritten. Always check `rowCount` after status-transition UPDATEs. (v8.26.7, Bug 11)

15. **One invoice per booking**: Each booking (simulator or conference room) has at most one Stripe invoice (`booking_requests.stripe_invoice_id`). Draft created at approval (if fees > $0), updated on roster changes, finalized at payment. If a booking is approved with $0 fees (no invoice created) and later gains fees through roster edits, `syncBookingInvoice()` creates the draft invoice on-the-fly. Conference rooms were migrated to the same invoice flow as simulators in v8.16.0 (2026-02-24). The invoice lifecycle is managed by `bookingInvoiceService.ts`.

16. **Roster lock after paid invoice**: Once a booking's Stripe invoice is paid, roster edits (add/remove participant, change player count) are blocked via `enforceRosterLock()`. Staff can override with a reason (logged via audit). The lock is fail-open: if the Stripe API check fails, edits proceed.

17. **Conflict check scope**: `checkBookingConflict()`, `checkAvailabilityBlockConflict()`, and `checkSessionConflict()` use simple time overlap logic (`start < end AND end > start`). This is correct because bookings and availability blocks are constrained to single-day, same-day time windows — cross-midnight bookings cannot be created (club closes at 10 PM). Only `hasTimeOverlap()` in closure checks needs overnight wrap-around handling, because facility closures CAN span midnight (e.g., 22:00–06:00). **`checkBookingConflict()` must check all 6 active statuses**: `pending`, `pending_approval`, `approved`, `confirmed`, `attended`, `cancellation_pending` — matching the inline SQL in `bookings.ts` line 568. Missing statuses allow double-bookings through code paths that use `checkAllConflicts`. (v8.26.7)

19. **Booking expiry includes pending_approval + Trackman guard + WebSocket broadcast**: The booking expiry scheduler (`bookingExpiryScheduler.ts`) targets both `pending` and `pending_approval` bookings. It waits 20 minutes past the booking `start_time` before acting. **Trackman-linked bookings** (those with a `trackman_booking_id`) are set to `cancellation_pending` instead of `expired`, so the Trackman hardware cleanup flow runs and the physical bay is unlocked. Non-Trackman bookings are set to `expired` directly. After each status change, the scheduler calls `broadcastAvailabilityUpdate()` for every affected booking with a `resourceId`, so front desk iPads and member phones update instantly. The scheduler tracks its `setTimeout` timer IDs and clears them on shutdown. The manual expiry endpoint also includes `pending_approval` in its query.

20. **Roster fetch race protection**: `fetchRosterData()` in `useUnifiedBookingLogic.ts` uses a `rosterFetchIdRef` counter to prevent stale roster data from overwriting current state. Each fetch increments the counter, and after `await`, checks that it hasn't been superseded by a newer fetch. This prevents rapid-fire WebSocket events from causing the UI to flicker with old data. (v8.26.7)

18. **Pending booking soft lock**: Pending booking requests (`pending`, `pending_approval`) on a specific bay block that bay/time slot in member-facing availability (`server/routes/availability.ts`), preventing other members from requesting the same slot. The soft lock is self-exclusive: a member's own pending requests are excluded from the lock so they see their own slot as accessible, not blocked. Only bay-assigned requests (`resource_id IS NOT NULL`) trigger the soft lock — preference-only requests (where the member chose "No Preference" for bay) do not block specific bays. The frontend renders soft-locked slots with an amber "Requested" indicator via `ResourceCard` (`requested` prop). The creation-time hard block in `server/routes/bays/bookings.ts` and `server/routes/staff/manualBooking.ts` checks `('pending', 'pending_approval', 'approved', 'confirmed', 'attended', 'cancellation_pending')` — all six active statuses must be present in both. The availability query cache key (`bookGolfKeys.availability`) includes `userEmail` to ensure self-exclusion is per-user when admins use "view as." Note: `cancellation_pending` is treated as an occupied booking (in the `bookedSlots` query alongside `approved`/`confirmed`), NOT as a soft lock — this ensures the slot stays blocked for ALL members (including the owner) per Rule 6 of booking-import-standards.

## Booking Event System

`bookingEvents` provides a pub/sub system via `publish()`:

| Event | When |
|---|---|
| `booking_created` | After booking request inserted |
| `booking_approved` | After staff approves booking |
| `booking_declined` | After staff declines booking |
| `booking_cancelled` | After cancellation (member or Trackman) |
| `booking_checked_in` | After staff marks attended |

Each event can trigger member notifications (push, WebSocket, email) and staff notifications independently via `PublishOptions`.

## Reconciliation

After a booking is completed, `findAttendanceDiscrepancies()` compares `declared_player_count` (from member's request) against `trackman_player_count` (from Trackman data). If they differ:

- Calculate potential fee adjustment based on additional players × duration × overage rate.
- Staff reviews via reconciliation UI: mark as `reviewed` or `adjusted`.
- Fee adjustments use `calculatePotentialFeeAdjustment()` — computes per-player minute allocation and applies 30-minute block overage rates.

See `references/trackman-sync.md` for reconciliation details.

### Audit Findings (Feb 2026)

#### Date/String Type Safety in Booking Events

`bookingEvents.publish()` receives `data.bookingDate` from database queries that may return a `Date` object instead of a string. The `formatBookingDateTime()` function now accepts `string | Date` and converts Date objects to ISO date strings before calling `formatDateDisplayWithDay()`. The `BookingEventData.bookingDate` type was updated to `string | Date`.

**Rule:** When passing date values from database query results to formatting functions that expect strings, always handle both `Date` and `string` types. Database ORMs may return Date objects for date columns.

#### Owner User ID Resolution (Trackman Auto-Link)

`ensureSessionForBooking()` now resolves the owner's `user_id` from the `users` table via email when callers don't pass `ownerUserId`. Previously, Trackman auto-link paths (e.g., `tryMatchByBayDateTime`) created owner participants with NULL `user_id`, making slots appear empty in the roster UI.

**Rule:** All session creation paths must ensure the owner participant has a valid `user_id`. If the caller doesn't provide `ownerUserId`, resolve it from the `users` table by email.

#### Link Member sessionId Scoping (Feb 2026)

`PUT /api/admin/booking/:bookingId/members/:slotId/link` declared `sessionId` with `const` inside the `if (session_id)` block. After the block closed, the notification and `broadcastBookingRosterUpdate()` code at the bottom of the handler referenced `sessionId` outside its scope, causing a `ReferenceError`. The DB write (participant INSERT) succeeded, so the member appeared after a page refresh, but the catch block returned HTTP 500 with "Failed to link member to slot."

**Fix:** Moved `sessionId` declaration to the outer function scope (`const sessionId = bookingResult.rows[0]?.session_id`) before the `if` block. This ensures `sessionId` is accessible throughout the handler for notifications, audit logging, and WebSocket broadcast.

**Rule:** Always declare variables at the scope where ALL consumers can access them. Block-scoped `const`/`let` inside `if` blocks are invisible to code after the closing brace.

#### Reassign Endpoint Invoice Sync (Feb 2026)

`PUT /api/admin/booking/:id/reassign` called `recalculateSessionFees()` after reassigning a booking to a new member, but did NOT call `syncBookingInvoice()`. The fee service correctly computed $0 fees (new member has allowance), but the Stripe draft invoice retained the original overage charge from the old member. This caused members like Kenneth Lee to see a $50 overage charge even after reassignment.

**Fix:** Added `syncBookingInvoice(bookingId, sessionId)` immediately after `recalculateSessionFees()` in the reassign endpoint. The invoice now reflects the recalculated $0 fees.

**Rule:** Every caller of `recalculateSessionFees()` that modifies the billing context (roster change, reassignment, participant add/remove) MUST also call `syncBookingInvoice()` afterward. `recalculateSessionFees()` only updates `booking_participants.cached_fee_cents` — it does NOT touch the Stripe invoice. See fee-calculation skill for the full list of callers.

#### Staff Assignment FK Violation

Frontend was sending `staff.user_id || staff.id` as `member_id` when assigning bookings to staff. If a staff member lacks a `users` record, `user_id` is null and `staff.id` (from `staff_users` table) is NOT a valid `users.id` UUID, causing a foreign key violation on `booking_requests.user_id`. Fixed: frontend sends `staff.user_id || null`, and backend resolves `user_id` from the owner's email via the `users` table when no valid `member_id` is provided.

**Rule:** Never use `staff_users.id` as a substitute for `users.id`. They are different tables with different ID spaces. Always use `staff.user_id` (which references `users.id`) or resolve from email.

#### Resolve Endpoint Usage Ledger and Invoice Sync (Feb 2026)

`PUT /api/admin/trackman/unmatched/:id/resolve` had two bugs:

1. **`usage_ledger.member_id` stored integer user ID instead of email.** `recordUsage()` was called with `member.id` (integer primary key from `users` table) instead of `member.email`. This violated invariant 10 (usage_ledger stores emails) and caused fee calculations to miss the resolved member's prior usage, producing incorrect overage charges.

2. **Missing `syncBookingInvoice()` after `recalculateSessionFees()`.** The resolve endpoint recalculated fees but never synced the Stripe draft invoice, leaving stale line items (e.g., $50 overage from the unmatched state) on the invoice even after the member was resolved with $0 fees.

**Fix:** Changed all three `recordUsage` calls (visitor path line 751, member path lines 808 and 816) to use `(member.email as string).toLowerCase()`. Added `syncBookingInvoice(booking.id, sessionId)` immediately after `recalculateSessionFees()`.

**Rule:** When writing to `usage_ledger.member_id`, ALWAYS use the member's email address (lowercased), never the integer user ID. This applies to `recordUsage()` calls and direct `UPDATE usage_ledger` statements.

#### Trackman Webhook SQL Safety

Trackman webhook handlers (`webhook-billing.ts`, `webhook-validation.ts`) had production SQL errors caused by `undefined` values in Drizzle `sql` template literals creating empty placeholders. Fixed by coalescing all optional values with `?? null`.

**Rule:** Always use `${value ?? null}` in Drizzle `sql` template literals for optional parameters. See `stripe-webhook-flow` skill for the full pattern.

## Roster Lock After Payment (v8.37.0)

Once a booking's invoice has been paid, the roster is **locked**. `enforceRosterLock()` in `rosterService.ts` checks `isBookingInvoicePaid(bookingId)` before allowing any roster mutation (add, remove, update player count). If the invoice is paid:
- Non-admin users are blocked outright (HTTP 403).
- Admin users can override with a required `lockOverrideReason` parameter.
- All overrides are audit-logged with the admin's email and reason.

This prevents fee discrepancies after payment — adding a player after payment would change fees but the invoice is already finalized.

## Trackman Booking Modification Webhooks (v8.34.0)

When Trackman sends a Booking Update webhook with changes (bay, time, date), the app automatically:

1. Detects the modification type (bay change, time change, date change) by comparing webhook data against the existing booking.
2. Updates the booking record, session, fees, and invoice to reflect the new slot.
3. Runs conflict detection on the new slot — if conflicts exist, staff are warned but the change is applied (Trackman is source of truth).
4. Notifies staff via WebSocket with details about what changed.
5. Sends push notifications to affected members.
6. Records the event as `booking.modified` in webhook stats (purple badge).

**Idempotency:** The webhook idempotency guard uses content-aware signatures (bay, time, status) so modification webhooks are not rejected as duplicates of the original creation webhook.

## References

**Core Reference Docs:**
- `references/server-flow.md` — Detailed API route → core service call chains for each booking operation.
- `references/trackman-sync.md` — Trackman webhook handling, CSV import matching, reconciliation, and delta billing.

**Key Implementation Files:**
- `server/core/bookingService/tierRules.ts` — Tier validation rules, daily minute limits, Social tier guest restrictions, and `TierValidationResult` interface. See `references/server-flow.md#tier-validation-rules` for detailed documentation.
- `server/core/bookingService/sessionManager.ts` — Session creation, participant linking, usage ledger recording, and guest pass deduction.
- `server/core/bookingService/bookingStateService.ts` — `BookingStateService` class: centralized cancellation (member and Trackman), `completePendingCancellation()`, side-effects manifest for atomic cleanup (Stripe refunds, balance restoration, invoice void, calendar deletion, guest pass release, notifications).
- `server/core/bookingService/conflictDetection.ts` — Booking conflict detection (owner and participant conflicts).
- `server/core/bookingValidation.ts` — Centralized booking conflict detection (`checkBookingConflict`). Used by booking creation for consistent conflict validation with advisory locks.
- `server/core/bookingService/availabilityGuard.ts` — Availability validation (closures, blocks, session overlaps).
- `server/core/billing/bookingInvoiceService.ts` — Draft invoice creation, line item sync, finalization, voiding, paid-status check. Key exports: `createDraftInvoiceForBooking`, `syncBookingInvoice`, `finalizeAndPayInvoice`, `finalizeInvoicePaidOutOfBand`, `voidBookingInvoice`, `isBookingInvoicePaid`. `finalizeAndPayInvoice()` includes terminal payment detection: before charging, it checks for existing terminal payments on the booking to avoid double-charging. It is also balance-aware: if the customer's Stripe balance fully covers the invoice amount, the invoice is finalized and auto-paid without requiring a card charge.
- `server/core/bookingService/rosterService.ts` — Roster changes with `enforceRosterLock()` guard. Exports: `addParticipant`, `removeParticipant`, `updateDeclaredPlayerCount`, `applyRosterBatch`.

**Related Skills:**
- Refer to `booking-import-standards` skill for CSV parsing rules, roster protection, and import data integrity rules.
