---
name: android-agent-skills
description: Production-ready Agent Skills framework for Android Kotlin development. Provides Clean Architecture patterns, Jetpack Compose best practices, validation DSL, MVI state management, error handling, and AI-powered code generation. Use when building Android apps with quality standards, generating ViewModels, Repositories, UseCases, Compose screens, or writing pure Kotlin Agent Skills.
license: MIT
compatibility: Requires Kotlin 1.9+, Android Studio, AI assistant (GitHub Copilot, Claude, or similar)
metadata:
  author: TrongLB
  version: "2.2.0"
  updated: "2026-01"
  category: android-development
allowed-tools: Read Write Bash(git:*) Bash(./gradlew:*)
---

# Android Agent Skills

> A production-ready Agent Skill framework for Android Kotlin development with Clean Architecture, Jetpack Compose, and AI-powered code generation.

## When to use this skill

Use this skill when:
- Building Android apps with Kotlin
- Implementing Clean Architecture (ViewModel → UseCase → Repository)
- Creating Jetpack Compose UIs with proper stability
- Generating ViewModels, Repositories, UseCases, or Compose Screens
- Writing Agent Skills (pure Kotlin modules)
- Implementing validation, error handling, or state management
- Reviewing code for architecture compliance

## Quick Start

1. **Read context first**: [skills/AI_CONTEXT.md](skills/AI_CONTEXT.md)
2. **Use generation commands**: `@gen-viewmodel`, `@gen-skill`, `@gen-screen`
3. **Follow templates**: [skills/templates/](skills/templates/)
4. **Check quality**: `@analyze-skill`, `@review`

## Available Commands

### Code Generation
| Command | Description |
|---------|-------------|
| `@gen-viewmodel [name]` | Generate MVI ViewModel with StateFlow |
| `@gen-skill [name]` | Generate pure Kotlin Agent Skill |
| `@gen-screen [name]` | Generate Compose Screen (Route + Screen) |
| `@gen-repository [name]` | Generate Repository with offline-first |
| `@gen-usecase [name]` | Generate UseCase with Result<T> |
| `@gen-pipeline [name]` | Generate skill pipeline |
| `@gen-test [component]` | Generate tests (auto-detect type) |

### Analysis & Review
| Command | Description |
|---------|-------------|
| `@analyze-skill` | Analyze skill quality (aim for 90+) |
| `@review [file]` | Review code for rule compliance |
| `@review-changes` | Review git changes for violations |

### Quick Access
| Command | Description |
|---------|-------------|
| `/context` | Load AI_CONTEXT.md |
| `/summary` | Load AGENT_SUMMARY.md (core rules) |
| `/decision` | Pattern selection decision tree |
| `/validate` | Validation DSL patterns |
| `/help` | Show all commands (English) |
| `/help-vi` | Hiển thị trợ giúp (Tiếng Việt) |

## Critical Rules (Never Break)

```kotlin
// 1. ALWAYS rethrow CancellationException
catch (e: CancellationException) { throw e }

// 2. State: private mutable, public immutable
private val _state = MutableStateFlow(UiState())
val state: StateFlow<UiState> = _state.asStateFlow()

// 3. Events use Channel, NOT SharedFlow
private val _events = Channel<Event>()

// 4. Mark all state classes @Immutable
@Immutable data class UiState(...)

// 5. Architecture: ViewModel → UseCase → Repository
// NEVER: ViewModel → Repository directly
```

## Key References

| Need | Reference |
|------|-----------|
| Quick context | [skills/AI_CONTEXT.md](skills/AI_CONTEXT.md) |
| Core rules | [skills/AGENT_SUMMARY.md](skills/AGENT_SUMMARY.md) |
| Decision tree | [skills/guides/00-decision-tree.md](skills/guides/00-decision-tree.md) |
| All prompts | [skills/AI_PROMPTS.md](skills/AI_PROMPTS.md) |
| Validation | [VALIDATION_CHEAT_SHEET.md](VALIDATION_CHEAT_SHEET.md) |
| Error handling | [ERROR_HANDLING_CHEAT_SHEET.md](ERROR_HANDLING_CHEAT_SHEET.md) |
| Testing | [TESTING_CHEAT_SHEET.md](TESTING_CHEAT_SHEET.md) |
| Compose | [COMPOSE_CHEAT_SHEET.md](COMPOSE_CHEAT_SHEET.md) |

## Templates

| Component | Template Location |
|-----------|-------------------|
| Skill | [skills/templates/examples/DomainSkillGuide.kt](skills/templates/examples/DomainSkillGuide.kt) |
| ViewModel | [skills/templates/examples/UseCaseViewModelExample.kt](skills/templates/examples/UseCaseViewModelExample.kt) |
| Repository | [skills/templates/examples/UserRepositoryImpl.kt](skills/templates/examples/UserRepositoryImpl.kt) |
| Compose | [skills/templates/compose/ComposeExample.kt](skills/templates/compose/ComposeExample.kt) |
| Pipeline | [skills/templates/examples/PipelineExamples.kt](skills/templates/examples/PipelineExamples.kt) |
| Tests | [skills/templates/testing/](skills/templates/testing/) |

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    UI Layer                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │   Screen    │  │   Route     │  │  ViewModel  │  │
│  │ (Stateless) │◄─│ (Stateful)  │◄─│   (MVI)     │  │
│  └─────────────┘  └─────────────┘  └──────┬──────┘  │
└───────────────────────────────────────────┼─────────┘
                                            │
┌───────────────────────────────────────────┼─────────┐
│                 Domain Layer              │         │
│  ┌─────────────┐  ┌─────────────┐  ┌──────▼──────┐  │
│  │   Skill     │  │  UseCase    │◄─│  Interface  │  │
│  │ (Pure Kotlin)│  │ invoke()   │  │ Repository  │  │
│  └─────────────┘  └─────────────┘  └──────┬──────┘  │
└───────────────────────────────────────────┼─────────┘
                                            │
┌───────────────────────────────────────────┼─────────┐
│                  Data Layer               │         │
│  ┌─────────────┐  ┌─────────────┐  ┌──────▼──────┐  │
│  │   Remote    │  │   Local     │  │  Repository │  │
│  │ DataSource  │  │ DataSource  │◄─│    Impl     │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────┘
```

## Guides (35 available)

All guides in [skills/guides/](skills/guides/):

| Guide | Topic |
|-------|-------|
| 01-architecture.md | Clean Architecture |
| 02-coding-conventions.md | Kotlin style |
| 05-jetpack-compose.md | Compose basics |
| 07-state-management.md | MVI, StateFlow |
| 10-error-handling.md | Result pattern |
| 11-testing.md | Unit/UI tests |
| 24-validation-rules.md | Validation DSL |
| 30-skill-analysis.md | Skill quality |
| 31-pipeline-patterns.md | Skill pipelines |
| 32-modern-kotlin-features.md | Kotlin 2.0+, K2 compiler |
| 33-agent-feedback-loop.md | Feedback tracking, metrics |
| 34-logging-analytics.md | Timber, analytics, crash reporting |
| 35-gradle-optimization.md | Build optimization, convention plugins |

## Validation Quick Reference

```kotlin
// Top 5 validation functions
requireNotBlank(input.name, "name")
requireValidEmail(input.email, "email")
requireInRange(input.age, 13..150, "age")
requireNotEmpty(input.items, "items")
require(a < b) { "a must be < b" }
```

## File Structure

```
android-agent-skills/
├── SKILL.md                    # This file - entry point
├── skills/
│   ├── AI_CONTEXT.md           # Quick reference (read first)
│   ├── AGENT_SUMMARY.md        # Core rules & patterns
│   ├── AI_PROMPTS.md           # All @gen-* prompts
│   ├── guides/                 # 35 detailed guides
│   └── templates/              # Code templates & examples
├── *_CHEAT_SHEET.md            # Quick reference sheets
└── README.md                   # GitHub documentation
```
