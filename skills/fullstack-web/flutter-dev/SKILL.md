---
name: flutter-dev
description: Expert guidance for Flutter and Dart development. Use when building Flutter apps, implementing state management, setting up routing, writing tests, or working with Flutter UI components. Provides access to detailed rules for Bloc, Riverpod, Provider, Mocktail, and more.
---

# Flutter Development Skill

## Overview

This Skill provides modular Flutter development guidance with access to detailed documentation for specific libraries and patterns.

**Quick Start**: When user mentions a specific library (e.g., "use Bloc", "Riverpod", "GoRouter"), read the corresponding `./rules/*.md` file before responding.

---

## Module 1: State Management

### Core Principles
- Prefer built-in solutions for simple state (ValueNotifier, ChangeNotifier, StreamBuilder)
- Choose state management based on complexity and team familiarity
- Read detailed rules when using specific libraries

### Extended Rules Index

| Priority | Library | Rule File | Trigger Keywords | Notes |
|----------|---------|-----------|------------------|-------|
| 🔥 High | Bloc | `./rules/bloc.md` | "bloc", "cubit", "blocprovider" | Event-driven, enterprise scale |
| 🔥 High | Riverpod | `./rules/riverpod.md` | "riverpod", "provider 2.0" | Modern, type-safe |
| 🟡 Medium | Provider | `./rules/provider.md` | "provider" | Classic, widely used |
| 🟢 Low | ChangeNotifier | `./rules/flutter_change_notifier.md` | "changenotifier" | Built-in, simple |

**Usage**: Read the rule file when user mentions trigger keywords.

### Quick Reference

**For simple state**: Start with ChangeNotifier or ValueNotifier
**For complex apps**: Use Bloc or Riverpod
**When unsure**: Ask user about their preferences and app complexity

---

## Module 2: Testing

### Core Principles
- Write tests that can actually fail if real code is broken
- Use Mocktail for mocks (preferred over Mockito)
- Always use `group()` for test organization

### Extended Rules Index

| Priority | Topic | Rule File | Trigger Keywords | Notes |
|----------|-------|-----------|------------------|-------|
| 🔥 High | Mocktail | `./rules/mocktail.md` | "mocktail", "mock", "fake" | Preferred mocking lib |
| 🟡 Medium | Mockito | `./rules/mockito.md` | "mockito" | Legacy mocking lib |
| 🟡 Medium | Testing | `./rules/testing.md` | "test", "testing", "widget test" | General testing guide |

### Testing Checklist

```
Test Quality Checklist:
- [ ] Test can fail if real code is broken
- [ ] Tests are grouped with group()
- [ ] Test names use "should" format
- [ ] Mocks are used appropriately
```

---

## Module 3: Navigation & Routing

### Core Principles
- Use GoRouter for all navigation (declarative approach)
- Prefer named routes over hardcoded paths
- Centralize route definitions

### Extended Rules Index

| Priority | Topic | Rule File | Trigger Keywords | Notes |
|----------|-------|-----------|------------------|-------|
| 🔥 High | GoRouter | `./rules/navigation.md` | "router", "navigation", "gorouter", "route" | Declarative routing |

### Quick Reference

```dart
// Navigation actions
context.goNamed('profile')      // Switch screen
context.pushNamed('details')     // Stack screen
context.pop()                    // Go back
```

---

## Module 4: Code Quality & Standards

### Core Principles
- Follow Effective Dart guidelines
- Use `dart analyze` and `dart format` regularly
- Write self-documenting code with clear naming

### Extended Rules Index

| Priority | Topic | Rule File | Trigger Keywords | Notes |
|----------|-------|-----------|------------------|-------|
| 🔥 High | Effective Dart | `./rules/effective_dart.md` | "style", "naming", "format", "effective dart" | Style guide |
| 🔥 High | Dart 3 Updates | `./rules/dart_3_updates.md` | "dart 3", "patterns", "records", "class modifiers" | New features |
| 🟡 Medium | Code Review | `./rules/code_review.md` | "review", "code review" | Review guidelines |

### Naming Standards

| Type | Convention | Example |
|------|------------|---------|
| Classes | PascalCase | `UserRepository` |
| Variables/Functions | camelCase | `fetchUserData` |
| Files | snake_case | `user_repository.dart` |

### Code Quality Checklist

```
- [ ] Functions < 20 lines
- [ ] Single responsibility per function
- [ ] Descriptive names, no abbreviations
- [ ] Proper error handling with developer.log()
- [ ] Run: dart analyze && dart format .
```

---

## Module 5: Architecture & Data

### Core Principles
- Separate concerns: Presentation, Business Logic, Data layers
- Use repositories to abstract data sources
- Keep business logic out of UI widgets

### Extended Rules Index

| Priority | Topic | Rule File | Trigger Keywords | Notes |
|----------|-------|-----------|------------------|-------|
| 🔥 High | App Architecture | `./rules/flutter_app_architecture.md` | "architecture", "project structure", "layer" | Project organization |
| 🟡 Medium | JSON Serialization | `./rules/json_serialization.md` | "json", "serializable", "json_serializable" | Data parsing |
| 🟡 Medium | Error Handling | `./rules/flutter_errors.md` | "error", "exception", "handling errors" | Error patterns |

### Architecture Layers

```
┌─────────────────────────────────┐
│   Presentation Layer (UI)        │
│   - Widgets                      │
│   - Blocs/Cubits/Providers       │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│   Business Logic Layer          │
│   - Use cases / Services         │
│   - Business rules               │
└─────────────────────────────────┘
              ↓
┌─────────────────────────────────┐
│   Data Layer                     │
│   - Repositories                 │
│   - Data sources (API, DB)       │
└─────────────────────────────────┘
```

---

## Common Workflows

### Workflow: New Flutter Feature

```
Feature Development Progress:
- [ ] 1. Define data models (with json_serializable if needed)
- [ ] 2. Create repository/service layer
- [ ] 3. Implement business logic (use appropriate state management)
- [ ] 4. Build UI widgets (break into small components)
- [ ] 5. Write tests (unit → widget → integration)
- [ ] 6. Run: dart analyze && dart format
```

**When user starts a new feature**: Reference this workflow and ask which step they're on.

### Workflow: Choosing State Management

```
State Management Decision Tree:
1. Is state simple and local? → ValueNotifier / ChangeNotifier
2. Is state complex/shared? → Bloc / Riverpod / Provider
3. Does user/event tracking matter? → Bloc
4. Is type safety a priority? → Riverpod
5. What does the team prefer? → Ask user
```

### Workflow: Setting Up Navigation

```
GoRouter Setup Checklist:
- [ ] 1. Add go_router dependency
- [ ] 2. Create app_router.dart with route definitions
- [ ] 3. Use MaterialApp.router in main.dart
- [ ] 4. Define routes with GoRoute (path, name, builder)
- [ ] 5. Implement guards/redirect if needed
- [ ] 6. Test navigation flows
```

---

## Anti-patterns to Avoid

| Anti-pattern | Why Bad | Correct Approach |
|--------------|--------|------------------|
| Network calls in `build()` | Blocks UI, causes repeated calls | Use initState, FutureBuilder, or state management |
| `setState()` in StatelessWidget | Compile error | Use StatefulWidget or state management |
| Ignoring null safety (`!` without check) | Runtime crashes | Use nullable types or verify with `?.` |
| Deep widget trees | Hard to read, maintain | Extract widgets into smaller components |
| Using `print()` for logging | Not configurable, no context | Use `dart:developer` log() |
| Direct bloc-to-bloc communication | Tight coupling | Use shared repository or listener pattern |

---

## Troubleshooting Guide

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `LateInitializationError` | Variable used before init | Initialize in constructor or make nullable |
| `RenderFlex overflowed` | Content exceeds bounds | Wrap in SingleChildScrollView or use ListView |
| `setState() called after dispose()` | Async callback after widget unmounted | Check: `if (mounted) setState(...)` |

### Debugging Commands

```bash
# Analyze code
dart analyze

# Format code
dart format .

# Run tests
flutter test

# Check dependencies
flutter pub deps
```

---

## Quick Command Reference

```bash
# Project creation
flutter create my_app

# Dependency management
flutter pub add <package>
flutter pub remove <package>
flutter pub upgrade

# Code quality
dart analyze
dart format .
dart fix --apply

# Testing
flutter test
flutter test --coverage
```

---

## References

- [evanca/flutter-ai-rules](https://github.com/evanca/flutter-ai-rules) - Source of these rules
- [Flutter Documentation](https://docs.flutter.dev/)
- [Dart Language Guide](https://dart.dev/guides)
