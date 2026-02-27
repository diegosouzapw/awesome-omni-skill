---
name: message-authentication-code-pattern
description: Security pattern for implementing Message Authentication Codes (MACs) to ensure data integrity and origin authentication. Use when implementing HMAC, CMAC, or other MAC algorithms, verifying message integrity, authenticating message origin with shared secrets, or when non-repudiation is NOT required. Specialization of Cryptographic action pattern.
---

# Message Authentication Code (MAC) Security Pattern

This pattern encapsulates common considerations for using Message Authentication Codes (MAC) to ensure the integrity of messages and authenticate the identity of the provider.

## What is a MAC?

A Message Authentication Code (MAC) is a tag computed from a message using a special hash function whose output depends on a secret cryptographic key. Generating a MAC requires:
1. The message itself
2. Possession of a secret key

Only parties possessing the agreed-upon secret key can generate and verify valid MACs.

## Properties Provided

Appending a MAC to a message provides two properties:

1. **Data Integrity**: Assurance that the received message is identical to the one used to calculate the MAC
2. **Data Origin Authentication**: Assurance of the identity of the party that originated the message (i.e., they possess the secret key)

## Properties NOT Provided

**Important Limitations**:
- Properties depend on **secrecy of the cryptographic key**
- Anyone possessing the key can generate valid MACs for any message
- An attacker with the secret can compute a new MAC after altering a message (undetectable)
- **MAC does NOT provide evidence that a specific party generated it** (all parties share the same key)
- **No non-repudiation**: Cannot prove which specific party created the MAC

**If non-repudiation is required**: Use digital signatures instead of MACs.

## Core Components

| Role | Type | Responsibility |
|------|------|----------------|
| **EntityA** | Entity | Wants to create a MAC for message(s) |
| **EntityB** | Entity | Wants to verify whether message was modified |
| **MAC Generator** | Cryptographic Primitive | Generates MAC for message and key |
| **MAC Verifier** | Cryptographic Primitive | Verifies message and MAC match |

**Note**: MAC Generator and MAC Verifier can be the same library instance. EntityA and EntityB can also be the same entity.

### Data Elements

- **m**: Message (plaintext data element or action request)
- **mac**: The generated MAC tag
- **keyInfo**: Information on the secret key
- **config**: Cipher configuration (optional)
- **confirmation**: Verification success indication
- **error**: Verification failure indication

### Actions

- **generate**: Create MAC for message m using secret key
- **verify**: Check if message m and MAC match using secret key

## Pattern Flow

### MAC Generation
```
EntityA → [generate(m, keyInfo)] → MAC Generator
MAC Generator → [mac] → EntityA
EntityA → [m + mac] → EntityB
```

### MAC Verification
```
EntityB → [verify(m, mac, keyInfo)] → MAC Verifier
MAC Verifier → [confirmation or error] → EntityB
```

The MAC Verifier checks whether the MAC generated from m using the given key is identical to the provided MAC. If so, confirms to EntityB; otherwise, returns error.

## Algorithm Recommendations

### Recommended Algorithms
- **HMAC-SHA-256** / **HMAC-SHA-384** / **HMAC-SHA-512**
- **AES-CMAC**
- **Blake2b**

### Deprecated/Avoid
- **HMAC-MD5**: Avoid except for verifying legacy MACs
- Ad hoc constructions (e.g., SHA(message || secret)): **Insecure - never use**

**Critical**: Always use dedicated MAC ciphers. Ad hoc constructions using unkeyed hash functions concatenated with secrets have been shown to be insecure.

### Key Length
- **Minimum 128 bits** for the secret key

### MAC Output Length

| Use Case | Recommended Length |
|----------|-------------------|
| Long-term (10+ years) | 256 bits |
| Standard (up to 10 years) | 128 bits |
| Short-lived (e.g., session tokens) | 64 bits minimum |

## Security Considerations

### Key Confidentiality and Integrity

A cryptographic key used to generate and verify MACs should:
- **Always be kept confidential**
- Have its **integrity protected**
- Be secured during persistent storage
- Be secured during transmission between entities

### Use Keys for Single Purpose

Specialization of Cryptographic action.Use keys for a single purpose:
- **Never use MAC keys for other purposes** (e.g., encryption)
- Dedicated key for MAC generation/verification only

### Design for Change

Specialization of Cryptographic action.Design for change:
- Algorithms may become deprecated
- Design for easy algorithm transitions

### Reuse Existing Libraries

Specialization of Cryptographic action.Reuse existing libraries:
- Use well-known cryptographic libraries
- Never implement custom MAC algorithms

### Authenticated Encryption Preference

**If protecting integrity of encrypted messages in transit**:
- Use **authenticated encryption** instead of separate encryption + MAC
- Authenticated encryption uses a single symmetric key for both confidentiality and authentication
- Separate encryption and MAC requires two distinct keys, complicating key management

## MAC vs. Digital Signature

| Aspect | MAC | Digital Signature |
|--------|-----|-------------------|
| Key type | Symmetric (shared secret) | Asymmetric (public/private) |
| Non-repudiation | No | Yes |
| Who can verify | Only key holders | Anyone with public key |
| Who can generate | Any key holder | Only private key holder |
| Performance | Faster | Slower |
| Use case | Internal integrity | External verification, legal |

## Implementation Checklist

- [ ] Using recommended algorithm (HMAC-SHA-256, AES-CMAC, Blake2b)
- [ ] **No HMAC-MD5** for new implementations
- [ ] **No ad hoc constructions** (hash(message || key))
- [ ] Secret key minimum 128 bits
- [ ] MAC output appropriate for use case (64-256 bits)
- [ ] Key used only for MAC (not encryption)
- [ ] Key confidentiality protected
- [ ] Key integrity protected
- [ ] Using authenticated encryption if combining with encryption
- [ ] Understood: MAC provides NO non-repudiation

## Related Patterns

- **Cryptographic action** (parent pattern)
- **Digital signature** (alternative when non-repudiation needed)
- **Encryption** (combine via authenticated encryption)
- **Cryptographic key management** (key handling)

## References

- Source: https://securitypatterns.distrinet-research.be/patterns/99_01_004__mac/
- P. C. van Oorschot, Computer Security and the Internet - Tools and Jewels, 2020
- E. Barker, 'Recommendation for Key Management: Part 1 – General', NIST SP 800-57 Part 1, May 2020
- Bundesamt für Sicherheit in der Informationstechnik, 'Cryptographic Mechanisms: Recommendations and Key Lengths', BSI TR-02102-1, Mar. 2020
- N. P. Smart et al., 'Algorithms, Key size and parameters report', ENISA, Nov. 2014
- S. Turner and L. Chen, 'Updated Security Considerations for the MD5 Message-Digest and the HMAC-MD5 Algorithms', IETF RFC 6151, Mar. 2011
- B. Preneel and P. C. van Oorschot, 'On the security of iterated message authentication codes', IEEE Transactions on Information Theory, vol. 45, no. 1, Jan. 1999
- Google Tink - Message Authentication Code (MAC): https://developers.google.com/tink/mac
