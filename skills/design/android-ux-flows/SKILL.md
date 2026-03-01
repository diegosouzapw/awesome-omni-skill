---
name: android-ux-flows
description: This skill is used to design Android user flows and screen structures that match the existing app patterns and keep forms, lists and navigation clear.
---

# Android UX and Interaction Design Skill

## Purpose
Turn a shaped product slice into a clear Android user journey and screen structure that feels natural in Compose, with simple navigation and discoverable actions.

## When to Use
Use this skill after product shaping is complete and before any implementation work starts.

## Outputs
- Entry → action → exit flow
- Screen and section structure
- Interaction rules, including tap areas and edit modes
- Empty states and navigation hints

## Procedure
1. Identify where the user enters the flow (for example, app launch, list tap, notification tap) and where they leave it.
2. Decide which existing screens are reused and whether a new screen, bottom sheet or dialog is required.
3. For each screen:
   - Define sections and hierarchy (headers, “create new” area, list of existing items, secondary actions).
   - Separate “create” forms from “view/edit existing items”.
4. Specify interaction patterns that avoid everything being an inline form:
   - Use rows or cards for existing items.
   - Use a single “new item” form or dialog rather than repeating forms per row.
   - Put rename or advanced actions behind menus or explicit buttons.
5. Define empty states that explain what to do and how to reach the next screen. Include hints such as:
   - “Create a list, then tap it to view tasks.”
6. Ensure navigation is obvious:
   - Entire rows or cards as tap targets.
   - Icons or labels that signal navigation (for example chevrons).
   - No hidden navigation that relies on guessing.

## Guardrails
- Do not invent new navigation paradigms. Use existing use of NavHost, top bars and bottom bars.
- Do not turn lists into stacked edit forms. Keep clear separation between viewing and editing.
- Do not specify colours or typography beyond hierarchy and grouping.
- Respect existing screen names, terminology and style.
