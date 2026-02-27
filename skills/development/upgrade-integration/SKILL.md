---
name: upgrade-integration
description: Integrate Carnegie Learning's UpGrade A/B testing platform into LMS and EdTech applications. Guides setup of decision points, experiment conditions, LTI/xAPI integration, and outcome logging. Use when asked to add A/B testing, experiments, or feature flags to educational software.
allowed-tools: Read, Write, Edit, Bash, WebFetch, Glob, Grep
---

# UpGrade A/B Testing Integration for EdTech

You are helping integrate [UpGrade](https://github.com/CarnegieLearningWeb/UpGrade), Carnegie Learning's open-source A/B testing platform designed specifically for educational technology.

## When This Skill Applies

- Adding A/B testing or experiments to educational apps
- Integrating feature flags in learning platforms
- Connecting EdTech apps to LMS via LTI with experiment support
- Setting up learning analytics with xAPI + experimentation
- Debugging experiment assignment issues

## Critical Context

UpGrade is NOT a generic A/B testing tool. It's designed for education with features like:
- **Group-level assignment** (whole classrooms get same condition)
- **Consistency rules** (students don't get re-randomized)
- **Educational context metadata** (grade, subject, proficiency)

**ALWAYS** ask these questions before implementing:

1. What's the assignment unit? (Individual student vs classroom vs school)
2. What happens if a student changes classes mid-experiment?
3. How will teachers access both conditions (or will they)?
4. Is the app used in LMS iframes? (Cookie issues!)

## Quick Reference

### Installation

```bash
# JavaScript/TypeScript
npm install upgrade_client_lib

# Java (Maven)
<dependency>
  <groupId>com.carnegielearning</groupId>
  <artifactId>upgrade-client</artifactId>
</dependency>
```

### Core Integration Pattern

```typescript
import UpgradeClient from 'upgrade_client_lib/dist/browser';
import { MARKED_DECISION_POINT_STATUS } from 'upgrade_client_lib';

// 1. Initialize (MUST complete before getDecisionPointAssignment)
const client = new UpgradeClient(userId, hostUrl, context);
await client.init(groupData, workingGroupData);

// 2. Get assignment at decision point
const assignment = await client.getDecisionPointAssignment('feature_name', 'target');
const condition = assignment.getCondition();
const payload = assignment.getPayload();

// 3. Mark that condition was applied
client.markDecisionPoint('feature_name', MARKED_DECISION_POINT_STATUS.CONDITION_APPLIED);

// 4. Log outcomes
client.log('score', 85);
client.log('completion_time_ms', 45000);
```

### Three SDK Modes

| Mode | Use Case | Requires init()? |
|------|----------|------------------|
| **Standard** | Pre-registered users with stored groups | Yes |
| **Ephemeral** | Runtime-provided groups, no DB lookup | No |
| **Merged** | Combine session + stored groups | Yes |

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         LMS                                  │
│                  (Canvas, Moodle, etc.)                     │
└─────────────────────┬───────────────────────────────────────┘
                      │ LTI 1.3 Launch
                      │ (user_id, context_id, roles)
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                    Your EdTech App                          │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Identity Mapper                                      │   │
│  │ LTI user_id → UpGrade userId + altUserIds           │   │
│  └─────────────────────┬───────────────────────────────┘   │
│                        │                                    │
│  ┌─────────────────────▼───────────────────────────────┐   │
│  │ UpGrade SDK                                          │   │
│  │ - init(groupData)                                    │   │
│  │ - getDecisionPointAssignment()                       │   │
│  │ - markDecisionPoint()                                │   │
│  │ - log()                                              │   │
│  └─────────────────────┬───────────────────────────────┘   │
└─────────────────────────┼───────────────────────────────────┘
                          │ HTTPS
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    UpGrade Backend                          │
│            (Self-hosted or Carnegie Learning)               │
└─────────────────────────────────────────────────────────────┘
```

## CRITICAL: Known Pitfalls

Before writing ANY integration code, read `resources/pitfalls-and-edge-cases.md`. The most dangerous issues:

### 1. Third-Party Cookie Blocking (CRITICAL)

If your app runs in an LMS iframe, Safari and Chrome WILL block your session cookies by default.

**Symptoms:**
- User gets re-randomized on every page load
- `init()` seems to work but assignments are inconsistent
- Works in development, fails in production LMS

**Solutions:**
- Set `SameSite=None; Secure` on all cookies
- Use localStorage with user-provided identifier (not session-based)
- Consider opening in new window instead of iframe

### 2. Race Condition: init() vs getDecisionPointAssignment()

```typescript
// WRONG - race condition
const client = new UpgradeClient(userId, hostUrl, context);
client.init(groupData); // async, not awaited!
const assignment = await client.getDecisionPointAssignment('feature');
// assignment may be NO_CONDITION_ASSIGNED
```

```typescript
// CORRECT
const client = new UpgradeClient(userId, hostUrl, context);
await client.init(groupData); // AWAIT this!
const assignment = await client.getDecisionPointAssignment('feature');
```

### 3. LTI Identity Mismatch

LTI provides `user_id` which is opaque and LMS-specific. Your app may use email or internal IDs.

```typescript
// Map all identities together
await client.init(groupData);
client.setAltUserIds([
  ltiUserId,           // from LTI launch
  userEmail,           // from your auth
  internalDatabaseId   // your user table PK
]);
```

### 4. Group Assignment Drift

Student transfers from Mrs. Smith's class (Treatment A) to Mr. Jones' class (Treatment B).

**Default behavior:** Student KEEPS original assignment (consistency rule)

**Problem:** Student now sees different UI than classmates, causing confusion.

**Solution:** Decide upfront:
- Accept drift (experimental purity)
- Re-assign on class change (classroom consistency)
- Exclude transferred students from analysis

### 5. Shared Device / Chromebook Cart Problem

```typescript
// WRONG - userId persists from previous student
const client = new UpgradeClient(getUserIdFromLocalStorage(), ...);

// CORRECT - validate identity on each session
const client = new UpgradeClient(getCurrentAuthenticatedUserId(), ...);
// Clear any cached assignments if userId changed
```

## Decision Point Patterns for Education

See `resources/decision-point-patterns.md` for detailed examples:

| Pattern | Example | Assignment Unit |
|---------|---------|-----------------|
| **UI Variant** | New vs old problem interface | Individual |
| **Pedagogical** | Worked examples vs practice problems | Classroom |
| **Adaptive Algorithm** | Mastery threshold 80% vs 90% | Individual |
| **Content Ordering** | Teach fractions before decimals vs after | School |
| **Feedback Timing** | Immediate vs delayed feedback | Classroom |

## Code Templates

See `templates/` directory for:
- `react-hook.tsx` - React hook with proper error handling
- `nextjs-middleware.ts` - Next.js API route pattern
- `vanilla-js.js` - Framework-agnostic implementation
- `lti-launch-handler.ts` - LTI 1.3 launch with UpGrade init

## Logging Best Practices

```typescript
// DO: Use consistent types
client.log('score', 85);              // number
client.log('completed', true);         // boolean
client.log('response', 'correct');     // string

// DON'T: Mix types for same metric
client.log('score', 85);
client.log('score', '85');  // String! Will corrupt analysis

// DON'T: Log before marking decision point
client.log('score', 85);  // Not associated with condition!
client.markDecisionPoint('quiz', MARKED_DECISION_POINT_STATUS.CONDITION_APPLIED);

// DO: Mark first, then log
client.markDecisionPoint('quiz', MARKED_DECISION_POINT_STATUS.CONDITION_APPLIED);
client.log('score', 85);  // Correctly associated
```

## Troubleshooting Decision Tree

```
Assignment returns NO_CONDITION_ASSIGNED
├── Did you await init()?
│   └── No → Add await before init()
├── Is experiment in ENROLLING state?
│   └── No → Check UpGrade UI, start experiment
├── Does user match segment criteria?
│   └── No → Check inclusion/exclusion rules
├── Is context correct?
│   └── No → Verify context string matches experiment config
└── Check network tab for failed requests
```

```
User gets different condition than classmates
├── Is assignment unit set to GROUP?
│   └── No → Change to GROUP in experiment settings
├── Was groupData provided to init()?
│   └── No → Pass { classId: 'xxx' } to init()
├── Did student join after experiment started?
│   └── Yes → Check POST_EXPERIMENT_RULE
└── Is student in multiple groups?
    └── Yes → Check group priority/override logic
```

## External Resources

- [UpGrade GitHub](https://github.com/CarnegieLearningWeb/UpGrade)
- [UpGrade Documentation](https://upgrade-platform.gitbook.io/docs/)
- [JavaScript SDK](https://github.com/CarnegieLearningWeb/UpGrade/tree/develop/clientlibs/js)
- [LTI 1.3 Spec](https://www.imsglobal.org/spec/lti/v1p3)
- [xAPI Spec](https://xapi.com/overview/)

## When to Escalate

Recommend the developer contact UpGrade support or file a GitHub issue if:
- Assignment algorithm seems incorrect (statistical anomaly)
- Data export shows impossible values
- Backend returns 5xx errors consistently
- Need custom consistency rules not supported by SDK
