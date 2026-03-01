---
name: android-engineering-core
description: This skill is used to implement Android features within the existing Kotlin, Compose, Room, Hilt and Navigation architecture, including data, navigation and background work.
---

# Android Engineering and Core Implementation Skill

## Purpose
Implement the shaped and designed Android feature in the existing architecture, covering data persistence, business logic, navigation, reminders and background behaviour.

## When to Use
Use this skill once product shaping and UX design are complete and the UI structure is known.

## Outputs
- Updated Room entities, DAOs and migrations when required
- Repository and use-case logic
- ViewModel state and event handling
- Navigation wiring
- Integration with AlarmManager, notifications or other platform components

## Procedure
1. Confirm the slice fits within the current stack:
   - Kotlin, Coroutines and Flow
   - Room for local data
   - Hilt for dependency injection
   - Jetpack Navigation and Compose for UI
2. Apply the smallest data changes needed:
   - Adjust existing entities where possible.
   - Add new entities only when the UX slice truly demands it.
   - Create safe Room migrations for any schema changes.
3. Map user actions from the UX design to ViewModel events:
   - Define clear intent functions (for example onAddTask, onToggleComplete).
   - Keep side effects (saving, scheduling reminders) in the ViewModel or use-case layer, not in composables.
4. Implement repositories or use-cases:
   - Keep them focused on data and business logic.
   - Expose Flows or suspend functions as appropriate.
5. Wire navigation:
   - Add or adjust routes in the NavHost.
   - Pass arguments via navigation arguments and SavedStateHandle.
6. Integrate platform services carefully:
   - Use AlarmManager, Notifications and BroadcastReceivers for reminders when required.
   - Handle boot rescheduling if reminders depend on time.
   - Avoid new background frameworks unless necessary.

## Guardrails
- Do not change the overall architecture or introduce new frameworks.
- Do not move logic into composables.
- Do not add network sync, backends or multi-device features unless explicitly required by shaping.
- Prefer small, incremental changes over broad refactors.
