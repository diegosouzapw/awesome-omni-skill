---
name: android-qa-verification
description: This skill is used to verify Android features against acceptance criteria, catch regressions and define tests that reflect real device behaviour.
---

# Android QA and Verification Skill

## Purpose
Ensure implemented Android features behave correctly, survive lifecycle changes and match acceptance criteria on real devices and emulators.

## When to Use
Use this skill after implementation work or when bugs or regressions are suspected.

## Outputs
- Behaviour-based test plan
- Unit, instrumentation and UI test ideas
- Edge-case scenarios
- Reproduction steps
- Focused fix suggestions

## Procedure
1. Take the acceptance criteria from product shaping and turn each into one or more test cases.
2. Identify Android-specific edge conditions:
   - Configuration changes (rotation, theme change).
   - Process death and recreation.
   - Timezone changes and daylight saving shifts if times are involved.
   - App backgrounding and resuming.
3. Plan tests at appropriate levels:
   - Unit tests for repositories and ViewModels.
   - Instrumentation or UI tests for flows through Composables and navigation.
   - Manual test steps where automation is difficult (for example notifications).
4. Validate migrations:
   - Describe steps to upgrade from an old database version to the new one.
   - Confirm existing data is preserved and new fields behave as expected.
5. For each bug:
   - Provide a minimal, deterministic reproduction path.
   - Map the likely layer (UI, ViewModel, repository, database, platform integration).
   - Suggest a targeted change, not a refactor.

## Guardrails
- Do not propose architectural rewrites.
- Do not rely on brittle UI tests that mirror implementation details closely.
- Focus on user-visible behaviour, data integrity and stability across lifecycle events.
