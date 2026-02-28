---
name: surrealdb-ffi-codec
description: A codec implementation pattern for high‑efficiency FFI data exchange between SurrealDB Embedded and Go/Swift/other languages, eliminating JSON by using FlatBuffers + MessagePack. Used for Rust FFI layer construction, SurrealDB query result conversion, and binary serialization.
---

# SurrealDB FFI Codec

## Overview

An efficient binary codec pattern for accessing SurrealDB Embedded from other languages (Go, Swift, Kotlin, etc.) via FFI.

**Core principle**: eliminate JSON and optimize with a two‑layer structure:
- **FlatBuffers**: fixed schema (error codes, status, operation types)
- **MessagePack**: dynamic schema (query results, parameters)

## When to Use

- Embedding SurrealDB in Rust and accessing it from other languages via FFI
- Eliminating JSON parse/serialize overhead
- Handling dynamic query results while preserving type safety
- Requiring zero‑copy access through FlatBuffers

## When NOT to Use

- HTTP/WebSocket client implementations (use official SurrealDB SDKs)
- Server-side API development without FFI requirements
- Simple single-language Rust projects (use surrealdb crate directly)
- MCP server implementations (this skill provides templates, not runtime)

**Important**: This skill provides implementation **templates and guidelines only**, not executable binaries or runtime libraries.

## Examples

**User request**: "Create an FFI layer to access SurrealDB from Go"

→ Generate Rust library with:
- FlatBuffers schemas from `resources/schemas/`
- Codec implementation from `resources/codec/`
- C ABI wrapper from `resources/ffi/`
- Go client code following `docs/integration_guide.md`

**User request**: "Convert SurrealDB query results to MessagePack"

→ Implement converter using `resources/codec/converter.rs.template` with proper TypeHints handling.

## Conversion Flow

See `docs/design_principle.md` for detailed architecture. Summary:

```
sql::Value → rmpv::Value → [u8] bytes (in FlatBuffers payload)
```

## Type Mapping

See `docs/conversion_table.md` for the complete type conversion reference.

Key points:
- `sql::Number::Decimal` → `f64` (precision loss beyond 15-17 digits)
- `sql::Datetime` → RFC3339 string (remove `d'...'` wrapper)
- `sql::Thing` → ID-only string (remove table prefix and brackets)

## Instructions

### Project Layout

Generate the following structure:

```
rust_lib/
├── Cargo.toml
├── schemas/
│   ├── request.fbs
│   └── response.fbs
└── src/
    ├── lib.rs
    ├── codec/
    │   ├── mod.rs
    │   ├── converter.rs
    │   ├── msgpack.rs
    │   └── generated/
    └── ffi/
        ├── mod.rs
        ├── wrapper.rs
        └── error_codes.rs
```

### Implementation

1. **FlatBuffers schemas** - Use `resources/schemas/*.fbs.template`, replace `{{NAMESPACE}}`
2. **Codec** - Use `resources/codec/*.rs.template` for type conversion
3. **FFI wrapper** - Use `resources/ffi/*.rs.template` for C ABI exports

### FlatBuffers Code Generation

```bash
flatc --rust -o src/codec/generated schemas/request.fbs schemas/response.fbs
```

### Dependencies

```toml
[dependencies]
surrealdb = { version = "2.x", features = ["kv-mem"] }  # or kv-rocksdb
flatbuffers = "24.x"
rmp-serde = "1.x"
rmpv = "1.x"
chrono = { version = "0.4", features = ["serde"] }
thiserror = "2.x"
log = "0.4"
```

## Guidelines

### Using TypeHints

Convert string parameters from clients into native SurrealDB types:

```rust
// Formats: "field_name" or "datetime:field_name" or "record:table:field"
let hints = TypeHints::parse(&[
    "created_at".to_string(),
    "record:user:owner_id".to_string(),
]);
let surreal_value = to_surreal_value_with_type_hints(msgpack_value, &hints);
```

### Decimal Precision Loss

`sql::Number::Decimal` is converted to f64, so precision beyond 15–17 digits is lost. If exact precision is required, treat it as a string.

### Thing ID Cleanup

SurrealDB 2.x wraps Thing IDs in angle brackets. Remove them during conversion:

```rust
// "user:⟨uuid-here⟩" -> "uuid-here"
let clean_id = id.trim_matches(|c| c == '\u{27E8}' || c == '\u{27E9}');
```

### Error Handling

- status_code = 0: success
- status_code = -1: Rust panic (fatal)
- status_code > 0: application error

All FFI functions must wrap logic with `std::panic::catch_unwind` to prevent panics from crossing the FFI boundary.

### Async Runtime Management

FFI functions are synchronous but SurrealDB SDK is async. Use `OnceLock` for global runtime singleton. See `docs/integration_guide.md` for the complete pattern.

### Applying RecordFieldHint

FlatBuffers `record_fields: [RecordFieldHint]` should be mapped into `TypeHints.record_fields`.

```
record_fields = [
  { field: "owner_id", table_name: "user" }
]
```

The above is equivalent to `record:user:owner_id`. Ensure client hints and Rust `TypeHints` are consistent.

### FFI Input Safety

- If `request_ptr` is null or `request_len == 0`, return `InvalidRequest` immediately.
- FFI callers must always invoke `free_response_buffer` and avoid double‑free.

### Error Payload Format

Use a structured MessagePack payload on error:

```
{ "code": i32, "message": String, "details": Option<String> }
```

Clients should ignore unknown fields for forward compatibility.
