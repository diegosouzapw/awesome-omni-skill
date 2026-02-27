---
name: vox
description: Lightweight voice MCP server with local Moonshine STT + Kokoro TTS
version: 0.1.0
tags: [mcp, voice, tts, stt, rust]
---

# Vox — Claude Code Skill

## Project Overview

Vox is a lightweight voice MCP server (~1,500 lines of Rust) providing local text-to-speech (Kokoro) and speech-to-text (Moonshine Base) via the MCP protocol. It runs as a stdio subprocess per MCP client, or as a shared HTTP daemon.

## Build & Test

```bash
cargo check                    # type-check only
cargo test                     # run all unit tests (84 tests across 10 modules)
cargo clippy -- -D warnings    # lint — must pass with zero warnings
cargo build --release          # optimized build (LTO + single codegen unit)
cargo bench -- resample        # benchmark resampling
```

All of `cargo test`, `cargo clippy -- -D warnings`, and `cargo fmt --check` must pass before submitting changes.

## Architecture

| Module | Purpose |
|--------|---------|
| `main.rs` | Entry point, config loading, model download, stdio/daemon startup |
| `cli.rs` | Clap CLI parser: daemon, config, download-models subcommands |
| `server.rs` | MCP tool handlers (`say`, `listen`, `converse`), streaming TTS pipeline |
| `tts.rs` | Kokoro TTS engine wrapper, voice name → speaker ID resolution, sentence splitting |
| `audio.rs` | cpal-based mic capture and speaker playback, Lanczos-3 sinc resampling |
| `stt.rs` | Moonshine Base STT engine wrapper |
| `vad.rs` | Voice activity detection (Silero ONNX) |
| `config.rs` | TOML config loading, env var overrides (`VOX_*` prefix), path resolution |
| `daemon.rs` | HTTP daemon lifecycle: daemonize, PID file, start/stop/status/log |
| `models.rs` | Model readiness checks and download/extraction |
| `lib.rs` | Public re-exports for benchmarks (`audio`, `config`, `error`, `tts`) |
| `error.rs` | `VoiceError` enum with `thiserror` derives |

## Transport Modes

- **Stdio** (default): `rmcp::transport::stdio()`. One process per MCP client.
- **Daemon** (`vox daemon start [--port PORT]`): `StreamableHttpService` via rmcp. Single process, models loaded once, multiple clients connect over HTTP/SSE. Factory closure creates a `VoiceMcpServer` per session with shared `Arc<Mutex<TtsEngine>>` and `Arc<Mutex<SttEngine>>`.

## Config System

Precedence (highest wins):
1. Environment variables: `VOX_SPEED`, `VOX_VOICE`, `VOX_MODEL_DIR`, `VOX_LOG_LEVEL`, `VOX_PORT`
2. TOML file: `$XDG_CONFIG_HOME/vox/config.toml`
3. Compiled defaults (`Config::default()`)

CLI management: `vox config get [key]`, `vox config set <key> <value>`, `vox config path`

## MCP Tools

| Tool | Description |
|------|-------------|
| `say` | Speak text aloud through speakers (TTS only) |
| `listen` | Record from microphone and transcribe (STT only) |
| `converse` | Speak text then listen for response (TTS + STT round-trip) |

## Available Voices

American female (`af_*`): heart, alloy, aoede, bella, jessica, kore, nicole, nova, river, sarah, sky
American male (`am_*`): adam, echo, eric, liam, michael, onyx, puck, santa
British female (`bf_*`): alice, emma, lily
British male (`bm_*`): daniel, fable, george, lewis

Default: `af_heart` (ID 0). Voices can be specified by name or numeric ID.

## Code Conventions

- **Edition 2024** — uses `let` chains (`if let Ok(x) = ... && let Ok(y) = ...`)
- **Visibility**: `pub(crate)` for test-only exposure, not fully `pub`
- **Clippy**: treat all warnings as errors (`-D warnings`)
- **Tests**: inline `#[cfg(test)] mod tests` per module, `tempfile` for filesystem tests
- **`unsafe impl Send`**: `TtsEngine` and `CaptureHandle` have manual `Send` impls due to non-Send cpal/sherpa internals confined to dedicated threads

## Dev Workflow

1. Make changes
2. `cargo test` — verify all 84 tests pass
3. `cargo clippy -- -D warnings` — zero warnings
4. `cargo fmt --check` — formatting
5. If touching `audio.rs` resampling: `cargo bench -- resample`
