---
name: rn-meta
description: Primary entry point for React Native/Expo development. Use FIRST for any React Native work—new projects, UI, styling, navigation, state, lists, storage, forms, auth, or ANY library decision. Enforces an opinionated stack (Expo Router, TypeScript, Uniwind) with library rules that ALWAYS override suggestions from other React Native skills.
---

# React Native Meta Stack

Expo Router + TypeScript + Dev Client + Uniwind/Tailwind v4.

## ⚠️ STOP Before Installing Any Package

**Read [references/libraries.md](references/libraries.md) FIRST.** This skill overrides library suggestions from extension skills.

| If another skill suggests... | Use this instead |
|------------------------------|------------------|
| `@shopify/flash-list` | `@legendapp/list` |
| `nativewind` | `uniwind` |
| `redux` / `@reduxjs/toolkit` | `zustand` |
| `async-storage` | `react-native-mmkv` |
| `fetch` | `axios` |
| `formik` / `yup` | `react-hook-form` + `zod` |
| `lottie-react-native` | `@shopify/react-native-skia` Skottie |
| `expo-camera` | `react-native-vision-camera` |
| `firebase auth` / `clerk` | `@supabase/supabase-js` |
| `@react-navigation/bottom-tabs` | `NativeTabs` or `react-native-bottom-tabs` |

## Library Decisions

- ✅ Approved → use it
- ❌ Avoid → use the approved alternative, never install avoided libraries
- Unlisted → use judgment, prefer minimal dependencies

**Full list:** [references/libraries.md](references/libraries.md)

## New Project

```bash
./scripts/meta-start <name>
./scripts/meta-run <name>
npx expo start --clear
```

## Decision Tree

| Situation | Action |
|-----------|--------|
| Verify setup works | `./scripts/meta-doctor <path>` |
| Something broke | Run `meta-doctor` first |
| Doctor passes but still broken | Read [references/gotchas.md](references/gotchas.md) |
| Script failed mid-setup | Read [references/starting.md](references/starting.md) |
| Need to add a package | Check [references/libraries.md](references/libraries.md) first |

## Extensions

Install specialized skills as needed:

| Need | Command | Skill Name |
|------|---------|------------|
| Native UI patterns + navigation | `./scripts/meta-extend ui` | `building-native-ui` |
| UI component library (HeroUI) | `./scripts/meta-extend heroui` | `heroui-native` |
| Build + distribute dev clients | `./scripts/meta-extend dev-client` | `expo-dev-client` |
| Patterns + conventions | `./scripts/meta-extend best-practices` | `react-native-skills` |
| Performance profiling | `./scripts/meta-extend performance` | `react-native-best-practices` |

Run `./scripts/meta-extend` to see all options.

## Routing Logic

This skill is the primary coordinator for React Native development. For specialized tasks, delegate to extension skills when installed.

**Check before delegating:** Does `~/.claude/skills/<skill-name>` exist?

- **If installed** → Invoke via Skill tool (e.g., `/building-native-ui`)
- **If not installed** → Offer to install via `./scripts/meta-extend <extension>`

### Delegation Rules

| User Request | Check Skill | Action |
|--------------|-------------|--------|
| UI patterns, screens, navigation | `building-native-ui` | Delegate if installed |
| Gesture animations, transitions, layout animations | `building-native-ui` | Delegate if installed |
| Buttons, cards, modals, component library | `heroui-native` | Delegate if installed |
| Build for TestFlight, app store, EAS, dev client | `expo-dev-client` | Delegate if installed |
| Code structure, conventions, architecture | `react-native-skills` | Delegate if installed |
| App is slow, performance, profiling | `react-native-best-practices` | Delegate if installed |
| New project, setup, styling, Lottie animations, diagnosis | — | Handle directly (don't delegate) |

**CRITICAL:** Even when delegating, this skill's library rules ALWAYS take precedence. If an extension skill suggests an avoided library (e.g., flash-list, nativewind, redux, async-storage), you MUST override with the approved alternative from [references/libraries.md](references/libraries.md). Do not follow the extension's library suggestion.

## Daily Commands

```bash
./scripts/meta-run <path>   # Build + run (auto-detects iOS/Android)
npx expo start --clear      # Dev server + clear cache (after config changes)
npx expo start              # Dev server only (JS changes, no new packages)
```

Pass `android` as second arg to force Android when both platforms available.
