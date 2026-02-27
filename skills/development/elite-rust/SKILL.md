---
name: elite-rust
description: Rust Elite Standards (Edition 2024, Safe & Robust)
---

# Rust Elite Standards (Edition 2024)

## Architecture
- **Edition**: Must use Rust Edition 2024.
- **Dependencies**: Use Standard Polars `0.52.x` (ensure patch version compatibility).

## Safety & Error Handling
- **No Panic Policy**: Strictly forbid `unwrap()` and `expect()` in business logic.
- **Typed Errors**: Use `thiserror` for library errors and `anyhow` for application-level errors.
- **Async**: Use `tokio` for async runtime unless specified otherwise.

## Testing & Quality (Atomic Simulator VAS)
- **TDD Requirement**: "No Test, No Commit". Every feature must have a "Red Phase" failing test before logic implementation.
- **Performance Benchmarks**: Core operations (e.g., Ledger queries) must meet sub-100ms targets for 1000 rows.
- **Verification**: Every Mission must conclude with a `walkthrough.md` documenting test results.

## Interoperability (Tauri/TS)
- **Data Contract**: All event payloads between Rust and TypeScript MUST use `camelCase`. Use `#[serde(rename_all = "camelCase")]` on structs.
- **Glass Panel Philosophy**: UI is strictly a "Glass Panel". Zero business logic, zero state inferencing, and zero data modification allowed in the Frontend. UI ONLY renders what the Core (Rust) dictates.
- **IPC Safety**: Use `#[tauri::arg(rename = "camelCase")]` or `#[allow(non_snake_case)]` to reconcile Rust's `snake_case` with Frontend's `camelCase` without breaking convention.

## Observability
- **Traceability**: Every result-bearing function must be traced or logged using the `tracing` crate. Use `#[tracing::instrument]` on core logic.

## Extraction (Unified Orchestrator)
- **Architecture**: Use the "Unified Extraction Orchestrator" pattern. Orchestrators must be stateless, purely gaging capability and dispatching to lanes.
- **Stability & Isolation**: Heavy FFI tasks (PDF/Office) MUST be isolated in sub-processes via `WorkerManager`.
- **Fate-sharing**: Workers must monitor `stdin`. If parent drops, worker must exit immediately to avoid "ghost processes".
- **Resource Governor**: Limit maximum concurrent workers using a `Semaphore`. Cap based on CPU core count or memory availability.
- **Contract Enforcement**: All lanes must speak the `ExtractionProduct` JSON protocol.

## Interoperability (IPC Protocol)
- **JSON Stream**: Communication between Main (Rust) and Worker (Python) must be conducted via JSON over `stdin/stdout`.
- **CamelCase Alignment**: All IPC payloads MUST be `camelCase`.
- **Timeout Policy**: Every worker task must have a hard timeout to prevent blocking the dispatcher.
- **Zero-Lag IPC**: Round-trip time (RTT) for IPC MUST NOT exceed 20ms. Use persistent worker pools to avoid cold-start overhead. Any refactoring that degrades IPC performance beyond this threshold MUST be rejected.

## Polars Integration (Mission 018+)

- **Version Lock**: Use Polars `0.52.x` for stability and API consistency.
- **DataFrame Construction**: 
  - Use `DataFrame::new(Vec<Column>)` (not `Vec<Series>`)
  - Convert `Series` → `Column` via `.into_column()` or `Column::from(series)`
- **Type Safety**: 
  - Column names must use `.into()` for `PlSmallStr` compatibility
  - Example: `Series::new((&col_name).into(), &values)`
  - Avoid `.unwrap()` on DataFrame operations — use `Result` propagation
- **Testing**: 
  - Every Polars operation must have a unit test with known input/output
  - Benchmark DataFrame creation < 50ms per table (per `LATENCY_BUDGET.md`)
- **Documentation**: 
  - Always check Polars docs for current version before implementation
  - Polars API changes frequently between minor versions

## Lessons Learned (Mission 018)

### ❌ What Went Wrong
1. **Ignored Skill Standards**: Used Polars `0.45` instead of `0.52.x` specified in skill
2. **No TDD**: Wrote implementation before tests, leading to trial-and-error debugging
3. **API Assumptions**: Guessed Polars API instead of reading docs, wasted time on type mismatches
4. **Missing Observability**: No `#[tracing::instrument]` on core functions

### ✅ What Was Fixed
1. **Version Alignment**: Upgraded to Polars `0.52` — clean build with no breaking changes
2. **Type Corrections**: 
   - `Vec<Series>` → `Vec<Column>` for DataFrame construction
   - Added `.into()` for `PlSmallStr` column names
3. **Dependency Management**: Added missing `chrono` for timestamp handling

### 📋 Process Improvements
- **Read Skill First**: Always check skill requirements before choosing dependencies
- **TDD Discipline**: Write failing test → implement → verify (Red-Green-Refactor)
- **Version Lock Early**: Pin exact versions in `Cargo.toml` to avoid API drift
- **Document Assumptions**: If deviating from skill, document why in commit message

## Refactor & Safety Audit (Elite Mandatory)

- **Unsafe Policy**
  - `unsafe` blocks are forbidden by default.
  - If unavoidable:
    - Must be isolated in a single module.
    - Must include SAFETY comments explaining invariants.
    - Must be reviewed and traced.

- **Refactor Discipline**
  - Eliminate all `unwrap()` / `expect()` (including tests & examples).
  - Replace with `Result<T, E>` and `?`.
  - No silent fallback.

- **Public API Contract**
  - Any public function that can fail MUST return `Result`.
  - Error types must be explicit at library boundaries.

- **Refactor Workflow**
  1. Analyze module responsibilities.
  2. Identify unsafe / panic / implicit failure.
  3. Propose refactor plan **before applying major logic changes**.
  4. Refactor module-by-module under `src/`.
  5. Update call sites, tests, examples, and documentation.
## Data Purity Protocol (The Janitor's Decree)

- **Architectural Separation**: Data cleaning (Janitor) is STRICTLY separated from data validation (TableTruth). 
- **Stateless & Pure**: The Janitor layer must be a pure transformer. It does not hold state or infer business logic.
- **Reporting Hierarchy**: `JanitorReport` is non-authoritative. It is an audit trail for the Dashboard, but MUST NEVER be used by the `Truth` layer to determine validity.
- **I/O Boundary Enforcement (Encoding)**: All external text (e.g., from PythonWorkers) must pass through an `EncodingGatekeeper` for UTF-8 and Mojibake validation *before* reaching the Janitor.
- **Ghost Rules**: 
    - Ejection of "Ghost Columns" is preferred for structural hallucinations.
    - Row ejection is only allowed for 100% empty rows with no semantic significance (e.g., header rows, spacer rows).
- **SIMD Usage**: Prefer standard library optimizations (Regex, Arrow) for SIMD. Avoid manual intrinsics unless explicitly authorized for a specific Mission.
- **Syntactic Cleaning Only**: Janitor cleans characters and formats (1.250,50 -> 1250.5). It NEVER performs semantic conversion (m3 -> liters). Parse failures are left "as-is" to be rejected by Truth.

## Iron Truth Contract & Clean Hands Doctrine (Mission 024-027)

- **LAW-07 (Fail Safe & Human-Gated)**: Systems MUST detect and reject anomalies (Mojibake, structural rot) but MUST NEVER attempt autonomous repair. All repairs are human-gated.
- **Clean Hands Doctrine**: The Truth Engine (`TableTruth`) must remain pure. It only validates data. Any repair logic resides in the `Adapter/Engine` layer and results must be re-submitted for validation. "The Judge does not write the Law, and the Janitor does not argue with the Truth."
- **Iron Truth Contract V1.0**:
  - `TableTruth` is the singular source of truth for structural validity.
  - `ProjectTruth` is the derived authority for cross-source reconciliation.
  - Any conflict between Reality and Truth results in `Rejected` status by default.
- **Global Singletons**: Use `std::sync::OnceLock` for global, thread-safe singletons (e.g., `EncodingNormalizer`). Avoid `lazy_static` unless complex macro execution is required.
- **UI Architecture**:
  - **4-Panel Arbiter Layout**: Mandatory for data-dense forensic tools.
  - **Virtualization**: Mandatory for tables > 100 rows to maintain sub-1s interactivity. Row height fixed at 32px for precision.
  - **Forbidden Patterns**: Non-deterministic loaders (spinners), softening of rejection language ("Check again" -> "TỪ CHỐI"), and autonomous "Fix" buttons are strictly forbidden.
  
## Project Intelligence & Lineage Protocol (Mission 030)

- **Truth Lineage**: Every aggregated metric (e.g., `total_cost`) MUST carry a `LineageMap`.
- **Shadow Columns**: Use a `_lineage_` prefix for forensic metadata columns in Polars. These columns are for sideboat metadata only and MUST NOT participate in primary business logic (sort/filter/calc).
- **Deterministic GlobalId**: `GlobalId` for any entity (cell, row, table) must be deterministic and hash-based: `hash(v1_id, v2_id...)`. Never use random UUIDs for forensic data.
- **Backward Compatibility**: When adding new fields to core data contracts (e.g., `TableCell`), use `#[serde(default)]` to ensure compatibility with existing serialized mocks and project files.
- **Polars String Handling (0.52)**: To check if a column name exists in `df.get_column_names()`, convert to a `Vec<String>` first or handle `PlSmallStr` types carefully to avoid `&str` vs `&&str` type mismatches in `.contains()`.
- **Forensic State Persistence**: Tauri backend `ForensicState` must act as the primary cache for derived truths (like `ProjectTruth`) to ensure consistency across independent IPC commands (Drill-down, Export).

## Tauri v2 & Serde Interop Protocol (RC-1 Legacy)

- **Trait Scoping (Emitter)**: 
  - To use `.emit()` on an AppHandle in Tauri v2, you MUST import the trait: `use tauri::Emitter;`. The `Manager` trait is no longer sufficient for event emission.
- **Serialization Mismatches ("The Camel Trap")**:
  - **Rule**: Tauri v2 often defaults to `camelCase` for serialized output.
  - **Risk**: Rust structs with `snake_case` fields (e.g., `cell_id`) may arrive at Frontend as `cellId` or `cell_id`, leading to `undefined` errors.
  - **Prescription**: 
    - Explicitly explicitly decorate structs with `#[serde(rename = "your_field_name")]` if the frontend expects a specific casing.
    - Or use `#[serde(rename_all = "camelCase")]` for the entire struct.
    - **Frontend Defense**: Always use optional chaining (e.g., `payload?.cell_id?.split(...)`) and verify field existence before operations.
- **Clean Hands Compilation**:
  - **Zero Tolerance**: `cargo check` warnings (unused imports, dead fields) are unacceptable in Release Candidates.
  - **Prefixing**: Unused fields in structs (required for FFI compatibility but not logic) MUST be prefixed with `_` (e.g., `_ctx`, `_start_time`).
  - **Comment Out**: If a function or import is truly unused, comment it out or remove it. Do not suppress warnings with `#[allow(...)]` unless it is a temporary bridge.
