---
name: android-ui-compose
description: This skill is used to implement Android UI in Jetpack Compose based on an existing UX flow, focusing on clear hierarchy, list vs form separation and discoverable navigation.
---

# Android UI Compose Implementation Skill

## Purpose
Translate UX designs into concrete Jetpack Compose layouts that feel clean and consistent. Keep lists as lists, forms as forms, and navigation discoverable.

## When to Use
Use this skill when a UX flow and section structure already exist and the UI needs to be implemented or refined in Kotlin with Compose.

## Outputs
- Composable functions for screens and components
- Layout that reflects hierarchy and sections
- Item rows or cards for existing entities
- Forms for creation or editing in a controlled place
- Empty states and hints

## Procedure
1. Take the UX specification and identify:
   - The main screen composable.
   - Reusable item composables (for example ListCard, TaskRow).
   - Places where dialogs, sheets or snackbars are required.
2. Implement a clear structure:
   - Top level scaffold with a top bar and optional bottom bar if used elsewhere.
   - A “create new” section that is visually separate from the list of existing items.
   - A LazyColumn or relevant container for lists of items.
3. For existing items:
   - Represent them as rows or cards, not full edit forms.
   - Include:
     - Primary text (for example list name or task title).
     - Optional secondary text (for example due date, list summary).
     - Tappable checkbox or icon where appropriate.
     - A clear tap target to open details or edit.
4. For editing:
   - Use either:
     - An inline expanded row for the item that is currently being edited, or
     - A dialog or bottom sheet with labelled fields.
   - Allow editing one item at a time to avoid visual overload.
5. Implement empty states and hints:
   - Use simple text and spacing to explain what to do.
   - Keep add buttons and initial actions visible in empty states.
6. Align with existing styling:
   - Use Material components already present in the project.
   - Reuse padding, shapes and typography from existing screens.
   - Keep component names consistent with the codebase.

## Guardrails
- Do not embed full edit forms inside every list row by default.
- Do not hide navigation in subtle click targets; ensure rows are clearly interactive.
- Do not introduce new design systems, libraries or icon packs.
- Do not alter ViewModel contracts or data models; treat state as given.
