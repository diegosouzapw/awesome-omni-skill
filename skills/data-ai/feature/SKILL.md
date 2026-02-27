---
name: feature
description: Creates a new feature module with minimal viable structure. Use when bootstrapping a new feature from scratch, scaffolding the Tuist module, Container, Feature entry point, DeepLinkHandler, and initial screen with placeholder Text view. Includes all unit tests, mocks, stubs, and app integration. For adding domain/data layers afterward, use /datasource, /repository, /usecase. For enhancing views, use /view, /viewmodel, /navigator.
---

# Skill: Feature

Create a new feature module with the minimum viable structure: Tuist module, Feature entry point, Container, DeepLinkHandler, and one screen with placeholder `Text`. Integrates into the app.

## Parameters

Gather from user before starting:

| Parameter | Format | Example |
|-----------|--------|---------|
| Feature | PascalCase | `Episode` |
| Screen | PascalCase | `EpisodeList` |
| Deep link host | lowercase | `episode` |
| Deep link path segment | lowercase, no slash | `list` |

Derived: module name = `Challenge{Feature}`, event prefix = snake_case of Screen.

## File Structure

```
Features/{Feature}/
├── Sources/
│   ├── {Feature}Feature.swift
│   ├── {Feature}Container.swift
│   └── Presentation/
│       ├── Navigation/
│       │   ├── {Feature}IncomingNavigation.swift
│       │   └── {Feature}DeepLinkHandler.swift
│       └── {Screen}/
│           ├── Navigator/
│           │   ├── {Screen}NavigatorContract.swift
│           │   └── {Screen}Navigator.swift
│           ├── Tracker/
│           │   ├── {Screen}TrackerContract.swift
│           │   ├── {Screen}Tracker.swift
│           │   └── {Screen}Event.swift
│           ├── ViewModels/
│           │   ├── {Screen}ViewModelContract.swift
│           │   └── {Screen}ViewModel.swift
│           └── Views/
│               └── {Screen}View.swift
└── Tests/
    ├── Unit/
    │   ├── Feature/
    │   │   └── {Feature}FeatureTests.swift
    │   └── Presentation/
    │       ├── Navigation/
    │       │   └── {Feature}DeepLinkHandlerTests.swift
    │       └── {Screen}/
    │           ├── Navigator/
    │           │   └── {Screen}NavigatorTests.swift
    │           ├── Tracker/
    │           │   ├── {Screen}TrackerTests.swift
    │           │   └── {Screen}EventTests.swift
    │           └── ViewModels/
    │               └── {Screen}ViewModelTests.swift
    └── Shared/
        ├── Mocks/
        │   ├── {Screen}NavigatorMock.swift
        │   └── {Screen}TrackerMock.swift
        └── Stubs/
            └── {Screen}ViewModelStub.swift
```

## Conventions

- **No `@Observable`** on minimal ViewModels (no observable state). Only add when ViewModel has `private(set) var`.
- **No `any` keyword** on internal protocol types. Only on public protocols from other modules (e.g., `any TrackerContract` in Container).
- **No imports** in ViewModel when all types are internal to the module.
- **Deep link paths** are scoped per host — `/list` under `episode` host is independent from `/list` under `character` host.
- **Deep links use path-based URLs** — parameters are embedded in the path (e.g., `challenge://character/detail/42`), never as query items (`?id=42`). Use `url.pathComponents` for parsing.
- Tuist module uses `\(appName)` string interpolation for target names.
- **Features always receive `HTTPClientContract`** as their network dependency — never specific clients like `GraphQLClientContract`. The Container is responsible for creating specific clients (e.g., `GraphQLClient`) internally from the `HTTPClientContract`. This keeps features decoupled from transport details.
- **Features that don't need networking** only receive `tracker: any TrackerContract`.
- **Features that need networking** receive `httpClient: any HTTPClientContract, tracker: any TrackerContract`.

## Workflow

### Step 1: Tuist Module

Create `Tuist/ProjectDescriptionHelpers/Modules/{Feature}Module.swift`:

```swift
import ProjectDescription

public let {feature}Module = Module.create(directory: "Features/{Feature}")
```

Register the module in **`Modules.swift`** — Add `{feature}Module` to the `Modules.all` array. This single registration automatically includes the module's package in the root project and its test target in the `Challenge.xctestplan`.

### Step 2: Source Files

Create all source files from [sources.md](references/sources.md).

Order: IncomingNavigation → DeepLinkHandler → NavigatorContract → Navigator → TrackerContract → Tracker → Event → ViewModelContract → ViewModel → View → Container → Feature.

### Step 3: Test Files

Create all test files from [tests.md](references/tests.md).

Order: Mocks → Stubs → Unit tests.

### Step 4: App Integration

Wire the feature into the app — 4 files to modify:

**`Tuist/ProjectDescriptionHelpers/Modules/AppKitModule.swift`** — Add dependency:
```swift
{feature}Module.targetDependency,
```

**`AppKit/Sources/AppContainer.swift`** — Three changes:
1. Add import: `import Challenge{Feature}`
2. Add property: `private let {feature}Feature: {Feature}Feature`
3. Initialize in `init`:
   - Without networking: `{feature}Feature = {Feature}Feature(tracker: self.tracker)`
   - With networking: `{feature}Feature = {Feature}Feature(httpClient: self.httpClient, tracker: self.tracker)`
4. Add to `features` array

**`AppKit/Tests/Unit/AppContainerTests.swift`** — No changes needed (`features` is private, tested indirectly).

### Step 5: Verify

```bash
xcodebuild test \
  -workspace Challenge.xcworkspace \
  -scheme "Challenge (Dev)" \
  -testPlan Challenge \
  -destination 'platform=iOS Simulator,name=iPhone 17 Pro,OS=latest'
```

## Extending the Feature

| Need | Skill |
|------|-------|
| REST API data source | `/datasource` |
| Repository + DTO mapping | `/repository` |
| Business logic | `/usecase` |
| Enhance ViewModel with state | `/viewmodel` |
| Enhance View with design system | `/view` |
| Add more navigation | `/navigator` |
| Snapshot tests | `/snapshot` |
| UI tests | `/ui-tests` |
