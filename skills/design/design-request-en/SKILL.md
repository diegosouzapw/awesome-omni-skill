---
name: design-request-en
description: Gather design change requests through interactive dialog and create a structured Issue.
user-invocable: true
---

**Language: Always interact with the user in 日本語.**

# design-request-en

Organize design change requests through dialog with design references (screenshots, mockups, verbal description) and create an actionable Issue. Usable by non-engineers.

## Prerequisites

- Claude Code environment
- `gh` CLI (for GitHub Issue output)

## Arguments

- **Text / image path** (e.g., `/design-request-en Want to change the header design`): Use as initial info, interview for missing details
- **No arguments**: Start interviewing from scratch

## Phase 1: Design Reference and Project Scan

1. Confirm design reference with `AskUserQuestion`:
   - **Screenshot / image**: Receive file path and read with `Read`
   - **Verbal description**: Receive as text
   - **Both**: Image + supplemental description
2. **Lightweight project scan** (to ground interview options in reality):
   - Identify screens/pages from directory structure (`pages/`, `views/`, `routes/`, etc.)
   - Identify component directory structure (`components/`, etc.)
   - Identify framework and CSS library
3. Confirm target screen/component with **scan-based options**

## Phase 2: Interview

**Ask in plain language without technical jargon.**
**Carefully clarify vague requests until they're concrete enough to implement.**

#### 2-1: Disambiguate Vague Requests

When user input is vague, make it concrete. **Prefer offering options over free-form input.**

| User input | What to clarify | How to ask |
|------------|----------------|-----------|
| "Ugly", "Meh", "Not great" | What specifically bothers them | Options: colors / layout / font / spacing / overall impression / other |
| "Make it nicer" | Ideal image | Options: simple-clean / modern-refined / warm-friendly / bold-impactful + reference sites |
| "Change this" (location unclear) | Specific area | Options from screen sections or ask for screenshot |
| "Hard to use" | What they tried to do | Ask specific scenario: which action / what happened / what they expected |
| "Want it aligned/consistent" | What's the reference | Options: match existing screen / new direction + reference screen if any |

#### 2-2: Gather Change Details

Use `AskUserQuestion` to gather (skip what's known from arguments/images):
1. **What to change**: Which part, how to change it (skip if clarified in 2-1)
2. **Reason/background**: Why the change is needed (optional)
3. **Priority**: Urgent or next release is fine

#### 2-3: Follow-up Questions

Ask **only what's needed** based on responses (max 3, don't ask what can be inferred):

| Situation | What to ask |
|-----------|-------------|
| Layout change | Behavior per screen size |
| Color/font change | Specific values or reference sites |
| New element | Placement, sizing, interactions |
| Animation | Timing, type, reference examples |
| Multi-screen impact | Impact scope confirmation |
| Conditional display | Which conditions, what display |
| "Leave it to you" | Investigate code then present multiple options |

#### 2-4: Confirm Understanding

After interview, **summarize understanding in bullet points and confirm with user**.
Repeat corrections until agreement, then proceed to Phase 3.

## Phase 3: Codebase Investigation

1. **Identify target component** — Search by screen/component name with `Grep`/`Glob`. Use `Explore` agent if needed
2. **Understand current state** — Check current styling, layout, component structure
3. **Check impact scope** — Other screens using same component, shared styles
4. **Check design foundation** — Theme settings, design tokens, available existing components

## Phase 4: Preview and Output Destination

1. Create draft from gathered info and investigation (see `templates/issue.md`)
2. Show preview to user
3. Apply revisions if any (repeat until satisfied)
4. Confirm output destination with `AskUserQuestion`:
   - **Create GitHub Issue** (recommended): `design: <change summary>` format
   - **Save as local MD file**: Generate in project root
   - **Append to existing Issue**: Specify Issue number to add as comment

## Phase 5: Output

#### GitHub Issue (new)
- Create with `gh issue create`, title `design: <change summary>`, label `design`
- Report URL to user

#### Append to Existing Issue
- Add as comment with `gh issue comment <number>`
- Report URL to user

#### Local MD
- Generate `design-request-<summary-kebab-case>.md` in project root
- Report path to user

## Priority Criteria

| Priority | Criteria |
|----------|----------|
| Urgent | Severely harms user experience, release blocker |
| High | Major screen appearance issue, brand inconsistency |
| Medium | Nice to improve, can be addressed next release |
| Low | Fine-tuning, address when time permits |

## Rules

- **No technical jargon** in interviews. Mirror user's own terms only
- `AskUserQuestion` options must be **based on project scan results**. Never include screens/components that don't exist in the project
- **Minimal questions**. Don't ask what can be inferred
- Code investigation must **verify against actual code**, not guess
- Issues must clearly state **before state and expected after state**
- **Always preview and get user approval** before creating Issue
- Do not modify code. Output is Issue only
