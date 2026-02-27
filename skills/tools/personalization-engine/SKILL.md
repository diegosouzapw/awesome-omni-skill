---
name: personalization-engine
description: Central hub for managing user preferences, learning patterns, and adapting skill behavior based on historical feedback. Enables "tell me once" paradigm where the system remembers and adapts.
allowed-tools: Read, Write, Bash
---

# Personalization Engine Skill

**Purpose:** Central preference management that enables skills to learn from user feedback, persist decisions across sessions, and adapt behavior based on individual patterns.

**When to use:**
- User asks about their preferences ("what have you learned about me?")
- User wants to adjust confidence thresholds
- User says "don't show this again" or "remember this preference"
- Skills need to read/write user preferences
- User wants to see personalization analytics

**Key Innovation:** "Tell me once" paradigm - decisions persist across sessions, skills learn and adapt.

**Location:** `~/.claude/user-preferences.json` (global, not per-project)

---

## The Personalization Problem

### Without Personalization

**Typical experience:**
```
Session 1:
User: "Skip that suggestion"
Claude: [Suggestion hidden for this session]

Session 2:
User: "Why are you showing me that again?!"
Claude: [No memory of previous decision]
```

**Problems:**
- Users repeat same decisions session after session
- Skills don't learn from feedback
- No persistence of "don't show again" choices
- Generic suggestions that don't match user style
- **40% productive time lost** to re-establishing context

### With Personalization Engine

**New experience:**
```
Session 1:
User: "Skip that suggestion"
Claude: [Records: user skipped 'verbose-logging' - now at 3 skips]
        "Got it. I've noted you prefer to skip verbose logging suggestions.
         After 3 skips, I won't suggest this again unless you ask."

Session 2:
Claude: [Reads preferences, sees 'verbose-logging' skipped 3x]
        [Automatically filters out verbose logging suggestions]
        [User never sees it again]

User: "Why don't you suggest verbose logging?"
Claude: "You've skipped verbose logging suggestions 3 times, so I stopped showing them.
         Say 'reset verbose-logging preference' if you'd like to see them again."
```

**Benefits:**
- "Tell me once" → System remembers forever
- Skills learn from accept/reject/skip patterns
- Confidence thresholds adapt to user style
- Personalized suggestions based on history
- Transparency: User can see what system learned

---

## Operation 1: Read User Preferences

**User Queries:**
- "What are my preferences?"
- "Show my personalization settings"
- "What have you learned about me?"
- "Show my skill usage analytics"

### Reading Steps

1. **Check for preferences file:**
   ```bash
   # Check if preferences file exists
   ls ~/.claude/user-preferences.json 2>/dev/null
   ```

2. **Load preferences:**
   - If exists: Parse JSON and return relevant sections
   - If not exists: Return defaults with "no preferences recorded yet" message

3. **Return formatted summary:**

### Response Template: Preferences Summary

```markdown
## Your Personalization Profile

**Learning Period:** 45 days (since 2025-11-01)
**Total Decisions Tracked:** 234
**Overall Acceptance Rate:** 72%

---

### Profile Settings

| Setting | Value |
|---------|-------|
| Experience Level | Intermediate |
| Primary Languages | TypeScript, Python |
| Proactivity Level | Medium |
| Preferred Workflow | Wizard |

---

### Confidence Thresholds

| Action | Your Threshold | Default | Difference |
|--------|---------------|---------|------------|
| Auto-apply | 95% | 90% | +5% (more conservative) |
| Suggest prominently | 75% | 75% | Same |
| Show as optional | 50% | 50% | Same |
| Hide below | 30% | 25% | +5% (less noise) |

---

### Top Skills by Usage

| Rank | Skill | Invocations | Last Used |
|------|-------|-------------|-----------|
| 1 | version-management | 78 | Today |
| 2 | commit-readiness-checker | 56 | Today |
| 3 | security-scanner | 34 | Yesterday |
| 4 | test-generator | 28 | 3 days ago |
| 5 | standards-enforcer | 22 | Today |

---

### Learned Preferences

**High Acceptance (>75%):**
- Null checks before dereferencing (92%)
- Conventional commit messages (88%)
- TypeScript strict mode (85%)

**Low Acceptance (<40%):**
- Early returns (23%) - You prefer explicit conditionals
- Verbose logging (12%) - You prefer minimal logging
- JSDoc comments (35%) - You prefer inline comments

**Permanently Skipped:**
- sequential-thinking MCP setup (skipped 3x, permanent)
- Git hook installation (skipped 2x, permanent)

---

### Recommendations

Based on your patterns:

1. **workflow-analyzer** (78% confidence)
   - You commit frequently (15 commits this week)
   - This skill could identify optimization patterns
   - Estimated time: 2 minutes to enable

2. **Raise auto-apply threshold to 97%** (65% confidence)
   - You rarely accept auto-applied changes
   - Higher threshold = fewer surprises

Would you like to adjust any of these settings?
```

### Response Template: No Preferences Yet

```markdown
## Personalization Status

**Status:** No preferences recorded yet

You're starting fresh! As you use Claude Code, I'll learn your preferences:

**What I'll Learn:**
- Which suggestions you accept vs reject
- Your preferred coding style patterns
- Which recommendations you skip
- Your confidence threshold preferences

**How It Works:**
1. Use skills normally (commit-readiness, security-scanner, etc.)
2. Accept, reject, or skip suggestions
3. Preferences are saved to `~/.claude/user-preferences.json`
4. Future sessions adapt based on your history

**Privacy:**
- All data stored locally on your machine
- Never synced to cloud
- You can view, edit, or delete anytime

Start using skills and I'll begin learning your preferences!
```

---

## Operation 2: Update Preferences

**User Queries:**
- "Remember that I prefer X"
- "Don't show this suggestion again"
- "Set my proactivity level to low"
- "Update my confidence threshold to 90%"
- "Skip this recommendation permanently"

### Update Types

#### Type 1: Direct Preference Setting

User explicitly sets a preference.

**Example:**
```
User: "Set my proactivity level to low"

Response:
✅ Updated proactivity level: medium → low

What this means:
- Only high-confidence suggestions will be shown
- Optional suggestions will be hidden
- Minimal interruptions during workflow

To revert: Say "set proactivity to medium"
```

#### Type 2: Feedback-Based Learning

Skills report user decisions, engine updates acceptance rates.

**Feedback Protocol:**

```typescript
interface PreferenceFeedback {
  skill: string;         // e.g., "standards-enforcer"
  category: string;      // e.g., "coding-style"
  item: string;          // e.g., "early-returns"
  action: "accept" | "reject" | "skip" | "dont-show-again";
  context?: string;      // Optional context
  timestamp: string;     // ISO date
}
```

**Learning Algorithm:**

```
New acceptance rate = (old_rate × old_samples + new_value) / (old_samples + 1)

Where:
- accept = 1.0
- reject = 0.0
- skip = 0.3 (weak negative signal)

Example:
- Old rate: 0.60 (60%), 10 samples
- User rejects suggestion
- New rate: (0.60 × 10 + 0.0) / 11 = 0.545 (54.5%)
```

#### Implicit Learning Signals (v4.23.0)

**Enhancement:** Detect signal strength from user language, not just explicit actions.

**Signal Taxonomy:**

| Signal Type | Keywords | Value | Weight | Example |
|-------------|----------|-------|--------|---------|
| **Strong Positive** | "exactly", "perfect", "love it", "that's great" | 1.0 | 2x | "That's exactly what I wanted!" → 2.0 |
| **Enthusiasm** | "wow", "awesome", "brilliant", "yes!" | 1.0 | 1.5x | "Wow, that's really cool" → 1.5 |
| **Neutral Accept** | (default accept) | 1.0 | 1x | (silence = acceptance) → 1.0 |
| **Weak Negative** | "skip", "not now", "later" | 0.3 | 1x | "Skip that" → 0.3 |
| **Correction** | "actually", "instead", "not that", "wrong" | 0.0 | 2x | "Actually, not that way" → -2.0 (double negative) |
| **Strong Negative** | "never", "don't", "stop", "no" | 0.0 | 2x | "Never do that" → -2.0 (double negative) |

**Enhanced Learning Algorithm:**

```
Base signal = action value (1.0, 0.3, or 0.0)
Detected weight = keyword multiplier (1x, 1.5x, or 2x)
Effective signal = base × weight

New acceptance rate = (old_rate × old_samples + effective_signal) / (old_samples + 1)

Examples:

1. Strong positive: "That's exactly what I wanted!"
   - Base: 1.0 (accept)
   - Weight: 2x (strong positive keyword)
   - Effective: 2.0
   - New rate: (0.60 × 10 + 2.0) / 11 = 0.727 (72.7%)

2. Correction: "Actually, let's not do early returns"
   - Base: 0.0 (reject)
   - Weight: 2x (correction keyword)
   - Effective: 0.0 (double weight on rejection = faster learning)
   - New rate: (0.60 × 10 + 0.0) / 11 = 0.545, then apply -1x correction
   - Adjusted: 0.545 - 0.1 = 0.445 (44.5%)

3. Neutral skip: "Skip that"
   - Base: 0.3 (skip)
   - Weight: 1x (no keyword)
   - Effective: 0.3
   - New rate: (0.60 × 10 + 0.3) / 11 = 0.573 (57.3%)
```

**Correction Signal Detection (v4.23.0):**

Detects when user immediately edits AI-generated content:

**Pattern:**
```
AI generates code → User edits within 1 minute → Correction signal
```

**Action:**
1. Record pattern: "Don't generate [pattern] for [file type]"
2. Apply high negative weight (2x)
3. Add to "learned patterns" with correction timestamp

**Example:**
```markdown
## Correction Detected

**What happened:**
1. I suggested: `if (!condition) return;`
2. You changed it to: `if (condition) { ... }`

**What I learned:**
- Pattern: early-returns
- File type: typescript
- Your preference: explicit conditionals
- Confidence: High (correction signal)

**Future behavior:**
I'll avoid suggesting early returns in TypeScript files. Current acceptance rate for early-returns: 45% → 25% (correction applied).
```

**Keyword Detection Process:**

1. **On user message:** Scan for signal keywords
2. **Match patterns:** Check against signal taxonomy
3. **Calculate weight:** Apply multiplier (1x, 1.5x, 2x)
4. **Update preferences:** Use enhanced algorithm
5. **Provide feedback:** Show what was learned

**Privacy:**
- Keyword matching only (no full message storage)
- Only sentiment detected (not message content)
- User can disable: "Don't detect implicit signals"

#### File-Context Memory (v4.24.0)

**Enhancement:** Tag preferences with file paths - recall patterns when editing the same file.

**The Problem:**
```
Session 1: User edits CLAUDE.md, prefers sentence-case headers
Session 2: User edits README.md, prefers title-case headers
Session 3: User edits CLAUDE.md again
          → System suggests title-case (wrong context!)
```

**The Solution:**
Tag preferences with file path or file pattern, recall when editing same file.

**File-Context Tagging:**

| Context Level | Pattern | Example |
|---------------|---------|---------|
| **Exact File** | Full path | `/path/to/CLAUDE.md` → sentence-case headers |
| **File Pattern** | Glob pattern | `*.md` → markdown linting rules |
| **Directory** | Directory path | `/docs/` → documentation style |
| **File Type** | Extension | `.ts` → TypeScript conventions |

**Storage Format:**

```json
{
  "fileContextPreferences": {
    "CLAUDE.md": {
      "header-style": {
        "preference": "sentence-case",
        "acceptanceRate": 0.95,
        "samples": 12,
        "lastUsed": "2025-12-22T10:30:00Z"
      }
    },
    "*.md": {
      "line-length": {
        "preference": "no-limit",
        "acceptanceRate": 0.80,
        "samples": 25
      }
    },
    "docs/**/*.md": {
      "emoji-usage": {
        "preference": "section-headers-only",
        "acceptanceRate": 0.90,
        "samples": 15
      }
    }
  }
}
```

**Matching Priority:**

1. **Exact file path** (highest priority)
2. **Specific glob pattern** (e.g., `docs/**/*.md`)
3. **General file type** (e.g., `*.md`)
4. **Directory pattern** (e.g., `/docs/`)
5. **Global preference** (fallback)

**Trigger Pattern:**

```
User edits file → System checks fileContextPreferences
                → Loads preferences for matching patterns
                → Applies to suggestions for this file
```

**Example:**

```markdown
## File-Context Detected

**File:** CLAUDE.md
**Context loaded:**
- Header style: sentence-case (95% acceptance, 12 samples)
- Line length: no-limit (80% acceptance, inherited from *.md)
- Emoji usage: section-headers-only (90% acceptance, inherited from docs/)

**Applying these preferences to suggestions for this file.**
```

**Learning:**

When user accepts/rejects suggestion while editing a file:

1. Update global preference (as before)
2. **Also tag with current file context**
3. Store both general + file-specific learning

**Conflict Resolution:**

If file-context preference conflicts with global:
- **File-context wins** (more specific)
- Show user: "Using CLAUDE.md preference (sentence-case) instead of global (title-case)"

**Commands:**

```
"Show file-context preferences for CLAUDE.md"
"What have you learned about this file?"
"Reset file-context for *.md"
"Disable file-context memory"
```

#### Recovery Pattern Learning (v4.24.0)

**Enhancement:** Learn from failure → success sequences to proactively suggest solutions.

**The Problem:**
```
User tries approach A → Fails
User tries approach B → Fails
User tries approach C → Success!
Next time: System doesn't remember C was the winner
```

**The Solution:**
Track task → attempt → outcome sequences, remember successful approaches.

**Recovery Pattern Detection:**

```
Pattern: Multiple attempts on same task before success
Trigger: 2+ failures followed by success within same session
Action: Record the successful approach as "proven solution"
```

**Pattern Structure:**

```json
{
  "recoveryPatterns": {
    "fix-typescript-import-error": {
      "context": {
        "errorType": "typescript-import",
        "attempts": 3,
        "failedApproaches": [
          "relative-path-import",
          "namespace-import"
        ],
        "successfulApproach": "default-import-with-type",
        "confidence": "high"
      },
      "metadata": {
        "firstAttempt": "2025-12-22T10:00:00Z",
        "resolved": "2025-12-22T10:15:00Z",
        "timeTaken": "15 minutes",
        "fileType": "typescript",
        "successRate": 1.0,
        "timesApplied": 3
      }
    }
  }
}
```

**Detection Logic:**

1. **Failure Detection:**
   - Test fails
   - Build error
   - User says "that didn't work", "still broken", "try again"
   - User immediately edits generated code (correction signal)

2. **Attempt Tracking:**
   - Same task attempted multiple times within 30 minutes
   - Different approaches each time
   - Track what was tried

3. **Success Detection:**
   - Tests pass after failures
   - User says "that worked!", "fixed!", "success"
   - No further edits after 5 minutes (silent success)

4. **Pattern Recording:**
   - Store: task type, failed approaches, successful approach
   - Tag with: file type, error type, context
   - Confidence: high (proven by recovery)

**Proactive Suggestions:**

Next time similar task is attempted:

```markdown
## Recovery Pattern Detected

**Task:** Fixing TypeScript import error
**I remember:** You solved this before in 3 attempts

**Failed approaches (avoid these):**
1. ❌ relative-path-import (didn't work)
2. ❌ namespace-import (didn't work)

**Successful approach (use this):**
✅ default-import-with-type (worked after 15 minutes)

**Apply proven solution?** This worked for you last time.
```

**Learning Enhancement:**

Recovery patterns have **higher confidence** than single-try learnings:
- Regular acceptance: 75% confidence
- Recovery pattern: 95% confidence (proven through struggle)

**Why this works:**

Hard-won solutions have higher value:
- User invested time (15 minutes in example)
- Multiple approaches tested
- Success validated through comparison
- **Psychological:** Struggling → success creates strong memory

**Commands:**

```
"Show recovery patterns"
"What solutions have I found through trial-and-error?"
"How did I solve [task] last time?"
"Forget recovery pattern for [task]"
```

#### Workflow Gap Detection (v4.25.0)

**Enhancement:** Identify related tasks that should be linked - automate repetitive workflows.

**The Problem:**
```
Pattern detected over 20 commits:
1. User edits version.json
2. User runs sync-version.sh
3. User edits CHANGELOG.md
4. User commits

Every single time, same sequence. Manual, repetitive, forgettable.
```

**The Solution:**
Detect workflow patterns from commit history, suggest automation or linking.

**Gap Detection:**

| Gap Type | Pattern | Suggestion |
|----------|---------|------------|
| **Sequential Tasks** | A always followed by B | "Link these tasks?" |
| **Forgotten Steps** | A → C (B missing) | "You usually do B between A and C" |
| **Manual Repetition** | Same sequence 5+ times | "Automate this workflow?" |
| **Context Switch** | Edit file A, then edit file B (unrelated) | "Consider grouping related edits" |

**Detection Logic:**

**1. Sequential Pattern Detection:**

```
Analyze last 50 commits:
- Find sequences: [edit version.json] → [run script] → [edit CHANGELOG]
- Frequency: 18/20 commits (90%)
- Time gap: Average 2 minutes between steps
→ Pattern detected: version-bump-workflow
```

**2. Forgotten Step Detection:**

```
Analyze current session:
- User edited version.json
- User edited CHANGELOG.md
- Missing: sync-version.sh (usually runs between these)
→ Gap detected: "Did you forget to run sync-version.sh?"
```

**3. Repetition Analysis:**

```
Same workflow detected 15 times:
1. git add .
2. git commit -m "..."
3. git push origin main

Time: 1-2 minutes per execution
→ Suggestion: "Create alias or git hook for this sequence?"
```

**Storage Format:**

```json
{
  "workflowPatterns": {
    "version-bump-workflow": {
      "steps": [
        {
          "action": "edit",
          "target": "version.json",
          "confidence": 0.95
        },
        {
          "action": "run",
          "target": "sync-version.sh",
          "confidence": 0.90
        },
        {
          "action": "edit",
          "target": "CHANGELOG.md",
          "confidence": 0.95
        },
        {
          "action": "commit",
          "pattern": "^(feat|fix|docs):",
          "confidence": 0.88
        }
      ],
      "metadata": {
        "frequency": 18,
        "totalCommits": 20,
        "patternStrength": 0.90,
        "avgTimePerStep": "2 minutes",
        "lastOccurrence": "2025-12-22T14:30:00Z"
      }
    }
  },
  "detectedGaps": {
    "missing-sync-script": {
      "workflow": "version-bump-workflow",
      "missingStep": "sync-version.sh",
      "frequency": 3,
      "impact": "medium"
    }
  }
}
```

**Proactive Detection:**

```markdown
## 🔗 Workflow Gap Detected

**Pattern:** version-bump-workflow
**Frequency:** 18/20 commits (90%)

**Your usual sequence:**
1. ✓ Edit version.json
2. ⚠️ Run sync-version.sh ← **You haven't done this yet**
3. ? Edit CHANGELOG.md (expected next)
4. ? Commit changes

**Suggestion:** Run sync-version.sh now to stay on pattern?

[Run now] [Skip this time] [Don't remind me]
```

**Automation Suggestions:**

After detecting high-frequency patterns (10+ occurrences):

```markdown
## 🤖 Automation Opportunity

**Workflow:** version-bump-workflow
**Frequency:** 18 times (manually repeated)
**Time cost:** ~36 minutes total (2 min × 18)

**I can help automate this:**

Option 1: Pre-commit hook
  - Auto-runs sync-version.sh before commit
  - Validates version consistency

Option 2: Bash script wrapper
  - Single command: ./bump-version.sh 4.25.0
  - Handles all 4 steps automatically

Option 3: Git alias
  - git bump-version "4.25.0"
  - Custom alias for this workflow

[Show me how] [Not now] [Never suggest for this workflow]
```

**Commands:**

```
"Show workflow patterns"
"What workflows have you detected?"
"Show gaps in current workflow"
"Suggest automation for [workflow]"
"Forget workflow pattern [name]"
```

#### Adaptive Threshold Tuning (v4.25.0)

**Enhancement:** Continuous micro-adjustments to confidence thresholds based on actual user behavior.

**The Problem:**
```
Current: Manual threshold adjustments
- User sets autoApply = 95%
- System uses 95% forever
- Even if user rejects 80% of auto-applied suggestions
→ Static threshold, not adapting to reality
```

**The Solution:**
Continuously monitor acceptance rates by confidence level, adjust thresholds automatically.

**Current (v3.10.0) AI-Suggested Tuning:**
- Runs every 7 days
- Manual review required
- Suggests threshold changes
- User must approve

**Enhanced (v4.25.0) Adaptive Tuning:**
- Runs continuously
- Micro-adjustments (±1-2%)
- No user approval needed
- Transparent feedback

**Tuning Logic:**

**1. Confidence Band Analysis:**

```
Analyze suggestions at each confidence level:

90-95%: 20 suggestions, 12 accepted (60% acceptance)
85-90%: 15 suggestions, 13 accepted (87% acceptance)
80-85%: 10 suggestions, 9 accepted (90% acceptance)

Observation: 85-90% band has HIGHER acceptance than 90-95%
→ Threshold too conservative, or 90-95% suggestions are wrong type
```

**2. Micro-Adjustment Rules:**

| Condition | Action | Magnitude |
|-----------|--------|-----------|
| 5 consecutive rejections at confidence X | Lower threshold by 2% | Gentle |
| 80%+ rejection rate in band | Lower threshold by 5% | Moderate |
| 90%+ acceptance rate in band | Raise threshold by 2% | Gentle |
| 95%+ acceptance, 20+ samples | Raise threshold by 5% | Moderate |

**3. Adjustment Limits:**

Safety constraints:
- **Max change per day:** ±10%
- **Min threshold:** 30% (never go below)
- **Max threshold:** 98% (never require perfection)
- **Cooldown:** 24 hours between major adjustments (>5%)

**Example Adaptive Flow:**

```
Day 1: autoApply threshold = 95%
       Suggestions at 95%+: 10 generated
       User rejects: 7/10 (70% rejection)

Day 2: System analyzes: "70% rejection is too high"
       Adjustment: 95% → 97% (+2%, gentle)
       Notification: "Raised autoApply to 97% (you rejected 70% at 95%)"

Day 3: Suggestions at 97%+: 5 generated
       User accepts: 4/5 (80% acceptance)

Day 5: System analyzes: "80% acceptance is good"
       No adjustment (stable)

Day 10: Suggestions at 97%+: 20 generated
        User accepts: 19/20 (95% acceptance)

Day 11: System analyzes: "95% acceptance, could be more permissive"
        Adjustment: 97% → 95% (-2%, gentle)
        Notification: "Lowered autoApply to 95% (you accept 95% at this level)"
```

**Storage Format:**

```json
{
  "adaptiveThresholds": {
    "autoApply": {
      "current": 95,
      "original": 95,
      "history": [
        {
          "timestamp": "2025-12-22T10:00:00Z",
          "value": 95,
          "reason": "user-set"
        },
        {
          "timestamp": "2025-12-23T10:00:00Z",
          "value": 97,
          "reason": "high-rejection-rate",
          "data": {
            "acceptanceRate": 0.30,
            "samples": 10
          }
        },
        {
          "timestamp": "2025-12-30T10:00:00Z",
          "value": 95,
          "reason": "high-acceptance-rate",
          "data": {
            "acceptanceRate": 0.95,
            "samples": 20
          }
        }
      ],
      "lastAdjustment": "2025-12-30T10:00:00Z",
      "adjustmentCount": 2
    }
  },
  "confidenceBandStats": {
    "95-100": {
      "suggestions": 30,
      "accepted": 27,
      "acceptanceRate": 0.90,
      "trend": "stable"
    },
    "90-95": {
      "suggestions": 45,
      "accepted": 39,
      "acceptanceRate": 0.87,
      "trend": "increasing"
    }
  }
}
```

**Transparency:**

Every adjustment is logged and explained:

```markdown
## 📊 Threshold Adjusted

**Threshold:** autoApply
**Change:** 95% → 93% (-2%)
**Reason:** High acceptance rate at this level

**Data:**
- Suggestions at 95%+: 25 in last 7 days
- Your acceptance: 24/25 (96%)
- Confidence: You're comfortable with slightly lower threshold

**Impact:** More suggestions will auto-apply
**Revert:** "Set autoApply to 95%" to undo

[View adjustment history] [Disable adaptive tuning]
```

**User Control:**

```
"Disable adaptive tuning"
"Enable adaptive tuning"
"Show threshold adjustment history"
"Revert threshold changes"
"Lock autoApply threshold at 95%"
"Show confidence band statistics"
```

**Safety Features:**

- **Preview mode:** Test adjustments for 24 hours before applying
- **Undo:** Revert to any previous threshold
- **Lock:** Prevent adjustments to specific thresholds
- **Notification:** Always inform user of changes
- **Limits:** Max ±10% per day, never below 30% or above 98%

**Integration with v4.23.0 & v4.24.0:**

```
Adaptive tuning uses:
- Implicit signals (v4.23.0): Keywords boost/lower confidence scores
- File-context (v4.24.0): Adjust thresholds per file pattern
- Recovery patterns (v4.24.0): Proven solutions get lower threshold

Example:
- Recovery pattern: 95% confidence baseline
- File-context (CLAUDE.md): 97% acceptance rate
- Adaptive tuning: Lowers threshold to 93% for CLAUDE.md
→ More recovery patterns auto-applied in this context
```

**Commands:**

```
"Show adaptive tuning status"
"Disable adaptive tuning for autoApply"
"Show threshold adjustment history"
"Why did you adjust [threshold]?"
"Revert to original thresholds"
"Lock all thresholds"
```

**Threshold Rules:**

| Acceptance Rate | Status | Behavior |
|-----------------|--------|----------|
| ≥75% | High acceptance | Show prominently |
| 50-74% | Medium | Show as optional |
| 25-49% | Low | Hide by default (proactivity=high shows) |
| <25% | Very low | Never show |

#### Type 3: Skip Counter

Tracks how many times user skips a recommendation.

```
Skip 1: Note preference
Skip 2: Lower priority
Skip 3: Mark as "permanent skip" (unless user says otherwise)
```

**Response Template: Skill Skipped**

```markdown
Got it. I've noted you skipped "sequential-thinking MCP setup".

**Skip History:**
- Skip 1: 2025-12-10
- Skip 2: 2025-12-12
- Skip 3: Today (permanent skip threshold reached)

**Action:** I won't suggest this again unless you ask.

To see it again: Say "reset sequential-thinking preference"
To see all skipped items: Say "show skipped recommendations"
```

#### Type 4: Don't Show Again

Explicit "never show this" request.

**Response Template: Don't Show Again**

```markdown
✅ Marked as "don't show again": Security scanner info-level findings

**What this affects:**
- Security scanner will hide informational findings
- Only Medium, High, and Critical will be shown

**To revert:**
Say "show security scanner info-level findings again"

**All "don't show again" items:**
1. Security scanner info-level findings
2. Test coverage reminder below 80%
3. Git hook installation prompt
```

### Update Steps

1. **Parse user intent:**
   - Direct setting change?
   - Feedback on a suggestion?
   - Skip request?
   - Don't show again?

2. **Load current preferences:**
   ```bash
   cat ~/.claude/user-preferences.json
   ```

3. **Apply update:**
   - Calculate new values (if learning-based)
   - Merge with existing preferences
   - Create backup if major change

4. **Write updated preferences:**
   ```bash
   cp ~/.claude/user-preferences.json ~/.claude/user-preferences.json.backup
   # Write new content
   ```

5. **Confirm to user:**
   - Show before/after
   - Explain impact
   - Provide revert instructions

---

## Operation 3: Adaptive Threshold Management

**User Queries:**
- "Why did you show me that suggestion?"
- "Why didn't you suggest X?"
- "Make suggestions more/less frequent"
- "Explain my threshold settings"

### How Thresholds Work

**Four threshold levels:**

```
100% ─────────────────────────────────────
      │ AUTO-APPLY ZONE                  │
 95%  ├─────────────────────────────────────  autoApply threshold
      │ PROMINENT SUGGESTION ZONE        │
 75%  ├─────────────────────────────────────  suggestProminently threshold
      │ OPTIONAL SUGGESTION ZONE         │
 50%  ├─────────────────────────────────────  showAsOptional threshold
      │ LOW CONFIDENCE ZONE              │
 30%  ├─────────────────────────────────────  hideBelow threshold
      │ HIDDEN (too low confidence)      │
  0% ─────────────────────────────────────
```

**Proactivity Level Modifiers:**

| Proactivity | autoApply | prominent | optional | hide |
|-------------|-----------|-----------|----------|------|
| Low | 98% | 85% | 70% | 50% |
| Medium | 95% | 75% | 50% | 30% |
| High | 90% | 60% | 40% | 20% |

### Response Template: Threshold Explanation

```markdown
## Why You Saw That Suggestion

**Suggestion:** "Consider using early returns"
**Confidence:** 65%
**Your Threshold:** Show as optional at ≥50%

**Decision Path:**
1. Confidence (65%) < Auto-apply (95%) → Won't auto-apply
2. Confidence (65%) < Prominent (75%) → Won't show prominently
3. Confidence (65%) ≥ Optional (50%) → ✅ Show as optional
4. Confidence (65%) ≥ Hide (30%) → Don't hide

**Result:** Shown as optional suggestion

**Your History with This Pattern:**
- Acceptance rate: 23% (5 accepted, 17 rejected)
- Trend: Declining (you've rejected last 5)

**Recommendation:**
Based on your history, I could stop showing "early returns" suggestions.
Say "don't suggest early returns" to apply this.
```

### Response Template: Why Not Suggested

```markdown
## Why I Didn't Suggest X

**Pattern:** Verbose logging
**Confidence:** 45%
**Your Threshold:** Hide below 50%

**Decision Path:**
1. Confidence (45%) < Hide threshold (50%)
2. **Result:** Hidden

**Your History:**
- Acceptance rate: 12% (2 accepted, 15 rejected)
- Status: Low acceptance (auto-hidden)

**If You Want to See It:**
1. Lower your hide threshold: "set hide threshold to 40%"
2. Reset this specific preference: "reset verbose-logging preference"
3. Ask explicitly: "suggest verbose logging for this code"
```

---

## Operation 4: Skill Usage Analytics

**User Queries:**
- "Which skills do I use most?"
- "Show my usage patterns"
- "What skills should I try?"
- "Analyze my workflow"

### Analytics Response Template

```markdown
## Your Claude Code Usage Analytics

**Analysis Period:** Last 30 days
**Total Skill Invocations:** 234

---

### Usage Distribution

```
version-management      ████████████████████ 78 (33%)
commit-readiness        ██████████████ 56 (24%)
security-scanner        █████████ 34 (15%)
test-generator          ███████ 28 (12%)
standards-enforcer      █████ 22 (9%)
other                   ███ 16 (7%)
```

---

### Usage Patterns

**Peak Usage Times:**
- Most active: Weekdays 9am-12pm
- Least active: Weekends

**Workflow Pattern Detected:** Release-focused
- High version-management usage suggests frequent releases
- commit-readiness checks before every commit
- security-scanner run before deployments

**Session Characteristics:**
- Average session length: 45 minutes
- Average skills per session: 3.2
- Most common sequence: commit-readiness → version-management → security-scanner

---

### Skill Recommendations

Based on your patterns, you might benefit from:

**1. workflow-analyzer** (85% match)
- You have 15 commits this week
- Pattern: Frequent small commits
- Benefit: Identify commit pattern optimizations
- Time to enable: 2 minutes

**2. documentation-sync-checker** (72% match)
- You use version-management frequently
- Pattern: Version updates might drift from docs
- Benefit: Catch documentation staleness early
- Time to enable: 3 minutes

**3. test-generator** (more) (65% match)
- Current usage: 28 invocations
- You run security-scanner frequently
- Pattern: Security-conscious but test coverage unknown
- Benefit: Ensure test coverage matches security focus

---

### Unused Skills

You haven't tried these skills yet:
- projects-registry (multi-project tracking)
- api-debugging (API troubleshooting)
- component-finder (React/Vue component search)

Would you like me to explain any of these?
```

---

## Operation 5: Reset Preferences

**User Queries:**
- "Reset my preferences"
- "Clear preference for X"
- "Start fresh with personalization"
- "Delete my learning history"

### Reset Options

#### Option 1: Reset Specific Preference

```markdown
✅ Reset preference: early-returns

**Before:**
- Acceptance rate: 23% (low)
- Sample size: 22 decisions
- Status: Hidden by default

**After:**
- Acceptance rate: 50% (neutral)
- Sample size: 0
- Status: Will be shown normally

The system will re-learn your preference as you use it.
```

#### Option 2: Reset Category

```markdown
✅ Reset category: coding-style

**Preferences Reset:**
- early-returns
- verbose-logging
- null-checks
- type-annotations
+ 8 more

**What Happens:**
- All coding-style suggestions return to default confidence
- System will re-learn from your decisions
- Other categories (workflow, quality, etc.) unchanged
```

#### Option 3: Full Reset

```markdown
⚠️ Full Preferences Reset

This will:
- Delete ~/.claude/user-preferences.json
- Reset all learned preferences to defaults
- Clear usage analytics
- Remove all "don't show again" items
- Clear skipped recommendations list

**Backup Created:** ~/.claude/user-preferences.json.backup.2025-12-15

Are you sure? Say "confirm reset" to proceed.
```

---

## Integration Protocol for Skills

Other skills integrate with Personalization Engine using this protocol:

### Reading Preferences (Skill Start)

```markdown
## Reading User Preferences

When a skill activates, it should:

1. **Check for preferences file:**
   - If `~/.claude/user-preferences.json` exists, read relevant sections
   - If not, use defaults

2. **Load skill-specific preferences:**
   ```json
   {
     "skillSpecificPreferences": {
       "security-scanner": {
         "showInfoLevel": true,
         "autoFixLowRisk": false
       }
     }
   }
   ```

3. **Load learned preferences for relevant categories:**
   ```json
   {
     "learnedPreferences": {
       "quality": {
         "security-scan-threshold": "high"
       }
     }
   }
   ```

4. **Load confidence thresholds:**
   ```json
   {
     "confidenceThresholds": {
       "suggestProminently": 75,
       "showAsOptional": 50
     }
   }
   ```

5. **Filter suggestions:**
   - Check `dontShowAgain.items` for items to skip
   - Check `skippedRecommendations.items` for skipped items
   - Apply confidence thresholds to each suggestion
```

### Writing Feedback (After User Decision)

```markdown
## Recording User Feedback

When user accepts/rejects/skips a suggestion:

1. **Construct feedback event:**
   ```json
   {
     "timestamp": "2025-12-15T10:30:00Z",
     "skill": "standards-enforcer",
     "category": "coding-style",
     "item": "early-returns",
     "action": "rejected",
     "context": "user preferred explicit conditional"
   }
   ```

2. **Update learned preferences:**
   - Calculate new acceptance rate
   - Update sample size
   - Record trend direction

3. **Update analytics:**
   - Increment decision counter
   - Update category breakdown
   - Add to learning history (max 100 events)

4. **Update skill usage:**
   - Increment usage counter
   - Update lastUsed timestamp

5. **Write to preferences file:**
   - Create backup first
   - Write updated JSON
   - Validate JSON structure
```

### Example: Standards Enforcer Integration

```markdown
## Standards Enforcer + Personalization

**Before (v3.7.0):**
- Standards Enforcer tracks acceptance rates internally
- Learning resets each session
- No cross-skill patterns

**After (v3.8.0):**

**On Activation:**
```
1. Read ~/.claude/user-preferences.json
2. Load learnedPreferences.coding-style
3. Filter suggestions where acceptance_rate < hideBelow threshold
4. Apply confidence modifiers from learned rates
```

**On User Decision:**
```
1. User rejects "early returns" suggestion
2. Calculate new acceptance rate
3. Write feedback to preferences file
4. Update analytics
```

**Result:**
- Learning persists across sessions
- Other skills benefit from coding-style learnings
- User sees consistent behavior
```

---

## Operation 6: Project Preferences (v3.9.0)

**User Queries:**
- "Show project preferences"
- "Set project proactivity to high"
- "Override security threshold for this project"
- "Create project preferences file"
- "Why is this project using different settings?"
- "Show effective preferences" (merged view)

### The Project Override Problem

**Global preferences don't fit all projects:**
- Security-critical project needs stricter thresholds
- Learning project wants more suggestions
- Legacy project has different coding standards
- Team project needs shared conventions

**Solution:** Project-level preference overrides that merge with global.

### Merge Logic

```
Final Preference = Global + Project Override

Merge Rules:
1. Project values override global values
2. Unspecified values inherit from global
3. Deep merge for nested objects
4. Arrays are replaced (not merged)

Example:
Global: { proactivityLevel: "medium", autoApply: 95, hideBelow: 30 }
Project: { proactivityLevel: "high" }
Result: { proactivityLevel: "high", autoApply: 95, hideBelow: 30 }
```

### File Locations

```
Global (personal):
~/.claude/user-preferences.json

Project (shared with team):
.claude/project-preferences.json

Reading order:
1. Read global preferences
2. Check for project preferences
3. Deep merge (project wins)
4. Return effective preferences
```

### Creating Project Preferences

**User says:** "Create project preferences for this project"

**Steps:**
1. Check if `.claude/project-preferences.json` exists
2. If not, create from template with sensible defaults
3. Prompt user for key overrides (proactivity, security level)
4. Save to `.claude/project-preferences.json`

**Response:**

```markdown
## Project Preferences Created

**File:** .claude/project-preferences.json

I've created a project preferences file with:
- Sparse format (only overrides, inherits rest from global)
- Reason fields for documentation
- Project context section

**Current Overrides:** None (using all global settings)

**To customize:**
- "Set project proactivity to high"
- "Set project autoApply threshold to 98%"
- "Enable strict security scanning for this project"

**Team sharing:**
This file can be committed to git so your team shares the same AI behavior.
```

### Viewing Project Preferences

**User says:** "Show project preferences"

**Response Template:**

```markdown
## Project Preferences: claude-config-template

**Status:** Project overrides active
**File:** .claude/project-preferences.json

---

### Overrides from Global

| Setting | Global | Project | Reason |
|---------|--------|---------|--------|
| proactivityLevel | medium | high | Documentation project - want thorough suggestions |
| autoApply | 95% | 98% | Be conservative with auto-changes |
| security.showInfoLevel | false | true | Show all security findings |

---

### Inherited from Global (no override)

- Experience level: intermediate
- Primary languages: typescript, python
- suggestProminently: 75%
- showAsOptional: 50%
- hideBelow: 30%

---

### Project Context

**Tech Stack:**
- Languages: TypeScript, Markdown
- Frameworks: None (documentation project)
- Testing: None

**Priorities:**
- Security: medium
- Documentation: high
- Performance: low

---

**To modify:** "Set project [setting] to [value]"
**To remove override:** "Remove project override for [setting]"
**To delete all:** "Delete project preferences"
```

### Viewing Effective Preferences

**User says:** "Show effective preferences" or "What settings am I actually using?"

**Response Template:**

```markdown
## Effective Preferences (Merged View)

**Source:** Global + Project overrides

---

### Profile
| Setting | Value | Source |
|---------|-------|--------|
| experienceLevel | intermediate | Global |
| proactivityLevel | high | **Project** |
| primaryLanguages | typescript, python | Global |

### Confidence Thresholds
| Threshold | Value | Source |
|-----------|-------|--------|
| autoApply | 98% | **Project** |
| suggestProminently | 75% | Global |
| showAsOptional | 50% | Global |
| hideBelow | 30% | Global |

### Skill Settings
| Skill | Setting | Value | Source |
|-------|---------|-------|--------|
| security-scanner | showInfoLevel | true | **Project** |
| test-generator | coverageTarget | 80% | Global |

---

**Legend:**
- Global = From ~/.claude/user-preferences.json
- **Project** = Override from .claude/project-preferences.json
```

### Modifying Project Preferences

**User says:** "Set project proactivity to low"

**Steps:**
1. Load current project preferences (or create if not exists)
2. Set `overrides.profile.proactivityLevel = "low"`
3. Add reason field if user provides one
4. Save to `.claude/project-preferences.json`

**Response:**

```markdown
✅ Updated project preference

**Setting:** proactivityLevel
**Global value:** medium
**Project override:** low

**Reason:** (none provided)

To add a reason: "Set project proactivity reason to 'Security-critical project'"

**Effective value for this project:** low
```

### Removing Project Override

**User says:** "Remove project override for proactivity"

**Steps:**
1. Load project preferences
2. Delete `overrides.profile.proactivityLevel`
3. Save changes

**Response:**

```markdown
✅ Removed project override

**Setting:** proactivityLevel
**Was:** low (project override)
**Now:** medium (global default)

This project will now use your global proactivity setting.
```

### Deleting Project Preferences

**User says:** "Delete project preferences"

**Response:**

```markdown
⚠️ Delete Project Preferences

This will:
- Delete .claude/project-preferences.json
- Revert to using global preferences only
- Remove all project-specific overrides

**Backup:** I'll create .claude/project-preferences.json.backup first

Are you sure? Say "confirm delete project preferences" to proceed.
```

### Project Context

The `projectContext` section provides hints to skills:

```json
{
  "projectContext": {
    "techStack": {
      "languages": ["typescript"],
      "frameworks": ["react"],
      "testing": ["jest"]
    },
    "priorities": {
      "security": "high",
      "performance": "medium"
    }
  }
}
```

**How skills use this:**
- **test-generator:** Uses `testing` to choose framework
- **security-scanner:** Uses `security` priority to adjust thoroughness
- **standards-enforcer:** Uses `techStack` for language-specific rules

### Team Sharing

**Key difference from global preferences:**

| Aspect | Global Preferences | Project Preferences |
|--------|-------------------|---------------------|
| Location | `~/.claude/` | `.claude/` in project |
| Scope | All projects | This project only |
| Git | Not committed | Can be committed |
| Sharing | Personal only | Team can share |
| Privacy | Private settings | Shared conventions |

**Recommendation:** Commit `.claude/project-preferences.json` to git so your team uses the same AI behavior.

---

## File Structure

```
~/.claude/
├── user-preferences.json         # Global preferences (personal)
├── user-preferences.json.backup  # Auto-backup before changes
└── ...

<project-root>/.claude/
├── project-preferences.json      # Project overrides (team-shareable)
└── ...

Templates:
├── templates/user-preferences.json.template
└── templates/project-preferences.json.template
```

---

## Privacy & Data

### What's Stored

- **Profile:** Experience level, languages, proactivity
- **Thresholds:** Confidence settings
- **Skill usage:** Which skills used, how often
- **Learned preferences:** Accept/reject patterns
- **Skipped items:** Things you've skipped
- **Learning history:** Last 100 events per category

### What's NOT Stored

- Actual code content
- File paths from your projects
- Any personally identifiable information
- Anything synced to cloud

### Data Location

- **File:** `~/.claude/user-preferences.json`
- **Backup:** `~/.claude/user-preferences.json.backup`
- **Scope:** Local machine only
- **Sync:** Never synced to cloud

### How to Delete

```bash
# Delete preferences (fresh start)
rm ~/.claude/user-preferences.json

# Delete backup too
rm ~/.claude/user-preferences.json.backup
```

---

## Troubleshooting

### Issue: Preferences Not Persisting

**Symptoms:** Decisions don't seem to be remembered

**Causes:**
1. File permissions issue
2. Invalid JSON in preferences file
3. Skill not writing feedback

**Solutions:**
1. Check file permissions: `ls -la ~/.claude/user-preferences.json`
2. Validate JSON: `cat ~/.claude/user-preferences.json | python -m json.tool`
3. Check backup: `cat ~/.claude/user-preferences.json.backup`

### Issue: Wrong Suggestions Appearing

**Symptoms:** Seeing suggestions that should be hidden

**Causes:**
1. Confidence threshold too low
2. Preference not recorded properly
3. Proactivity level overriding

**Solutions:**
1. Check thresholds: "show my confidence thresholds"
2. Check specific preference: "show preference for [item]"
3. Check proactivity: "show my proactivity level"

### Issue: Preferences File Corrupted

**Symptoms:** JSON parse errors when reading preferences

**Solutions:**
1. Restore from backup:
   ```bash
   cp ~/.claude/user-preferences.json.backup ~/.claude/user-preferences.json
   ```
2. If backup also corrupted, delete and start fresh:
   ```bash
   rm ~/.claude/user-preferences.json
   ```

### Issue: Too Many/Few Suggestions

**Symptoms:** Overwhelmed by suggestions OR not seeing useful ones

**Solutions:**
1. Adjust proactivity level:
   - Too many: "set proactivity to low"
   - Too few: "set proactivity to high"
2. Adjust specific thresholds:
   - "set hide threshold to 40%" (see more)
   - "set hide threshold to 50%" (see fewer)

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| "Show my preferences" | Display full preference summary |
| "Show my thresholds" | Display confidence thresholds |
| "Show my skill usage" | Display usage analytics |
| "Set proactivity to [low/medium/high]" | Adjust proactivity |
| "Set [threshold] to [value]" | Adjust specific threshold |
| "Don't show [item] again" | Permanently hide item |
| "Reset [item] preference" | Clear learned preference |
| "Reset all preferences" | Full reset (with backup) |
| "Show project preferences" | Display project overrides |
| "Set project [setting] to [value]" | Create/update project override |
| "Show effective preferences" | Display merged view |
| "Delete project preferences" | Remove project overrides |

### Proactivity Levels

| Level | Description | Best For |
|-------|-------------|----------|
| Low | Minimal suggestions, only high-confidence | Experienced users, speed |
| Medium | Balanced approach | Most users |
| High | Maximum suggestions, all options shown | Learning, exploration |

### Confidence Thresholds

| Threshold | Default | Description |
|-----------|---------|-------------|
| autoApply | 95% | Auto-apply actions (careful!) |
| suggestProminently | 75% | Show prominently |
| showAsOptional | 50% | Show as optional |
| hideBelow | 30% | Hide suggestions |

---

## Operation 7: AI-Suggested Tuning (v3.10.0)

**User Queries:**
- "Suggest preference improvements"
- "Analyze my preferences"
- "Why am I seeing so many suggestions?"
- "Optimize my settings"
- "Tune my preferences"
- "Show tuning suggestions"

**Auto-trigger:** When `analysisIntervalDays` has passed since `lastAnalyzedAt` and `totalDecisionsTracked >= minimumSampleSize`

### The Tuning Problem

**Without AI-Suggested Tuning:**
- Users set preferences once and forget
- High rejection rate = wasted suggestions
- High acceptance rate = could be more proactive
- No feedback on whether settings are optimal
- Preferences drift from actual behavior

**With AI-Suggested Tuning:**
- System analyzes decision patterns
- Generates confidence-scored recommendations
- User applies with one command or dismisses
- Preferences evolve based on actual usage

### Analysis Algorithm

```
1. CHECK PREREQUISITES:
   - totalDecisionsTracked >= minimumSampleSize (default: 20)
   - Time since lastAnalyzedAt > analysisIntervalDays (default: 7)
   - If not met: Return "insufficient data" response

2. ANALYZE THRESHOLDS:
   For each confidence level (autoApply, suggestProminently, showAsOptional):

   a. Calculate acceptance rate for suggestions at that level
   b. If rejectionRate > 40%:
      → SUGGEST raising threshold
      → Rationale: "You rejected X% of suggestions at this level"
   c. If acceptanceRate > 90%:
      → SUGGEST lowering threshold (more automation)
      → Rationale: "You accept X% at this level - could automate more"

3. ANALYZE PROACTIVITY:
   a. If overallAcceptanceRate < 60%:
      → SUGGEST lowering proactivity
      → Rationale: "Overall acceptance low - reduce suggestion frequency"
   b. If overallAcceptanceRate > 85% AND skillUsageFrequency is high:
      → SUGGEST raising proactivity
      → Rationale: "High acceptance + active usage = can show more"

4. ANALYZE CATEGORY VARIANCE:
   For each category in categoryBreakdown:

   a. Calculate category-specific acceptance rate
   b. If variance > 25% from overall:
      → SUGGEST category-specific settings
      → Example: "coding-style: 92%, documentation: 41%"

5. ANALYZE SKILLS:
   For each skill in skillUsageFrequency:

   a. If skill acceptance < 50% AND sampleSize > 10:
      → SUGGEST disabling or threshold override
      → Rationale: "This skill's suggestions aren't matching your preferences"
   b. If skill acceptance > 95% AND sampleSize > 20:
      → SUGGEST skill can be more proactive
      → Rationale: "You accept almost all - could auto-apply"

6. SCORE AND RANK SUGGESTIONS:
   confidence = High (samples >= 50), Medium (>= 20), Low (< 20)
   impact = deviation from optimal rate
   priority = confidence × impact

   Sort by priority, return top 3

7. UPDATE tuningSuggestions:
   - Set lastAnalyzedAt to now
   - Store pendingSuggestions
   - Preserve suggestionHistory
```

### Confidence Scoring

| Sample Size | Confidence | Reliability |
|-------------|------------|-------------|
| ≥ 50 | High | Strong pattern, reliable suggestion |
| 20-49 | Medium | Emerging pattern, moderate confidence |
| < 20 | Low | Insufficient data, tentative suggestion |

### Response Template: Tuning Suggestions Found

```markdown
## Preference Tuning Suggestions

Based on analyzing **{totalDecisions}** decisions over **{learningPeriodDays}** days:

---

### Suggestion 1: Raise Auto-Apply Threshold (High Confidence)

**Current:** autoApply = 95%
**Suggested:** autoApply = 97%

**Why:** You rejected 43% of auto-applied actions (86 of 200).
Raising the threshold will reduce unwanted automatic changes.

**Data:**
| Metric | Value |
|--------|-------|
| Sample size | 200 decisions |
| Rejection rate | 43% |
| Threshold trigger | > 40% |
| Trend | Stable |

**Actions:**
- Apply: "Set autoApply to 97"
- Dismiss: "Dismiss suggestion 1"
- Snooze: "Snooze suggestion 1 for 2 weeks"

---

### Suggestion 2: Lower Proactivity for Documentation (Medium Confidence)

**Current:** proactivityLevel = medium (global)
**Suggested:** Set documentation category to low proactivity

**Why:** You accept 92% of coding suggestions but only 41% of documentation suggestions.
The variance suggests category-specific settings would help.

**Data:**
| Category | Acceptance | Sample Size |
|----------|------------|-------------|
| coding-style | 92% | 95 |
| documentation | 41% | 29 |
| Variance | 51% | - |

**Actions:**
- Apply: "Set documentation proactivity to low"
- Dismiss: "Dismiss suggestion 2"

---

### Suggestion 3: Consider Disabling test-generator (Low Confidence)

**Current:** test-generator enabled
**Suggested:** Disable or override thresholds

**Why:** You've rejected 67% of test-generator suggestions (8 of 12).
This skill may not match your testing workflow.

**Data:**
| Metric | Value |
|--------|-------|
| Sample size | 12 decisions |
| Rejection rate | 67% |
| Confidence | Low (needs more data) |

**Actions:**
- Apply: "Disable test-generator skill"
- Override: "Set test-generator threshold to 90%"
- Wait: "Keep collecting data"

---

### Summary

| Category | Acceptance | Trend | Suggested Action |
|----------|------------|-------|------------------|
| coding-style | 92% | improving | None needed |
| workflow | 78% | stable | None needed |
| quality | 85% | stable | None needed |
| documentation | 41% | declining | **Lower proactivity** |

**Next analysis:** {nextAnalysisDate} (in 7 days)

---

**Quick Actions:**
- "Apply suggestion 1" - Apply first suggestion
- "Apply all suggestions" - Apply all (use with caution)
- "Dismiss all" - Dismiss all suggestions
- "Show suggestion details" - More information
```

### Response Template: No Suggestions Needed

```markdown
## Preference Analysis Complete

**Status:** Your preferences are well-tuned!

**Analysis Period:** {learningPeriodDays} days
**Decisions Analyzed:** {totalDecisions}
**Last Analysis:** {lastAnalyzedAt}

---

### Current Performance

| Metric | Value | Status |
|--------|-------|--------|
| Overall acceptance | 82% | Good |
| Decisions tracked | 156 | Sufficient |
| Learning period | 21 days | Mature |

---

### Category Breakdown

| Category | Acceptance | Trend | Status |
|----------|------------|-------|--------|
| coding-style | 89% | stable | ✓ Well-tuned |
| workflow | 78% | improving | ✓ Well-tuned |
| quality | 85% | stable | ✓ Well-tuned |
| documentation | 76% | stable | ✓ Well-tuned |

---

### Analysis Summary

All metrics within optimal ranges:
- ✓ No thresholds triggering high rejection
- ✓ No categories with significant variance
- ✓ No underperforming skills detected

**Next analysis:** {nextAnalysisDate}

---

**Tip:** You can always manually adjust preferences:
- "Set autoApply to 98%"
- "Set proactivity to low"
- "Show my current thresholds"
```

### Response Template: Insufficient Data

```markdown
## Preference Analysis: Insufficient Data

**Status:** Not enough data for meaningful analysis

**Current Data:**
| Metric | Value | Required |
|--------|-------|----------|
| Decisions tracked | {totalDecisions} | ≥ 20 |
| Days since last analysis | {daysSinceAnalysis} | ≥ 7 |

---

### What's Needed

To generate tuning suggestions, we need:
1. **Minimum 20 decisions** - Accept, reject, or skip suggestions from skills
2. **At least 7 days** since last analysis

**Current Progress:**
```
Decisions: [{totalDecisions}/20] ████████░░░░░░░░░░░░ {percentage}%
```

---

### How to Build Data Faster

Use skills that track decisions:
- **security-scanner** - Accept/reject security findings
- **standards-enforcer** - Accept/reject style suggestions
- **test-generator** - Accept/reject test suggestions
- **commit-readiness-checker** - Accept/reject pre-commit checks

Each decision you make helps the system learn your preferences.

**Check again:** "Analyze my preferences" (after more usage)
```

### Applying Suggestions

**User says:** "Apply suggestion 1" or "Set autoApply to 97"

**Steps:**
1. Find pending suggestion by ID or parse direct command
2. Update relevant preference in `user-preferences.json`
3. Move suggestion from `pendingSuggestions` to `suggestionHistory`
4. Set status to "applied"

**Response:**

```markdown
✅ Applied Tuning Suggestion

**Change:** autoApply threshold
**Before:** 95%
**After:** 97%

**Rationale:** Reduces unwanted auto-applied actions based on your 43% rejection rate.

**Effect:** Actions now need ≥97% confidence to auto-apply.

**To revert:** "Set autoApply to 95%"

---

**Remaining suggestions:** 2
- "Show tuning suggestions" to see others
```

### Dismissing Suggestions

**User says:** "Dismiss suggestion 1" or "Dismiss all"

**Steps:**
1. Find suggestion(s) by ID
2. Move to `suggestionHistory` with status "dismissed"
3. Record `actionAt` timestamp

**Response:**

```markdown
✅ Dismissed Suggestion

**Suggestion:** Raise Auto-Apply Threshold
**Status:** Dismissed (won't be suggested again)

**Note:** If your rejection pattern continues, a similar suggestion may appear in future analyses with fresh data.

**Remaining suggestions:** 2
```

### Snoozing Suggestions

**User says:** "Snooze suggestion 1 for 2 weeks"

**Steps:**
1. Calculate `snoozeUntil` date
2. Set status to "snoozed"
3. Move to history with snooze timestamp

**Response:**

```markdown
✅ Snoozed Suggestion

**Suggestion:** Raise Auto-Apply Threshold
**Status:** Snoozed until {snoozeUntilDate}

I'll remind you about this suggestion after the snooze period if the pattern persists.

**Remaining suggestions:** 2
```

### Viewing Suggestion History

**User says:** "Show suggestion history" or "What suggestions have I applied?"

**Response:**

```markdown
## Tuning Suggestion History

### Applied

| Date | Suggestion | Change |
|------|------------|--------|
| 2025-12-10 | Raise autoApply | 95% → 97% |
| 2025-12-03 | Lower doc proactivity | medium → low |

### Dismissed

| Date | Suggestion | Reason |
|------|------------|--------|
| 2025-12-08 | Disable test-generator | Wanted to keep trying |

### Snoozed

| Suggestion | Snooze Until |
|------------|--------------|
| Category-specific thresholds | 2025-12-25 |

---

**Total:** 2 applied, 1 dismissed, 1 snoozed
```

### Configuration

Users can adjust tuning behavior in `~/.claude/user-preferences.json`:

```json
{
  "tuningSuggestions": {
    "analysisIntervalDays": 7,
    "minimumSampleSize": 20,
    "thresholds": {
      "triggerAnalysis": {
        "rejectionRateHigh": 0.4,
        "acceptanceRateLow": 0.6,
        "acceptanceRateHigh": 0.9
      }
    },
    "autoTuning": {
      "enabled": false,
      "requireConfirmation": true
    }
  }
}
```

**Configurable options:**

| Setting | Default | Description |
|---------|---------|-------------|
| analysisIntervalDays | 7 | Days between auto-analyses |
| minimumSampleSize | 20 | Min decisions before suggestions |
| rejectionRateHigh | 0.4 | Suggest raising threshold if rejection > this |
| acceptanceRateHigh | 0.9 | Suggest lowering threshold if acceptance > this |
| autoTuning.enabled | false | Auto-apply high-confidence suggestions |

---

## Operation 8: Cross-Project Intelligence (v3.12.0)

**User Queries:**
- "Analyze patterns across my projects"
- "What patterns do I use consistently?"
- "Are my projects configured consistently?"
- "Apply my preferences to this new project"
- "Show cross-project insights"
- "Propagate [setting] to all projects"
- "Show consistency report"

**Auto-trigger:** When `aggregationIntervalDays` has passed since `lastAggregatedAt` and registry has `minProjectsForAnalysis` projects

### The Cross-Project Problem

**Without Cross-Project Intelligence:**
- Learning is siloed per project
- User prefers "conventional commits" in 8/10 projects - new project doesn't know
- Quality standards vary unintentionally
- Workflow preferences rediscovered per project
- No "compound interest" on learning investment

**With Cross-Project Intelligence:**
- Patterns aggregate across all registered projects
- New projects inherit established preferences
- Divergences are detected and flagged
- One command standardizes across projects

### Analysis Algorithm

```
1. CHECK PREREQUISITES:
   - Projects Registry exists at ~/.claude/projects-registry.json
   - At least minProjectsForAnalysis (3) projects registered
   - Time since lastAggregatedAt > aggregationIntervalDays (7)
   - If not met: Return "insufficient data" response

2. LOAD DATA:
   a. Read ~/.claude/projects-registry.json (project list)
   b. Read ~/.claude/user-preferences.json (global learnings)
   c. For each project in registry:
      - Check if <project-path>/.claude/project-preferences.json exists
      - Load project-specific overrides

3. AGGREGATE PATTERNS:
   For each preference category (workflow, coding-style, quality, documentation):
   For each preference item:

   a. Count projects using this pattern (explicit override or inherited default)
   b. Calculate adoption rate: projects_using / total_projects
   c. If adoptionRate >= propagateConfidence (0.8):
      → Mark as "established pattern"
   d. Track which projects have different values

4. DETECT CONSISTENCY:
   For key settings (commit style, branch naming, quality thresholds):

   a. Identify global preference (most common OR explicitly set in global prefs)
   b. Find divergent projects (different value than global)
   c. Score severity:
      - Production project diverges = "action-recommended"
      - Staging project diverges = "warning"
      - Personal/experimental = "info"
   d. Check if divergence has documented reason (in project-preferences)

5. GENERATE SUGGESTIONS:
   Type A: PROPAGATE
   - Pattern used in 80%+ of projects
   - Some projects don't have it
   - Suggestion: "Apply X to remaining Y projects"

   Type B: STANDARDIZE
   - Multiple projects use different values for same setting
   - No clear majority or global preference differs
   - Suggestion: "Standardize X across projects"

   Type C: DETECT-OUTLIER
   - Project falls below established standard
   - E.g., test coverage 50% when standard is 80%
   - Suggestion: "Project Y below your standard for X"

6. SCORE AND RANK:
   confidence = based on sample size and adoption rate
   impact = severity × project_count
   priority = confidence × impact

   Return top 5 suggestions sorted by priority

7. UPDATE TRACKING:
   - Set lastAggregatedAt to now
   - Store detected patterns in projectPatterns
   - Record divergences in consistencyReport
   - Add new suggestions to crossProjectSuggestions.pending
```

### Confidence Scoring for Cross-Project

| Adoption Rate | Project Count | Confidence | Reliability |
|---------------|---------------|------------|-------------|
| ≥ 90% | 5+ projects | High | Strong established pattern |
| 80-89% | 3-4 projects | High | Established pattern |
| 70-79% | 3+ projects | Medium | Emerging pattern |
| < 70% | Any | Low | No clear pattern |

### Response Template: Cross-Project Intelligence Report

```markdown
## Cross-Project Intelligence Report

**Analysis Date:** {date}
**Projects Analyzed:** {count}
**Patterns Detected:** {patternCount}

---

### Established Patterns (80%+ Adoption)

| Pattern | Adoption | Projects | Category |
|---------|----------|----------|----------|
| Conventional commits | 90% (9/10) | All except legacy-app | workflow |
| TypeScript strict mode | 80% (8/10) | All except scripts, tools | coding-style |
| 80% test coverage | 80% (8/10) | All except prototype | quality |
| Auto-update changelog | 90% (9/10) | All except archived | documentation |

---

### Consistency Report

**Overall Consistency:** {percentage}%
**Divergences Found:** {count}

#### Divergence 1: Commit Style (Warning)

**Your standard:** conventional
**Divergent projects:**

| Project | Current Value | Reason |
|---------|---------------|--------|
| legacy-app | simple | Not documented |

**Impact:** Production project using non-standard commit style

**Suggestion:** "Standardize legacy-app to conventional commits"

---

#### Divergence 2: Test Coverage (Info)

**Your standard:** 80%
**Divergent project:**

| Project | Current Value | Reason |
|---------|---------------|--------|
| prototype | 50% | "Experimental - lower coverage acceptable" |

**Impact:** Low (experimental project with documented reason)

**Action:** None recommended (reason documented)

---

### Cross-Project Suggestions

#### Suggestion 1: Propagate TypeScript Strict Mode (High Confidence)

**Type:** Propagate established pattern
**Pattern:** TypeScript strict mode
**Adoption:** 80% (8/10 projects)

**Source projects:** api, web-app, mobile, dashboard, admin, lib, cli, backend
**Target projects:** scripts, tools

**Rationale:** You enable TypeScript strict mode in 80% of projects.
These 2 utility projects may benefit from the same type safety.

**Actions:**
- Apply: "Propagate strict mode to scripts and tools"
- Skip: "Skip this suggestion"
- More info: "Why strict mode?"

---

#### Suggestion 2: Standardize Branch Naming (Medium Confidence)

**Type:** Standardize inconsistent settings
**Pattern:** Branch naming convention
**Current state:** Mixed

| Convention | Projects | Count |
|------------|----------|-------|
| feature/ | api, web-app, mobile | 3 |
| feat/ | dashboard, admin | 2 |
| No prefix | scripts, tools, lib | 3 |

**Rationale:** No clear standard. Consider aligning to most common (feature/).

**Actions:**
- Apply feature/: "Standardize branch naming to feature/"
- Keep varied: "Dismiss this suggestion"

---

### Summary

| Metric | Value | Status |
|--------|-------|--------|
| Pattern consistency | 87% | Good |
| Established patterns | 12 | |
| Projects up-to-date | 8/10 | |
| Suggested actions | 3 | |
| Critical divergences | 0 | ✓ |
| Warnings | 1 | |
| Info items | 2 | |

**Next analysis:** {nextDate} (in 7 days)

---

**Quick Actions:**
- "Apply suggestion 1" - Propagate pattern
- "Apply all propagate suggestions" - Apply all propagate-type suggestions
- "Show divergence details" - More about inconsistencies
- "Ignore project [name] for cross-project" - Exclude from analysis
```

### Response Template: Insufficient Data

```markdown
## Cross-Project Intelligence

**Status:** Insufficient data for cross-project analysis

**Current State:**

| Requirement | Current | Required |
|-------------|---------|----------|
| Projects in registry | {count} | ≥ 3 |
| Projects with preferences | {prefCount} | ≥ 1 |
| Days since last analysis | {days} | ≥ 7 |

---

### How to Enable Cross-Project Analysis

1. **Register more projects:**
   ```
   "Register this project"
   ```
   Or use CLI: `./scripts/register-project.sh`

2. **Build preference history:**
   - Use skills that track decisions
   - Accept/reject suggestions to build patterns
   - Set explicit preferences

3. **Wait for data accumulation:**
   - Minimum 3 projects needed
   - Analysis runs every 7 days

**Check registry:** "Show my projects"
**Check preferences:** "Show my preferences"
```

### Applying Cross-Project Suggestions

**User says:** "Apply suggestion 1" or "Propagate strict mode to scripts and tools"

**Steps:**

1. **Identify target projects:**
   - Parse suggestion or command
   - Get project paths from registry

2. **For each target project:**
   a. Check if `.claude/project-preferences.json` exists
   b. If not, create from template
   c. Add/update the relevant override
   d. Set reason: "Applied from cross-project suggestion"

3. **Update tracking:**
   - Move suggestion to history with status "applied"
   - Record affected projects and timestamp

4. **Confirm to user**

**Response:**

```markdown
✅ Applied Cross-Project Suggestion

**Pattern:** TypeScript strict mode
**Applied to:**

| Project | Status |
|---------|--------|
| scripts | ✓ Created project-preferences.json |
| tools | ✓ Updated existing preferences |

**Changes made:**
```json
{
  "overrides": {
    "skillSpecificPreferences": {
      "standards-enforcer": {
        "typescriptStrictMode": true,
        "_reason": "Applied from cross-project suggestion (80% adoption)"
      }
    }
  }
}
```

**Remaining suggestions:** 2
```

### Propagate to All Projects

**User says:** "Propagate [setting] to all projects"

**Response:**

```markdown
## Propagate Setting to All Projects

**Setting:** Conventional commits
**Current adoption:** 9/10 projects (90%)

**This will update:**

| Project | Current | New |
|---------|---------|-----|
| legacy-app | simple | conventional |

**Confirm:** "Yes, propagate to all"
**Cancel:** "Cancel propagation"
```

### Excluding Projects from Analysis

**User says:** "Ignore [project] for cross-project analysis"

**Steps:**
1. Add project to exclusion list in user preferences
2. Remove from future pattern calculations

**Response:**

```markdown
✅ Excluded from Cross-Project Analysis

**Project:** prototype
**Reason:** User requested exclusion

This project will be:
- ❌ Excluded from pattern aggregation
- ❌ Excluded from consistency reports
- ❌ Not suggested for propagation

**To re-include:** "Include prototype in cross-project analysis"
```

### Configuration

Users can adjust cross-project behavior in `~/.claude/user-preferences.json`:

```json
{
  "crossProjectLearning": {
    "enabled": true,
    "aggregationIntervalDays": 7,
    "thresholds": {
      "minProjectsForPattern": 3,
      "divergenceAlertThreshold": 0.3,
      "propagateConfidence": 0.8,
      "minProjectsForAnalysis": 3
    }
  }
}
```

**Configurable options:**

| Setting | Default | Description |
|---------|---------|-------------|
| enabled | true | Enable/disable cross-project analysis |
| aggregationIntervalDays | 7 | Days between auto-analyses |
| minProjectsForPattern | 3 | Min projects to establish a pattern |
| divergenceAlertThreshold | 0.3 | Alert if >30% diverge |
| propagateConfidence | 0.8 | Min adoption to suggest propagation |
| minProjectsForAnalysis | 3 | Min projects before enabling analysis |

### Integration with Projects Registry

Cross-Project Intelligence uses the Projects Registry (v3.2.0) to:

1. **Enumerate projects:** Get list of all registered projects
2. **Resolve paths:** Find project directories for preference files
3. **Get metadata:** Use tags for severity scoring (production > staging > personal)
4. **Track status:** Consider project update status in recommendations

**Registry location:** `~/.claude/projects-registry.json`

### Integration with Project Preferences (v3.9.0)

When propagating patterns, Operation 8 uses the merge logic from v3.9.0:

1. **Read existing:** Load current project-preferences.json (if exists)
2. **Merge pattern:** Add new override using sparse format
3. **Document reason:** Include why override was added
4. **Preserve custom:** Don't overwrite user's explicit overrides

---

## Operation 9: Import/Export Preferences (v3.13.0)

### Overview

Export your carefully tuned preferences for backup, sharing, or transfer to new machines. Import preferences from exports with validation, preview, and merge strategies.

### User Queries

**Export:**
- "Export my preferences"
- "Export preferences to [path]"
- "Create a preferences backup"
- "Export preferences as template"
- "Export anonymized preferences"

**Import:**
- "Import preferences from [file]"
- "Import preferences"
- "Restore preferences from backup"
- "Restore from last backup"

**Management:**
- "Show export/import history"
- "Compare my preferences with [file]"
- "Show preference diff with [file]"

### Export Types

| Type | Contents | Use Case |
|------|----------|----------|
| **full** | All settings + analytics + history | Personal backup |
| **partial** | User-selected sections only | Share specific settings |
| **anonymized** | Settings without history/timestamps | Team sharing |
| **template** | Core settings only (no learned data) | Reusable starter config |

### Export Algorithm

```
1. DETERMINE EXPORT TYPE
   - Check user request for type specification
   - Default: "full" if not specified

2. GATHER CONTENTS
   - Read ~/.claude/user-preferences.json
   - Apply filter based on export type:
     - full: Include everything
     - partial: Prompt user for sections to include
     - anonymized: Strip analytics, learningHistory, timestamps
     - template: Keep only profile, thresholds, skillSpecificPreferences

3. BUILD EXPORT STRUCTURE
   {
     "$schema": "claude-preferences-export-v1",
     "exportVersion": "1.0.0",
     "exportedAt": "<ISO timestamp>",
     "sourceVersion": "<current version>",
     "exportType": "<type>",
     "compatibility": {
       "minVersion": "3.8.0",
       "maxVersion": "<current version>"
     },
     "contents": { <filtered preferences> },
     "metadata": {
       "checksum": "<sha256 of contents>",
       "exportOptions": { <options used> }
     }
   }

4. WRITE EXPORT FILE
   - Use user-specified path or default (~/.claude/claude-preferences-export.json)
   - Format with 2-space indentation for readability

5. UPDATE TRACKING
   - Add entry to importExport.exportHistory
   - Set importExport.lastExportAt

6. RETURN CONFIRMATION
   - Show export summary
   - Provide import instructions
```

### Import Algorithm

```
1. VALIDATE FILE
   - Check file exists at specified path
   - Parse JSON and validate structure
   - Check $schema matches "claude-preferences-export-v1"
   - Verify version compatibility (minVersion <= current <= maxVersion)

2. CHECK SAFETY LIMITS
   - If safetySettings.maxImportsPerDay exceeded, warn and require override
   - Count today's imports from importHistory

3. CREATE BACKUP
   - If safetySettings.createBackupBeforeImport is true
   - Copy current preferences to timestamped backup
   - Record backup path for potential rollback

4. DETERMINE MERGE STRATEGY
   - Ask user if not specified:
     - "overwrite": Replace all matching sections entirely
     - "merge": Deep merge (import values win on conflict)
     - "selective": User chooses per-section

5. PREVIEW CHANGES
   - Show diff of what will change
   - Highlight conflicts (current vs import values)
   - List new items that will be added
   - Show items that will be removed (overwrite mode)

6. REQUIRE CONFIRMATION
   - If safetySettings.requireConfirmation is true
   - Wait for explicit user approval

7. APPLY IMPORT
   - Execute merge based on strategy
   - Preserve importExport section (never overwrite tracking)
   - Preserve backup section

8. UPDATE TRACKING
   - Add entry to importExport.importHistory
   - Set importExport.lastImportAt
   - Record merge strategy and sections imported

9. RETURN CONFIRMATION
   - Show what was imported
   - Provide rollback command
   - Show backup location
```

### Response Templates

#### Export Success

```markdown
## Export Successful

**Export Type:** Full
**File:** ~/claude-preferences-export.json
**Exported At:** 2025-12-16 10:30:00
**Size:** 4.2 KB

### Contents Exported

| Section | Items | Status |
|---------|-------|--------|
| Profile | 5 settings | ✅ Included |
| Confidence Thresholds | 4 settings | ✅ Included |
| Learned Preferences | 12 items | ✅ Included |
| Skill Preferences | 5 skills | ✅ Included |
| Skipped Recommendations | 3 items | ✅ Included |
| Analytics | 156 decisions | ✅ Included |
| Learning History | 100 events | ❌ Excluded |
| Tuning Suggestions | 2 pending | ✅ Included |
| Cross-Project Patterns | 8 patterns | ✅ Included |

**Checksum:** sha256:a1b2c3...

---

### To Import on Another Machine

1. Copy file to new machine
2. Run: "Import preferences from ~/claude-preferences-export.json"

### Other Export Options

- **Template:** "Export preferences as template"
- **Anonymized:** "Export anonymized preferences"
- **Partial:** "Export only [section] preferences"
```

#### Export as Template

```markdown
## Template Export Successful

**Export Type:** Template
**File:** ~/claude-preferences-template.json

### Contents (Clean Template)

| Section | Items | Included |
|---------|-------|----------|
| Profile | 5 settings | ✅ |
| Confidence Thresholds | 4 settings | ✅ |
| Skill Preferences | 5 skills | ✅ |
| Analytics | - | ❌ (template mode) |
| Learning History | - | ❌ (template mode) |
| Learned Preferences | - | ❌ (template mode) |

**Perfect for:** New team members, new machines, sharing best practices

---

**To use template:**
"Import preferences from ~/claude-preferences-template.json"
```

#### Import Preview

```markdown
## Import Preview

**Source:** ~/claude-preferences-export.json
**Source Version:** 3.12.0 ✅ Compatible
**Merge Strategy:** merge

---

### Changes to Apply

| Setting | Current | Import | Action |
|---------|---------|--------|--------|
| profile.proactivityLevel | medium | high | 🔄 Update |
| confidenceThresholds.autoApply | 95 | 90 | 🔄 Update |
| learnedPreferences.workflow.commit-style | conventional | conventional | ✓ Same |
| skillSpecificPreferences.test-generator.coverageTarget | 80 | 90 | 🔄 Update |

### New Items (will be added)

- learnedPreferences.coding-style.early-returns (acceptanceRate: 0.85)
- skillSpecificPreferences.custom-skill (new configuration)

### Conflicts (import wins in merge mode)

| Setting | Your Value | Import Value |
|---------|------------|--------------|
| confidenceThresholds.autoApply | 95 | 90 |

---

**Backup will be created at:** ~/.claude/user-preferences.json.backup.2025-12-16

**Proceed?**
- "Yes, apply import" - Apply these changes
- "No, cancel" - Cancel import
- "Use selective merge" - Choose per section
- "Show full diff" - More details
```

#### Import Success

```markdown
## Import Successful

**Source:** ~/claude-preferences-export.json
**Strategy:** merge
**Imported At:** 2025-12-16 10:35:00

### Summary

| Category | Imported | Unchanged | Conflicts |
|----------|----------|-----------|-----------|
| Profile | 2 | 3 | 0 |
| Thresholds | 3 | 1 | 1 |
| Learned Preferences | 8 | 4 | 0 |
| Skill Preferences | 2 | 3 | 0 |
| **Total** | **15** | **11** | **1** |

---

**Backup available:** ~/.claude/user-preferences.json.backup.2025-12-16

**To rollback:** "Restore preferences from backup"
**To view changes:** "Show my preferences"
```

#### Import Validation Error

```markdown
## Import Failed: Validation Error

**File:** ~/old-preferences.json
**Error:** Version incompatible

### Details

| Check | Status |
|-------|--------|
| File exists | ✅ |
| Valid JSON | ✅ |
| Schema match | ✅ |
| Version compatible | ❌ |

**Problem:** Export version 2.5.0 is below minimum supported (3.8.0)

### Options

1. **Export fresh:** "Export my preferences" from the source machine
2. **Manual migration:** Copy specific settings by hand
3. **Force import:** "Import preferences from [file] --force" (may cause issues)
```

#### Restore from Backup

```markdown
## Restore from Backup

**Available Backups:**

| # | Date | Size | Reason |
|---|------|------|--------|
| 1 | 2025-12-16 10:30 | 4.1 KB | Pre-import backup |
| 2 | 2025-12-15 14:22 | 3.9 KB | Pre-import backup |
| 3 | 2025-12-14 09:15 | 3.8 KB | Manual backup |

**Select backup to restore:**
- "Restore backup 1" - Most recent
- "Restore backup from 2025-12-15"
- "Show backup 1 contents"
```

#### Backup Restored

```markdown
## Backup Restored

**Restored from:** ~/.claude/user-preferences.json.backup.2025-12-16
**Original date:** 2025-12-16 10:30:00

### What Changed

| Category | Restored Values |
|----------|-----------------|
| Thresholds | autoApply: 95 (was 90) |
| Profile | proactivityLevel: medium (was high) |

**Current backup created:** ~/.claude/user-preferences.json.backup.restore.2025-12-16

Your preferences are back to the state before the last import.
```

#### Show History

```markdown
## Import/Export History

### Recent Exports

| Date | Type | File | Sections |
|------|------|------|----------|
| 2025-12-16 10:30 | full | ~/claude-preferences-export.json | 9 sections |
| 2025-12-10 15:45 | template | ~/team-template.json | 3 sections |
| 2025-12-05 09:00 | anonymized | ~/shared-prefs.json | 6 sections |

### Recent Imports

| Date | Source | Strategy | Imported |
|------|--------|----------|----------|
| 2025-12-16 10:35 | export.json | merge | 15 settings |
| 2025-12-08 14:20 | team-prefs.json | selective | 8 settings |

**Today's imports:** 1 of 5 allowed

---

**Actions:**
- "Export my preferences" - Create new export
- "Clear export history" - Remove history entries
```

#### Compare/Diff

```markdown
## Preferences Comparison

**Current** vs **~/other-preferences.json**

### Differences Found: 8

#### Profile

| Setting | Current | Other |
|---------|---------|-------|
| proactivityLevel | medium | high |
| experienceLevel | intermediate | advanced |

#### Confidence Thresholds

| Setting | Current | Other |
|---------|---------|-------|
| autoApply | 95 | 90 |
| suggestProminently | 75 | 70 |

#### Learned Preferences

| Setting | Current | Other |
|---------|---------|-------|
| commit-style | conventional | semantic |

### Only in Current (3 items)
- skillSpecificPreferences.custom-skill
- learnedPreferences.coding-style.early-returns
- skippedRecommendations.item-xyz

### Only in Other (2 items)
- learnedPreferences.workflow.auto-stage
- skillSpecificPreferences.other-skill

---

**Actions:**
- "Import preferences from ~/other-preferences.json" - Apply these
- "Import only thresholds from ~/other-preferences.json" - Partial import
```

### Configuration

Users can configure import/export behavior in `~/.claude/user-preferences.json`:

```json
{
  "importExport": {
    "safetySettings": {
      "createBackupBeforeImport": true,
      "validateSchemaVersion": true,
      "requireConfirmation": true,
      "maxImportsPerDay": 5
    },
    "defaults": {
      "exportPath": "~/claude-preferences-export.json",
      "defaultExportType": "full",
      "includeAnalytics": true,
      "includeLearningHistory": false,
      "includeProjectPatterns": true
    }
  }
}
```

### Export File Format

```json
{
  "$schema": "claude-preferences-export-v1",
  "exportVersion": "1.0.0",
  "exportedAt": "2025-12-16T10:30:00Z",
  "exportedBy": "personalization-engine",
  "sourceVersion": "3.13.0",

  "exportType": "full",

  "compatibility": {
    "minVersion": "3.8.0",
    "maxVersion": "3.13.0",
    "breakingChanges": []
  },

  "contents": {
    "profile": { },
    "confidenceThresholds": { },
    "learnedPreferences": { },
    "skillSpecificPreferences": { },
    "skippedRecommendations": { },
    "dontShowAgain": { }
  },

  "optionalContents": {
    "analytics": { },
    "learningHistory": { },
    "tuningSuggestions": { },
    "crossProjectLearning": { }
  },

  "metadata": {
    "checksum": "sha256:...",
    "exportOptions": {
      "includeAnalytics": true,
      "includeLearningHistory": false,
      "includeProjectPatterns": true
    }
  }
}
```

### Best Practices

**For Backups:**
- Export weekly for regular backups
- Use "full" type to capture everything
- Store exports in cloud storage for safety

**For Team Sharing:**
- Use "anonymized" to share without personal data
- Use "template" for onboarding new team members
- Document which settings are included

**For Machine Migration:**
- Export "full" from old machine
- Import with "merge" on new machine
- Review diff before applying

---

## Operation 10: Preference Templates (v3.14.0)

### Overview

Apply pre-built preference configurations for common use cases. Templates provide curated starting points so new users can get optimized defaults immediately.

### User Queries

**List/Browse:**
- "Show available templates"
- "List preference templates"
- "What templates are available?"

**Apply:**
- "Apply [template-name] template"
- "Apply balanced template"
- "Apply security-first template"
- "Use the learning-mode template"

**Help/Compare:**
- "Help me choose a template"
- "Compare templates"
- "Compare balanced and speed-focused"
- "What's the difference between templates?"

### Available Templates

| Template | Description | Best For |
|----------|-------------|----------|
| **balanced** | Sensible defaults - good suggestions without overwhelm | Most developers |
| **security-first** | Conservative automation, thorough security scanning | Security-conscious |
| **speed-focused** | Maximum automation, fewer interruptions | Experienced devs |
| **learning-mode** | Maximum guidance and suggestions | Beginners |
| **minimal** | Minimal interruptions, maximum autonomy | Experts |

### Template Locations

Templates are stored in: `templates/preference-templates/`

- `balanced.json` - Default recommended template
- `security-first.json` - Security-focused configuration
- `speed-focused.json` - Productivity-optimized configuration
- `learning-mode.json` - Beginner-friendly configuration
- `minimal.json` - Expert/minimal configuration

### Algorithm

```
1. LIST TEMPLATES
   - Read template files from templates/preference-templates/
   - Extract templateMetadata from each
   - Show name, description, recommendedFor
   - Highlight recommended template based on user's experienceLevel

2. RECOMMEND TEMPLATE
   If user asks "help me choose":
   - Read current profile.experienceLevel (if exists)
   - Match to template.recommendedFor
   - Ask clarifying question if needed:
     - "Do you prioritize speed or security?"
     - "How much guidance do you want?"
   - Recommend matching template with explanation

3. APPLY TEMPLATE
   - Load selected template file
   - Use Operation 9 import mechanism
   - Default merge strategy: "merge" (preserves learned preferences)
   - Show preview of changes
   - Create backup before applying
   - Update template tracking in importExport.templates

4. COMPARE TEMPLATES
   - Load both template files
   - Show side-by-side comparison table
   - Highlight key differences
   - Explain trade-offs in plain language
```

### Response Templates

#### List Templates

```markdown
## Available Preference Templates

| Template | Description | Best For |
|----------|-------------|----------|
| **balanced** | Sensible defaults | Most developers |
| **security-first** | Conservative automation | Security-conscious |
| **speed-focused** | Maximum automation | Experienced devs |
| **learning-mode** | Maximum guidance | Beginners |
| **minimal** | Minimal interruptions | Experts |

---

**Your profile:** intermediate
**Recommended:** balanced

---

**Actions:**
- "Apply balanced template"
- "Compare balanced and speed-focused"
- "Help me choose a template"
```

#### Help Choose Template

```markdown
## Help Me Choose a Template

Based on your profile and preferences, I have a few questions:

### 1. Experience Level

What's your experience with Claude Code?
- **Beginner** → learning-mode
- **Intermediate** → balanced
- **Advanced/Expert** → speed-focused or minimal

### 2. Priority

What matters more to you?
- **Safety & thoroughness** → security-first
- **Speed & minimal interruptions** → speed-focused
- **Balance of both** → balanced

### 3. Learning

Do you want to learn from suggestions?
- **Yes, show me everything** → learning-mode
- **Some guidance is fine** → balanced
- **I know what I'm doing** → minimal

---

**Based on typical intermediate developers:**

**Recommended: balanced**

Sensible defaults that provide helpful suggestions without being overwhelming.

**Apply:** "Apply balanced template"
```

#### Apply Template Preview

```markdown
## Apply Template: Balanced Developer

**Template:** balanced
**Description:** Sensible defaults - good suggestions without overwhelm
**Version:** 1.0.0

### Changes to Apply

| Setting | Current | Template | Action |
|---------|---------|----------|--------|
| experienceLevel | - | intermediate | Set |
| proactivityLevel | - | medium | Set |
| autoApply | - | 95 | Set |
| suggestProminently | - | 75 | Set |
| showAsOptional | - | 50 | Set |
| hideBelow | - | 30 | Set |

### Skill Preferences

| Skill | Setting | Value |
|-------|---------|-------|
| security-scanner | showInfoLevel | false |
| test-generator | coverageReminders | true |
| commit-readiness | conventionalCommits | true |

**Merge strategy:** merge
- Template values applied to matching settings
- Your existing learned preferences preserved
- Analytics and history unchanged

---

**Backup will be created at:** ~/.claude/user-preferences.json.backup

**Proceed?**
- "Yes, apply template"
- "No, cancel"
- "Show other templates"
```

#### Template Applied

```markdown
## Template Applied Successfully

**Template:** balanced
**Applied At:** 2025-12-16 11:00:00

### Summary

| Category | Settings Applied |
|----------|------------------|
| Profile | 3 settings |
| Thresholds | 4 settings |
| Skill Preferences | 3 skills |
| **Total** | **10 settings** |

**Backup created:** ~/.claude/user-preferences.json.backup

---

**Your preferences are now configured!**

The balanced template provides:
- Medium proactivity (helpful but not overwhelming)
- Conservative auto-apply (95% confidence required)
- Changelog and conventional commits enabled
- Security scanning at high level only

**To customize further:** "Show my preferences"
**To try different template:** "Apply speed-focused template"
**To restore backup:** "Restore from backup"
```

#### Compare Templates

```markdown
## Template Comparison

**Comparing:** balanced vs speed-focused

### Profile Settings

| Setting | balanced | speed-focused |
|---------|----------|---------------|
| experienceLevel | intermediate | advanced |
| proactivityLevel | medium | high |
| preferredWorkflow | wizard | automated |

### Confidence Thresholds

| Threshold | balanced | speed-focused |
|-----------|----------|---------------|
| autoApply | 95 | 90 |
| suggestProminently | 75 | 65 |
| showAsOptional | 50 | 40 |
| hideBelow | 30 | 25 |

### Key Differences

**balanced:**
- More conservative automation (95% vs 90% for auto-apply)
- More suggestions shown (lower hideBelow threshold)
- Wizard-guided workflows
- Good for: Most developers, learning Claude Code

**speed-focused:**
- More aggressive automation (90% auto-apply)
- Fewer interruptions (higher hideBelow threshold)
- Automated workflows preferred
- Good for: Experienced devs who want speed

---

**Choose:**
- "Apply balanced template"
- "Apply speed-focused template"
- "Compare other templates"
```

### Integration with Operation 9

Templates are applied using the Operation 9 import mechanism:

1. Template files use the same `claude-preferences-export-v1` schema
2. `exportType: "template"` indicates it's a template
3. Apply uses the same preview/backup/merge flow as regular imports
4. Template tracking recorded in `importExport.templates`

### Template Tracking

Applied templates are tracked in user preferences:

```json
{
  "importExport": {
    "templates": {
      "appliedTemplate": "balanced",
      "appliedAt": "2025-12-16T11:00:00Z",
      "templateHistory": [
        {
          "templateId": "balanced",
          "appliedAt": "2025-12-16T11:00:00Z",
          "mergeStrategy": "merge"
        }
      ]
    }
  }
}
```

### Creating Custom Templates

Users can create their own templates:

1. Configure preferences as desired
2. Export as template: "Export preferences as template"
3. Share the exported file
4. Others can import: "Import preferences from [file]"

Custom templates use the same format as built-in templates.

---

## Template Inheritance (v4.1.0)

### Overview

Templates can extend other templates using the `extends` field. Child templates inherit all settings from the base and can override specific values.

### The `extends` Field

Add `extends` to `templateMetadata` to inherit from a base template:

```json
{
  "templateMetadata": {
    "id": "team-frontend",
    "name": "Team Frontend",
    "extends": "team-standard",
    "_extendsHint": "Base template ID or URL to inherit from"
  }
}
```

### Inheritance Sources

| Source Type | Example | Resolution |
|-------------|---------|------------|
| Built-in ID | `"extends": "balanced"` | Lookup in built-in templates |
| Remote URL | `"extends": "https://..."` | Fetch via Operation 11 |
| Relative Path | `"extends": "./base.json"` | Resolve relative to template location |

### Inheritance Resolution Algorithm

```
RESOLVE_INHERITANCE(template, chain = []):
  1. CHECK FOR EXTENDS
     - If template.templateMetadata.extends is null/undefined
       → Return template unchanged

  2. CYCLE DETECTION
     - If template.templateMetadata.id in chain
       → ERROR: "Circular inheritance detected: {chain} → {id}"
     - If chain.length >= 5
       → ERROR: "Maximum inheritance depth exceeded (5 levels)"

  3. RESOLVE BASE TEMPLATE
     - extendsValue = template.templateMetadata.extends

     a. If extendsValue is built-in ID (balanced, security-first, etc.)
        → baseTemplate = load from templates/preference-templates/{id}.json

     b. If extendsValue is HTTPS URL
        → baseTemplate = fetch via Operation 11 mechanism
        → Verify checksum if available

     c. If extendsValue is relative path (starts with ./)
        → baseTemplate = load relative to template's location

     - Validate baseTemplate has claude-preferences-export-v1 schema

  4. RECURSIVE RESOLUTION
     - chain.push(template.templateMetadata.id)
     - resolvedBase = RESOLVE_INHERITANCE(baseTemplate, chain)

  5. DEEP MERGE
     - result = DEEP_MERGE(resolvedBase.contents, template.contents)

     DEEP_MERGE(base, child):
       For each key in child:
         - If both base[key] and child[key] are objects (not arrays)
           → result[key] = DEEP_MERGE(base[key], child[key])
         - Else (arrays, primitives, or type mismatch)
           → result[key] = child[key]  // Child wins
       For each key in base not in child:
         → result[key] = base[key]  // Inherit from base

  6. PRESERVE CHILD METADATA
     - result.templateMetadata = template.templateMetadata
     - result._inheritanceChain = chain
     - result._baseTemplate = resolvedBase.templateMetadata.id

  7. RETURN RESOLVED TEMPLATE
```

### Inheritance Triggers

- "Show template inheritance chain for [template]"
- "What does [template] inherit from?"
- "Preview resolved [template]"
- "Show [template] with inheritance resolved"

### Inheritance Chain Response

```markdown
## Template Inheritance Chain: team-frontend

```
team-frontend (applying)
    └── extends: team-standard
        └── extends: balanced (built-in)
```

### Resolved Template Preview

| Setting | Value | Source |
|---------|-------|--------|
| experienceLevel | intermediate | team-standard |
| proactivityLevel | high | team-frontend (override) |
| autoApply | 95 | team-standard |
| primaryFrameworks | ["react", "nextjs"] | team-frontend (override) |

---

**Inheritance Summary:**
- Base layers: 2 (balanced → team-standard)
- Overrides in team-frontend: 4 settings
- Total effective settings: 23

**Actions:**
- "Apply team-frontend template"
- "Show full resolved template"
- "Compare with current preferences"
```

### Error Handling

**Circular Inheritance:**
```markdown
## Error: Circular Inheritance Detected

Cannot resolve template `team-a`:

```
team-a → team-b → team-c → team-a (circular!)
```

**Fix:** Remove the circular reference in one of the templates.
```

**Max Depth Exceeded:**
```markdown
## Error: Maximum Inheritance Depth Exceeded

Template `deep-child` exceeds maximum inheritance depth (5 levels):

```
deep-child → level-4 → level-3 → level-2 → level-1 → base (6 levels!)
```

**Fix:** Flatten your template hierarchy to reduce nesting.
```

**Base Template Not Found:**
```markdown
## Error: Base Template Not Found

Template `team-frontend` extends `team-standard`, but `team-standard` was not found.

**Checked locations:**
- Built-in templates: Not found
- Remote sources: Not found
- Relative path: ./team-standard.json - File not found

**Fix:** Ensure the base template exists and is accessible.
```

### Integration with Operations 9, 10, 11

| Operation | Inheritance Integration |
|-----------|------------------------|
| Operation 9 (Import) | Resolve inheritance before preview |
| Operation 10 (Templates) | Show inheritance chain in template listing |
| Operation 11 (Remote) | Fetch base templates for remote extends |

When applying a template with inheritance:
1. Resolve full inheritance chain
2. Show resolved preview (merged result)
3. User confirms
4. Apply resolved template using Operation 9 import

---

## Template Parameters (v4.2.0)

### Overview

Templates can include parameters that are resolved when applied. Parameters enable reusable templates that adapt to specific values like company name, team, or configuration settings.

### The `parameters` Field

Add `parameters` to `templateMetadata` to declare template parameters:

```json
{
  "templateMetadata": {
    "id": "team-parameterized",
    "name": "Team Parameterized",
    "parameters": {
      "company": {
        "type": "string",
        "required": true,
        "description": "Your company name"
      },
      "team": {
        "type": "string",
        "required": true,
        "description": "Your team name"
      },
      "coverageTarget": {
        "type": "number",
        "default": 80,
        "description": "Test coverage target percentage"
      },
      "strictMode": {
        "type": "boolean",
        "default": false,
        "description": "Enable strict quality checks"
      }
    }
  }
}
```

### Parameter Schema

| Field | Type | Description |
|-------|------|-------------|
| `type` | string | `"string"`, `"number"`, or `"boolean"` |
| `required` | boolean | If true, user must provide value |
| `default` | any | Default value if not provided (required if not required) |
| `description` | string | Human-readable description for prompts |

### Variable Syntax

Use `${varName}` syntax in template contents:

```json
{
  "contents": {
    "learnedPreferences": {
      "workflow": {
        "preferred-commit-style": "feat(${company}/${team}): "
      },
      "quality": {
        "test-coverage-target": "${coverageTarget}"
      }
    }
  }
}
```

| Syntax | Description |
|--------|-------------|
| `${varName}` | Simple variable reference |
| `${varName:defaultValue}` | Variable with inline default |

### Parameter Resolution Algorithm

```
RESOLVE_PARAMETERS(template, providedValues = {}):
  1. EXTRACT PARAMETERS
     - parameters = template.templateMetadata.parameters || {}
     - If no parameters → return template unchanged

  2. VALIDATE REQUIRED PARAMETERS
     - For each param where required = true:
       - If param not in providedValues
         → Prompt user: "Enter value for {param}: {description}"
       - Store response in providedValues

  3. APPLY DEFAULTS
     - For each param with default value:
       - If param not in providedValues
         → providedValues[param] = default

  4. TYPE COERCION
     - For each param:
       - If type = "number" → parseFloat(value)
       - If type = "boolean" → value === "true" || value === true
       - If type = "string" → String(value)

  5. SUBSTITUTE IN CONTENTS
     - contentsJson = JSON.stringify(template.contents)
     - For each ${varName} or ${varName:default} in contentsJson:
       - If varName in providedValues
         → Replace with providedValues[varName]
       - Else if inline default exists
         → Replace with inline default
       - Else
         → ERROR: "Unresolved parameter: ${varName}"
     - template.contents = JSON.parse(contentsJson)

  6. RECORD RESOLUTION
     - template._resolvedParameters = providedValues
     - template._parameterSource = { paramName: "user" | "default" | "inline" }

  7. RETURN RESOLVED TEMPLATE
```

### Parameter Triggers

- "Apply [template] template" (with parameters)
- "Show template parameters for [template]"
- "Set parameter [name] to [value]"
- "Apply template with company=[value] team=[value]"

### Parameter Prompt Response

```markdown
## Applying Template: team-parameterized

This template has the following parameters:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| company | string | Yes | - | Your company name |
| team | string | Yes | - | Your team name |
| coverageTarget | number | No | 80 | Test coverage % |
| strictMode | boolean | No | false | Strict quality checks |

**Required parameters:** Please provide values for `company` and `team`.

---

**Actions:**
- "Apply with company=Acme team=Frontend"
- "Show parameter descriptions"
- "Cancel"
```

### Parameter Resolution Response

```markdown
## Template Parameters Resolved: team-parameterized

| Parameter | Value | Source |
|-----------|-------|--------|
| company | "Acme Corp" | user-provided |
| team | "Frontend" | user-provided |
| coverageTarget | 80 | default |
| strictMode | false | default |

### Preview (values substituted)

```json
{
  "learnedPreferences": {
    "workflow": {
      "preferred-commit-style": "feat(Acme Corp/Frontend): "
    },
    "quality": {
      "test-coverage-target": 80
    }
  }
}
```

---

**Actions:**
- "Apply resolved template"
- "Change parameter company to [value]"
- "Cancel"
```

### Integration with Inheritance

**Order of Operations:**
1. Resolve inheritance chain (v4.1.0)
2. Merge parameters from all levels (child overrides parent)
3. Prompt for required parameters
4. Apply defaults
5. Substitute in final merged content

**Parameter Inheritance:**
```
Base: { company: required, team: optional(default: "General") }
Child: { team: required, project: optional }
Merged: { company: required, team: required, project: optional }
```

**Rules:**
- Child can make optional parameter required
- Child can change default value
- Child can add new parameters
- Child cannot make required parameter optional

### Error Handling

**Missing Required Parameter:**
```markdown
## Error: Missing Required Parameter

Template `team-parameterized` requires parameter `company` but no value was provided.

**Fix:** Provide the parameter value:
- "Apply template with company=YourCompany"
```

**Invalid Parameter Type:**
```markdown
## Error: Invalid Parameter Type

Parameter `coverageTarget` expects type `number` but received `"high"`.

**Expected:** A number (e.g., 80)
**Received:** "high"

**Fix:** Provide a numeric value:
- "Set parameter coverageTarget to 90"
```

**Unresolved Parameter:**
```markdown
## Error: Unresolved Parameter

Template contains `${unknownParam}` but no parameter `unknownParam` is declared.

**Fix:** Either:
1. Declare the parameter in templateMetadata.parameters
2. Remove the `${unknownParam}` reference from contents
```

---

## Operation 11: Remote Template Sources (v4.0.0)

### Purpose

Enable teams to share preference templates via git repositories or URLs without authentication complexity.

### Core Concept: Git-Native Template Sharing

Instead of building authentication, sync, and conflict resolution:
- Templates are JSON files (already portable)
- Share via git repos or raw URLs
- Teams curate template catalogs
- No server infrastructure needed

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Remote Source** | URL pointing to a template catalog JSON file |
| **Template Catalog** | JSON file listing available templates with metadata |
| **Team Repository** | Git repo containing team's curated templates |

### Triggers

**Add Source:**
- "Add template source [URL]"
- "Add team templates from [URL]"
- "Configure template source"

**Browse:**
- "Browse template catalog"
- "List remote templates"
- "Show available remote templates"
- "What templates are available from [source]?"

**Fetch/Import:**
- "Import template from [URL]"
- "Fetch [template-name] template"
- "Download [template-name] from [source]"

**Manage:**
- "List template sources"
- "Remove template source [name]"
- "Refresh template sources"
- "Check for template updates"

### Algorithm

```
OPERATION 11: REMOTE TEMPLATE SOURCES

1. ADD TEMPLATE SOURCE
   Input: catalogUrl (HTTPS URL to catalog.json)

   Steps:
   a. Validate URL format
      - Must be HTTPS (unless allowHttpSources = true)
      - Must end in .json or be a valid URL

   b. Fetch catalog JSON
      - HTTP GET request to catalogUrl
      - Parse JSON response

   c. Validate catalog schema
      - Must have $schema: "claude-template-catalog-v1"
      - Must have templates array
      - Check compatibility.minVersion

   d. Store source in user preferences
      - Add to remoteSources.sources array
      - Generate unique source ID
      - Record addedAt timestamp

   e. Cache template metadata
      - Store template list in remoteSources.cache
      - Record lastRefreshed timestamp

   Output: Success message with template count

2. BROWSE CATALOG
   Input: sourceId (optional, browse all if not specified)

   Steps:
   a. Read configured remote sources
   b. For each source:
      - Load cached template metadata
      - Check if cache is stale (> refreshIntervalDays)
      - Refresh if needed
   c. Display templates with:
      - Name and description
      - Category and recommended level
      - Version and status (available/installed)
      - Source name
   d. Show compatibility warnings

   Output: Template listing table with actions

3. FETCH REMOTE TEMPLATE
   Input: templateId, sourceId

   Steps:
   a. Look up template in source catalog
   b. Download template JSON from sourceUrl
   c. Verify checksum (if provided)
      - Calculate SHA256 of downloaded content
      - Compare with catalog checksum
      - Warn if mismatch
   d. Validate template schema
      - Must be claude-preferences-export-v1
      - Check compatibility.minVersion/maxVersion
   e. Store in ~/.claude/remote-templates/
   f. Update local cache

   Output: Template ready for import

4. IMPORT REMOTE TEMPLATE
   Input: templateId (fetched template)

   Steps:
   a. Load fetched template from cache
   b. Use Operation 9 import mechanism:
      - Show preview diff
      - Create backup
      - Apply with merge strategy
   c. Track source URL in import history
   d. Update template tracking

   Output: Template applied with confirmation

5. REFRESH SOURCES
   Input: sourceId (optional, refresh all if not specified)

   Steps:
   a. For each source:
      - Re-fetch catalog JSON
      - Compare with cached version
      - Identify new/updated templates
   b. Update cache with new metadata
   c. Check for version updates on installed templates
   d. Report changes

   Output: Update summary
```

### Template Catalog Schema

```json
{
  "$schema": "claude-template-catalog-v1",
  "version": "1.0.0",
  "name": "Catalog Display Name",
  "description": "What's in this catalog",
  "maintainer": "team-or-person",
  "lastUpdated": "ISO-8601 date",

  "compatibility": {
    "minVersion": "4.0.0"
  },

  "templates": [
    {
      "id": "unique-template-id",
      "name": "Human Readable Name",
      "description": "What this template does",
      "category": "general|security|productivity|educational|team",
      "recommendedFor": ["beginner", "intermediate", "advanced", "expert"],
      "sourceUrl": "https://...template.json",
      "version": "1.0.0",
      "checksum": "sha256:...",
      "author": "who-made-it",
      "tags": ["searchable", "keywords"]
    }
  ]
}
```

### Response Templates

#### List Remote Sources
```markdown
## Configured Template Sources

| Source | Templates | Last Updated | Status |
|--------|-----------|--------------|--------|
| **Official** (built-in) | 5 | bundled | Active |
| **team-templates** | 3 | 2025-12-15 | Active |
| **community** | 12 | 2025-12-10 | Active |

**Total:** 20 templates available from 3 sources

---

**Actions:**
- "Add template source [URL]"
- "Browse template catalog"
- "Refresh template sources"
- "Remove template source [name]"
```

#### Browse Catalog
```markdown
## Available Templates

### From: team-templates (github.com/team/templates)

| Template | Category | Version | Status |
|----------|----------|---------|--------|
| **team-standard** | team | 1.2.0 | Available |
| **team-security** | security | 1.0.0 | Available |
| **frontend-react** | productivity | 1.1.0 | Installed |

### From: Official (built-in)

| Template | Category | Version | Status |
|----------|----------|---------|--------|
| **balanced** | general | 1.0.0 | Available |
| **security-first** | security | 1.0.0 | Installed |

---

**Actions:**
- "Import team-standard template"
- "Compare team-standard with current"
- "View template details team-standard"
```

#### Add Source Success
```markdown
## Template Source Added

**Source:** team-templates
**URL:** https://raw.githubusercontent.com/team/templates/main/catalog.json
**Templates found:** 3

| Template | Category |
|----------|----------|
| team-standard | team |
| team-security | security |
| frontend-react | productivity |

---

**Next steps:**
- "Browse template catalog" to see all templates
- "Import team-standard template" to apply one
```

#### Fetch Template
```markdown
## Template Fetched: team-standard

**Source:** team-templates
**Version:** 1.2.0
**Checksum:** Verified

**Ready to import.** This template will configure:
- Confidence thresholds (balanced automation)
- Commit style (conventional commits)
- Security scanning (high threshold)

---

**Actions:**
- "Import team-standard template" (preview before applying)
- "Compare with current preferences"
- "Cancel"
```

### Security Considerations

**URL Validation:**
- Only HTTPS sources by default
- `allowHttpSources: false` in settings
- Warn for non-HTTPS sources

**Checksum Verification:**
- SHA256 hash comparison
- Warn if checksum missing
- Fail if checksum mismatch

**Preview Before Import:**
- Show diff of what will change
- Require confirmation
- Create backup automatically

**Audit Trail:**
- Track source URL in import history
- Record when templates were fetched
- Log source additions/removals

### User Preferences Schema

```json
{
  "remoteSources": {
    "enabled": true,
    "sources": [
      {
        "id": "source-uuid",
        "name": "team-templates",
        "catalogUrl": "https://...",
        "addedAt": "ISO date",
        "lastFetched": "ISO date",
        "templateCount": 3,
        "trusted": false
      }
    ],
    "cache": {
      "lastRefreshed": "ISO date",
      "templates": [ /* cached metadata */ ]
    },
    "settings": {
      "autoRefresh": true,
      "refreshIntervalDays": 7,
      "verifyChecksums": true,
      "allowHttpSources": false
    }
  }
}
```

### Example Workflow

**Team Lead Setup:**
```
1. Create git repo with team templates
2. Add catalog.json listing templates
3. Share URL with team
```

**Team Member Usage:**
```
User: "Add template source https://github.com/team/templates/catalog.json"
Skill: Fetches catalog, validates, caches templates

User: "Browse template catalog"
Skill: Shows all available templates with status

User: "Import team-standard template"
Skill: Shows preview, creates backup, applies template
```

### Integration with Other Operations

| Operation | Integration |
|-----------|-------------|
| Operation 9 (Import/Export) | Remote templates use same import mechanism |
| Operation 10 (Templates) | Remote templates appear in template listing |
| Operation 8 (Tuning) | Can suggest remote templates based on usage |

### File Locations

| Type | Location |
|------|----------|
| Remote template cache | `~/.claude/remote-templates/` |
| Source configuration | `~/.claude/user-preferences.json` (remoteSources) |
| Downloaded templates | `~/.claude/remote-templates/{source-id}/{template-id}.json` |

---

## Version History

- **v4.2.0** (2025-12-16): Template Parameters
  - `parameters` field for declaring template variables
  - `${varName}` syntax for variable substitution
  - Parameter types: string, number, boolean
  - Required vs optional with defaults
  - Parameter resolution algorithm
  - Integration with inheritance (merged parameters)
  - Type coercion and validation
  - Error handling for missing/invalid parameters

- **v4.1.0** (2025-12-16): Template Inheritance
  - `extends` field for template composition
  - Inheritance resolution algorithm with cycle detection
  - Support for built-in, remote URL, and relative path bases
  - Deep merge with child override semantics
  - Maximum depth enforcement (5 levels)
  - Inheritance chain visualization
  - Integration with Operations 9, 10, 11

- **v4.0.0** (2025-12-16): Remote Template Sources
  - Operation 11: Team template sharing via URLs
  - Template catalog schema (claude-template-catalog-v1)
  - Add, browse, fetch, import remote templates
  - Checksum verification for security
  - Git-native sharing without authentication

- **v3.14.0** (2025-12-16): Preference Templates
  - Operation 10: Pre-built preference configurations
  - Five curated templates: balanced, security-first, speed-focused, learning-mode, minimal
  - Template listing, comparison, and recommendation
  - Integration with Operation 9 import mechanism
  - Template tracking in user preferences

- **v3.13.0** (2025-12-16): Import/Export Preferences
  - Operation 9: Portable configuration management
  - Four export types: full, partial, anonymized, template
  - Three merge strategies: overwrite, merge, selective
  - Version compatibility checking
  - Preview before import with diff view
  - Automatic backup before import
  - Export/import history tracking
  - Rollback capability from backups

- **v3.12.0** (2025-12-16): Cross-Project Intelligence
  - Operation 8: Cross-project pattern aggregation
  - Consistency detection across projects
  - Propagate/standardize/detect-outlier suggestions
  - Integration with Projects Registry
  - Learning leverage across multiple projects

- **v3.10.0** (2025-12-16): AI-Suggested Tuning
  - Operation 7: Intelligent preference tuning
  - Confidence-scored recommendations
  - Apply/dismiss/snooze workflow
  - Tuning suggestions tracking in preferences
  - Analysis algorithm for thresholds, proactivity, skills

- **v3.9.0** (2025-12-15): Project-level preferences
  - Operation 6: Project preferences management
  - Merge logic (project overrides global)
  - project-preferences.json.template
  - Team sharing support (committable to git)
  - Effective preferences view (merged)

- **v3.8.0** (2025-12-15): Initial implementation
  - Core operations: Read, Update, Thresholds, Analytics
  - Integration protocol for skills
  - user-preferences.json schema
  - Privacy-first local storage
