---
name: swiftui-components
description: SwiftUI component expert for building reusable views, custom modifiers, view compositions, and Liquid Glass integration. Use when creating new SwiftUI views, refactoring UI code, applying design tokens, or building production-ready component libraries for macOS Tahoe (26) and iOS 26.
---

# SwiftUI Components

## Lifecycle Position

Phase 3 (Implement). After design from `macos-app-design`. Use templates and patterns during implementation. Next: `swiftui-view-refactor` for Phase 5 review.

## Quick Start

For standard patterns, see [PATTERNS.md](references/PATTERNS.md).
For API reference, see [REFERENCE.md](references/REFERENCE.md).

## Instructions

When creating or refactoring SwiftUI components:

1. **Analyze the required component functionality**
   - What data does the component display?
   - What interactions does it support?
   - Does it need Liquid Glass effects?

2. **Check existing components for reuse**
   - Grep the project for similar views before creating new ones
   - Check if a custom `ViewModifier` already covers the styling need
   - Look for shared `PreferenceKey` definitions

3. **Apply templates from `templates/` directory**
   - `view-template.swift` — Standard view with @Observable ViewModel, .task lifecycle, accessibility
   - `viewmodel-template.swift` — @Observable @MainActor ViewModel with repository DI pattern
   - `modifier-template.swift` — Custom ViewModifier with View extension and conditional application
   - `glass-component-template.swift` — Liquid Glass component with GlassEffectContainer, glassEffectID, morph transitions, backgroundExtensionEffect, and toolbar grouping
   - `templates/onboarding/` — Complete paged onboarding flow with persistence and view modifier
   - `templates/settings/` — Cross-platform settings screen (iOS + macOS) with `@Observable` model

4. **Follow project styling conventions**
   - Use semantic colors (`.primary`, `.secondary`, `.tertiary`) over hardcoded colors
   - Prefer `.font(.body)` system text styles over fixed font sizes
   - Use `RoundedRectangle(cornerRadius:)` and `ConcentricRectangle` for shapes
   - Apply `.accessibilityLabel` to all icon-only buttons
   - Use `.task {}` instead of `.onAppear { Task { } }` for async work

## Component Checklist

Before submitting a new component, verify:

- [ ] View is under 100 lines (extract subviews if larger)
- [ ] `@Observable` used for view models (not `ObservableObject`)
- [ ] `@MainActor` on view model class
- [ ] Repository/service dependencies injected via init (not hardcoded)
- [ ] `.task {}` used for async loading (auto-cancels on disappear)
- [ ] `accessibilityLabel` on icon-only buttons and decorative images
- [ ] Dynamic Type supported (no hardcoded font sizes)
- [ ] Dark mode tested (use semantic colors)
- [ ] Reduce Transparency and Reduce Motion tested for glass effects
- [ ] `[weak self]` in stored closures and Timer callbacks
- [ ] `Sendable` conformance verified for types crossing isolation boundaries
- [ ] Preview provided with `#Preview { }` macro

## Liquid Glass Guidelines

When applying Liquid Glass to custom components:

1. **Use sparingly** — only on the most important functional elements
2. **Always wrap in `GlassEffectContainer`** — required for performance and morph animations
3. **Assign `glassEffectID`** when glass views should morph during transitions
4. **Apply `.backgroundExtensionEffect()`** to hero images in split views
5. **Group toolbar items** with `ToolbarSpacer(.fixed)` for separate glass backgrounds
6. **Remove custom backgrounds** on toolbars, tab bars, and navigation bars — they block glass
7. **Use `.buttonStyle(.glass)`** instead of custom glass effects on buttons
8. **Test with accessibility** — Reduce Transparency removes glass; ensure fallback looks good

## Templates

### Base Templates (`templates/`)
- **`view-template.swift`** — Standard view with @Observable ViewModel, .task lifecycle, accessibility
- **`viewmodel-template.swift`** — @Observable @MainActor ViewModel with repository DI pattern
- **`modifier-template.swift`** — Custom ViewModifier with View extension and conditional application
- **`glass-component-template.swift`** — Liquid Glass component with GlassEffectContainer, glassEffectID, morph transitions, backgroundExtensionEffect, and toolbar grouping

### Onboarding Flow (`templates/onboarding/`)
Complete paged onboarding with persistence and view modifier:
- **`OnboardingView.swift`** — TabView-based paged flow with page indicators
- **`OnboardingPage.swift`** — Data model for onboarding page content
- **`OnboardingPageView.swift`** — Individual page layout (icon, title, description)
- **`OnboardingStorage.swift`** — `@AppStorage`-backed completion tracking
- **`OnboardingModifier.swift`** — `.onboarding()` view modifier for one-time display

### Settings Screen (`templates/settings/`)
Cross-platform settings with iOS + macOS support:
- **`SettingsView.swift`** — Root settings view with platform-adaptive layout
- **`AppSettings.swift`** — `@Observable` settings model with persistence
- **`SettingsRow.swift`** — Reusable row component for settings sections
- **`AboutSettingsView.swift`** — App info, version, credits
- **`AccountSettingsView.swift`** — Account management section

## File Structure

```
skills/swiftui-components/
├── SKILL.md              ← You are here (entry point)
├── references/           ← Deep documentation by category (scraped from Apple docs)
│   ├── PATTERNS.md       ← ViewModifier, ViewBuilder, PreferenceKey, Layout, composition patterns
│   ├── REFERENCE.md      ← API quick-reference tables for SwiftUI protocols, wrappers, and modifiers
│   ├── views_builtin.md
│   ├── modifiers_styling.md
│   ├── modifiers_interaction.md
│   ├── layout_system.md
│   ├── shapes_and_drawing.md
│   ├── animation_and_transitions.md
│   ├── data_flow.md
│   ├── presentations.md
│   └── index.md
└── templates/            ← Copy-paste-ready Swift templates
    ├── view-template.swift
    ├── viewmodel-template.swift
    ├── modifier-template.swift
    ├── glass-component-template.swift
    ├── onboarding/       ← Complete onboarding flow (5 files)
    │   ├── OnboardingView.swift
    │   ├── OnboardingPage.swift
    │   ├── OnboardingPageView.swift
    │   ├── OnboardingStorage.swift
    │   └── OnboardingModifier.swift
    └── settings/         ← Cross-platform settings screen (5 files)
        ├── SettingsView.swift
        ├── AppSettings.swift
        ├── SettingsRow.swift
        ├── AboutSettingsView.swift
        └── AccountSettingsView.swift
```

## Companion Skills & Agents

- **macos-architect** — Consult for architectural decisions (navigation patterns, multi-window, state management hierarchy)
- **swift-reviewer** — Run code reviews against Swift 6 concurrency, memory, Liquid Glass, and accessibility standards
- **apple-liquid-glass-design** — Deep reference for all Liquid Glass APIs and the Landmarks sample app

## Data Source

This skill was generated from 170+ Apple documentation pages via:

```bash
skill-seekers scrape --config configs/swiftui-components.json
```

To refresh with updated documentation, re-run the scrape.
