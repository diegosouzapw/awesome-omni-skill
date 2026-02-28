---
name: HexCore Binary Analysis
description: Skill para analise de binarios com ferramentas HexCore integradas ao editor
---

# HexCore Binary Analysis Skill — v3.5.4

## Overview

HexCore is a VS Code fork for reverse engineering and binary analysis (HikariSystem HexCore). It includes 20 extensions with 5 native engines (Capstone, Unicorn, Remill, LLVM MC, better-sqlite3) and a full automation pipeline.

> **Current version:** v3.5.4 "Stability & Isolation" (2026-02-19)
> **Engine versions:** capstone 1.3.2 | unicorn 1.2.1 | llvm-mc 1.0.0 | better-sqlite3 2.0.0 | remill 0.1.2

---

## Extensions

### Native Engines (no VS Code commands — pure API)

| Engine | Version | Purpose | Architectures |
|--------|---------|---------|---------------|
| **hexcore-capstone** | 1.3.2 | Disassembly | x86, x64, ARM, ARM64, MIPS, PPC, SPARC, M68K, RISC-V |
| **hexcore-unicorn** | 1.2.1 | CPU emulation | x86, x64, ARM, ARM64, MIPS, SPARC, PPC, RISC-V |
| **hexcore-remill** | 0.1.2 | LLVM IR lifting | x86, x64, ARM64 only |
| **hexcore-llvm-mc** | 1.0.0 | Assembly/encoding | x86, x64, ARM, ARM64, MIPS, RISC-V, PPC, SPARC |
| **hexcore-better-sqlite3** | 2.0.0 | SQLite database | N/A |

### Disassembler (`hexcore-disassembler` v1.3.0)

Professional disassembler with Capstone engine, ELF/PE parsing, CFG, xrefs, patching, and the pipeline runner.

**Headless commands (pipeline-safe):**
- `hexcore.disasm.analyzeAll` — Deep analysis (prolog scan + xrefs)
- `hexcore.disasm.buildFormula` — Symbolic expression extraction (**x86/x64 only**)
- `hexcore.disasm.checkConstants` — Validate numeric annotations
- `hexcore.disasm.searchStringHeadless` — Search string references
- `hexcore.disasm.exportASMHeadless` — Export assembly to file
- `hexcore.pipeline.runJob` — Run automation job
- `hexcore.pipeline.listCapabilities` — Export capability map
- `hexcore.pipeline.validateJob` — Preflight validation
- `hexcore.pipeline.validateWorkspace` — Batch validation
- `hexcore.pipeline.createPresetJob` — Generate job from preset
- `hexcore.pipeline.saveJobAsProfile` — Save job as profile
- `hexcore.pipeline.doctor` — Diagnose health

**Interactive commands (need UI):**
- `hexcore.disasm.openFile`, `analyzeFile`, `goToAddress`, `findXrefs`, `addComment`, `renameFunction`, `showCFG`, `searchString`, `exportASM`, `patchInstruction`, `nopInstruction`, `assemble`, `assembleMultiple`, `savePatchedFile`, `setSyntax`, `showLlvmVersion`, `nativeStatus`

**Experimental:**
- `hexcore.disasm.liftToIR` — Lift to LLVM IR (requires Remill, x86/x64/ARM64 only)

**Architecture auto-detection:** Reads ELF `e_machine` / PE `Machine` headers. Supports x86, x64, ARM, ARM64, MIPS. Defaults to x64 for raw files.

### Debugger (`hexcore-debugger` v2.0.1)

Emulation-based debugger using Unicorn engine with PE/ELF loading, API hooking, syscall handling, and API call tracing.

**Process isolation & Smart Sync:** x64 ELF and ARM64 ELF emulation run in dedicated child processes (`x64ElfWorker.js`, `arm64Worker.js`) to prevent Unicorn heap corruption from crashing the VS Code extension host. The worker communicates via JSON-RPC over IPC. A unique **Smart Sync** architecture instantly synchronizes heap memory (e.g. dynamically allocated strings) from the Worker to the Host before evaluating any API hook (such as `__printf_chk`, `getline`, or `puts`), guaranteeing flawless validation of complex obfuscated VMs (like active advanced HTB CTFs). PE emulation and other architectures run in-process.

**Headless commands (pipeline-safe):**
- `hexcore.debug.emulateFullHeadless` — **Unified single-shot emulation** (load → configure → run → collect → dispose). Recommended for pipeline jobs. Args: `{ file, arch?, stdin?, maxInstructions?, breakpoints?, keepAlive?, output?, quiet? }`. Aliases: `hexcore.debug.emulate.full`, `hexcore.debug.run`
- `hexcore.debug.writeMemoryHeadless` — Write data to emulation memory. Args: `{ address, data, output?, quiet? }`. Data accepts base64 or `0x`-prefixed hex.
- `hexcore.debug.setRegisterHeadless` — Set CPU register value. Args: `{ name, value, output?, quiet? }`. Value accepts hex string or decimal.
- `hexcore.debug.setStdinHeadless` — Set STDIN buffer for emulation. Args: `{ input, output?, quiet? }`. Supports escape sequences (`\n`, `\t`, `\r`, `\\`).
- `hexcore.debug.disposeHeadless` — Dispose emulation session (idempotent, safe to call without active session). Args: `{ output?, quiet? }`
- `hexcore.debug.snapshotHeadless` — Save emulation snapshot
- `hexcore.debug.restoreSnapshotHeadless` — Restore emulation snapshot
- `hexcore.debug.exportTraceHeadless` — Export API/libc call trace as JSON

**Interactive commands (need UI):**
- `hexcore.debug.emulate` — Start emulation (auto-detect arch)
- `hexcore.debug.emulateWithArch` — Start with manual arch selection
- `hexcore.debug.emulationStep` — Step one instruction
- `hexcore.debug.emulationContinue` — Continue to breakpoint/end
- `hexcore.debug.emulationBreakpoint` — Set breakpoint
- `hexcore.debug.emulationReadMemory` — Read memory region
- `hexcore.debug.setStdin` — Set STDIN buffer for ELF emulation
- `hexcore.debug.saveSnapshot` — Save emulation snapshot
- `hexcore.debug.restoreSnapshot` — Restore snapshot
- `hexcore.debug.unicornStatus` — Show Unicorn status

**Internal engine capabilities (programmatic, not exposed as headless commands):**
- PE loading with import resolution and Windows API hooks
- ELF loading with PLT stubs and Linux API hooks (libc emulation)
- Linux syscall handler (x86/x64: int 0x80, syscall instruction; ARM64: SVC #0)
- Architecture auto-detection from ELF/PE headers
- Deterministic ELF continue (250K instruction budget)
- STDIN buffer injection for scanf/read emulation
- Snapshot save/restore via Unicorn context
- x64 ELF worker process isolation with Smart Sync (prevents host heap corruption & guarantees dynamic string visibility)
- ARM64 ELF worker process isolation (same pattern)

**Architecture support in debugger:**

| Feature | x86 | x64 | ARM64 | ARM | MIPS |
|---------|-----|-----|-------|-----|------|
| Unicorn init | Yes | Yes | Yes | Yes | Yes |
| Register read/write | Yes | Yes | Yes | No | No |
| ELF loading | Yes | Yes | Yes | No | No |
| PE loading | Yes | Yes | No | No | No |
| Stack initialization | Yes | Yes | Yes | No | No |
| Syscall handler | Yes | Yes | Yes | No | No |
| API hooks (Linux) | Yes | Yes | Yes | No | No |
| API hooks (Windows) | Yes | Yes | No | No | No |
| Worker process isolation | No | Yes (ELF) | Yes | No | No |

### Other Extensions

| Extension | Version | Headless | Commands |
|-----------|---------|----------|----------|
| **hexcore-peanalyzer** | — | Yes | `peanalyzer.analyze`, `peanalyzer.analyzeActive` |
| **hexcore-elfanalyzer** | 1.0.0 | Yes | `elfanalyzer.analyze`, `elfanalyzer.analyzeActive` |
| **hexcore-hexviewer** | — | Yes | `hexview.dumpHeadless`, `hexview.searchHeadless`, `openHexView`, `goToOffset`, `searchHex`, `copyAsHex`, `copyAsC`, `copyAsPython`, `addBookmark`, `applyTemplate`, `toggleEdit` |
| **hexcore-strings** | — | Yes | `strings.extract`, `strings.extractAdvanced` (now with multi-byte XOR, rolling XOR, increment XOR) |
| **hexcore-entropy** | — | Yes | `entropy.analyze` |
| **hexcore-filetype** | — | Yes | `filetype.detect` |
| **hexcore-hashcalc** | — | Yes | `hashcalc.calculate`, `hashcalc.quick`, `hashcalc.verify` |
| **hexcore-base64** | — | Yes | `base64.decodeHeadless`, `base64.decode` |
| **hexcore-yara** | — | Partial | `yara.scan` (headless), `yara.updateRules` (headless), rest interactive |
| **hexcore-ioc** | — | Yes | `ioc.extract`, `ioc.extractActive` |
| **hexcore-minidump** | — | Yes | `minidump.parse`, `minidump.threads`, `minidump.modules`, `minidump.memory` |
| **hexcore-report-composer** | 1.0.0 | Yes | `pipeline.composeReport` — aggregates reports into unified Markdown |
| **hexcore-common** | — | N/A | Utility library (formatBytes, loadNativeModule, etc.) |

---

## Pipeline Automation

### Creating Jobs

1. **From preset:** Run `hexcore.pipeline.createPresetJob` — choose quick-triage, full-static, or ctf-reverse
2. **Manual:** Create `.hexcore_job.json` in workspace root (see `docs/HEXCORE_JOB_TEMPLATES.md`)
3. **Save profile:** Run `hexcore.pipeline.saveJobAsProfile` to store in `.hexcore_profiles.json`

### Running Jobs

- **Auto:** HexCore watches `.hexcore_job.json` and runs on create/change
- **Manual:** Run `hexcore.pipeline.runJob`
- **Validate first:** Run `hexcore.pipeline.validateJob` for preflight check

### Job Contract

Every headless command receives:
- `file` — path to target binary
- `quiet` — suppress UI notifications
- `output` — `{ path, format }` for writing results

### Output

Jobs produce in `outDir`:
- `hexcore-pipeline.log` — execution log with timestamps
- `hexcore-pipeline.status.json` — structured status per step (ok/failed/timed-out)
- Per-step output files (JSON or MD)

---

## Architecture Support Matrix

| Component | x86 | x64 | ARM | ARM64 | MIPS |
|-----------|-----|-----|-----|-------|------|
| Disassembly (Capstone) | Yes | Yes | Yes | Yes | Yes |
| Emulation (Unicorn) | Yes | Yes | Yes | Yes | Yes |
| IR Lifting (Remill) | Yes | Yes | No | Yes | No |
| Assembly (LLVM MC) | Yes | Yes | Yes | Yes | Yes |
| Debugger (full) | Yes | Yes | No | Yes | No |
| PE Analysis | Yes | Yes | No | No | No |
| Minidump | Yes | Yes | No | No | No |
| buildFormula | Yes | Yes | No | No | No |

---

## Known Gaps (Critical for Agents)

1. ~~**Debugger interactive commands still need UI**~~ — **MOSTLY RESOLVED**: `emulateFullHeadless` provides full headless emulation (load → run → collect → dispose) without UI. `writeMemoryHeadless`, `setRegisterHeadless`, `setStdinHeadless`, and `disposeHeadless` fill remaining gaps. Only `emulateWithArch` (manual arch picker) remains interactive.
2. ~~**Debugger ARM64 ELF is incomplete**~~ — **RESOLVED in v3.5.1**: Full ARM64 DebugEngine with stack initialization, process stack layout (argc/argv via X0/X1/X2), SVC syscall handler, register state mapping, and 20+ Linux syscalls.
3. **Debugger + static ELF** — statically-linked binaries have no PLT stubs, so LinuxApiHooks cannot intercept libc calls. Only direct syscall interception works (and only for x86/x64/ARM64).
4. **buildFormula is x86/x64 only** — the register regex doesn't recognize ARM64 registers (x0-x30, sp, lr). *(ARM64 formulaBuilder added in v3.5.1 but limited to 15 mnemonics)*
5. ~~**No ELF analyzer extension**~~ — **RESOLVED in v3.5.2**: `hexcore-elfanalyzer` provides full ELF analysis (sections, segments, symbols, security mitigations).
6. ~~**Base64 decode has no headless mode**~~ — **RESOLVED in v3.5.2**: `hexcore.base64.decodeHeadless` is pipeline-safe.
7. ~~**Hex viewer has no headless dump**~~ — **RESOLVED in v3.5.2**: `hexcore.hexview.dumpHeadless` and `hexcore.hexview.searchHeadless` are pipeline-safe.
8. ~~**Strings XOR is 1-byte only**~~ — **RESOLVED in v3.5.2**: `extractAdvanced` now supports multi-byte XOR (2, 4, 8, 16 bytes), rolling XOR, and XOR with increment.
9. **Prebuilds are win32-x64 only** — Linux/macOS need `node-gyp rebuild` fallback.

---

## What Agents CAN Do

1. **Create `.hexcore_job.json`** files and run analysis via `hexcore.pipeline.runJob`
2. **Read pipeline output** from `hexcore-pipeline.status.json` and step output files
3. **Interpret results** — entropy reports, string extractions, YARA matches, IOC lists
4. **Validate jobs** with `hexcore.pipeline.validateJob` before execution
5. **Use presets** via `hexcore.pipeline.createPresetJob` for quick setup
6. **Search strings headlessly** via `hexcore.disasm.searchStringHeadless`
7. **Export assembly headlessly** via `hexcore.disasm.exportASMHeadless`
8. **Analyze ELF binaries** via `hexcore.elfanalyzer.analyze` (sections, segments, symbols, security)
9. **Decode Base64** via `hexcore.base64.decodeHeadless`
10. **Dump hex ranges** via `hexcore.hexview.dumpHeadless`
11. **Search hex patterns** via `hexcore.hexview.searchHeadless`
12. **Run full emulation headlessly** via `hexcore.debug.emulateFullHeadless` (single-shot: load → configure → run → collect → dispose)
13. **Write emulation memory** via `hexcore.debug.writeMemoryHeadless`
14. **Set CPU registers** via `hexcore.debug.setRegisterHeadless`
15. **Set STDIN buffer** via `hexcore.debug.setStdinHeadless`
16. **Dispose emulation sessions** via `hexcore.debug.disposeHeadless`
17. **Save/restore emulation snapshots** via `hexcore.debug.snapshotHeadless` / `restoreSnapshotHeadless`
18. **Export API call traces** via `hexcore.debug.exportTraceHeadless`
19. **Compose unified reports** via `hexcore.pipeline.composeReport`

## What Agents CANNOT Do

1. ~~**Start emulation**~~ — **RESOLVED**: Use `hexcore.debug.emulateFullHeadless` (or aliases `hexcore.debug.emulate.full` / `hexcore.debug.run`) for headless emulation. Interactive `emulateWithArch` still requires UI for manual arch selection.
2. **See webviews** — CFG graph, hex viewer, debugger view are visual only
3. **Use interactive commands** — file pickers, input boxes, quick-picks
4. **Patch binaries** — `patchInstruction`, `nopInstruction`, `savePatchedFile` need the disassembler UI open
5. **Run YARA quick scan** — requires prior UI context

---

## Workflow: Static Analysis

```
1. hexcore.filetype.detect         → Identify file type
2. hexcore.hashcalc.calculate      → Compute hashes (VT lookup)
3. hexcore.entropy.analyze         → Detect packing/encryption
4. hexcore.strings.extract         → Extract strings
5. hexcore.strings.extractAdvanced → XOR deobfuscation (1-byte + multi-byte + rolling + increment) + stack strings
6. hexcore.base64.decodeHeadless   → Detect Base64 encoded strings
7. hexcore.hexview.dumpHeadless    → Inspect file header bytes
8. hexcore.peanalyzer.analyze      → PE headers/imports (PE files only)
9. hexcore.elfanalyzer.analyze     → ELF sections/segments/symbols/security (ELF files only)
10. hexcore.disasm.analyzeAll      → Deep disassembly + xrefs
11. hexcore.yara.scan              → Threat detection
12. hexcore.ioc.extract            → IOC extraction
13. hexcore.pipeline.composeReport → Unified report
```

## Workflow: CTF Reverse Engineering

```
1. hexcore.filetype.detect              → Verify binary format
2. hexcore.disasm.analyzeAll            → Function discovery + xrefs
3. hexcore.disasm.exportASMHeadless     → Full disassembly export
4. hexcore.disasm.searchStringHeadless  → Find flag patterns
5. hexcore.strings.extractAdvanced      → Find obfuscated strings (multi-byte XOR, rolling, increment)
6. hexcore.base64.decodeHeadless        → Find Base64 encoded data
7. hexcore.hexview.searchHeadless       → Search for flag byte patterns
8. hexcore.disasm.buildFormula          → Extract key computations (x86/x64 only)
```

## Workflow: Dynamic Analysis (Emulation)

```
1. hexcore.debug.emulateFullHeadless    → Single-shot emulation (recommended for pipeline jobs)
   Args: { file, arch?, stdin?, maxInstructions?, breakpoints?, keepAlive?, output? }
   Returns: FullEmulationResult with registers, apiCalls, stdout, memoryRegions, crash status

For advanced multi-step emulation (keepAlive: true):
2. hexcore.debug.emulateFullHeadless    → Start with keepAlive: true
3. hexcore.debug.writeMemoryHeadless    → Patch memory (base64 or 0x hex data)
4. hexcore.debug.setRegisterHeadless    → Modify CPU registers
5. hexcore.debug.setStdinHeadless       → Inject STDIN input
6. hexcore.debug.snapshotHeadless       → Save state checkpoint
7. hexcore.debug.disposeHeadless        → Clean up session
```

---

## File Format Support

| Format | Extensions | PE Analysis | ELF Analysis | Disassembly | Emulation |
|--------|-----------|-------------|--------------|-------------|-----------|
| PE32 | .exe, .dll | Yes | No | Yes (x86) | Yes |
| PE64 | .exe, .dll | Yes | No | Yes (x64) | Yes |
| ELF32 | .elf, .so, .o | No | Yes | Yes (x86/ARM) | Yes |
| ELF64 | .elf, .so | No | Yes | Yes (x64/ARM64/MIPS) | Yes (worker isolated) |
| Raw | .bin, .raw | No | No | Yes (default x64) | Yes |
| Minidump | .dmp | N/A | N/A | N/A | N/A |

---

*HexCore v3.5.4 "Stability & Isolation" — Powered by Capstone 1.3.2 / Unicorn 1.2.1 / LLVM MC 1.0.0 / Remill 0.1.2*
