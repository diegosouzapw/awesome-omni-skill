---
name: security-validation-checklist
description: Guides security validation checklist: Signal protocol security, encryption standards, authentication patterns, data protection. Use when validating security, reviewing security implementations, or ensuring security compliance.
---

# Security Validation Checklist

## Security Components

### Authentication Security
- [ ] Secure authentication implemented (OAuth 2.0)
- [ ] Token security validated
- [ ] Session management secure
- [ ] Password hashing validated (SHA-256)

### Encryption Security
- [ ] Signal protocol encryption validated
- [ ] Key management secure
- [ ] Data encryption at rest
- [ ] Data encryption in transit

### Access Control
- [ ] User authentication required
- [ ] Role-based access control
- [ ] Permission checks in place
- [ ] Data access restrictions enforced

### Privacy Protection
- [ ] Personal data anonymized
- [ ] No PII in logs
- [ ] Secure data storage
- [ ] GDPR compliance verified

## Validation Pattern

```dart
/// Security Validator
/// 
/// Validates security implementations
class SecurityValidator {
  /// Validate authentication security
  Future<SecurityResult> validateAuthenticationSecurity() async {
    // Check password hashing
    final passwordHash = sha256.convert(utf8.encode('test'));
    final isPasswordHashingValid = passwordHash.bytes.length == 32;
    
    // Check session management
    final isSessionManagementValid = await _checkSessionManagement();
    
    // Check token security
    final isTokenSecurityValid = await _checkTokenSecurity();
    
    return SecurityResult(
      isCompliant: isPasswordHashingValid && 
                   isSessionManagementValid && 
                   isTokenSecurityValid,
      details: 'Authentication security validation',
    );
  }
  
  /// Validate encryption security
  Future<SecurityResult> validateEncryptionSecurity() async {
    // Check Signal protocol implementation
    // Check key management
    // Check encryption at rest
    // Check encryption in transit
  }
}
```

## Security Checklist

### Authentication
- [ ] OAuth 2.0 implementation
- [ ] Secure token storage
- [ ] Session expiration
- [ ] Password hashing (SHA-256)

### Encryption
- [ ] Signal protocol validated
- [ ] Key rotation implemented
- [ ] Secure key storage
- [ ] End-to-end encryption

### Access Control
- [ ] Authentication required
- [ ] Role-based permissions
- [ ] Data access restrictions
- [ ] Audit logging

### Privacy
- [ ] Data anonymization
- [ ] No PII in logs
- [ ] Secure storage
- [ ] GDPR compliance

## Reference

- `lib/core/services/security_validator.dart` - Security validation service
- `test/security/` - Security test suite
- `docs/security/` - Security documentation
