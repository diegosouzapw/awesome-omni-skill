---
name: review:ux
description: UX Review - analyzes feature for efficiency-first UX patterns, keyboard navigation, and pro-tool experience
---

# UX Review

Performs a UX review optimized for **productivity tools and admin interfaces**. Based on the principle: **"Hands stay on keyboard, mouse is optional."**

This review evaluates features against efficiency-first UX patterns used in best-in-class productivity tools: command palettes, keyboard-first navigation, inline editing, optimistic UI, and minimal context switching.

## When to use

Use this skill when the user needs to:
- Validate that a feature feels fast and professional
- Check if keyboard shortcuts and command palette are properly integrated
- Verify inline editing vs unnecessary page transitions
- Ensure optimistic UI and instant feedback
- Review a feature for "pro tool" experience quality

## Core UX Principles

> The interface should think faster than the user

1. **Command Palette** — single entry point for all actions (⌘K / Ctrl+K)
2. **Keyboard-First** — frequent actions have shortcuts, arrow navigation works
3. **Inline Editing** — no modals for simple edits, Enter saves, Esc cancels
4. **Minimal States** — side panels over pages, overlays without context loss
5. **Search-First** — search is the primary navigation method
6. **Strong Defaults** — zero decisions needed for common actions
7. **Optimistic UI** — instant response, no spinners when avoidable
8. **List-Centric** — lists as main screens, saved views, fast filters
9. **Context > Settings** — actions next to objects, not buried in settings
10. **Visual Clarity** — no noise, color = meaning only

## Instructions

### Step 1: Determine Review Target

Parse `<args>` to determine what to review:

1. **Spec name** — if provided, read documents from `.specs/<spec-name>/` (requirements, design, tasks)
2. **File paths** — if provided, review specific files or directories
3. **Branch diff** — if `branch` is specified, review all changes on the current branch vs main

If no arguments provided:
1. Check if there are specs in `.specs/` and list them
2. Check if there are uncommitted changes or branch changes
3. Ask the user what to review

### Step 2: Gather Context

Read all available materials depending on the target:

#### From spec documents (if available):
- `requirements.md` — extract user stories, acceptance criteria, and described user flows
- `design.md` — extract UI components, data flow diagrams, and interaction patterns

#### From codebase:
Use **parallel sub-agents** (`subagent_type: "Explore"`) to investigate:

1. **Command palette agent** — find command palette implementation, registered commands, keyboard shortcut definitions
2. **UI patterns agent** — find forms, dialogs, pages, lists; identify inline editing vs modal patterns
3. **Navigation agent** — find route definitions, side panels, drawers, overlay patterns
4. **Feedback agent** — find optimistic updates, loading states, toast notifications, undo mechanisms

Launch all four agents in a **single message** (parallel tool calls).

### Step 3: Evaluate Efficiency-First Patterns

For each pattern, evaluate the current implementation level:

#### 3a. Command & Control

| Pattern | ❌ Missing | ⚠️ Basic | ✅ Good | 🚀 Excellent |
|---------|-----------|----------|---------|--------------|
| **Command Palette** | No ⌘K | Opens but limited actions | Search + navigation + actions | Context-aware commands, recent items, fuzzy search |
| **Keyboard Shortcuts** | None | Only global (save, close) | CRUD shortcuts (C/E/D) + navigation (↑↓) | Full coverage, discoverable hints, customizable |
| **Focus Management** | Random focus | Focus on page load | Logical tab order | Keyboard trap-free, skip links, roving tabindex |

#### 3b. Editing & Data Entry

| Pattern | ❌ Missing | ⚠️ Basic | ✅ Good | 🚀 Excellent |
|---------|-----------|----------|---------|--------------|
| **Inline Editing** | Always navigate away | Modal for every edit | Click-to-edit in lists | Enter=save, Esc=cancel, blur=save, multi-field inline |
| **Strong Defaults** | All fields required | Some defaults | Smart defaults based on context | Auto-fill from context, minimal required fields |
| **Form Efficiency** | One field per page | Standard form | Logical grouping, clear optionals | Auto-advance, paste-to-fill, batch entry |

#### 3c. Navigation & Context

| Pattern | ❌ Missing | ⚠️ Basic | ✅ Good | 🚀 Excellent |
|---------|-----------|----------|---------|--------------|
| **Minimal States** | Full page transitions | Modals for everything | Side panels for details | Overlays preserve context, split views, peek preview |
| **Search-First** | No search | Basic text match | Instant search, multiple entities | Filters, saved searches, search by ID/status/user |
| **List-Centric** | Card grids only | Basic lists | Sortable, filterable lists | Saved views, column customization, bulk actions |

#### 3d. Feedback & Performance

| Pattern | ❌ Missing | ⚠️ Basic | ✅ Good | 🚀 Excellent |
|---------|-----------|----------|---------|--------------|
| **Optimistic UI** | Wait for server | Spinner on every action | Optimistic for reads | Full optimistic with rollback, background sync |
| **Response Time** | Visible delays | < 500ms responses | < 200ms, skeleton loading | < 100ms, no spinners, prefetching |
| **Undo/Redo** | Confirm dialogs only | Toast with undo for delete | Undo for all destructive | Full undo stack, Ctrl+Z support |

#### 3e. Visual Design

| Pattern | ❌ Missing | ⚠️ Basic | ✅ Good | 🚀 Excellent |
|---------|-----------|----------|---------|--------------|
| **Visual Clarity** | Cluttered, many icons | Some organization | Clean, purposeful layout | Typography-driven, color = status only |
| **Context > Settings** | Actions in settings page | Actions in dropdown menu | Actions visible on hover | Inline actions, contextual toolbars |
| **Information Density** | Too sparse or too dense | Acceptable density | Scannable, grouped info | Progressive disclosure, smart truncation |

### Step 4: Cross-Reference with Spec

If spec documents are available, verify:

1. **Keyboard flows documented** — spec mentions keyboard shortcuts and navigation
2. **Inline editing specified** — design prefers inline over modal where appropriate
3. **Performance requirements** — response time and optimistic UI mentioned
4. **Command palette integration** — new actions registered in command palette

### Step 5: Generate UX Review Report

Present a structured report:

```markdown
# UX Review: [Feature Name]

**Scope:** [What was reviewed — spec docs, files, branch diff]
**UX Maturity:** [🚀 Pro-tool / ✅ Good / ⚠️ Basic / ❌ Needs Work]

---

## Efficiency-First Assessment

### Command & Control
| Pattern | Level | Evidence |
|---------|-------|----------|
| Command Palette | [emoji] | [What was found] |
| Keyboard Shortcuts | [emoji] | [What was found] |
| Focus Management | [emoji] | [What was found] |

### Editing & Data Entry
| Pattern | Level | Evidence |
|---------|-------|----------|
| Inline Editing | [emoji] | [What was found] |
| Strong Defaults | [emoji] | [What was found] |
| Form Efficiency | [emoji] | [What was found] |

### Navigation & Context
| Pattern | Level | Evidence |
|---------|-------|----------|
| Minimal States | [emoji] | [What was found] |
| Search-First | [emoji] | [What was found] |
| List-Centric | [emoji] | [What was found] |

### Feedback & Performance
| Pattern | Level | Evidence |
|---------|-------|----------|
| Optimistic UI | [emoji] | [What was found] |
| Response Time | [emoji] | [What was found] |
| Undo/Redo | [emoji] | [What was found] |

### Visual Design
| Pattern | Level | Evidence |
|---------|-------|----------|
| Visual Clarity | [emoji] | [What was found] |
| Context > Settings | [emoji] | [What was found] |
| Information Density | [emoji] | [What was found] |

---

## 🔴 Critical Issues

> Blocks keyboard users or forces unnecessary navigation

### [Issue Title]
**Pattern violated:** [Which efficiency pattern]
**Location:** `path/to/component.tsx`

[What the user experiences and why it slows them down]

**Fix:** [Specific recommendation]

---

## 🟠 Friction Points

> Slows down power users

- **[Location]** — [What's slow and why]
- **[Location]** — [What's slow and why]

---

## 🟡 Polish Opportunities

> Would elevate to pro-tool level

- [Opportunity 1]
- [Opportunity 2]

---

## ✅ What's Already Great

- [Positive finding 1]
- [Positive finding 2]

---

## Edge Cases

| Scenario | Status | Notes |
|----------|--------|-------|
| Empty state | [emoji] | [Has CTA? Keyboard accessible?] |
| Error state | [emoji] | [Dismissible? Undo available?] |
| Loading state | [emoji] | [Skeleton? Optimistic?] |
| Bulk operations | [emoji] | [Keyboard select? Batch actions?] |

---

## Quick Wins

> Low effort, high impact improvements

1. [ ] [Quick win 1]
2. [ ] [Quick win 2]
3. [ ] [Quick win 3]

---

## Summary

| Category | Score |
|----------|-------|
| Command & Control | [X/3 patterns at Good+] |
| Editing & Data Entry | [X/3 patterns at Good+] |
| Navigation & Context | [X/3 patterns at Good+] |
| Feedback & Performance | [X/3 patterns at Good+] |
| Visual Design | [X/3 patterns at Good+] |
| **Overall** | [X/15 patterns at Good+] |
```

### Step 6: Offer Next Steps

After presenting the report, offer actions:

1. **Add keyboard shortcuts** — help implement missing shortcuts
2. **Convert to inline editing** — refactor modals to inline patterns
3. **Add to command palette** — register new actions in ⌘K
4. **Implement optimistic UI** — add optimistic updates for slow operations
5. **Re-review** — run the review again after changes

## Severity Levels

| Level | Criteria | Examples |
|-------|----------|----------|
| 🔴 Critical | Keyboard users blocked; forced full-page navigation for simple actions | No keyboard access to primary action; modal required for single-field edit |
| 🟠 Friction | Power users slowed down; inconsistent with app patterns | Missing shortcut for frequent action; spinner where optimistic would work |
| 🟡 Polish | Good but not pro-level; minor efficiency gains possible | Could add to command palette; could show undo toast |

## Arguments

- `<args>` - Spec name, file paths, or review scope
  - `<spec-name>` — review UX based on spec documents and related code
  - `<file-path>` — review UX of specific files or directories
  - `branch` — review UX of all changes on current branch vs main

Examples:
- `review:ux user-auth` — review UX for the user-auth feature spec
- `review:ux src/pages/settings/` — review UX of the settings pages
- `review:ux branch` — review UX of all UI changes on the current branch
