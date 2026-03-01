---
name: plan
description: Creates detailed implementation plan from validated research. Produces task breakdown with dependencies.
---

# Plan Skill

Creates detailed implementation plan from validated research.

---

## Purpose

The Plan skill transforms validated research into an actionable implementation plan:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         PLANNING FRAMEWORK                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌────────────┐  │
│  │  RESEARCH   │──▶│  ARCHITECT  │──▶│    TASK     │──▶│   VERIFY   │  │
│  │   INPUT     │   │  DECISIONS  │   │  BREAKDOWN  │   │    PLAN    │  │
│  └─────────────┘   └─────────────┘   └─────────────┘   └────────────┘  │
│        │                 │                  │                │         │
│        ▼                 ▼                  ▼                ▼         │
│   • Requirements    • Approach        • Atomic tasks    • Traceability│
│   • Codebase map    • Patterns        • Dependencies    • Completeness│
│   • Risks           • Components      • Sequence        • Feasibility │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Agent Compatibility

- AskUserQuestion: use the tool in Claude Code; in Codex CLI, ask the user directly and record the answer.
- OUTPUT_DIR: `.claude/output` for Claude Code, `.codex/output` for Codex CLI.

## Planning Phases

### Phase 1: Architectural Decisions

Before task breakdown, make key architectural decisions:

### Phase 1.5: MANDATORY Edge Case Review

**CRITICAL: This phase is BLOCKING. Cannot proceed to Phase 2 until all edge cases are clarified.**

Before proceeding to task breakdown:

1. **List all edge cases** discovered during planning:
   - Empty states
   - Error states
   - Boundary conditions
   - Null/zero values
   - Missing data scenarios

---

#### P-Checkpoints (Plan Questions)

**P1: Edge Case Behavior**
For each edge case without explicit PRD guidance:

```
AskUserQuestion(
  questions: [
    {
      question: "Edge case: '{scenario}'. What should happen?",
      header: "Edge Case",
      options: [
        { label: "Show empty state", description: "Display 'No data' or similar message" },
        { label: "Show default value", description: "Display '--' or 0" },
        { label: "Hide element", description: "Don't render the component at all" },
        { label: "Show error", description: "Display error message to user" }
      ],
      multiSelect: false
    }
  ]
)
```

**P2: Architecture Decision**
For each architectural choice with trade-offs:

```
AskUserQuestion(
  questions: [
    {
      question: "For '{component}', should we use '{Option A}' or '{Option B}'?",
      header: "Architecture",
      options: [
        { label: "Option A", description: "Benefits: X. Trade-off: Y" },
        { label: "Option B", description: "Benefits: Y. Trade-off: X" },
        { label: "Discuss further", description: "Need more context to decide" }
      ],
      multiSelect: false
    }
  ]
)
```

**P3: Scope Boundary**
When scope is unclear:

```
AskUserQuestion(
  questions: [
    {
      question: "Should '{feature X}' be included in this implementation?",
      header: "Scope",
      options: [
        { label: "Yes, include", description: "Add to current implementation scope" },
        { label: "No, defer", description: "Create follow-up ticket for later" },
        { label: "Partial", description: "Include basic version only, enhance later" }
      ],
      multiSelect: false
    }
  ]
)
```

---

2. **Document in plan:**
   ```markdown
   ## Edge Case Decisions (User Confirmed)
   | Checkpoint | Edge Case | User Decision | Date |
   |------------|-----------|---------------|------|
   | P1 | All items defective | Show "–" | 2024-01-01 |
   | P2 | State management | Use existing controller | 2024-01-01 |
   | P3 | Export feature | Defer to next sprint | 2024-01-01 |
   ```

**Rules:**
1. NEVER assume edge case behavior - ASK
2. NEVER proceed with unconfirmed edge cases
3. Document all user decisions with checkpoint ID
4. Each P-checkpoint MUST be resolved before Phase 2

**Plan Phase Complete Criteria:**
```
□ All P1 checkpoints resolved (all edge cases have defined behavior)
□ All P2 checkpoints resolved (architecture decisions made)
□ All P3 checkpoints resolved (scope boundaries clear)
```

---

```
Decision Framework:
├── Approach Selection
│   ├── What pattern to follow?
│   ├── Create new vs extend existing?
│   └── Which components to use?
│
├── Data Flow Design
│   ├── API → Repository → Service → Controller → UI
│   └── State management approach
│
├── UI Architecture
│   ├── Screen structure
│   ├── Widget decomposition
│   └── Navigation flow
│
└── Integration Points
    ├── Existing services to use
    ├── New services needed
    └── External dependencies
```

### Phase 2: Task Decomposition

Break implementation into atomic, sequential tasks:

#### Task Properties

```
Task {
  id: string              // T1, T2, T3...
  title: string           // Short description
  description: string     // Detailed description
  type: enum {
    model,               // Data models (Equatable + ReturnValue)
    api,                 // API client methods
    repository,          // Repository layer
    service,             // Service layer
    controller,          // StateNotifier controller
    state,               // State class
    screen,              // Screen widget
    widget,              // Reusable widget
    navigation,          // Routing
    test,                // Unit/widget tests
    integration          // Integration/cleanup
  }
  layer: enum {
    data,
    domain,
    application,
    presentation
  }
  files: string[]         // Files to create/modify
  requirements: string[]  // Requirement IDs this addresses
  dependencies: string[]  // Task IDs this depends on
  complexity: enum {
    trivial,             // < 30 min
    low,                 // 30 min - 1 hr
    medium,              // 1-2 hrs
    high                 // 2+ hrs
  }
  risks: string[]
  acceptanceCriteria: string[]
}
```

#### Task Ordering Rules

```
1. Foundation First
   - Models before services
   - Services before controllers
   - Controllers before screens

2. Data Layer → Domain → Application → Presentation
   - Response models (data/)
   - Domain models (domain/)
   - Services & mappers (application/)
   - Controllers & screens (presentation/)

3. Dependencies Respected
   - Task cannot start until dependencies complete
   - No circular dependencies

4. Test Adjacent
   - Unit tests with related code
   - Integration tests after feature complete
```

### Phase 3: File Planning

For each file to create/modify:

```
File Plan {
  path: string           // Full file path
  action: enum {
    create,              // New file
    modify,              // Existing file
    delete               // Remove file (rare)
  }
  purpose: string        // Why this file
  contentOutline: string // High-level structure
  patterns: string[]     // Patterns to apply (from AGENTS.md)
  references: string[]   // Similar files to reference
}
```

### Phase 4: Test Strategy

Define testing approach:

```
Test Strategy {
  unitTests: [
    {
      target: string,        // Class/function to test
      scenarios: string[],   // Test scenarios
      mocks: string[]        // Dependencies to mock
    }
  ],
  widgetTests: [
    {
      target: string,        // Widget to test
      interactions: string[] // User interactions to test
    }
  ],
  integrationTests: [
    {
      flow: string,          // User flow to test
      steps: string[]        // Test steps
    }
  ]
}
```

---

## Task Templates by Type

### Model Task (Equatable + ReturnValue)

```markdown
### Task: Create {ModelName} Response Model

**Type**: model | **Layer**: data | **Complexity**: low

**Files**:
- Create: `lib/src/features/{feature}/data/{model_name}_response.dart`

**Description**:
Create response model for {API endpoint} using Equatable + ReturnValue pattern.

**Implementation**:
```dart
class {ModelName}Response extends Equatable {
  // Properties from API

  const {ModelName}Response({...});

  factory {ModelName}Response.fromJson(Map<String, dynamic> json) {
    return {ModelName}Response(
      // Use ReturnValue for all fields
    );
  }

  Map<String, dynamic> toJson() => {...};

  {ModelName}Response copyWith({...}) => {...};

  @override
  List<Object?> get props => [...];
}
```

**Acceptance Criteria**:
- [ ] All API fields mapped
- [ ] Uses ReturnValue for JSON parsing
- [ ] Has copyWith method
- [ ] Has props for Equatable
```

### Controller Task (StateNotifier)

```markdown
### Task: Create {Feature}Controller

**Type**: controller | **Layer**: presentation | **Complexity**: medium

**Files**:
- Create: `lib/src/features/{feature}/presentation/{name}/{name}_controller.dart`
- Create: `lib/src/features/{feature}/presentation/{name}/{name}_state.dart`

**Description**:
Create StateNotifier controller for {feature} screen.

**Implementation Pattern**:
```dart
// State
class {Feature}State {
  final AsyncValue<Data> dataValue;
  // other state properties

  const {Feature}State({...});

  {Feature}State copyWith({...}) => {...};
}

// Controller
class {Feature}Controller extends StateNotifier<{Feature}State> {
  {Feature}Controller({required this.service}) : super(const {Feature}State());

  final {Feature}Service service;

  Future<void> loadData() async {
    state = state.copyWith(dataValue: const AsyncLoading());
    final result = await service.getData();
    result.when(
      success: (data) => state = state.copyWith(dataValue: AsyncData(data)),
      failure: (error) => state = state.copyWith(errorMessage: error.message),
    );
  }
}

// Provider
final {feature}ControllerProvider = StateNotifierProvider<{Feature}Controller, {Feature}State>((ref) {
  return {Feature}Controller(service: ref.read({feature}ServiceProvider));
});
```

**Acceptance Criteria**:
- [ ] Follows StateNotifier pattern
- [ ] Has loading/error/success states
- [ ] Provider properly defined
```

### Screen Task

```markdown
### Task: Create {Feature}Screen

**Type**: screen | **Layer**: presentation | **Complexity**: medium

**Files**:
- Create: `lib/src/features/{feature}/presentation/{name}/{name}_screen.dart`

**Description**:
Create screen widget for {feature}.

**Implementation Pattern**:
```dart
class {Feature}Screen extends ConsumerWidget {
  const {Feature}Screen({super.key});

  static const routeName = '/{feature}';

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch({feature}ControllerProvider);

    return Scaffold(
      appBar: AppBarWidget(...),
      body: state.dataValue.when(
        data: (data) => _buildContent(data),
        loading: () => const LoadingWidget(),
        error: (e, _) => ErrorWidget(message: e.toString()),
      ),
    );
  }
}
```

**UI Components** (separate widget classes):
- {Feature}Header
- {Feature}Content
- {Feature}Footer

**Acceptance Criteria**:
- [ ] Uses ConsumerWidget
- [ ] Handles loading/error/data states
- [ ] Uses project styling (TypographyTheme, ColorApp, Gap)
- [ ] Separate widget classes (no _buildX methods)
```

---

## Output Template

Generate `OUTPUT_DIR/plan-{feature}.md`:

```markdown
# Implementation Plan: {Feature Name}

## Metadata
- **Date**: {YYYY-MM-DD}
- **Based On**: research-{feature}.md
- **PRD Reference**: {URL/source}
- **Estimated Tasks**: {count}
- **Complexity**: {low/medium/high}

---

## 1. Executive Summary

### Feature Overview
{Brief description of what will be implemented}

### Scope
- **In Scope**: {what's included}
- **Out of Scope**: {what's not included}

### Key Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| {decision} | {choice} | {why} |

---

## 2. Architecture

### 2.1 Component Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    PRESENTATION                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Screen    │  │  Controller │  │    State    │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
├─────────────────────────────────────────────────────────┤
│                    APPLICATION                           │
│  ┌─────────────┐  ┌─────────────┐                       │
│  │   Service   │  │   Mapper    │                       │
│  └─────────────┘  └─────────────┘                       │
├─────────────────────────────────────────────────────────┤
│                      DOMAIN                              │
│  ┌─────────────┐                                        │
│  │    Model    │                                        │
│  └─────────────┘                                        │
├─────────────────────────────────────────────────────────┤
│                       DATA                               │
│  ┌─────────────┐  ┌─────────────┐                       │
│  │  Response   │  │ Repository  │                       │
│  └─────────────┘  └─────────────┘                       │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Data Flow

```
API Response
    ↓
{Feature}Response (data/)
    ↓
{Feature}Model (domain/) [via Mapper]
    ↓
{Feature}Service (application/)
    ↓
{Feature}Controller (presentation/)
    ↓
{Feature}Screen (presentation/)
```

### 2.3 State Management

```dart
// State structure
{Feature}State {
  dataValue: AsyncValue<{Model}>
  // other fields
}
```

---

## 3. Task Breakdown

### 3.1 Task Summary

| ID | Task | Type | Layer | Complexity | Dependencies |
|----|------|------|-------|------------|--------------|
| T1 | ... | model | data | low | - |
| T2 | ... | service | application | medium | T1 |

### 3.2 Task Sequence

```
Phase 1: Data Layer
├── T1: Create response models
└── T2: Create/update repository

Phase 2: Domain Layer
└── T3: Create domain models

Phase 3: Application Layer
├── T4: Create mapper
└── T5: Create service

Phase 4: Presentation Layer
├── T6: Create state
├── T7: Create controller
├── T8: Create widgets
└── T9: Create screen

Phase 5: Integration
├── T10: Add navigation
├── T11: Add tests
└── T12: Integration testing
```

### 3.3 Detailed Tasks

---

#### T1: {Task Title}

**Type**: {type} | **Layer**: {layer} | **Complexity**: {complexity}

**Dependencies**: {none or task IDs}

**Requirements Addressed**: R1, R2

**Files**:
| Action | Path |
|--------|------|
| Create | `lib/src/features/{feature}/...` |

**Description**:
{Detailed description of what to implement}

**Implementation Notes**:
- {specific guidance}
- {patterns to follow}

**Acceptance Criteria**:
- [ ] {criterion 1}
- [ ] {criterion 2}

---

#### T2: {Next Task}
...

---

## 4. File Inventory

### 4.1 New Files

| Path | Purpose | Template |
|------|---------|----------|
| `lib/src/features/{feature}/data/{name}_response.dart` | API response | Equatable |
| `lib/src/features/{feature}/domain/{name}_model.dart` | Domain model | Equatable |
| `lib/src/features/{feature}/application/{name}_service.dart` | Business logic | - |
| `lib/src/features/{feature}/presentation/{name}/{name}_screen.dart` | UI | ConsumerWidget |
| `lib/src/features/{feature}/presentation/{name}/{name}_controller.dart` | State mgmt | StateNotifier |
| `lib/src/features/{feature}/presentation/{name}/{name}_state.dart` | State class | copyWith |

### 4.2 Modified Files

| Path | Changes |
|------|---------|
| `lib/src/routing/app_router.dart` | Add route |
| `lib/src/services/remote/api/{api}_client.dart` | Add endpoint |

### 4.3 Reference Files

| Path | Why Reference |
|------|---------------|
| `lib/src/features/{similar}/...` | Similar pattern |

---

## 5. Test Strategy

### 5.1 Unit Tests

| Target | Test File | Scenarios |
|--------|-----------|-----------|
| {Service} | `test/.../service_test.dart` | success, error, edge cases |
| {Controller} | `test/.../controller_test.dart` | state transitions |

### 5.2 Widget Tests

| Target | Scenarios |
|--------|-----------|
| {Screen} | loading, data display, error, interactions |

### 5.3 Integration Tests

| Flow | Steps |
|------|-------|
| {User flow} | {step sequence} |

---

## 6. Requirement Traceability

| Requirement | Tasks | Status |
|-------------|-------|--------|
| R1: {desc} | T1, T2, T5 | Planned |
| R2: {desc} | T3, T6 | Planned |

---

## 7. Risk Mitigation

| Risk | Mitigation | Tasks Affected |
|------|------------|----------------|
| {risk} | {mitigation} | T1, T2 |

---

## 8. Checklist Before Implementation

- [ ] Research validated (audit passed)
- [ ] All requirements traced to tasks
- [ ] Dependencies are clear
- [ ] File paths verified (no conflicts)
- [ ] Patterns match AGENTS.md
- [ ] User approved plan
```

---

## Prompt

When user invokes `/plan`, execute:

```
I will now create an implementation plan from the validated research.

## Prerequisites Check

1. Checking for research file: research-{feature}.md
2. Verifying research was audited and passed

## Phase 1: Architectural Decisions

Making key decisions:
- Pattern approach: {decision}
- Data flow: {decision}
- Component structure: {decision}

## Phase 2: Task Decomposition

Breaking down implementation into atomic tasks...

[Generate task list with dependencies]

## Phase 3: File Planning

Identifying files to create/modify:
- New files: {count}
- Modified files: {count}

## Phase 4: Test Strategy

Defining test approach:
- Unit tests: {count}
- Widget tests: {count}
- Integration tests: {count}

## Output

[Generate plan-{feature}.md]

## Summary

| Metric | Value |
|--------|-------|
| Total Tasks | {count} |
| Complexity | {level} |
| New Files | {count} |
| Modified Files | {count} |

Ready for plan audit? [Proceeding to /audit plan]
```

---

## Progress Tracking (MANDATORY when called from RPI)

**If this skill is invoked as part of an RPI workflow, you MUST update progress:**

### On Plan Start
```bash
~/.claude/skills/scripts/rpi-progress.sh --phase plan --status in_progress --last "Starting planning" --next "Complete implementation plan"
```

### On Plan Complete (before audit)
```bash
~/.claude/skills/scripts/rpi-progress.sh --phase plan --status complete --last "Plan complete" --next "Plan audit"
```

### After Plan Audit Pass
```bash
~/.claude/skills/scripts/rpi-progress.sh --audit plan --passed true --score {score} --last "Plan audit passed" --next "Await user approval"
```

### Progress Values
- Plan started: 20%
- Plan complete: 25%
- Plan audit pass: 30%
- User approval: 35%
