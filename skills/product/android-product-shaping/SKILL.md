---
name: android-product-shaping
description: This skill is used to turn Android app ideas into small, well-bounded product slices with clear value, ready for UX and implementation.
---

# Android Product Shaping Skill

## Purpose
Define small, self-contained slices of Android functionality that deliver clear user value without expanding scope. Provide a stable specification before UX or engineering work begins.

## When to Use
Use this skill whenever a new Android feature, flow or improvement is proposed and needs to be turned into something buildable.

## Outputs
- Problem statement and context
- 3–5 user stories
- Acceptance criteria
- Explicit non-goals
- Minimal slice description

## Procedure
1. State who the Android user is and what problem they face.
2. Write 3–5 user stories that describe value from the user’s perspective.
3. Turn these into testable acceptance criteria based only on observable behaviour in the app.
4. List non-goals, including behaviours that must not be implemented in this slice.
5. Reduce the work to the smallest slice that:
   - Can be implemented inside the existing Kotlin, Compose, Room, Hilt and Navigation stack.
   - Does not require new backend systems, sync engines or AI providers unless explicitly requested.

## Guardrails
- Do not describe specific UI widgets or layouts. That belongs in UX and UI skills.
- Do not change or extend the overall architecture.
- Do not introduce sharing, sync, multi-provider AI or new storage layers unless the problem statement explicitly demands it.
- Keep scope limited to what can be delivered as a single Android release.
