---
name: android-ui-design-guide
description: Apply Android/Jetpack Compose design principles following Material Design 3 when building any Android UI component. Only execute this when the current project is an Android project and involves UI-related work. Use this skill for Compose layouts, Material components, or Android app development. Ensures Material You compliance with Dynamic Color, expressive theming, 4dp grid spacing, Roboto typography with Type Scale, and native Android patterns. Prevents common anti-patterns like hardcoded colors, Dark Mode neglect, and touch target violations.
---

# Android UI Design Guide

## Overview

Build native Android interfaces that follow Material Design 3 (Material You) by applying systematic design principles. This skill provides comprehensive guidelines for Jetpack Compose-first development with Dynamic Color, spacing, typography, and component-specific patterns optimized for Android.

## When to Use This Skill

Activate this skill when:
- Building Android UI with Jetpack Compose or XML
- Creating Android app screens, layouts, or components
- Working with Android development (Kotlin, Compose, Android Studio)
- Receiving requests like:
  - "Create a login screen in Compose"
  - "Design an Android settings view"
  - "Build a card layout for Android"
  - "Make this Android UI follow Material Design"
  - "Style this Compose screen"

**Do NOT activate** for:
- Web development
- iOS development
- Backend/server code
- Non-visual Android tasks

## Core Design Philosophy

Follow Material Design 3 principles + Flexible extensions:

### Material Design 3 Principles

1. **Expressive** - Dynamic Color from user wallpaper, personalized theming
2. **Adaptive** - Responsive layouts for phones, tablets, foldables
3. **Cohesive** - Consistent Material components and interactions

### Flexible Extensions

4. **Simplicity** - Remove unnecessary elements, focus on core features
5. **Accessibility** - 48x48dp touch targets, WCAG AA compliance, TalkBack support

## Framework Priorities

- **Primary**: Jetpack Compose (declarative, modern, recommended)
- **Secondary**: XML Views (legacy support)

## How to Use This Skill

### Step 1: Load Relevant Reference

```
Read references/design-principles.md - Material Design 3 principles
Read references/color-system.md - Dynamic Color, Material Theme
Read references/spacing-system.md - 4dp grid, touch targets
Read references/typography.md - Roboto, Material Type Scale
Read references/component-patterns.md - Compose component best practices
Read references/anti-patterns.md - Common Android design mistakes
```

### Step 2: Apply Component-Specific Patterns

- **Button**: `component-patterns.md` → Button section
- **Card**: `component-patterns.md` → Card section
- **List**: `component-patterns.md` → LazyColumn section
- **TextField**: `component-patterns.md` → TextField section
- **Navigation**: `component-patterns.md` → Navigation section
- **Dialog**: `component-patterns.md` → AlertDialog section

### Step 3: Validate Against Anti-Patterns

- ❌ Hardcoded colors (no Dark Mode)
- ❌ Material Theme ignored
- ❌ Touch targets smaller than 48x48dp
- ❌ Fixed text sizes (no Type Scale)
- ❌ 4dp grid violations
- ❌ Nested scrolling

### Step 4: Ensure System Consistency

**4dp grid system**:
- Use only: 4dp, 8dp, 12dp, 16dp, 24dp, 32dp, 48dp, 64dp
- Compose: `16.dp`, `.padding(16.dp)`

**Material Theme colors**:
- Labels: `colorScheme.onSurface`, `colorScheme.onSurfaceVariant`
- Backgrounds: `colorScheme.surface`, `colorScheme.surfaceVariant`
- Primary actions: `colorScheme.primary`
- Dynamic Color support (Android 12+)

**Material Type Scale** (REQUIRED):
- Display: `displayLarge`, `displayMedium`, `displaySmall`
- Headline: `headlineLarge`, `headlineMedium`, `headlineSmall`
- Title: `titleLarge`, `titleMedium`, `titleSmall`
- Body: `bodyLarge`, `bodyMedium`, `bodySmall`
- Label: `labelLarge`, `labelMedium`, `labelSmall`

## Resources

### references/

- **design-principles.md** - Material Design 3 principles (Expressive, Adaptive, Cohesive)
- **color-system.md** - Dynamic Color, Material Theme, dark mode
- **spacing-system.md** - 4dp grid, touch targets, Compose modifiers
- **typography.md** - Roboto, Material Type Scale, font weights
- **component-patterns.md** - Compose patterns for Button, Card, List, TextField, Navigation, Dialog
- **anti-patterns.md** - Common mistakes: hardcoded colors, touch targets, Material Theme neglect

## Quick Decision Tree

```
Android UI Component Request
│
├─ What component? → Load component-patterns.md section
│
├─ What spacing? → Use 4dp grid (spacing-system.md)
│
├─ What colors? → Material Theme + Dynamic Color (color-system.md)
│
├─ What typography? → Material Type Scale (typography.md)
│
├─ Compose or XML? → Compose first
│
└─ Validation → Check anti-patterns.md
```

## Examples

**Good Request Flow**:
```
User: "Create a login form in Compose"
→ Read references/component-patterns.md (TextField section)
→ Apply: OutlinedTextField, Material Type Scale, 4dp spacing
→ Validate against anti-patterns.md
→ Implement with Column { OutlinedTextField, Button }
```

**Component Implementation Checklist**:
- ✅ Spacing uses 4dp multiples
- ✅ Material Type Scale used (bodyMedium, titleLarge, etc.)
- ✅ Material Theme colors (auto Dark Mode)
- ✅ Touch targets minimum 48x48dp
- ✅ Material components used
- ✅ Dynamic Color support
- ✅ Accessibility (TalkBack contentDescription)

## Compose Code Examples

### ✅ Good: Material-Compliant Button

```kotlin
Button(onClick = { }) {
    Text("확인")
}
```

### ✅ Good: Material Theme Colors

```kotlin
Card {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "제목",
            style = MaterialTheme.typography.titleLarge,
            color = MaterialTheme.colorScheme.onSurface
        )
        Text(
            text = "내용",
            style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
    }
}
```

### ❌ Bad: Hardcoded Colors, No Type Scale

```kotlin
Text(
    text = "제목",
    fontSize = 24.sp,  // ❌ No Type Scale
    color = Color.Black  // ❌ No Dark Mode
)
```

## Platform-Specific Considerations

### Dynamic Color (Android 12+)
```kotlin
val colorScheme = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    dynamicLightColorScheme(LocalContext.current)
} else {
    lightColorScheme()
}
```

### Dark Mode (REQUIRED)
- Test both light and dark themes
- Use Material Theme colors
- Never hardcode `Color.Black`, `Color.White`

### Touch Targets
- Minimum 48x48dp for all interactive elements

## Reference Documentation

- [Material Design 3](https://m3.material.io/)
- [Jetpack Compose](https://developer.android.com/jetpack/compose)
- [Material Theme Builder](https://m3.material.io/theme-builder)
