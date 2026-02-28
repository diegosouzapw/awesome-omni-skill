---
name: Proactive Orchestrator
description: |
  Event-driven orchestrator that chains existing skills, edge functions, and agent capabilities
  into automated workflows triggered by real-time events. Receives events from webhooks
  (MeetingBaaS, email, calendar), cron jobs (morning brief, pipeline scan), and Slack interactions
  (button clicks, approvals), then executes multi-step sequences with context-aware routing,
  HITL approval gates, and self-invocation for long-running chains.
  This is the conductor — it connects 30+ existing Slack functions, 6 specialist agents,
  and the sequence executor into coherent, event-driven sales workflows.
  NOT triggered by user chat messages. Triggered by system events only.
metadata:
  author: sixty-ai
  version: "1"
  category: agent-sequence
  skill_type: sequence
  is_active: true
  context_profile: orchestration
  agent_affinity:
    - pipeline
    - outreach
    - meetings
    - crm_ops
    - research
    - prospecting
  triggers:
    - pattern: "meeting_ended"
      intent: "post_meeting_workflow"
      confidence: 0.95
      examples:
        - "meetingbaas webhook received"
        - "transcript ready for processing"
        - "recording complete"
    - pattern: "pre_meeting_90min"
      intent: "meeting_prep_workflow"
      confidence: 0.95
      examples:
        - "upcoming meeting in 90 minutes"
        - "meeting prep trigger"
        - "calendar event approaching"
    - pattern: "email_received"
      intent: "email_triage_workflow"
      confidence: 0.90
      examples:
        - "new email from prospect"
        - "email webhook fired"
        - "inbound email detected"
    - pattern: "proposal_generation"
      intent: "proposal_pipeline_workflow"
      confidence: 0.90
      examples:
        - "generate proposal for deal"
        - "proposal requested by rep"
        - "intent detected: send proposal"
    - pattern: "calendar_find_times"
      intent: "scheduling_workflow"
      confidence: 0.90
      examples:
        - "find available meeting times"
        - "scheduling request detected"
        - "calendar availability needed"
    - pattern: "stale_deal_revival"
      intent: "deal_revival_workflow"
      confidence: 0.85
      examples:
        - "stale deal detected in pipeline scan"
        - "deal inactive for 14+ days"
        - "pipeline scan flagged deal"
    - pattern: "campaign_check"
      intent: "campaign_monitoring_workflow"
      confidence: 0.85
      examples:
        - "daily campaign check"
        - "instantly campaign metrics"
        - "outreach performance review"
    - pattern: "coaching_digest"
      intent: "coaching_workflow"
      confidence: 0.85
      examples:
        - "weekly coaching analysis"
        - "meeting pattern review"
        - "rep performance digest"
    - pattern: "morning_brief"
      intent: "morning_briefing_workflow"
      confidence: 0.95
      examples:
        - "daily morning briefing"
        - "8am daily digest"
        - "start of day summary"
    - pattern: "slack_approval_received"
      intent: "resume_paused_sequence"
      confidence: 0.95
      examples:
        - "rep clicked approve button"
        - "HITL approval received"
        - "slack action: approve"
  keywords:
    - "orchestrator"
    - "event-driven"
    - "workflow"
    - "chain"
    - "sequence"
    - "proactive"
    - "automation"
    - "webhook"
    - "post-meeting"
    - "pipeline"
    - "approval"
  required_context:
    - org_id
    - user_id
  inputs:
    - name: event_type
      type: string
      description: "The event that triggered this orchestration (meeting_ended, email_received, etc.)"
      required: true
    - name: event_source
      type: string
      description: "Origin of the event (webhook:meetingbaas, cron:morning, slack:button_approve, orchestrator:chain)"
      required: true
    - name: payload
      type: object
      description: "Event-specific data (meeting_id, email_id, contact_id, deal_id, etc.)"
      required: true
    - name: parent_job_id
      type: string
      description: "If this event was chained from another sequence, the parent sequence_jobs.id"
      required: false
    - name: resume_step
      type: number
      description: "If resuming a paused sequence (after HITL approval), the step index to resume from"
      required: false
  outputs:
    - name: sequence_job_id
      type: string
      description: "The sequence_jobs.id for tracking this orchestration run"
    - name: steps_completed
      type: array
      description: "List of skill/action names that completed successfully"
    - name: outputs
      type: object
      description: "Accumulated outputs from each step, keyed by step name"
    - name: pending_approvals
      type: array
      description: "Actions waiting for HITL approval via Slack"
    - name: queued_followups
      type: array
      description: "Follow-up events queued for future execution"
    - name: errors
      type: array
      description: "Any errors encountered during execution"
  requires_capabilities:
    - meetings
    - crm
    - email
    - messaging
    - calendar
    - enrichment
  priority: critical
  linked_skills:
    - detect-intents
    - find-available-slots
    - email-send-as-rep
    - monitor-campaigns
    - coaching-analysis
    - meeting-digest-truth-extractor
    - post-meeting-followup-drafter
    - sales-sequence
    - proposal
    - lead-research
    - daily-brief-planner
    - deal-slippage-diagnosis
  tags:
    - agent-sequence
    - orchestrator
    - proactive
    - event-driven
    - automation
    - pipeline
    - meetings
    - critical-path
---

## Available Context & Tools
@_platform-references/org-variables.md
@_platform-references/capabilities.md
@references/event-sequences.md
@references/context-tiers.md
@references/hitl-patterns.md

# Proactive Orchestrator

The single edge function that transforms isolated capabilities into an autonomous sales copilot. Every event flows through this orchestrator — it decides what sequence of existing skills to run, in what order, with what context, and where to pause for human approval.

## Design Principle

**Extend, don't rebuild.** The orchestrator calls existing edge functions and skills. It does NOT duplicate their logic. It is a router, context loader, and sequence runner — nothing more.

- No new queue tables — extends `sequence_jobs` with 3 columns
- No new preference tables — reads existing `slack_user_preferences`, `notification_feature_settings`, `slack_user_mappings`
- No polling heartbeat — every event has a clear trigger source
- Self-invokes when approaching the 150s edge function timeout

## Architecture

```
EVENT SOURCE                     ORCHESTRATOR                        EXISTING CAPABILITIES
                                 (agent-orchestrator edge fn)

meetingbaas-webhook ────┐        ┌────────────────────┐        ┌──────────────────────────┐
                        │        │                    │        │ extract-action-items      │
proactive-meeting-prep ─┤        │ 1. Receive event   │───────>│ suggest-next-actions      │
                        │        │ 2. Load context    │        │ copywriter skill          │
slack-interactive ──────┤───────>│    (3 tiers)       │        │ generate-proposal         │
                        │        │ 3. Select sequence │        │ proactive-pipeline-anal.  │
email webhook ──────────┤        │ 4. Chain skills    │        │ proactive-meeting-prep    │
                        │        │ 5. Route output    │        │ Slack delivery modules    │
calendar webhook ───────┤        │ 6. Queue followups │        │ CRM update functions      │
                        │        │                    │        │ Apollo / Apify / Instantly│
cron: morning brief ────┘        └────────────────────┘        │ sequence executor         │
                                                               └──────────────────────────┘
```

## How It Works

### Step 1: Receive and Validate Event

Every orchestration starts with an `OrchestratorEvent`:

```typescript
interface OrchestratorEvent {
  type: string          // 'meeting_ended' | 'email_received' | 'slack_action' | 'pre_meeting' | etc.
  source: string        // 'webhook:meetingbaas' | 'cron:morning' | 'slack:button_approve' | etc.
  org_id: string
  user_id: string
  payload: Record<string, any>
  parent_job_id?: string  // If chained from another sequence
}
```

Validate the event:
1. Confirm `org_id` and `user_id` exist
2. Check `notification_feature_settings` — is this event type enabled for the org?
3. Check `slack_user_preferences` — is the user within active hours? (Quiet hours = defer, don't drop)
4. Check cost budget via `costTracking.ts` — halt if budget exceeded
5. Check deduplication via `dedupe.ts` — skip if identical event processed within window

If validation fails at any step, log the reason and exit gracefully. Never silently drop events — always log why they were skipped.

### Step 2: Load Context (Tiered)

Context loading uses the existing `_shared/proactive/` modules, formalised into explicit tiers. Only load what the sequence needs — never load tier 3 data speculatively.

See `references/context-tiers.md` for the full tier specification.

**Tier 1 — Always loaded** (every orchestration):
- Org profile, user preferences, feature settings, ICP, product profile, cost budget
- Source: existing `_shared/proactive/` modules + `costTracking.ts`
- Latency: ~200ms (cached in most cases)

**Tier 2 — Per-contact** (when event involves a specific contact/deal):
- CRM contact, company, deal, meeting history, email thread, activities
- Source: existing CRM service functions
- Latency: ~500ms (multiple queries)
- Only loaded when `payload.contact_id` or `payload.deal_id` is present

**Tier 3 — On-demand** (loaded by specific skills that need external data):
- Apollo enrichment, LinkedIn via Apify, company news, proposal templates, campaign metrics
- Source: external API calls
- Latency: 2-15s per source
- Only loaded when a skill explicitly requires it (declared in `requires_context`)

### Step 3: Select Event Sequence

Map the event type to its sequence of skills/actions. The full event-sequence mapping is in `references/event-sequences.md`.

```typescript
const sequence = EVENT_SEQUENCES[event.type]
if (!sequence) {
  log.warn(`No sequence defined for event type: ${event.type}`)
  return
}
```

Each sequence is an ordered array of steps. Each step either:
- Invokes an **existing skill** (by `skill_key`)
- Calls an **existing edge function** (by function name)
- Executes a **CRM action** (create task, update deal, etc.)
- **Branches** based on previous step output
- **Queues a follow-up event** for the orchestrator to process next

### Step 4: Execute Sequence Steps

Process steps sequentially. After each step:

1. **Check result** — did the skill/action succeed?
2. **Store output** — add to `state.outputs[step_name]`
3. **Check for branches** — does this step's output trigger conditional paths?
4. **Check for approval gates** — does the next step require HITL?
5. **Check timeout** — if approaching 150s, persist state and self-invoke
6. **Check cost** — call `checkCostBudget()` before any AI-intensive step

```typescript
interface SequenceState {
  event: OrchestratorEvent
  context: {
    tier1: OrgContext
    tier2?: ContactContext
    tier3?: Record<string, any>
  }
  steps_completed: string[]
  current_step: number
  outputs: Record<string, any>
  pending_approvals: PendingAction[]
  queued_followups: OrchestratorEvent[]
  started_at: string
  cost_accumulated: number
}
```

**Execution rules:**
- Each step gets the accumulated `state.outputs` from all previous steps
- Steps with `requires_approval: true` pause the sequence and send a Slack message
- Steps with `on_failure: 'continue'` log the error and proceed to the next step
- Steps with `on_failure: 'stop'` halt the sequence and notify the user
- Steps with `condition` are only executed if the condition evaluates to true

### Step 5: Handle Approval Gates (HITL)

When a step requires approval:

1. Format the approval request using the appropriate HITL pattern (see `references/hitl-patterns.md`)
2. Create a `slack_pending_actions` record with the full context needed to resume
3. Send the Slack message via existing `_shared/proactive/deliverySlack.ts`
4. Persist the current `SequenceState` to `sequence_jobs.context`
5. Record `sequence_jobs.status = 'awaiting_approval'`
6. Exit the edge function — the sequence will resume when the rep acts

**Resuming after approval:**
When the rep clicks a button in Slack → `slack-interactive` edge function → routes to orchestrator with:
```typescript
{
  type: 'slack_approval_received',
  source: 'slack:button_approve',
  payload: {
    action: 'approve' | 'edit' | 'skip' | 'cancel',
    pending_action_id: string,
    sequence_job_id: string,
    resume_step: number,
    user_modifications?: any  // If they edited the draft
  }
}
```

The orchestrator reloads `SequenceState` from `sequence_jobs.context` and resumes from `resume_step`.

### Step 6: Queue Follow-up Events

Some steps produce follow-up events. For example, `detect-intents` might detect "I'll send you a proposal" — this queues a `proposal_generation` event.

```typescript
// After detect-intents completes:
const intents = state.outputs['detect-intents']
for (const commitment of intents.commitments) {
  if (commitment.speaker === 'rep' && commitment.intent === 'send_proposal' && commitment.confidence >= 0.8) {
    state.queued_followups.push({
      type: 'proposal_generation',
      source: 'orchestrator:chain',
      org_id: state.event.org_id,
      user_id: state.event.user_id,
      payload: {
        meeting_id: state.event.payload.meeting_id,
        contact_id: state.context.tier2?.contact?.id,
        trigger_phrase: commitment.phrase,
      },
      parent_job_id: currentJobId,
    })
  }
}
```

After the current sequence completes (or at a natural pause point), process queued follow-ups:
- Each follow-up becomes a new `sequence_jobs` record with `event_chain.parent` linking to the originator
- Follow-ups are processed by self-invoking the orchestrator edge function
- Chain depth is limited to 3 (prevent infinite loops)

### Step 7: Self-Invocation for Long Sequences

Edge functions timeout at 150s. Long sequences must checkpoint and continue:

```typescript
const TIMEOUT_BUFFER_MS = 15_000  // 15s safety buffer
const startTime = Date.now()

for (let i = state.current_step; i < sequence.length; i++) {
  // Check if we're running out of time
  if (Date.now() - startTime > (150_000 - TIMEOUT_BUFFER_MS)) {
    // Persist state and self-invoke
    await persistState(state, sequenceJobId)
    await selfInvoke({
      type: state.event.type,
      source: 'orchestrator:continuation',
      payload: { sequence_job_id: sequenceJobId, resume_step: i },
      parent_job_id: sequenceJobId,
    })
    return  // Exit current invocation
  }

  await executeStep(sequence[i], state)
}
```

## State Storage: sequence_jobs

All state lives in `sequence_jobs` — the existing table extended with 3 new columns:

```sql
ALTER TABLE sequence_jobs ADD COLUMN IF NOT EXISTS event_source TEXT;
ALTER TABLE sequence_jobs ADD COLUMN IF NOT EXISTS event_chain JSONB;
ALTER TABLE sequence_jobs ADD COLUMN IF NOT EXISTS trigger_payload JSONB;
```

- `event_source`: Where the event came from (`webhook:meetingbaas`, `cron:morning`, `slack:button_approve`, `orchestrator:chain`)
- `event_chain`: Links parent/child sequences (`{ parent: uuid, children: uuid[] }`)
- `trigger_payload`: The raw event payload that started this sequence
- `context` (existing JSONB column): Stores the full `SequenceState` for resumption
- `status` (existing): `pending` | `in_progress` | `awaiting_approval` | `completed` | `failed`

## Event Routing Table

Every event has a clear, non-polling trigger:

| Event | Trigger Mechanism | Source Value |
|---|---|---|
| Meeting ended | MeetingBaaS webhook -> `meetingbaas-webhook` edge fn | `webhook:meetingbaas` |
| Pre-meeting prep | `proactive-meeting-prep` cron (90min before) | `cron:pre_meeting` |
| Morning brief | pg_cron -> `call_proactive_edge_function()` RPC | `cron:morning` |
| Pipeline scan | pg_cron -> `proactive-pipeline-analysis` | `cron:pipeline_scan` |
| Email received | Gmail/O365 push notification or poll webhook | `webhook:email` |
| Slack button click | `slack-interactive` edge fn | `slack:button_approve` |
| Calendar event created | Google Calendar webhook | `webhook:calendar` |
| Sequence step completed | Orchestrator self-invokes | `orchestrator:chain` |
| Sequence continuation | Self-invocation after timeout | `orchestrator:continuation` |
| Campaign check | pg_cron daily | `cron:campaign_check` |
| Coaching digest | pg_cron weekly | `cron:coaching_weekly` |

## Error Handling

### Per-Step Errors

Each step declares its failure mode:
- `on_failure: 'stop'` — Halt the entire sequence. Notify the user via Slack with error context.
- `on_failure: 'continue'` — Log the error, skip this step, proceed to the next.
- `on_failure: 'retry'` — Retry once with exponential backoff. If retry fails, treat as `stop`.

### Sequence-Level Errors

- **Budget exceeded**: Halt immediately. Send Slack notification: "AI budget limit reached. Pausing proactive workflows until next billing period."
- **Rate limited**: Defer the sequence. Re-queue with a 5-minute delay.
- **External API failure** (Apollo, Instantly, etc.): Skip the enrichment step, continue with available data.
- **Edge function timeout approaching**: Checkpoint state and self-invoke (see Step 7).
- **Chain depth exceeded** (>3): Log warning, do not create further follow-ups. Complete current sequence.
- **Duplicate event**: Skip silently (handled by `dedupe.ts` in Step 1).

### Observability

Every orchestration run creates an audit trail:

```typescript
{
  sequence_job_id: string,
  event_type: string,
  event_source: string,
  steps_attempted: number,
  steps_completed: number,
  steps_failed: number,
  approval_gates_hit: number,
  followups_queued: number,
  total_duration_ms: number,
  total_ai_cost: number,
  chain_depth: number,
}
```

This is stored in `sequence_jobs.context.execution_metadata` for debugging and performance monitoring.

## Integration Points

### Existing Edge Functions Called by Orchestrator

| Function | When Called |
|---|---|
| `extract-action-items` | `meeting_ended` step 1 |
| `suggest-next-actions` | `meeting_ended` step 3 |
| `generate-proposal` | `proposal_generation` step 3 |
| `proactive-pipeline-analysis` | `morning_brief` pipeline section |
| `proactive-meeting-prep` | `pre_meeting_90min` briefing |
| `slack-post-meeting` | Post-meeting Slack notification |
| `meeting-workflow-notifications` | Meeting status updates |
| `create-task-from-action-item` | Task creation from commitments |

### New Skills Called by Orchestrator

| Skill | When Called |
|---|---|
| `detect-intents` | `meeting_ended` step 2 — commitment and signal extraction |
| `find-available-slots` | `calendar_find_times` — mutual availability |
| `email-send-as-rep` | After approval of any email draft |
| `monitor-campaigns` | `campaign_check` — daily Instantly metrics |
| `coaching-analysis` | `coaching_digest` — weekly or per-meeting |

### Existing Shared Modules Used

| Module | Purpose |
|---|---|
| `_shared/proactive/recipients.ts` | Determine who receives notifications |
| `_shared/proactive/deliverySlack.ts` | Send Slack messages with Block Kit |
| `_shared/proactive/dedupe.ts` | Prevent duplicate event processing |
| `_shared/proactive/settings.ts` | Load feature settings and preferences |
| `_shared/proactive/types.ts` | Shared type definitions |
| `_shared/costTracking.ts` | AI cost logging and budget checking |
| `_shared/corsHelper.ts` | Origin-validated CORS for the edge function |

## Safeguards

### Rate Limiting
- Respects `slack_user_preferences.max_notifications_per_hour` from existing table
- Respects quiet hours from `slack_user_preferences.quiet_hours_start/end`
- Defers (does not drop) messages outside active hours — queues for next active window

### Cost Control
- Calls `checkCostBudget(org_id)` before every AI-intensive step
- Halts entire sequence if budget exceeded — does not proceed to "just one more step"
- Logs cumulative cost per orchestration run in `execution_metadata`

### Chain Depth
- Maximum chain depth: 3 (event -> follow-up -> follow-up -> STOP)
- Prevents: meeting_ended -> detect_intents -> proposal_generation -> email_send -> (blocked)
- Each chain creates a new `sequence_jobs` record with `event_chain.parent` for full traceability

### Approval Gates
- Every email send requires HITL approval (no exceptions, even if user preferences say "auto-approve")
- Every CRM field update that changes deal stage requires HITL approval
- Task creation from detected intents requires approval
- Slack summaries are inform-only (no approval needed)

### Data Privacy
- Orchestrator runs with user-scoped Supabase client (respects RLS)
- Service role only for cross-user operations (org-wide pipeline scan) — documented and audited
- No conversation data stored outside existing `copilot_conversations` table
- Meeting transcripts referenced by ID, not copied into orchestrator state

## Quality Checklist

Before considering an orchestration run successful, verify:

- [ ] All required context tiers loaded without error
- [ ] Event deduplication check passed
- [ ] Cost budget check passed before each AI step
- [ ] Each step's output matches expected schema
- [ ] Approval gates sent correct Slack messages with action buttons
- [ ] Paused sequences have full state persisted in `sequence_jobs.context`
- [ ] Follow-up events have valid `parent_job_id` links
- [ ] Chain depth did not exceed 3
- [ ] Execution metadata recorded (duration, cost, steps completed)
- [ ] No sensitive data leaked into logs (emails, transcript content)
- [ ] Rate limits respected (Slack notifications, quiet hours)
- [ ] Edge function completed within 150s (or self-invoked for continuation)

## Morning Brief: Enhanced Format

The orchestrator upgrades the existing morning brief with richer context by chaining multiple data sources:

```
Good morning, {name}. Here's your {day_of_week}.

MEETINGS TODAY ({count})
{for each meeting}
  {time} -- {contact_name}, {company} ({meeting_type})
  {if prep_ready} [Prep ready ->]
  {if prep_in_progress} [Prepping now...]
  {if no_prep_needed} No prep needed
{end}

NEEDS YOUR ATTENTION ({count})
{for each attention_item}
  {contact_name} ({company}) -- {summary}
  [{primary_action} ->] [{secondary_action} ->]
{end}

PIPELINE PULSE
  {deals_needing_action} deals need action today
  {at_risk_count} deal(s) at risk ({deal_name} -- {days_inactive} days no activity)
  {proposal_value} in proposals awaiting response

OVERNIGHT UPDATES
{for each update}
  {icon} {update_summary}
{end}

[Expand pipeline ->] [Show full inbox ->]
```

Data sources chained:
1. `calendar_events` (today's meetings with attendees)
2. `proactive-meeting-prep` (prep status per meeting)
3. Email inbox scan (threads needing response)
4. `proactive-pipeline-analysis` (deal health, stale deals)
5. Campaign metrics from Instantly (if campaigns active)
6. CRM activity log (overnight changes)

## File Locations

### New Files (Build 1)
- `supabase/functions/agent-orchestrator/index.ts` — The edge function
- `supabase/functions/_shared/orchestrator/types.ts` — TypeScript interfaces
- `supabase/functions/_shared/orchestrator/contextLoader.ts` — Tiered context loading
- `supabase/functions/_shared/orchestrator/eventSequences.ts` — Event-to-sequence mapping
- `supabase/functions/_shared/orchestrator/runner.ts` — Step execution engine

### New Adapter Files (Build 1)
- `supabase/functions/_shared/orchestrator/adapters/actionItems.ts` — Wraps `extract-action-items`
- `supabase/functions/_shared/orchestrator/adapters/nextActions.ts` — Wraps `suggest-next-actions`
- `supabase/functions/_shared/orchestrator/adapters/createTasks.ts` — Wraps `create-task-from-action-item`
- `supabase/functions/_shared/orchestrator/adapters/emailClassifier.ts` — Email intent classification
- `supabase/functions/_shared/orchestrator/adapters/detectIntents.ts` — Wraps `detect-intents` skill
- `supabase/functions/_shared/orchestrator/adapters/proposalGenerator.ts` — Wraps `generate-proposal`

### Migration
- `supabase/migrations/YYYYMMDD_extend_sequence_jobs_orchestrator.sql`

### Existing Files Modified (Build 1)
- `supabase/functions/meetingbaas-webhook/index.ts` — Add orchestrator call after transcript processing
- `supabase/functions/proactive-meeting-prep/index.ts` — Upgrade to call orchestrator for richer pipeline
- `supabase/functions/slack-interactive/index.ts` — Route approval actions back to orchestrator
