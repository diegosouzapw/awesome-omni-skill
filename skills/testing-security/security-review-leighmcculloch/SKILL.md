---
name: security-review
description: Review code for security vulnerabilities. Use when asked to "security review", "audit code", "find vulnerabilities", "check for security issues", "pentest review", or "security audit" a codebase or set of files.
allowed-tools: Read, Grep, Glob, Bash(git *), Task
---

# Security Review Skill

Perform a security-focused code review that identifies real, exploitable vulnerabilities and provides proof-of-concept demonstrations. Produces a high-signal report with two distinct sections: confirmed vulnerabilities with PoCs, and separate security hardening recommendations.

## Critical Principles

1. **Only report real vulnerabilities.** Every vulnerability in the report must be grounded in how the code actually works, not how it theoretically could be misused. Trace data flow from attacker-controlled input to the dangerous operation. If the path is blocked by validation, type constraints, or other controls, it is not a vulnerability.

2. **Investigate before reporting.** If you encounter a pattern that looks suspicious but you are not certain it is exploitable, DO NOT include it in the report as a vulnerability. Instead, investigate it:
   - Use the Task tool with `subagent_type: "general-purpose"` to spawn an investigation agent. Instruct it to trace the data flow, read all relevant code paths, and determine whether the issue is actually exploitable.
   - Only include the finding if the investigation confirms it is a real problem.

3. **Every vulnerability needs a proof of concept.** If you cannot write a PoC that demonstrates the vulnerability, it does not belong in the Vulnerabilities section. If it is a genuine concern but you cannot prove exploitability, put it in the Hardening section instead.

4. **Separate vulnerabilities from hardening.** Vulnerabilities are confirmed exploitable issues rated High, Medium, or Low. Hardening recommendations are defense-in-depth improvements that do not address a specific confirmed vulnerability.

## Workflow

### 1. Determine Scope

Identify what to review:

- If the user specifies files or directories, use those.
- If the user provides a PR number or branch, get the diff:
  ```bash
  git diff main...HEAD --name-only
  ```
- If no scope is given, ask the user what to review.

Detect the primary language(s) in scope to determine which language-specific checks apply.

Determine what kind of software is being reviewed (CLI tool, library, server, etc.) as this affects how PoCs are written.

### 2. Enumerate Attack Surface

Before reading code, identify the attack surface by searching for:

| Surface | Search Patterns |
|---------|----------------|
| Network listeners | `net.Listen`, `http.ListenAndServe`, `TcpListener::bind`, `hyper::Server` |
| HTTP handlers | `HandleFunc`, `Handler`, `#[get`, `#[post`, `axum::Router`, `actix_web` |
| User input parsing | `json.Unmarshal`, `serde::Deserialize`, `ParseForm`, `FromStr` |
| File system access | `os.Open`, `os.Create`, `std::fs::`, `ioutil.ReadFile` |
| Database queries | `db.Query`, `db.Exec`, `sqlx::query`, `diesel::` |
| External commands | `exec.Command`, `std::process::Command`, `os/exec` |
| Cryptographic operations | `crypto/`, `ring::`, `sha`, `hmac`, `aes`, `rand` |
| Authentication/authorization | `token`, `jwt`, `session`, `auth`, `permission`, `rbac` |
| Environment/config | `os.Getenv`, `std::env::var`, `viper`, `config` |

### 3. Review Code

Read each file in scope. For every file, check against:

1. **General security checklist** (Section: General Checks)
2. **Language-specific checklist** (Section: Go Checks or Rust Checks)
3. **Context-specific concerns** based on the attack surface found in Step 2

For every potential finding, ask yourself:

- **Is this reachable from attacker-controlled input?** Trace the data flow backwards from the dangerous operation to an input source. Read the actual code at each step. If you cannot trace a path from untrusted input to the dangerous operation, it is not a vulnerability.
- **Are there existing controls that mitigate this?** Read the surrounding code, middleware, validation layers, and type constraints. If the issue is already mitigated, it is not a vulnerability.
- **Can I demonstrate this with a concrete PoC?** If not, it belongs in Hardening at most.

### 4. Investigate Uncertain Findings

For any finding where you are not confident it is exploitable:

Spawn an investigation agent using the Task tool:

```
Task tool with subagent_type: "general-purpose"

Prompt: "Investigate whether [description of the suspicious pattern] in [file:line]
is actually exploitable. Trace the data flow from any attacker-controlled input to
this operation. Read all relevant validation, middleware, type constraints, and
calling code. Determine:
1. Can an attacker reach this code path with controlled input?
2. Are there existing mitigations that prevent exploitation?
3. If exploitable, what is the concrete attack scenario?
Report back with your conclusion: CONFIRMED, NOT EXPLOITABLE, or NEEDS MORE CONTEXT."
```

If the agent reports NOT EXPLOITABLE, drop the finding entirely. Do not include it in the report. If CONFIRMED, proceed to write a PoC. If NEEDS MORE CONTEXT, make a judgment call: either investigate further or move it to Hardening with a note about what is uncertain.

### 5. Write Proof of Concept for Each Vulnerability

Every vulnerability must have a minimal PoC that demonstrates exploitability. The format depends on the type of software:

**For a CLI tool:**
```bash
# PoC: [vulnerability name]
# Expected: [what should happen safely]
# Actual: [what happens due to the vulnerability]
command --flag "$(malicious_payload)"
```

**For a library:**
```go
// PoC: [vulnerability name]
package main

import "vulnerable/package"

func main() {
    // Minimal code that triggers the vulnerability
    result := package.VulnerableFunction(maliciousInput)
    // Demonstrate the impact
}
```

```rust
// PoC: [vulnerability name]
use vulnerable_crate::vulnerable_function;

fn main() {
    // Minimal code that triggers the vulnerability
    let result = vulnerable_function(malicious_input);
    // Demonstrate the impact
}
```

**For a server:**
```bash
# PoC: [vulnerability name]
# Step 1: [setup if needed]
curl -X POST http://localhost:8080/endpoint \
  -H "Content-Type: application/json" \
  -d '{"malicious": "payload"}'
# Expected response: [what safe behavior looks like]
# Actual response: [what the vulnerability produces]
```

The PoC must be minimal — only the code necessary to trigger the issue and nothing else.

### 6. Classify Findings

All confirmed vulnerabilities are rated on this scale:

| Severity | Criteria |
|----------|----------|
| **High** | Exploitable with no or minimal preconditions. Leads to RCE, authentication bypass, privilege escalation, data breach, or significant data exposure. |
| **Medium** | Exploitable but requires specific conditions (authenticated attacker, particular configuration, race condition timing). Impact is meaningful but bounded. |
| **Low** | Exploitable but impact is limited (minor information disclosure, DoS requiring sustained effort, issues only exploitable in non-default configurations). |

Hardening recommendations are not rated on this scale. They are listed separately without severity ratings.

### 7. Write Report

Write the report to `SECURITY_REVIEW.md` in the current working directory with this structure:

```markdown
# Security Review

**Scope:** {files or directories reviewed}
**Date:** {date}
**Languages:** {detected languages}
**Software type:** {CLI tool / library / server / etc.}

## Summary

{1-3 sentences. State the number of confirmed vulnerabilities found and their severity breakdown. If none found, say so clearly.}

---

## Vulnerabilities

{If no vulnerabilities were found, state: "No confirmed vulnerabilities were identified."}

### HIGH-001: {Finding Title}

**File:** `path/to/file.go:42`
**Category:** {e.g., Injection, Authentication, Cryptography}

**Description:**
{What the vulnerability is. Be specific about the data flow: what attacker-controlled input reaches what dangerous operation, and why existing controls do not prevent it.}

**Vulnerable Code:**
{The specific code snippet containing the vulnerability}

**Proof of Concept:**
{Minimal PoC demonstrating the vulnerability — commands, code, or HTTP requests}

**Recommendation:**
{How to fix it, with a corrected code example}

---

### MED-001: {Finding Title}

{same structure}

---

### LOW-001: {Finding Title}

{same structure}

---

## Security Hardening

{Defense-in-depth recommendations that do not address a specific confirmed vulnerability. These are improvements to security posture.}

### {Recommendation Title}

**File:** `path/to/file.go:42` (if applicable)

**Description:**
{What could be improved and why it strengthens security posture, even though it does not address a specific confirmed vulnerability.}

**Suggestion:**
{Concrete code or configuration change}
```

### 8. Present to User

After writing the report, summarize the confirmed vulnerabilities inline. Do not summarize hardening items — they are secondary. Ask if the user wants to discuss any specific finding in detail.

---

## General Checks

Apply these checks regardless of language. Remember: only flag items where you can trace attacker-controlled input to a dangerous operation without existing mitigation.

### Input Validation

- Untrusted input used directly in SQL queries, commands, file paths, or URLs
- Missing or insufficient length/size limits on inputs
- Improper handling of Unicode, null bytes, or encoding edge cases
- Path traversal via `../` or absolute paths in user-supplied filenames
- SSRF via user-controlled URLs used in outbound requests

### Authentication and Authorization

- Missing authentication on sensitive endpoints
- Authorization checks that can be bypassed (IDOR, missing ownership checks)
- Hard-coded credentials, API keys, or tokens
- Timing-safe comparison not used for secrets (`==` instead of constant-time compare)
- Session tokens with insufficient entropy or predictable generation

### Cryptography

- Use of broken algorithms (MD5, SHA1 for security purposes, DES, RC4)
- Hard-coded keys, IVs, or nonces
- Nonce reuse in AEAD ciphers
- Use of ECB mode
- Custom cryptographic implementations instead of vetted libraries
- Insufficient key lengths (RSA < 2048, symmetric < 128-bit)
- Missing integrity protection (encryption without authentication)

### Error Handling

- Stack traces, internal paths, or debug info exposed to users
- Errors that reveal whether a user/resource exists (enumeration)
- Catch-all error handlers that swallow security-relevant failures
- Panics or crashes reachable from untrusted input

### Secrets Management

- Secrets logged to stdout/stderr or log files
- Secrets in source code, config files committed to VCS, or environment variable defaults
- Secrets not zeroed from memory after use
- `.env` files, private keys, or credential files in the repository

### Concurrency

- Race conditions in authentication or authorization checks (TOCTOU)
- Shared mutable state without synchronization
- Double-spend or duplicate-action vulnerabilities from concurrent requests

### Dependency and Supply Chain

- Dependencies with known CVEs (check go.sum, Cargo.lock)
- Pinned to mutable refs (branches, `latest`) instead of exact versions or hashes
- Vendored code that has been modified from upstream

---

## Go Checks

Apply these when reviewing Go code.

### Injection

- `fmt.Sprintf` used to build SQL queries instead of parameterized queries
- `os/exec.Command` with user-controlled arguments not validated against an allowlist
- `text/template` used where `html/template` is needed (XSS)
- String concatenation in `database/sql` query construction

### Error Handling

- Errors discarded with `_ =` on security-critical operations (file permissions, crypto, auth)
- `defer` closing a resource but ignoring the close error (can mask write failures)
- `recover()` catching panics broadly, masking security failures
- Missing error checks on `io.Copy`, `http.Response.Body.Close`

### HTTP and Networking

- Missing TLS configuration or insecure `TLSClientConfig` (`InsecureSkipVerify: true`)
- `http.DefaultClient` used without timeouts (enables slowloris DoS)
- Missing `http.MaxBytesReader` on request bodies (memory exhaustion)
- CORS headers set to `*` or reflecting origin without validation
- Missing CSRF protection on state-changing endpoints
- `http.Redirect` with user-controlled URL (open redirect)

### Concurrency

- Race conditions on maps (concurrent read/write without `sync.Map` or mutex)
- Goroutine leaks from missing context cancellation or unbounded channel sends
- `sync.WaitGroup` misuse (Add/Done mismatch)
- Shared slice/map modified across goroutines without synchronization

### Unsafe Operations

- Use of `unsafe` package for pointer arithmetic or type punning
- `reflect` used to bypass unexported field protections
- `cgo` calls with unchecked buffer sizes or missing null termination
- `//go:linkname` used to access internal runtime functions

### Cryptographic Pitfalls

- `math/rand` used where `crypto/rand` is required (token generation, nonces)
- `crypto/subtle.ConstantTimeCompare` not used for secret comparison
- Custom `hash.Hash` usage without proper reset between uses
- `crypto/tls` config missing `MinVersion: tls.VersionTLS12`

### Integer and Memory

- Integer overflow in `make([]byte, userControlledSize)` causing allocation issues
- Slice bounds not checked before indexing with external values
- `strconv.Atoi` result used without range validation

### File and OS

- `os.MkdirAll` with overly permissive mode (0777)
- `os.OpenFile` without validating symlink targets (symlink following)
- Temporary files created in shared directories without `os.CreateTemp`
- Missing `filepath.Clean` on user-supplied paths

---

## Rust Checks

Apply these when reviewing Rust code.

### Unsafe Code

- Every `unsafe` block: verify the safety invariants documented and upheld
- Raw pointer dereferencing without null/alignment checks
- `std::mem::transmute` used for type punning (verify layout compatibility)
- `unsafe impl Send/Sync` on types containing raw pointers or non-thread-safe internals
- `from_raw_parts` / `from_raw_parts_mut` with unchecked length or alignment
- `ManuallyDrop` or `std::mem::forget` causing resource leaks (file handles, locks)
- FFI boundaries: missing null checks, buffer size validation, lifetime guarantees

### Error Handling

- `.unwrap()` or `.expect()` on `Result`/`Option` reachable from untrusted input
- `panic!` in library code reachable from external callers
- `?` propagation that discards context needed for security decisions
- `catch_unwind` used as a substitute for proper error handling

### Memory Safety (even in safe Rust)

- Logic errors in index arithmetic leading to out-of-bounds via `.get_unchecked()`
- `Vec::set_len()` on uninitialized memory
- Stack overflow via unbounded recursion on attacker-controlled input
- Large allocations from user-controlled sizes (`Vec::with_capacity(user_input)`)

### Serialization and Parsing

- `serde` deserialization of untrusted input without size limits or depth limits
- `#[serde(deny_unknown_fields)]` missing on security-critical config structs
- Custom `Deserialize` implementations with unchecked invariants
- Deserializing into `enum` variants that trigger different privilege levels

### Web and Network (axum, actix-web, hyper, tokio)

- Missing request body size limits (`tower_http::limit::RequestBodyLimitLayer`)
- Timeout not configured on server or client connections
- Missing TLS certificate validation on HTTP clients (`danger_accept_invalid_certs`)
- Shared mutable state in handlers without `Arc<Mutex<_>>` or `Arc<RwLock<_>>`
- Missing authentication middleware on protected routes
- `tower` layers ordered incorrectly (auth after handler, rate limit after auth)

### Concurrency

- `Mutex` poisoning not handled (`lock().unwrap()` on potentially poisoned mutex)
- `Arc` reference cycles causing memory leaks
- Deadlock potential from inconsistent lock ordering
- `tokio::spawn` without `JoinHandle` tracking (silent task failures)
- Blocking operations in async context (`std::fs` in tokio runtime, `std::thread::sleep`)

### Cryptographic Pitfalls

- `rand::thread_rng()` used where `rand::rngs::OsRng` is needed for cryptographic randomness
- `ring` or `rustcrypto` primitives used without authenticated encryption
- Custom serialization of cryptographic keys without constant-time comparison
- Timing side-channels in comparison operations (`==` on secrets)

### Supply Chain and Build

- `build.rs` executing arbitrary code at compile time
- `proc_macro` dependencies with network access or file system writes
- `[patch]` or `[replace]` in `Cargo.toml` overriding dependencies
- Feature flags that silently disable security features (`#[cfg(not(feature = "tls"))]`)

### Integer Safety

- Arithmetic operations that can overflow/underflow (use `checked_*`, `saturating_*`, or `wrapping_*`)
- `as` casts that truncate or sign-extend (`u64 as u32`, `i32 as u32`)
- `usize` used for serialized data sizes across architectures (32 vs 64-bit mismatch)
