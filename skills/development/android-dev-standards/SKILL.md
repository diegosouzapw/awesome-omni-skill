---
name: android-dev-standards
description: Standards, architecture patterns, and best practices for Android app development with Kotlin, Jetpack Compose, and Android Jetpack libraries using clean architecture and MVVM. Use for any Android coding, review, refactor, or design task, especially when acting as an AI coding agent that must follow established project conventions.
---

# Android Development Standards

These standards apply to all Android application code unless a specific project document explicitly says otherwise. When in doubt, follow these conventions.

## 1. Technology and language

- Prefer Kotlin for all new Android code.
- Use Jetpack Compose for UI in new features. Use XML layouts only when:
  - Modifying existing XML based screens.
  - Integrating legacy components that are not Compose ready.
- Use Android Jetpack libraries where appropriate:
  - `ViewModel`, `LiveData` or `StateFlow`, `Navigation`, `Room`, `DataStore`, `WorkManager`.

When a module is already heavily based on Java or XML, follow the existing style in that module and avoid mixing paradigms without a clear and incremental migration plan.

## 2. Architecture

Follow clean architecture with clear separation of concerns.

- Use MVVM as the default presentation pattern.
- Structure code into layers:
  - `presentation` for UI and ViewModels.
  - `domain` for use cases and business logic.
  - `data` for repositories and data sources.
- ViewModels:
  - Expose UI state via `StateFlow`, `SharedFlow` or `MutableState` for Compose.
  - Keep Android framework types out of the domain layer.
- Repositories:
  - Hide data source and caching details from the rest of the app.
  - Expose suspend functions or reactive streams for consumption.

## 3. Project structure

Prefer a modular structure in larger apps.

- Split into modules such as:
  - `core` for shared utilities, design system, common models.
  - `feature-*` modules per screen or feature.
  - `data-*` modules per data source when complex enough.
- Avoid creating new modules for trivial features. Default to placing code in existing relevant modules.

Use consistent and meaningful package names:

- `com.company.app.feature.name`
- `com.company.app.core.ui`
- `com.company.app.data.api`

## 4. UI and Compose standards

- Maintain a single source of truth for UI state per screen.
- Write composable functions that:
  - Are small, focused and composable.
  - Receive all dynamic data and callbacks via parameters.
  - Do not access repositories, data sources or navigation directly.
- Use a central design system:
  - Centralise colours, typography, shapes and spacing in a `designsystem` or `core/ui` module.
  - Avoid hard coded values in composables. Use theme tokens instead.
- Model screen states explicitly, for example:
  - Loading
  - Content
  - Empty
  - Error
- Accessibility:
  - Provide `contentDescription` for important images and icons.
  - Ensure tap targets are large enough and layouts support dynamic font sizes.

## 5. Navigation

- Use the Navigation Component or Compose Navigation for in app navigation.
- Define routes or destinations in a single place per feature.
- Pass only minimal arguments between screens. Prefer IDs over full objects.
- Do not store `NavController` in ViewModels. Pass it from composables or use navigation events from ViewModels that are handled in the UI layer.

## 6. Data, networking and persistence

- Use a dedicated networking layer:
  - Prefer Retrofit with OkHttp for HTTP APIs when suitable.
  - Model responses with data classes and map them into domain models.
- Error handling:
  - Wrap remote calls in a result type, such as `Result<T>` or a sealed class.
  - Distinguish clearly between:
    - Network errors.
    - Server errors.
    - Business rule or validation errors.
- Persistence:
  - Use Room for structured data and DataStore for key value preferences.
  - Keep database entities separate from domain models. Map explicitly between layers.
- Do not block the main thread:
  - Use coroutines with `Dispatchers.IO` for IO and heavy work.
  - Ensure any expensive computation runs off the UI thread.

## 7. Dependency injection

- Use Hilt as the default dependency injection framework in new projects.
- Provide Hilt modules for:
  - Network clients.
  - Repositories.
  - Use cases and other core services.
- Do not manually construct dependencies inside ViewModels or composables.

## 8. Coroutines and concurrency

- Use structured concurrency:
  - Launch coroutines in appropriate scopes such as `viewModelScope` or `lifecycleScope`.
  - Ensure work is cancelled when the owning scope is cancelled.
- Use `suspend` functions for long running or IO operations.
- Avoid `GlobalScope` except for initialisation that must live for the entire process lifetime and is explicitly safe.

## 9. Testing

- Types of tests:
  - Unit tests for ViewModels, use cases and pure logic.
  - Instrumentation tests for persistence and integration with Android components.
  - UI tests using Compose testing APIs, or Espresso for legacy views.
- Testing rules:
  - Cover new ViewModel and use case logic with tests.
  - Mock external dependencies instead of calling real services.
  - Keep tests deterministic, isolated and fast.

## 10. Performance and reliability

- Avoid unnecessary recompositions in Compose:
  - Hoist state where possible and use stable data structures.
  - Use `remember` only where it avoids work and does not hide state.
- Use pagination for lists that can grow large.
- Use `WorkManager` for background work that must be guaranteed or persisted.
- Monitor startup time and memory usage. When optimisation is needed, measure first, then change code based on data.

## 11. Security and privacy

- Do not store secrets in source code or in the app binary.
- Use secure storage for sensitive user data according to platform guidance.
- Follow platform recommendations for permissions:
  - Request permissions as late as possible, near the feature that needs them.
  - Provide rationale dialogs when appropriate.
- When handling personal data, adhere to project specific privacy, compliance and regulatory requirements.

## 12. AI coding agent guidance

When acting as an AI coding agent on Android tasks:

- Respect existing project patterns and constraints before introducing new libraries or frameworks.
- When adding dependencies:
  - Justify why they are needed.
  - Update relevant Gradle files consistently, including version catalogues if present.
- Prefer small, incremental changes over large refactors. Preserve behaviour unless a change is explicitly requested.
- Clearly explain:
  - Breaking changes.
  - Migration steps.
  - Any required manual changes or configuration updates for the team.

