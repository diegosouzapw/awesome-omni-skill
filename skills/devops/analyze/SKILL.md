---
name: analyze
description: Analyze Vela and liFS feature completeness compared to macOS/XNU, producing a completion scorecard
disable-model-invocation: true
---

Perform a full feature completeness analysis of the Vela OS and liFS filesystem, comparing against macOS (XNU kernel + APFS). No argument is needed — always analyze both:

- **Vela kernel (Carina):** `~/claude/vela/kernel/src/`
- **liFS filesystem:** `~/claude/lifs/lifs-core/src/`

Read the relevant source files before producing the report. Do not guess — check the actual code.

---

## What to look for

For each feature, read the file(s) and classify honestly:

- **✅ Complete** — fully implemented, no TODOs, actually runs
- **🔧 In Progress** — real logic present but missing pieces, stubbed I/O, or TODO blocks
- **🪵 Stub** — file exists but is mostly scaffolding, empty bodies, `unimplemented!()`, or type definitions only
- **❌ Missing** — no file, no code, feature doesn't exist at all

---

## Feature categories to evaluate

### 1. Memory Management
Compare to: XNU VM subsystem (Mach VM, compressor, UPL, anonymous memory, file-backed mappings)

- Physical frame allocator (buddy allocator)
- Kernel heap (slab + linked-list)
- Virtual memory areas / mmap
- Page tables + TLB management
- Guard pages + stack overflow detection
- Copy-on-write (CoW)
- Page cache (file-backed pages)
- Swap / compressed memory
- Huge pages / 2MB mappings
- NUMA awareness
- Memory pressure handling / OOM killer

### 2. Process & Thread Management
Compare to: BSD/Mach process model, pthreads, QoS classes

- Task/process creation and destruction
- Kernel threads
- User threads (multi-thread per process)
- Thread join / detach
- Process groups / sessions
- Zombie reaping
- CPU affinity / binding
- Priority / nice levels
- QoS / scheduling classes

### 3. Scheduler
Compare to: XNU scheduler (mach timeshare + realtime + fixed priority)

- Preemptive multitasking
- SMP scheduling (multi-core)
- Interactivity-aware fairness (ARIA vs XNU timeshare)
- Real-time scheduling class
- Work stealing
- Per-CPU run queues
- Idle threads / halt
- Timer-based preemption
- SMT / hyperthreading awareness

### 4. IPC
Compare to: Mach ports, XPC, pipes, Unix domain sockets, signals

- Kernel pipes (byte stream + message mode)
- Typed event system (replaces POSIX signals)
- IPC channels / ports
- Shared memory segments
- Unix domain sockets
- Message queues
- Dispatch queues (GCD equivalent)

### 5. System Calls & User Interface
Compare to: Mach traps + BSD syscalls + commpage

- Syscall fast path (SYSCALL/SYSRET)
- User memory access safety (SMAP guards)
- vDSO / commpage (fast time queries)
- ioctl / device control interface
- sysctl / kernel parameter interface
- Async I/O interface (io_queue / io_uring equivalent)

### 6. File System
Compare to: APFS (journaling, copy-on-write, snapshots, encryption, compression, clones)

- VFS abstraction layer
- liFS: superblock + format/mount
- liFS: inode management
- liFS: B-tree directory layout
- liFS: extent-based file data
- liFS: journaling / crash consistency
- liFS: block allocator
- liFS: compression (LZ4)
- liFS: extended attributes (xattr)
- liFS: symlinks
- liFS: permissions / capabilities
- liFS: snapshots
- liFS: encryption
- liFS: file cloning (reflinks)
- liFS: online resize
- RAM disk fallback
- FSEvents / filesystem notifications
- FUSE driver (lifs-linux)

### 7. Drivers & Hardware
Compare to: IOKit framework, Apple Silicon / Intel driver support

- PCI / PCIe enumeration
- SATA / AHCI disk
- NVMe disk
- USB 3.x host controller (xHCI)
- USB HID (keyboard / mouse)
- Real-time clock (RTC)
- Framebuffer / display output
- GPU / graphics acceleration (Metal equivalent)
- Network interface (Ethernet)
- WiFi
- Bluetooth
- Audio (CoreAudio equivalent)
- IOMMU / DMA protection

### 8. Security
Compare to: SMEP/SMAP, SIP, Gatekeeper, code signing, Sandbox, TCC, KASLR

- SMEP (no kernel execution of user pages)
- SMAP (no kernel access to user memory without guard)
- NX bit / data execution prevention
- W^X enforcement
- Kernel guard pages
- Capability-based ACL (vs macOS DAC + SIP)
- KASLR (kernel address space layout randomization)
- Sandboxing / entitlements (sandbox.rs)
- Secure boot / firmware trust
- Code signing / executable verification
- User authentication (login, password hashing)

### 9. Networking
Compare to: BSD TCP/IP stack, Network Kernel Extensions, pf firewall

- TCP/IP stack
- UDP
- IPv4 / IPv6
- BSD socket API
- Unix domain sockets
- Network filtering / firewall
- DNS resolution (userspace, but kernel stubs needed)

### 10. Power Management
Compare to: IOPMrootDomain, ACPI, sleep/wake, P-states, T-states

- ACPI table parsing
- CPU frequency scaling (P-states)
- Sleep / hibernate / wake
- Thermal management
- Power assertions (prevent sleep)
- Battery / power source monitoring

### 11. Time & Clocks
Compare to: mach_absolute_time, gettimeofday, timers, commpage clock

- Wall clock (RTC-backed)
- Monotonic clock
- High-resolution timer (LAPIC-backed)
- vDSO / commpage fast time access
- Timer coalescing
- POSIX timer API (timer_create)
- Time zones

### 12. Shell & Userspace
Compare to: zsh (macOS default), launchd, dyld

- hesh shell (built-ins + external commands)
- Init system / pid 1 (init.rs)
- Dynamic linker (dyld equivalent)
- Standard library / libc equivalent
- TTY / PTY subsystem (Pyxis — planned)

### 13. Debugging & Observability
Compare to: DTrace, kdebug, syslog, crash reporter, Instruments

- Serial console / kernel logging
- Kernel crash reporter
- DTrace / tracing framework
- Performance counters (PMU)
- Kernel debugger (LLDB/KDP equivalent)
- System profiling

### 14. SMP / Multi-Core
Compare to: XNU SMP bringup, per-CPU data, IPI, CPU hotplug

- ACPI MADT parsing (CPU topology discovery)
- Local APIC (per-CPU timer + IPI)
- AP startup (SIPI trampoline)
- Per-CPU data structures
- IPI (inter-processor interrupts)
- CPU hotplug

---

## Output format

For each category, print a subsection heading and a table:

| Feature | Status | Notes |
|---|---|---|
| Physical frame allocator | ✅ Complete | Buddy allocator, 13 orders, double-free detection |
| Copy-on-write | 🔧 In Progress | VmaBacking::Cow structure exists, no page fault handler |

After all categories, print:

---

## Completion Score

A table:

| Category | Complete | In Progress | Stub | Missing | Score |
|---|---|---|---|---|---|
| Memory Management | 5 | 2 | 0 | 4 | 45% |
| ... | | | | | |
| **Total** | | | | | **XX%** |

Score formula per category: `(Complete × 1.0 + InProgress × 0.4 + Stub × 0.1) / Total × 100`

Then a single **Overall Score: XX%** line, weighted by category importance:
- Memory, Scheduler, Process Management, File System, Security: weight 2×
- IPC, Drivers, Syscalls, SMP: weight 1.5×
- Networking, Power, Debugging, Shell, Time: weight 1×

End with a **Verdict** paragraph: what Vela is today (research OS? daily-driver kernel? prototype?) and the top 3 features to complete next for maximum impact.
