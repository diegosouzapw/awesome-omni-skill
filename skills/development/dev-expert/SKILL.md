---
name: dev-expert
description: "Development patterns for React, Vue, Laravel, Next.js, React Native - state management, forms, API integration"
autoInvoke: false
priority: medium
triggers:
  - "coding pattern"
  - "state management"
  - "form handling"
  - "api integration"
---

# Skill: Dev Expert

**Category:** Reference Skill
**Version:** 2.0.0
**Used By:** All development agents

---

## Framework-Specific Patterns

For framework-specific patterns, use the auto-invoking expert skills:

```toon
expert_skills[8]{skill,triggers}:
  react-expert,React/JSX/hooks
  vue-expert,Vue/Composition API/Pinia
  react-native-expert,React Native/Expo/mobile
  nextjs-expert,Next.js/App Router/SSR
  laravel-expert,Laravel/PHP/Eloquent
  nodejs-expert,Node.js/Express/NestJS
  angular-expert,Angular/NgRx/RxJS
  flutter-expert,Flutter/Dart/widgets
```

---

## Generic Patterns (This Skill)

```toon
patterns[3]{file,purpose}:
  state-management.md,Zustand/Redux/Pinia patterns
  form-handling.md,Form validation + submission
  api-integration.md,API calls + error handling + caching
```

---

## When to Use

- Phase 5b (Implementation) - Generic coding patterns
- State management decisions (cross-framework)
- API integration patterns (cross-framework)
- Form handling patterns (cross-framework)

---

## Quick Reference

```toon
state_types[3]{type,use_when,solution}:
  Local state,Component-specific UI,useState/ref
  Global state,Shared app state,Zustand/Pinia/Redux
  Server state,API data,TanStack Query/SWR/RTK Query
```

---

**Version:** 2.0.0 | **Last Updated:** 2025-12-23
