---
name: rust
description: Best practices for Rust development including project structure, error handling, and testing
goal: พัฒนาระบบด้วย Rust ตาม best practices
outcome: Projects ที่ใช้ Rust มีคุณภาพและ maintainable
---

# Rust Development

## When to Execute

Use this skill when you need to develop systems with Rust focusing on safety, performance, and maintainability.

### Folder Structure Summary

| Folder | Purpose | When to Use |
|--------|---------|-------------|
| `rules/` | Specific guidelines and patterns | Create actionable rules |
| `rust-*.md` | Implementation guides | Step-by-step setup instructions |

### Entry Points

1. **New Rust Project** - Start with `cargo new` and configure Cargo.toml
2. **Error Handling** - Use thiserror for robust error management
3. **Project Structure** - Follow components, services, and adapters patterns
4. **Configuration** - Set up config files and Cargo config
5. **Testing & CI** - Implement comprehensive testing strategies

## Quick Start

1. สร้างโปรเจกต์ Rust ใหม่ด้วย `cargo new my-project`
2. ตั้งค่า Cargo.toml ตาม [rust-cargo-toml.md](./rules/rust-cargo-toml.md)
3. ตั้งค่า error handling ด้วย thiserror ตาม [rust-error-rs.md](./rules/rust-error-rs.md)
4. จัดโครงสร้างโปรเจกต์ตาม [rust-components.md](./rules/rust-components.md)
5. รัน `cargo test` เพื่อทดสอบการตั้งค่า

## Rules

| Priority | Impact | Reference | Name | Description | Prefix | Condition |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `CRITICAL` | [rust-cargo-toml.md](./rules/rust-cargo-toml.md) | Cargo.toml | ตั้งค่า Cargo.toml ตาม best practices | `rust-` | เมื่อสร้าง project |
| 1 | `CRITICAL` | [rust-cargo-config.md](./rules/rust-cargo-config.md) | Cargo Config | ตั้งค่า .cargo/config.toml | `rust-` | เมื่อตั้งค่า config |
| 1 | `CRITICAL` | [rust-config-file.md](./rules/rust-config-file.md) | Config File | ตั้งค่า Config.toml | `rust-` | เมื่อตั้งค่า config |
| 2 | `HIGH` | [rust-error-rs.md](./rules/rust-error-rs.md) | Error Handling | การจัดการ errors ด้วย thiserror | `rust-` | เมื่อ handle errors |
| 2 | `HIGH` | [rust-config-rs.md](./rules/rust-config-rs.md) | Config Module | การจัดการ configuration ด้วย figment | `rust-` | เมื่อจัดการ config |
| 2 | `HIGH` | [rust-telemetry-rs.md](./rules/rust-telemetry-rs.md) | Telemetry | การจัดการ logging ด้วย tracing | `rust-` | เมื่อ setup telemetry |
| 2 | `HIGH` | [rust-prelude-rs.md](./rules/rust-prelude-rs.md) | Prelude | การจัดการ common imports | `rust-` | เมื่อจัดการ imports |
| 2 | `HIGH` | [rust-lib-rs.md](./rules/rust-lib-rs.md) | Library Entry | Expose Public API | `rust-` | เมื่อสร้าง library |
| 2 | `HIGH` | [rust-main-rs.md](./rules/rust-main-rs.md) | Main Entry | Application Entry Point | `rust-` | เมื่อสร้าง application |
| 3 | `HIGH` | [rust-components.md](./rules/rust-components.md) | Components | Pure Layer: Domain logic | `rust-` | เมื่อสร้าง components |
| 3 | `HIGH` | [rust-services.md](./rules/rust-services.md) | Services | Effect Layer: I/O | `rust-` | เมื่อสร้าง services |
| 3 | `HIGH` | [rust-adapters.md](./rules/rust-adapters.md) | Adapters | Wrappers สำหรับ external libraries | `rust-` | เมื่อสร้าง adapters |
| 3 | `HIGH` | [rust-utils.md](./rules/rust-utils.md) | Utils | Pure helpers | `rust-` | เมื่อสร้าง utils |
| 3 | `HIGH` | [rust-types.md](./rules/rust-types.md) | Types | Data Structures | `rust-` | เมื่อสร้าง types |
| 3 | `HIGH` | [rust-constants.md](./rules/rust-constants.md) | Constants | ค่าคงที่ | `rust-` | เมื่อสร้าง constants |
| 4 | `MEDIUM` | [rust-testing.md](./rules/rust-testing.md) | Testing | Unit และ Integration Tests | `rust-` | เมื่อทดสอบ |
| 4 | `MEDIUM` | [rust-imports.md](./rules/rust-imports.md) | Imports | การจัดการ imports | `rust-` | เมื่อจัดการ imports |
| 4 | `MEDIUM` | [rust-documentation.md](./rules/rust-documentation.md) | Documentation | Documentation Standard | `rust-` | เมื่อเขียน docs |
| 4 | `MEDIUM` | [rust-prohibitions.md](./rules/rust-prohibitions.md) | Prohibitions | สิ่งที่ห้าม | `rust-` | เมื่อเขียน code |
| 4 | `MEDIUM` | [rust-code-quality.md](./rules/rust-code-quality.md) | Code Quality | Best practices | `rust-` | เมื่อเขียน code |
| 4 | `MEDIUM` | [rust-security.md](./rules/rust-security.md) | Security | Best practices | `rust-` | เมื่อ ensure security |
| 4 | `MEDIUM` | [rust-performance.md](./rules/rust-performance.md) | Performance | Best practices | `rust-` | เมื่อ optimize |

## Knowledge

| Reference | Name | Description | Prefix |
| :--- | :--- | :--- | :--- |
| [core-concept.md](./knowledge/core-concept.md) | Core Concepts | ความรู้เกี่ยวกับ concepts หลักของ Rust | `rust-` |
| [all-features.md](./knowledge/all-features.md) | All Features | ความรู้เกี่ยวกับฟีเจอร์ทั้งหมดของ Rust | `rust-` |
| [best-practices.md](./knowledge/best-practices.md) | Best Practices | แนวทางปฏิบัติที่ดีที่สุดสำหรับ Rust | `rust-` |

## References

- [Rust Documentation](https://doc.rust-lang.org/)
