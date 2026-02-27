---
name: token-pacing
description: Calculate the optimal token usage burn rate to reach exactly 100% usage by reset. Use when the user asks about token budget, usage limits, spending speed, or "will I run out". Supports Claude, Gemini, Codex, VS Code, and other providers.
---

# Token Pacing Calculator

Analyzes current token usage against the reset deadline to provide a specific daily burn rate target.

**Why?** Optimizes resource utility by ensuring the user neither runs out of tokens too early nor leaves unused budget on the table.

## Quick Start

This skill calculates optimal token burn rate from current usage data.

**To use this skill, provide:**
1. **Provider name** (Claude, Gemini, Codex, VS Code, Cursor, Windsurf, etc.)
2. **Current usage %** (or % remaining)
3. **Time until reset** (for weekly providers)

**Example:** "I'm at 31% in Claude, resets in 4 days"

**Then the skill will:**
1. Apply provider profile to normalize metric
2. Calculate pacing (Steps 1-4 below)
3. Output the Token Pacing Report

---

## How to Get Usage Data (Optional)

If you have `ai-usage` installed, run it for a quick overview of all services.
Otherwise, check your provider directly:

| Provider | How to Check |
|----------|--------------|
| **Claude Code** | IDE status bar, or `/usage` command |
| **OpenAI Codex** | ChatGPT settings, or IDE status bar |
| **Gemini CLI** | `gcli quota` or Google Cloud dashboard |
| **VS Code Copilot** | Copilot icon in status bar → "View quota usage" |
| **Cursor** | Settings → Copilot → Usage |
| **Windsurf** | Codeium status panel |

**Web dashboards:**
- Claude/Codex: Provider settings pages
- Gemini: https://console.cloud.google.com/
- Copilot: https://github.com/settings/copilot

---

## Provider Profiles

Auto-detect the provider and apply these defaults:

| Provider | Aliases | Metric Shown | Reset Period | Reset Day |
|----------|---------|--------------|--------------|-----------|
| **Claude** | Claude Code, Anthropic | Usage % | Weekly | User-specific (ask) |
| **Gemini** | Google Gemini, Gemini Code | Usage % | Weekly | User-specific (ask) |
| **Codex** | OpenAI Codex, ChatGPT | % Remaining | Weekly | User-specific (ask) |
| **VS Code** | VS Code Copilot, GitHub Copilot (VS Code) | Usage % | Monthly | 1st of month |
| **Cursor** | Cursor AI | Usage % | Monthly | 1st of month |
| **Windsurf** | Codeium Windsurf | Usage % | Monthly | 1st of month |

> [!NOTE]
> Weekly providers (Claude, Gemini, Codex) have **personal reset cycles** — there's no universal reset day. Ask the user (unless provided): "When does your week reset?"

### Metric Normalization

```
If provider shows "% remaining" (e.g., Codex):
    usage_% = 100 - remaining_%
Else:
    usage_% = reported value
```

### Reset Date Calculation

**Weekly providers (Claude, Gemini, Codex):**
```
# Reset day is USER-SPECIFIC — must ask if not provided
# Example: "My Claude resets on Thursdays" or "resets in 3 days"
cycle_days = 7
reset = user's next reset date/time
days_remaining = (reset - now) in days
days_elapsed = cycle_days - days_remaining
```

**Monthly providers (VS Code, Cursor, Windsurf):**
```
# Reset is predictable — 1st of next month
cycle_days = days in current month (28-31)
reset = 1st of next month, 12:00am
days_remaining = (reset - now) in days
days_elapsed = current day of month
```

> [!TIP]
> For weekly providers, if user doesn't know their reset day, suggest they check their usage dashboard — it usually shows "resets in X days".

---

## Inputs Required

**For monthly providers (VS Code, Cursor, Windsurf):**
- Just the metric value — reset date is predictable
- Example: "16% in VS Code"

**For weekly providers (Claude, Gemini, Codex):**
- Metric value + reset info (days remaining or reset day)
- Example: "31% in Claude, resets in 4 days" or "Codex 40% left, resets Thursday"

> [!TIP]
> If user gives weekly provider without reset info, ask: "When does your usage reset? (e.g., 'in 3 days' or 'on Friday')"

> [!CAUTION]
> **Ambiguous phrasing:** If user says "X% left" for a non-Codex provider, clarify immediately: "Is that X% *used* or X% *remaining*?" — this changes the calculation entirely.

---

## Calculation Steps

Perform these calculations directly — no external script needed.

### Step 0: Validate Inputs

Before calculating, confirm:
1. **Provider** is identified → if not, ask
2. **Metric type** is clear (usage % vs % remaining) → if ambiguous, clarify
3. **Reset info** is available → for weekly providers, ask if missing

### Step 1: Calculate Time Metrics

```
now = current date/time
reset = reset date/time (from provider profile or user input)
days_remaining = (reset - now) in days (decimal, e.g., 4.4)
cycle_days = 7 (weekly) or days_in_month (monthly)
days_elapsed = cycle_days - days_remaining
time_elapsed_% = (days_elapsed / cycle_days) × 100
```

### Step 2: Calculate Usage Metrics

```
# Normalize metric based on provider
if provider shows "% remaining":
    usage_% = 100 - reported_value
else:
    usage_% = reported_value

remaining_% = 100 - usage_%
daily_target = remaining_% / days_remaining
```

### Step 3: Determine Status

Compare `usage_%` to `time_elapsed_%`:

| Condition | Status | Meaning |
|-----------|--------|---------|
| `usage_% < time_elapsed_% - 3` | **Under Budget** | Saving tokens, can spend more freely |
| `usage_% > time_elapsed_% + 3` | **Over Budget** | Burning too fast, need to slow down |
| Otherwise | **On Track** | Balanced pace |

> [!TIP]
> The ±3% buffer avoids false alarms for minor deviations.

### Step 4: Calculate Buffer

```
buffer_% = time_elapsed_% - usage_%
```

- **Positive buffer** = tokens banked (ahead of schedule)
- **Negative buffer** = tokens owed (behind schedule)

---

## Response Template

Report the following:

```
## Token Pacing Report

**Provider:** [Provider Name] ([weekly/monthly] reset)
**Status:** [Under Budget / On Track / Over Budget]

| Metric | Value |
|--------|-------|
| Used | X% |
| Time Elapsed | Y% |
| Buffer | ±Z% |
| Remaining | W% over N days |
| Daily Target | D%/day |

[One sentence recommendation based on status]
```

### Recommendation by Status

- **Under Budget:** "You're saving tokens. You can increase usage to [daily_target]%/day to hit 100% by reset."
- **On Track:** "You're pacing well. Maintain ~[daily_target]%/day."
- **Over Budget:** "You're burning fast. Limit usage to [daily_target]%/day to last until reset."

---

## Examples

### Example 1: VS Code (Monthly Reset)

**Input:** "16% in VS Code" (current date: Jan 11)

```
Provider: VS Code → monthly reset, 1st of next month
cycle_days = 31 (January)
days_elapsed = 11
days_remaining = 20
time_elapsed_% = (11 / 31) × 100 = 35.5%

usage_% = 16%
remaining_% = 84%
daily_target = 84 / 20 = 4.2%/day

buffer_% = 35.5 - 16 = +19.5% (well under budget)
status = Under Budget
```

**Output:**
```
## Token Pacing Report

**Provider:** VS Code (monthly reset)
**Status:** Under Budget

| Metric | Value |
|--------|-------|
| Used | 16% |
| Time Elapsed | 35.5% |
| Buffer | +19.5% |
| Remaining | 84% over 20 days |
| Daily Target | 4.2%/day |

You're saving tokens. Increase to 4.2%/day to fully utilize your allocation.
```

---

### Example 2: Claude (Weekly Reset)

**Input:** "31% used in Claude" (current: Jan 10, reset: Jan 15 Wed 9am PT)

```
Provider: Claude → weekly reset, Wednesday 9am PT
cycle_days = 7
days_remaining = 4.4 days
days_elapsed = 2.6 days
time_elapsed_% = (2.6 / 7) × 100 = 37%

usage_% = 31%
remaining_% = 69%
daily_target = 69 / 4.4 = 15.7%/day

buffer_% = 37 - 31 = +6%
status = Under Budget
```

**Output:**
```
## Token Pacing Report

**Provider:** Claude (weekly reset)
**Status:** Under Budget

| Metric | Value |
|--------|-------|
| Used | 31% |
| Time Elapsed | 37% |
| Buffer | +6% |
| Remaining | 69% over 4.4 days |
| Daily Target | 15.7%/day |

You're saving tokens. You can increase usage to 15.7%/day to hit 100% by reset.
```

---

### Example 3: Codex (% Remaining Format)

**Input:** "Codex shows 40% left" (current: Thursday, reset: Sunday 12am)

```
Provider: Codex → weekly reset, shows % REMAINING (invert!)
usage_% = 100 - 40 = 60%

cycle_days = 7
days_remaining = 2.5 days (Thu to Sun)
days_elapsed = 4.5 days
time_elapsed_% = (4.5 / 7) × 100 = 64%

remaining_% = 40%
daily_target = 40 / 2.5 = 16%/day

buffer_% = 64 - 60 = +4%
status = On Track
```

**Output:**
```
## Token Pacing Report

**Provider:** Codex (weekly reset)
**Status:** On Track

| Metric | Value |
|--------|-------|
| Used | 60% (40% remaining) |
| Time Elapsed | 64% |
| Buffer | +4% |
| Remaining | 40% over 2.5 days |
| Daily Target | 16%/day |

You're pacing well. Maintain ~16%/day.
```

---

### Example 4: Over Budget (The Spender)

**Input:** "80% in Gemini" (current: Tuesday, reset: Monday 12am)

```
Provider: Gemini → weekly reset, Monday 12am PT
cycle_days = 7
days_remaining = 6 days
days_elapsed = 1 day
time_elapsed_% = (1 / 7) × 100 = 14%

usage_% = 80%
remaining_% = 20%
daily_target = 20 / 6 = 3.3%/day

buffer_% = 14 - 80 = -66% (way behind)
status = Over Budget
```

**Output:**
```
## Token Pacing Report

**Provider:** Gemini (weekly reset)
**Status:** Over Budget

| Metric | Value |
|--------|-------|
| Used | 80% |
| Time Elapsed | 14% |
| Buffer | -66% |
| Remaining | 20% over 6 days |
| Daily Target | 3.3%/day |

You're burning fast. Limit usage to 3.3%/day to last until reset.
```

---

### Example 5: On Track (The Balanced)

**Input:** "50% in Claude" (mid-week)

```
Provider: Claude → weekly reset
cycle_days = 7
days_remaining = 3.5 days
days_elapsed = 3.5 days
time_elapsed_% = 50%

usage_% = 50%
remaining_% = 50%
daily_target = 50 / 3.5 = 14.3%/day

buffer_% = 0%
status = On Track
```

**Output:**
```
## Token Pacing Report

**Provider:** Claude (weekly reset)
**Status:** On Track

| Metric | Value |
|--------|-------|
| Used | 50% |
| Time Elapsed | 50% |
| Buffer | 0% |
| Remaining | 50% over 3.5 days |
| Daily Target | 14.3%/day |

You're pacing well. Maintain ~14.3%/day.
```

---

## The 5pm Rule (Advanced Pacing)

**Problem:** Standard pacing aims for 100% at the exact reset time. But if your reset is at 11:59 AM and you hit 100% at 11:58 AM, you've perfectly paced... but also can't use tokens all morning. Similarly, evening resets (10 PM) mean you can't use tokens during dinner/evening work.

**Solution:** Aim to hit 100% by **5:00 PM** on the "effective reset day" — this eliminates "dead time" when you'd otherwise be token-limited during productive hours.

### Effective Reset Day

| Reset Time | Effective Target |
|------------|------------------|
| **Morning reset** (before 5 PM) | 5:00 PM the day **before** |
| **Evening reset** (5 PM or later) | 5:00 PM the **same day** |

**Examples:**
- Reset at 11:59 AM Wednesday → Target: 5:00 PM Tuesday
- Reset at 10:56 PM Thursday → Target: 5:00 PM Thursday
- Reset at 12:00 AM (midnight) Monday → Target: 5:00 PM Sunday

### Adjusted Calculation

When using the 5pm Rule, replace the `reset` datetime in Step 1:

```
# Standard calculation
reset = actual_reset_datetime

# 5pm Rule calculation
if reset.time < 17:00:
    effective_reset = (reset.date - 1 day) at 17:00
else:
    effective_reset = reset.date at 17:00

# Then use effective_reset for all calculations
days_remaining = (effective_reset - now) in days
```

> [!TIP]
> The 5pm Rule is optional but recommended for users who want to maximize productive usage hours. Always mention when using it in the report.

### Report Addition for 5pm Rule

When using the 5pm Rule, add to the report:

```
**Target:** 5:00 PM [Day] (5pm Rule applied)
```

---

## Quality Guidelines

- Round percentages to 1 decimal place
- **Always show both metrics:** Time passed % vs % used — this makes pacing intuitive at a glance
- If `daily_target > 50%`, warn that this is an extremely heavy workload
- If `days_remaining < 1`, switch to hourly targets:
  ```
  hours_remaining = days_remaining × 24
  hourly_target = remaining_% / hours_remaining
  ```

> [!WARNING]
> If `daily_target > 50%/day`, add this warning: "This requires very heavy usage. Consider whether you can realistically sustain this pace."

### Pre-Calculation Checklist

Before running calculations, verify:

- [ ] Provider identified (or asked)
- [ ] Metric type confirmed (usage % vs % remaining)
- [ ] Reset period known (weekly vs monthly)
- [ ] For weekly providers: reset day/time obtained from user
- [ ] Ambiguous phrasing clarified ("50% left" → usage or remaining?)

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Provider not specified | Ask: "Which tool? (Claude, Gemini, VS Code, Codex, Cursor, Windsurf)" |
| User gives ambiguous date ("next Wednesday") | Parse as the next upcoming occurrence from current date |
| Reset time appears to be in the past | Ask user to confirm the next reset date |
| Usage reported as > 100% | Report status as "Exhausted" with 0% remaining, 0%/day target |
| User doesn't know exact reset time | Use provider default from profiles table |
| Very short time remaining (< 4 hours) | Switch to per-hour targets and emphasize urgency |
| Unknown provider | Ask for reset period (weekly/monthly) and metric type (usage % or % remaining) |
| User reports "X% left" for non-Codex | Clarify: "Is that usage or remaining?" then normalize |

---

## Common Mistakes to Avoid

| Mistake | Correct Approach |
|---------|------------------|
| Assuming weekly reset for all providers | Check provider profile — VS Code/Cursor/Windsurf are monthly |
| Not inverting "% remaining" for Codex | Codex shows remaining; convert: `usage = 100 - remaining` |
| Using 7 days for monthly providers | Use actual days in month (28-31) |
| Forgetting to convert "days left" to decimal | Use precise decimal (e.g., 4.4 days, not 4 days) |
| Reporting buffer as absolute tokens | Buffer is always a percentage |
| Not adjusting for partial days | Include fractional days in calculations |
