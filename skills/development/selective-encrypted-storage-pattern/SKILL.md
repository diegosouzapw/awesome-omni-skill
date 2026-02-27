---
name: selective-encrypted-storage-pattern
description: Security pattern for field-level encryption at rest. Use when encrypting specific sensitive data fields before storage, implementing application-level encryption for databases, or when only certain data elements need encryption at rest. Addresses "Leak data at rest" problem.
---

# Selective Encrypted Storage Security Pattern

Application explicitly encrypts specific sensitive data elements before persisting them to storage. Application controls which data is encrypted and manages encryption operations.

## Problem Addressed

**Leak data at rest**: Sensitive data stored in databases, files, or other storage could be accessed by unauthorized parties (database breach, stolen backups, etc.).

## Core Components

| Role | Type | Responsibility |
|------|------|----------------|
| **Application** | Entity | Decides what to encrypt, invokes encryption |
| **Cryptographer** | Cryptographic Primitive | Performs encryption/decryption |
| **Storage** | Storage | Persists data (encrypted and plaintext) |

### Data Elements

- **d**: Plaintext sensitive data
- **{d}_k**: Ciphertext
- **keyInfo**: Key identification/material
- **config**: Cipher configuration

## Pattern Flow

### Storage
```
Application → [encrypt(d, keyInfo, config)] → Cryptographer
Cryptographer → [{d}_k] → Application
Application → [store({d}_k)] → Storage
```

### Retrieval
```
Application → [retrieve] → Storage
Storage → [{d}_k] → Application
Application → [decrypt({d}_k, keyInfo, config)] → Cryptographer
Cryptographer → [d] → Application
```

## Key Characteristics

### Application-Controlled
- Application decides WHAT data to encrypt
- Application invokes encryption before storage
- Application invokes decryption after retrieval

### Field-Level Granularity
- Encrypt specific fields (SSN, credit cards, etc.)
- Non-sensitive data stored plaintext
- Enables partial data access

### Key Per Data Type
- Different keys for different sensitivity levels
- Key compromise limits exposure
- Supports key rotation per data category

## When to Use

### Use Selective Encryption When:
- Only specific fields are sensitive
- Different data needs different protection levels
- Need to query non-sensitive fields
- Application must control encryption

### Consider Transparent Encryption When:
- All data equally sensitive
- Simpler implementation preferred
- Database/filesystem encryption sufficient

## Security Considerations

### Key Management Critical
- Keys separate from encrypted data
- Use Key Management Service (KMS) or HSM
- Implement key rotation
- Audit key access

### Algorithm Selection
- AES-256-GCM (authenticated encryption)
- RSA-3072+ for key encryption
- Follow Encryption pattern guidelines

### What to Encrypt
Typically encrypt:
- Personally Identifiable Information (PII)
- Payment card data
- Health information
- Authentication credentials
- Cryptographic keys

### Index/Search Challenges
Encrypted data cannot be:
- Searched directly
- Indexed efficiently
- Sorted

Solutions:
- Blind indexes (hash-based)
- Searchable encryption (advanced)
- Encrypt only display fields, index separately

### Data Flow Analysis
Trace plaintext through entire flow:
- Application memory
- Logs (never log plaintext!)
- Caches
- Temporary files
- Error messages
- Backups

### Performance Impact
- Encryption/decryption adds latency
- Consider caching decrypted values (securely)
- Batch operations where possible

## Implementation Approaches

### Application-Level
```
// Before storage
encryptedSSN = encrypt(ssn, ssnKey)
db.store(record with encryptedSSN)

// After retrieval
record = db.retrieve()
ssn = decrypt(record.encryptedSSN, ssnKey)
```

### ORM/Framework Integration
Many frameworks support field-level encryption:
- Django encrypted fields
- Hibernate encryption
- ActiveRecord attr_encrypted

### Database Features
Some databases offer column-level encryption:
- SQL Server Always Encrypted
- Oracle TDE column encryption
- PostgreSQL pgcrypto

## Key Rotation Strategy

1. Generate new key
2. Re-encrypt data with new key (background)
3. Update key reference
4. Deprecate old key
5. Eventually delete old key

Consider:
- Dual-key period during rotation
- Performance impact of mass re-encryption
- Backup/restore implications

## Implementation Checklist

- [ ] Identified all sensitive data fields
- [ ] Using strong algorithm (AES-256-GCM)
- [ ] Keys stored separately from data
- [ ] Key management system in place
- [ ] Key rotation procedure defined
- [ ] Plaintext never logged
- [ ] Caches secured
- [ ] Backup encryption addressed
- [ ] Search/index strategy defined
- [ ] Performance tested

## Related Patterns

- Transparent encrypted storage (alternative: encrypt everything)
- Encryption (underlying operations)
- Cryptographic key management (key handling)
- Selective encrypted transmission (encryption in transit)

## References

- Source: https://securitypatterns.distrinet-research.be/patterns/07_01_001__selective_encrypted_storage/
- OWASP Cryptographic Storage Cheat Sheet
