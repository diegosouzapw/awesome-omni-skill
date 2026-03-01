---
name: date-validation
description: Use when editing Planning Hubs, timelines, calendars, or any file with day-name + date combinations (Wed Nov 12), relative dates (tomorrow), or countdowns (18 days until) - validates day-of-week accuracy, relative date calculations, and countdown math with two-source ground truth verification before allowing edits
metadata:
  created: 2025-11-12
  version: 1.0.0
  author: Jack Reis
---

# Date Validation

## Overview

Prevent day-of-week errors, countdown mistakes, and timeline inconsistencies before they reach files.

**Core principle:** Trust but verify. Establish independent ground truth, validate every date systematically, block edits until all pass.

**CRITICAL:** This skill exists because baseline testing showed agents:
- Trust anchor dates without verification (propagates errors)
- Spot-fix obvious errors, miss related ones (incomplete fixes)
- Trust sequential patterns without individual validation (consistent ≠ correct)
- Only verify when suspicious (errors slip through)

## When to Use

### Auto-Triggers (Mandatory)

This skill MUST activate before Edit/Write when:

**File patterns:**
- `*Planning-Hub.md`
- `*Timeline*.md`
- `*Checklist*.md`
- `calendar/day/*.md`
- `calendar/week/*.md`
- Filename contains: "schedule", "deadline", "milestone"

**Content patterns:**
- Day-name + date: `Wed Nov 12`, `Monday, November 11`
- Relative dates: `today`, `tomorrow`, `yesterday`, `in X days`
- Countdowns: `X days until`, `X days remaining`, `X days from today`

### Manual Use

Invoke when:
- User says "check the dates" or "verify timeline"
- You notice day-name + date combinations while reading
- After making date-related errors (systematic revalidation)

---

## Ground Truth Protocol

**CRITICAL: Verify "today" using TWO independent sources. If they disagree, BLOCK ALL EDITS.**

### Step 1: Get Both Sources

**Source A - Environment Tag:**
```
Parse: <env>Today's date: YYYY-MM-DD</env>
Example: "2025-11-12"
```

**Source B - System Command:**
```bash
date +"%Y-%m-%d %A"
# Output: "2025-11-12 Wednesday"
```

### Step 2: Cross-Validate

Extract components:
- `env_date`: Date from `<env>` tag
- `sys_date`: Date from system command
- `sys_day`: Day-of-week from system command

**If dates MATCH:**
```
✓ Ground truth established
Today is: Wednesday, November 12, 2025
```

**If dates DON'T MATCH:**
```
❌ GROUND TRUTH MISMATCH - BLOCKING ALL EDITS

Environment says: 2025-11-11
System says: 2025-11-12 Wednesday

Reason: <env> tag likely stale from context compression
Action: User must restart session

STOP: Do not proceed with validations or edits
```

**Why two sources:** Baseline testing showed agents "trust user-provided context completely" without verification. If anchor is wrong, all calculations are wrong.

### Step 3: Calculate Reference Values

Once verified:
- `today_date`: 2025-11-12
- `today_day`: Wednesday
- `tomorrow_date`: 2025-11-13 (today + 1)
- `yesterday_date`: 2025-11-11 (today - 1)

---

## Validation Checklist

Run ALL validations before presenting errors. Don't stop at first error.

### ☐ Check 1: Day-Name ↔ Date Matching

**What to find:**

Patterns where day-name appears with date:
- `Wed Nov 12`
- `Wednesday, November 12`
- `Thu 11/14/2025`
- `Mon Nov 11, Tue Nov 12, Wed Nov 13` (sequences)

**Regex:**
```python
r'(Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday)\s*,?\s*([A-Z][a-z]+)\s+(\d{1,2})'
r'(Mon|Tue|Wed|Thu|Fri|Sat|Sun)\s+([A-Z][a-z]+)\s+(\d{1,2})'
```

**For each match:**
1. Parse date (US MM/DD format for numeric dates)
2. Calculate actual day-of-week:
   ```python
   from datetime import datetime
   date_obj = datetime.strptime("2025-11-12", "%Y-%m-%d")
   actual_day = date_obj.strftime("%A")  # "Wednesday"
   ```
3. Compare stated vs. actual

**If mismatch:**
```
❌ Error: Day-name mismatch
Found: "Wed Nov 12"
Truth: November 12, 2025 is Tuesday

Options:
1. Tue Nov 12 (correct day name)
2. Wed Nov 13 (correct date)
3. Something else (I'll type it)
```

**CRITICAL:** Validate EACH date individually. Baseline showed "sequential pattern feels right" even when all wrong.

---

### ☐ Check 2: Relative Date Calculations

**What to find:**
- `today` → should match ground truth
- `tomorrow` → ground truth + 1 day
- `yesterday` → ground truth - 1 day
- `in X days` → ground truth + X days

**Validation:**
For each term, check surrounding text (~50 chars) for date mentions. Verify alignment.

**Example error:**
```
Text: "tomorrow (Wed Nov 13)"
Today: Wednesday, Nov 12
Tomorrow should be: Thursday, Nov 13 (not Wed)
```

**If mismatch:**
```
❌ Error: Relative date incorrect
Found: "tomorrow (Wed Nov 13)"
Truth: Tomorrow is Thursday, Nov 13

Options:
1. tomorrow (Thu Nov 13)
2. today (Wed Nov 12)
3. Something else
```

---

### ☐ Check 3: Countdown Math

**What to find:**
- `X days until [date]`
- `X days remaining`
- `X days to [date]`

**Regex:**
```python
r'(\d+)\s+days?\s+(until|to|before)\s+([A-Za-z]+\s+\d{1,2})'
r'(\d+)\s+days?\s+remaining'
```

**Validation:**
1. Extract countdown number
2. Find target date
3. Calculate: `actual = target_date - ground_truth_date`
4. Compare stated vs. actual

**If mismatch:**
```
❌ Error: Countdown math incorrect
Found: "December 1, 2025 (18 days from today!)"
Truth: Dec 1 - Nov 12 = 19 days

Options:
1. (19 days from today!)
2. (18 days from tomorrow!)
3. Something else
```

---

### ☐ Check 4: Comprehensive Scan

**CRITICAL:** Baseline showed agents "fix obvious error" but miss others.

After finding first error, DON'T stop. Scan ENTIRE content:
- Check ALL lines with date references
- Include lines without day-names (might need them)
- Check headers, footers, tables, lists

Example from baseline: Agent fixed line 239 but missed line 74 (complex) and line 285 (no day-name).

**Solution:** Regex scan catches ALL occurrences before presenting errors.

---

## Error Correction Flow

### Step 1: Collect All Errors

Run all validations. Group results:
```
Found 3 errors:

CRITICAL:
- Error #1: Day-name mismatch at line 174
- Error #2: Countdown math at line 163

WARNINGS:
- Warning #1: Sequential inconsistency at line 240
```

### Step 2: Present Interactive Corrections

For each error:
1. Show context
2. Show what was found
3. Show what ground truth says
4. Offer 2-3 correction options
5. Wait for user choice

### Step 3: Apply and Re-Validate

1. Apply all corrections
2. Re-run ALL validations
3. If new errors → Return to Step 1
4. If all pass → Allow Edit/Write

---

## Red Flags - STOP and Verify

If you catch yourself thinking:

- ❌ "Trust the user-provided context completely"
- ❌ "Would NOT naturally use a calendar tool"
- ❌ "Only check if something felt obviously wrong"
- ❌ "Fix the most obvious error"
- ❌ "Quick scan with shallow verification"
- ❌ "Might have trusted the sequential pattern"
- ❌ "Internal consistency feels right"
- ❌ "Not verified each date individually"
- ❌ "Without explicit contradiction, accept at face value"

**All of these mean: Use two-source ground truth, validate systematically, check every date.**

---

## Common Rationalizations

These are from baseline testing - agents said these things when NOT using this skill:

| Excuse | Reality | Why It Fails |
|--------|---------|--------------|
| "Trust user-provided context" | Context can be wrong. Verify independently. | Anchor errors propagate |
| "Wouldn't naturally use calendar tool" | Tools exist for a reason. Use bash date. | Manual calculation fails |
| "Only check if feels wrong" | Errors don't "feel" wrong. Systematic check required. | Misses consistent errors |
| "Fix most obvious error" | Related errors exist. Comprehensive scan required. | Incomplete fixes |
| "Quick scan, shallow verification" | Shallow = misses errors. Deep required. | Misses edge cases |
| "Trust sequential pattern" | Sequential errors look consistent. Validate each. | All wrong together |
| "Internal consistency feels right" | Consistent ≠ correct. All could be wrong. | False confidence |
| "Not verified individually" | Individual check catches propagation. | Sequential bias |
| "Accept at face value" | Errors exist without contradictions. Always verify. | Needs suspicion |

---

## Edge Cases

### Ambiguous Dates

**Problem:** `11/12` could be Nov 12 or Dec 11

**Solution:**
1. Default: US format (MM/DD) = November 12
2. Check context for month names
3. If ambiguous: Prompt user

### Past Dates in Future Tense

**Problem:** `tomorrow (Nov 13)` but today is Nov 15

**Solution:**
- Warning only (don't block)
- Show: `⚠️  Note: Nov 13 was 2 days ago`
- Ask: `Is this an old document? (y/n)`

### Year Boundaries

**Problem:** Dec 30 → Jan 2 crosses years

**Solution:**
- Show year explicitly in corrections
- Handle leap years with Python datetime

---

## Success Criteria

Before allowing Edit/Write:
- ✅ Ground truth verified (env + system agree)
- ✅ ALL date patterns scanned
- ✅ EACH date validated individually
- ✅ All errors corrected by user
- ✅ Re-validation passed

Only after ALL criteria → Proceed with edit.

---

## Example Session

```
User: "Update Move Planning Hub with Nov 13 events"

[Skill auto-triggers on "Planning Hub" filename]

Step 1: Ground truth
  ENV: 2025-11-12
  SYS: 2025-11-12 Wednesday
  ✓ Verified: Wednesday, November 12, 2025

Step 2: Scan content
  Found: 15 date references

Step 3: Validate
  ✓ Day-name matching (8/8)
  ❌ Relative dates (1 error)
  ❌ Countdown math (1 error)

Found 2 errors:

❌ Error #1: "TODAY - NOVEMBER 13"
Truth: Today is November 12
Options: 1. TODAY - NOVEMBER 12  2. TOMORROW - NOVEMBER 13

User choice: 2

❌ Error #2: "(18 days from today)"
Truth: Dec 1 - Nov 12 = 19 days
Options: 1. (19 days)  2. Keep 18

User choice: 1

Step 4: Re-validate
  ✓ All validations passed

✅ Proceeding with Edit...
```

---

## Anti-Patterns from Baseline

**DON'T:**
- Trust anchor without two-source verification
- Fix one error and move on (scan comprehensively)
- Trust sequential patterns (validate each)
- Only verify when suspicious (always verify)
- Do mental math instead of bash date calculation
- Skip re-validation after corrections

**DO:**
- Establish ground truth first (env + system)
- Scan ALL content before presenting errors
- Validate EACH date individually
- Auto-trigger before edits (no manual check needed)
- Use Python datetime for calculations
- Re-validate after applying corrections
