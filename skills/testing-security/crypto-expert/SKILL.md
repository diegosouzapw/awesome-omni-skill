---
name: crypto-expert
description: Crypto best-practices guidance and review across languages and domains. Use whenever cryptography, encryption, hashing, signatures, key/nonce/IV handling, randomness, password storage, TLS/PKI, secure channels, token formats, or "roll your own crypto" is mentioned, including high-level questions or code/design reviews. Trigger broadly to prevent subtle security mistakes.
---

# Crypto Expert

## Overview

Provide language-agnostic cryptography guidance, highlight unsafe patterns, and steer toward proven constructions and libraries. Optimize for correctness, clear threat assumptions, and long-term maintainability.

## Operating Rules (do not skip)

1. Avoid bespoke crypto. Prefer vetted, high-level library APIs and standard protocols.
2. Treat all crypto changes as production-impacting; call out risk and required migration steps.
3. Never recommend broken or obsolete algorithms/modes; verify any algorithm choice against current guidance.
4. Default to AEAD (e.g., AES-GCM/ChaCha20-Poly1305) for encryption; use separate primitives for hashing/signatures.
5. Nonce/IV misuse is catastrophic: never reuse with the same key; ensure uniqueness and correct length.
6. Prefer KDFs for key derivation and password hashing (Argon2id/scrypt/bcrypt/PBKDF2 with safe parameters).

## Workflow (quick triage)

1. Identify goal: confidentiality, integrity, authenticity, key agreement, or password storage.
2. Define context: data at rest/in transit, threat model, adversary capabilities, compliance constraints.
3. Choose construction: standard protocol or library recipe; avoid piecemeal assembly.
4. Validate key/nonce/IV handling, randomness, encoding, and storage.
5. Plan migrations and versioning; avoid silent behavior changes.

## Core Best Practices

### Key and Nonce/IV Management
- Never reuse a nonce/IV with the same key (AEAD or CTR/GCM/ChaCha20). Require uniqueness.
- Use cryptographically secure RNGs from the OS; never `rand()` or timestamps.
- Separate keys by purpose (encryption vs MAC vs signing); derive via HKDF if needed.
- Zeroize sensitive material when feasible; avoid logging secrets.

### Encryption
- Use AEAD for authenticated encryption; avoid "encrypt-then-hash" DIY unless a standard mandates.
- Specify and validate: algorithm, key size, nonce length, tag length, and associated data.
- Never use ECB; avoid raw CBC unless you also use a standard authenticated construction.

### Hashing, MACs, Signatures
- Hash: SHA-256/512 for general-purpose; avoid MD5/SHA-1.
- MAC: HMAC with SHA-256/512 or AEAD tags for integrity.
- Signatures: Ed25519 or ECDSA with safe curves; manage key formats and validation.

### Passwords and Secrets
- Password storage: Argon2id preferred; scrypt/bcrypt acceptable with tuned parameters.
- Never encrypt passwords for storage; always hash with salt and appropriate KDF.
- Store keys in KMS/HSM or OS keychain; rotate and version keys.

### Protocols and Data Formats
- Prefer standard protocols (TLS, Noise, JOSE, age) rather than inventing formats.
- Include versioning and algorithm identifiers in ciphertext metadata.
- Encode data unambiguously (base64/hex); avoid ad-hoc string concatenation.

## Red Flags (call out explicitly)

- "We can just roll our own crypto"
- "Reuse the IV/nonce to save space"
- "Encrypt then sign with the same key"
- "Use MD5/SHA-1 because it's faster"
- "Store passwords encrypted so we can recover them"
- "Hardcode keys or keep them in repo"

## Questions to Ask (when underspecified)

- What is the security goal and threat model?
- What data is sensitive, how long must it be protected, and who are the adversaries?
- Is there a standard protocol or library recipe already required?
- How are keys generated, stored, rotated, and revoked?
- What are the performance and compatibility constraints?

## Output Guidelines

- Lead with a concise assessment of risk and the safest recommended construction.
- Provide migration-safe advice (versioning, dual-write/dual-read, staged rollout).
- If recommending parameters, explain their trade-offs and compatibility constraints.
- When deprecations or standards might have changed, verify before final guidance.

## References

Use these files to keep answers concise and consistent:
- `skills/crypto-expert/references/pitfalls.md` for red flags during review.
- `skills/crypto-expert/references/recipes.md` for goal-based constructions and compliance notes.
