---
name: compliance-manager
description: Guardrails for edits to core/security/compliance-manager.js that preserve PCI/GDPR/PSD2/SOX/HIPAA controls (masking, encryption, SCA, consent checks, and audit logging). Use when changing compliance validators, security handling, or audit flows.
---

# Compliance Manager Guardian

## Purpose & Scope

Apply this skill when modifying `core/security/compliance-manager.js`.

The Compliance Manager provides:
- PCI-DSS data protection (card data masking, encryption)
- GDPR compliance (pseudonymization, consent management, data minimization)
- PSD2 compliance (Strong Customer Authentication)
- SOX audit trail requirements
- HIPAA health data protection
- Multi-regulation validation framework
- Secure audit logging

## Non-Negotiables (Never Do)

### Compliance Validators
- Never disable or bypass compliance validators.
- Never weaken validation rules (for example, making required checks optional).
- Never skip validation for "trusted" sources.
- Never add bypass flags or debug modes that skip compliance.

### PCI-DSS Rules
- Never log these PCI fields (even in debug mode):
  - `cvv`, `cvv2`, `cvc`, `cvc2`, `cid`, `cav2`
  - `pin`, `pinBlock`
  - `track1`, `track2`, `magneticStripe`
- Never weaken card masking:
  - Must show only first 6 and last 4 digits.
  - Middle digits must be masked with `*`.
- Never reduce encryption below AES-256-GCM.
- Never store CVV/PIN after authorization.

### GDPR Rules
- Never process personal data without consent check.
- Never skip pseudonymization for personal identifiers.
- Never retain personal data beyond retention period.
- Never disable data minimization for analytics.

### PSD2 Rules
- Never reduce SCA requirements below 2 factors.
- Never bypass SCA for amounts over threshold.
- Never skip transaction monitoring for high-value transactions.
- Never disable cumulative amount tracking.

### Audit Logging
- Never skip audit logging for sensitive operations.
- Never delete or modify existing audit entries.
- Never log sensitive data in audit trails (mask first).
- Never disable audit persistence.

### Security Rollback
- Never rollback security fixes without security team approval.
- Never lower security levels in production.

## Required Patterns (Must Follow)

### Card Number Masking
```javascript
// Must mask showing only first 6 and last 4
maskCardNumber(cardNumber) {
    const cleaned = cardNumber.replace(/\D/g, '');
    const first6 = cleaned.substring(0, 6);
    const last4 = cleaned.substring(cleaned.length - 4);
    const masked = '*'.repeat(cleaned.length - 10);
    return `${first6}${masked}${last4}`;
}
// Example: 4111111111111111 -> 411111******1111
```

### Data Encryption
```javascript
// Must use AES-256-GCM
encryptSensitiveData(data) {
    const algorithm = 'aes-256-gcm';  // Do not change
    const key = process.env.ENCRYPTION_KEY;
    if (!key) throw new Error('ENCRYPTION_KEY is required');

    // 12-byte IV is recommended for GCM
    const iv = crypto.randomBytes(12);

    // Prefer @onasis/security-sdk for key handling if available
    // If ENCRYPTION_KEY is a passphrase, derive a 32-byte key via scrypt.
    const keyBuf = (key.length === 64 && /^[0-9a-f]+$/i.test(key))
        ? Buffer.from(key, 'hex')
        : crypto.scryptSync(key, 'onasis-gateway', 32);

    const cipher = crypto.createCipheriv('aes-256-gcm', keyBuf, iv);
    cipher.setAAD(Buffer.from('compliance-encryption'));

    const plaintext = typeof data === 'string' ? data : JSON.stringify(data);
    const ciphertext = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
    const authTag = cipher.getAuthTag();

    return {
        encrypted: ciphertext.toString('base64'),
        iv: iv.toString('hex'),
        authTag: authTag.toString('hex'),
        algorithm
    };
}
```

### Strong Customer Authentication
```javascript
// Must require 2+ factors
validateSCA(data) {
    const factors = [];

    if (data.password || data.pin) factors.push('knowledge');
    if (data.deviceId || data.token) factors.push('possession');
    if (data.biometric || data.fingerprint) factors.push('inherence');

    return factors.length >= 2;  // PSD2 requirement
}
```

### Defense in Depth
```javascript
// Must apply all applicable protections
enforceDataHandling(serviceId, data, operation) {
    let processedData = { ...data };

    if (service?.compliance?.pci) {
        processedData = this.applyPCIProtections(processedData, operation);
    }
    if (service?.compliance?.gdpr) {
        processedData = this.applyGDPRProtections(processedData, operation);
    }
    if (service?.compliance?.psd2) {
        processedData = this.applyPSD2Protections(processedData, operation);
    }

    return processedData;
}
```

### Audit Entry Creation
```javascript
// Must create audit entry for all compliance events
logAuditEntry(action, details) {
    const entry = {
        timestamp: new Date(),
        action,
        details,
        id: crypto.randomUUID()
    };

    this.auditLog.push(entry);
    this.emit('audit:logged', entry);
    this.persistAuditEntry(entry);  // Must persist
}
```

## Prohibited Fields Registry

| Field | Regulation | Storage | Logging | Transmission |
|-------|------------|---------|---------|--------------|
| cvv, cvv2, cvc, cvc2 | PCI-DSS 3.2 | Never | Never | HTTPS only |
| pin, pinBlock | PCI-DSS 3.4 | Never | Never | Encrypted |
| track1, track2 | PCI-DSS 3.2 | Never | Never | Never |
| magneticStripe | PCI-DSS 3.2 | Never | Never | Never |
| Full card number | PCI-DSS 3.4 | Encrypted | Masked | Encrypted |

## Integration Points

| Component | Integration Method |
|-----------|-------------------|
| Base Client | Data passed through `enforceDataHandling()` |
| Metrics Collector | `compliance_violations_total` metric |
| API Routes | Middleware for request validation |
| Database | Audit entries persisted to `audit.compliance_log` |

## Compliance Validation Checklist

Before deploying changes:
- [ ] Card data properly masked (first 6, last 4 only).
- [ ] CVV/PIN never logged or stored.
- [ ] Encryption uses AES-256-GCM.
- [ ] SCA requires 2+ factors.
- [ ] Audit entries created for all operations.
- [ ] GDPR consent check in place.
- [ ] Data minimization applied for analytics.
- [ ] No PII in metric labels.
- [ ] Audit log persisted to secure storage.
