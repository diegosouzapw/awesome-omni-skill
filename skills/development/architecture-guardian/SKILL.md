---
name: architecture-guardian
description: Review architecture decisions and code changes against documented architecture principles. Use this skill before implementing new features or refactoring existing code to ensure compliance with layer boundaries, separation of concerns, and design patterns. Should be invoked when user requests "architecture review" or before making significant changes.
---

# Architecture Guardian

## Purpose

Act as the **architect** in the engineer-architect review process. Before writing code, review the proposed changes against the documented architecture in `docs/ARCHITECTURE.md` to ensure:
- Layer boundaries are respected
- Separation of concerns is maintained
- Existing patterns are followed
- New patterns align with architectural principles

## When to Invoke

### Proactive (Before Coding)
- User requests new feature implementation
- User asks for architectural review
- Planning refactoring of existing systems
- Adding new subsystems or components
- Modifying cross-cutting concerns

### Reactive (Before Committing)
- After implementing significant changes
- Before creating pull requests
- When user says "review architecture" or "check compliance"

## Workflow

### Phase 1: Understand the Intent

**Ask clarifying questions:**
1. What is the goal of this change? (feature, bug fix, refactor)
2. Which components will be affected?
3. What new code will be written? (classes, functions, files)
4. Are there existing patterns that should be followed?
5. What layer should this logic live in? (UI, Service, Middleware, Engine, Data)

### Phase 2: Review Architecture Documentation

**Read relevant sections:**
1. `docs/ARCHITECTURE.md` - Core principles and layer descriptions
2. `docs/ARCHITECTURE.md` sections 6-7 - Service Layer and Middleware patterns
3. `docs/ARCHITECTURE.md` "Architecture Evolution" - Recent refactorings and rationale
4. `docs/DESIGN_PATTERNS.md` - Comprehensive catalog of all design patterns used in codebase
5. `.claude/CLAUDE.md` - Development standards

**Identify applicable patterns (see `docs/DESIGN_PATTERNS.md` for details):**
- Is this a combat action? → Must use middleware (Chain of Responsibility pattern)
- Is this context assembly? → Must use service layer (Builder pattern)
- Is this AI decision? → Must be in systems/ai/ (Strategy pattern)
- Is this game state? → Must be in game engine, not UI
- Is this object creation? → Use Factory pattern (CharacterFactory, LLMProviderFactory)
- Is this cross-system communication? → Use Observer pattern (EventBus)
- Is this content/data? → Use Registry pattern (DataLoader) with JSON files
- Is this time-based effect? → Use TimeManager
- Is this resource tracking? → Use ResourcePool system

### Phase 3: Evaluate Proposed Approach

**Check against architectural rules:**

#### Rule 1: Layer Boundaries (CRITICAL)
```
❌ VIOLATIONS:
- UI layer contains game logic (combat resolution, dice rolling, AI decisions)
- Game engine imports from UI layer
- Service layer contains presentation logic
- Data layer contains business logic

✅ CORRECT:
- UI → Service → Engine → Data (dependency flow)
- Each layer only depends on layers below it
- No circular dependencies
```

#### Rule 2: Combat Action Pattern (REQUIRED)
```
❌ VIOLATIONS:
- Direct validation in action handlers (if not in_combat, if not is_turn)
- Manual resource refunding on failure
- Inconsistent action consumption
- Missing logging

✅ CORRECT:
- All combat actions use CombatActionExecutor
- Middleware handles validation, logging, cleanup
- Action handlers focus only on action logic
- Resources auto-refund on failure
```

#### Rule 3: Context Assembly Pattern (REQUIRED)
```
❌ VIOLATIONS:
- CLI queries data_loader directly (items_data, monsters_data, races_data)
- Manual assembly of attack/spell context (60+ lines)
- Repeated data queries across handlers

✅ CORRECT:
- Use CombatContextService.get_attack_parameters()
- Use CombatContextService.get_combatant_context()
- Use CombatContextService.build_attack_context()
- Context assembly in one reusable place
```

#### Rule 4: AI and Game State (REQUIRED)
```
❌ VIOLATIONS:
- AI decisions in CLI (target selection, condition removal)
- Combat history stored in CLI
- Enemy numbering in CLI instance variables
- Battlefield state built in UI layer

✅ CORRECT:
- AI decisions in systems/ai/
- Combat history in GameState
- Enemy numbering in InitiativeTracker/CombatEngine
- Battlefield state from CombatEngine.get_battlefield_state()
```

#### Rule 5: Separation of Concerns (FUNDAMENTAL)
```
✅ RESPONSIBILITIES BY LAYER:
- UI Layer: Display, input, formatting
- Service Layer: Data assembly, context building
- Middleware Layer: Validation, cross-cutting concerns
- Game Engine: Mechanics, state, deterministic rules
- Data Layer: JSON content, no logic
```

### Phase 4: Provide Recommendations

**Output format:**

```markdown
# Architecture Review: [Feature/Change Name]

## Intent
[What the user wants to accomplish]

## Proposed Approach
[High-level description of the planned implementation]

## Architecture Analysis

### ✅ Compliant Aspects
- [What follows architecture correctly]
- [Existing patterns being used properly]

### ⚠️ Concerns
- [Potential violations or anti-patterns]
- [Areas that need adjustment]

### ❌ Violations
- [Clear architecture violations]
- [Specific rules being broken]

## Recommendations

### 1. [Recommendation Title]
**Issue**: [What's wrong with current approach]
**Fix**: [How to align with architecture]
**Pattern**: [Which documented pattern to use]
**Example**: [Code snippet or reference]

### 2. [Next Recommendation]
[Same structure]

## Decision

**APPROVED** ✅ - Proceed with implementation (may have minor suggestions)
**APPROVED WITH CHANGES** ⚠️ - Proceed after addressing concerns
**REJECTED** ❌ - Requires redesign to meet architecture principles

## Next Steps
1. [Action item]
2. [Action item]
```

### Phase 5: Validate After Implementation (Optional)

If invoked after code is written:

1. **Read modified files** - Use git diff or direct file reads
2. **Check for patterns** - Search for anti-patterns
3. **Validate layer boundaries** - Ensure correct imports
4. **Report findings** - Same format as Phase 4

**Anti-pattern detection:**
```python
# Search for these in wrong places:
- "resolve_attack" in ui/ files → Game logic in UI
- "data_loader.load" in cli.py (not in service) → Direct data access
- "if not in_combat" in handle_* methods → Validation bypass
- "pool.current += 1" after failures → Manual resource refund
- "min(living_party, key=lambda" in ui/ → AI in UI
```

## Example Interactions

### Example 1: New Combat Action

**User**: "I want to add a 'Dash' action that lets characters move twice"

**Guardian Response**:
```
# Architecture Review: Dash Action

## Intent
Add Dash action (use action to double movement speed until end of turn)

## Proposed Approach
Add handle_dash() method to CLI that consumes an action and sets movement buff

## Architecture Analysis

### ✅ Compliant Aspects
- Adding to existing CLI command structure
- Will use action economy system

### ⚠️ Concerns
- Need to ensure middleware pattern is used
- Movement buff should be in game state, not CLI variable

### ❌ Violations
None identified yet

## Recommendations

### 1. Use Combat Middleware Pattern
**Issue**: New combat actions must go through validation pipeline
**Fix**: Use CombatActionExecutor with ActionType.ACTION
**Pattern**: See ARCHITECTURE.md Section 7 - Middleware Layer
**Example**:
```python
def handle_dash(self):
    context = self.action_executor.execute(
        actor=current_character,
        action_type=ActionType.ACTION,
        action_name="dash",
        action_handler=lambda ctx: self._execute_dash(ctx)
    )
```

### 2. Store Movement Buff in Game State
**Issue**: Movement modifiers are game state, not UI state
**Fix**: Add movement buff to Character or create timed effect
**Pattern**: Use TimeManager for temporary effects
**Example**: Similar to concentration spells

## Decision
**APPROVED WITH CHANGES** ⚠️

Follow middleware pattern and store state in game engine.

## Next Steps
1. Implement handle_dash() using middleware
2. Add _execute_dash() with buff logic
3. Store movement modifier in Character or TimeManager
4. Write unit tests for dash action
```

### Example 2: Context Assembly

**User**: "I need to show weapon details in the attack display"

**Guardian Response**:
```
# Architecture Review: Enhanced Attack Display

## Intent
Show weapon properties (damage type, properties) in combat output

## Architecture Analysis

### ⚠️ Concerns
- CLI already uses CombatContextService for attacks
- Need to extend AttackContext, not query data_loader in CLI

## Recommendations

### 1. Extend CombatContextService
**Issue**: Don't add data queries to CLI
**Fix**: Add weapon properties to AttackParameters dataclass
**Pattern**: Service layer handles all data assembly
**Example**:
```python
# In services/combat_context.py
@dataclass
class AttackParameters:
    # ... existing fields ...
    weapon_properties: List[str]  # NEW: ["finesse", "versatile"]

# Update get_attack_parameters() to populate this field
```

## Decision
**APPROVED** ✅

Extend service layer, don't add queries to UI.
```

## Key Architectural Principles to Enforce

From `docs/ARCHITECTURE.md` and `docs/DESIGN_PATTERNS.md`:

1. **Separation of Concerns**: Game rules, content, narrative, and UI are completely separated
2. **Data-Driven Design**: All content in JSON, not hardcoded (Registry pattern via DataLoader)
3. **Event-Driven Architecture**: Components communicate via event bus (Observer pattern)
4. **Extensibility**: Easy to add content, features, systems (Strategy, Factory patterns)
5. **Deterministic Core + Creative Enhancement**: Game engine is deterministic; LLM only enhances narrative
6. **Dependency Injection**: Make dependencies explicit and testable (constructor injection)
7. **Inversion of Control**: Systems don't control each other; control flows through events

## Common Violations to Watch For

1. **Game logic creeping into UI**
   - Combat resolution in CLI
   - AI decisions in UI layer
   - Dice rolling in presentation code

2. **Bypassing established patterns**
   - Combat actions without middleware
   - Context assembly without service layer
   - Direct data queries in UI

3. **State management confusion**
   - Game state in UI layer
   - Combat history in CLI
   - Display logic in game engine

4. **Breaking layer boundaries**
   - UI importing from UI (circular)
   - Engine importing from UI (upward dependency)
   - Service importing from UI (wrong direction)

## Success Criteria

A successful architecture review should:
- ✅ Identify layer boundary violations before implementation
- ✅ Guide engineer to correct pattern/layer
- ✅ Reference specific sections of ARCHITECTURE.md
- ✅ Provide concrete code examples
- ✅ Make a clear approve/reject decision
- ✅ Prevent technical debt from accumulating

The goal is **prevention, not just detection** - catch issues at design time, not commit time.
