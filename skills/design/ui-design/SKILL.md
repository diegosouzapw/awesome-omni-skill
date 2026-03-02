---
name: ui-design
description: Applies consistent renderer UI/UX implementation patterns using a Vercel-inspired white theme, strong accessibility defaults, and repository component conventions.
license: MIT
opencode:
  compatibility: opencode
  metadata:
    author: nabby27
    version: 1.0
copilot:
  user-invokable: true
---

# Skill: UI Design

This skill defines how to implement or review renderer UI so it looks unified with the existing app and follows a high-quality product design baseline.

Primary sources:

- `docs/frontend/components.md`
- `docs/guidelines/code-standards.md`

## Scope

### Use when

- Implementing or refining renderer UI components, pages, or visual states.
- Harmonizing new UI with existing app layout, spacing, typography, and interaction patterns.
- Validating design consistency before final review.

### Do NOT use when

- Working on main/preload architecture without renderer UI impact.
- Defining test-suite routing or coverage strategy.

## Design baseline

1. Vercel-inspired white theme direction
   - Bright base surfaces and subtle neutral layers.
   - Clean borders and restrained shadows.
   - High text contrast and clear hierarchy.
2. Repository-first consistency
   - Reuse existing Tailwind and shadcn patterns.
   - Respect established component behavior and spacing rhythm.
   - Prefer extending current primitives over introducing custom one-off controls.
3. UX quality requirements
   - Clear information hierarchy and predictable navigation.
   - Visible interactive states (hover, focus, active, disabled).
   - Keyboard-friendly flows for primary actions.
4. Accessibility defaults
   - Sufficient contrast in text and controls.
   - Focus states are always visible.
   - Meaning is not conveyed by color alone.
5. Responsive behavior
   - UI must remain usable on desktop and mobile for touched flows.
   - Avoid layout breakpoints that cause clipped content or inaccessible actions.

## Implementation rules

- Use Tailwind classes and shadcn primitives as primary styling stack.
- Keep style tokens coherent: spacing, radius, border, and typography should match nearby screens.
- Prefer minimal purposeful motion; avoid decorative animation noise.
- Avoid inline styles for regular component styling.
- Do not introduce new visual language in isolated pages without explicit feature requirement.

## Consistency checklist

For each changed UI file, verify:

1. Visual alignment
   - Matches surrounding pages and shared components.
2. Typography and spacing
   - Preserves hierarchy and spacing cadence.
3. Interaction quality
   - States are visible and behavior is predictable.
4. Accessibility
   - Focus, contrast, and keyboard path remain valid.
5. Responsive stability
   - Main flow remains functional on mobile and desktop.

## Anti-patterns (blocker)

- New UI that ignores existing app visual language.
- Hard-to-read text or missing focus states.
- One-off style overrides that conflict with shared component patterns.
- Large decorative changes without product requirement.

## Definition of done

- Changed UI is visually consistent with the app and follows white-theme baseline.
- Accessibility and responsive checks pass for touched behavior.
- Design adjustments are minimal, intentional, and scope-bounded.

## Output contract

Return a structured YAML report including:

- `status`: `ok` | `needs_clarification` | `error`
- `summary`: short UI consistency and UX quality assessment
- `scope`: task IDs, files reviewed, assumptions
- `decisions`: key design decisions and rationale
- `consistency_checks`: checklist results by file or module
- `findings`: issues with severity, evidence, and minimal fix
- `verdict`: `ready` | `ready_with_changes` | `not_ready`
- `next_actions`: minimal action list
- `warnings`: unresolved risks or assumptions
