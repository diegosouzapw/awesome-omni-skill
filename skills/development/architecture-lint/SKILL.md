---
name: architecture-lint
description: Detect clean architecture layer violations and check dependency rule violations. Verify dependency direction of events→facades→services→repository.
---

# Architecture Lint Skill

This skill detects code that violates clean architecture rules in the project.

## Detected Violations

### 1. Cross-Layer Direct Access
- **Direct calls from Events layer to Service/Repository layers** ❌
  - Events layer should only call Facade layer

- **Direct calls from Facade layer to Repository layer** ❌
  - Facade layer should access Repository layer through Service layer

- **Direct calls from Service layer to other Service layers** ❌
  - Service coordination should be done in Facade layer

### 2. Transaction Management Violations
- **Transaction start/commit/rollback in Service layer** ❌
  - `db.begin()`, `txn.commit()`, `txn.rollback()` only in Facade layer

- **Transaction start/commit/rollback in Repository layer** ❌
  - Repository layer only receives and uses transactions

### 3. Database Connection Violations
- **Creating individual DB connections in each layer** ❌
  - DB connections should be obtained from AppState and shared

## Detection Method

### Step 1: Identify files in each layer
```
src/events/**/*.rs       → Events layer
src/facades/**/*.rs      → Facade layer
src/services/**/*.rs     → Service layer
src/repository/**/*.rs   → Repository layer
```

### Step 2: Analyze use statements and method calls
- Search Events layer files for `use crate::services::` or `use crate::repository::`
- Search Facade layer files for `use crate::repository::`
- Search Service layer files for `.begin()`, `.commit()`, `.rollback()`

### Step 3: Report violations
For each violation:
- File path
- Line number
- Violation details
- Recommended fix

## Output Format

```markdown
## Clean Architecture Violation Report

### 🔴 Critical Violations (X items)

#### 1. Direct access from Events layer to Repository layer
- **File**: src/events/recruitment/join.rs:42
- **Violation**: `use crate::repository::recruitment::RecruitmentRepository;`
- **Recommendation**: Access through Facade layer

### 🟡 Warning-level Violations (Y items)

#### 2. Transaction management in Service layer
- **File**: src/services/quest/quest_service.rs:128
- **Violation**: `let txn = db.begin().await?;`
- **Recommendation**: Manage transactions in Facade layer, receive as argument in Service layer

### ✅ No Violations Detected

All layers are properly separated!
```

## Usage Example

```
User: Check architecture violations
Claude: [Executes architecture-lint skill]
```

## Implementation Guidelines

When using this skill, Claude should:

1. **Search for violation patterns with Grep tool**
   - Events layer: `use crate::(services|repository)::`
   - Facade layer: `use crate::repository::`
   - Service/Repository layers: `\.begin\(\)|\.commit\(\)|\.rollback\(\)`

2. **Analyze detection results**
   - Exclude legitimate uses (e.g., in test code)
   - Determine violation severity

3. **Generate fix suggestions**
   - Provide specific fix methods for each violation
   - Provide code examples if needed

## Notes

- Code within `#[cfg(test)]` is excluded from inspection
- `main.rs` and setup code treated as exceptions
- `use` statements in type/trait definitions don't count as violations (only actual calls)
