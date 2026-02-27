---
name: security-patterns
description: Zero-trust security patterns for frontend and backend
triggers:
  - security
  - auth
  - authentication
  - authorization
  - token
  - XSS
  - OWASP
---

# Security Patterns Skill

Enforce zero-trust security across frontend and backend.

## When to Use

Automatically activate when:
- Implementing authentication/authorization
- Handling tokens or sessions
- Writing API endpoints
- Storing sensitive data

---

## Zero Trust Principles

> **Never Trust, Always Verify, Least Privilege**

1. Every request is authenticated
2. Every action is authorized
3. Minimal permissions granted
4. All data encrypted at rest and in transit
5. Complete audit trail

---

## Frontend Security (Angular)

### Token Storage — Memory ONLY

```typescript
// ✅ CORRECT - Store in memory
@Injectable({ providedIn: 'root' })
export class AuthService {
  private accessToken = signal<string | null>(null);
  private refreshToken = signal<string | null>(null);

  setTokens(access: string, refresh: string) {
    this.accessToken.set(access);
    this.refreshToken.set(refresh);
  }

  clearTokens() {
    this.accessToken.set(null);
    this.refreshToken.set(null);
  }
}

// ❌ WRONG - Never use localStorage
localStorage.setItem('token', token);  // FORBIDDEN
sessionStorage.setItem('token', token);  // FORBIDDEN
```

### XSS Prevention

```typescript
// ✅ Use Angular's built-in sanitization
import { DomSanitizer } from '@angular/platform-browser';

@Component({...})
class SafeComponent {
  private sanitizer = inject(DomSanitizer);

  // Only when absolutely necessary
  trustedHtml = this.sanitizer.bypassSecurityTrustHtml(userContent);
}

// ❌ WRONG - Never bypass without sanitization
[innerHTML]="userContent"  // DANGEROUS
```

### Auto-Logout on Inactivity

```typescript
@Injectable({ providedIn: 'root' })
export class InactivityService {
  private readonly TIMEOUT_MS = 15 * 60 * 1000; // 15 minutes
  private timeoutId?: number;

  resetTimer() {
    clearTimeout(this.timeoutId);
    this.timeoutId = window.setTimeout(() => this.logout(), this.TIMEOUT_MS);
  }

  private logout() {
    this.authService.clearTokens();
    this.router.navigate(['/login']);
  }
}
```

### Clear PII on Logout

```typescript
logout() {
  // Clear all sensitive data
  this.accessToken.set(null);
  this.refreshToken.set(null);
  this.userProfile.set(null);
  
  // Clear IndexedDB
  await this.indexedDb.clear();
  
  // Clear service worker cache
  if ('serviceWorker' in navigator) {
    const caches = await window.caches.keys();
    await Promise.all(caches.map(c => window.caches.delete(c)));
  }
}
```

---

## Backend Security (Spring Boot)

### Method-Level Authorization

```java
@RestController
@RequestMapping("/api/v1/learners")
@RequiredArgsConstructor
public class LearnerController {

    @GetMapping
    @PreAuthorize("hasAnyRole('TEACHER', 'ADMIN')")
    public List<LearnerResponse> listLearners() { ... }

    @PostMapping
    @PreAuthorize("hasRole('ADMIN')")
    public LearnerResponse createLearner(@Valid @RequestBody request) { ... }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('SUPER_ADMIN')")
    public void deleteLearner(@PathVariable UUID id) { ... }
}
```

### PII Masking in Responses

```java
public record LearnerResponse(
    UUID id,
    @JsonSerialize(using = MaskedStringSerializer.class)
    String lrn,          // Returns: "****8901"
    String firstName,
    @JsonSerialize(using = MaskedStringSerializer.class)
    String birthDate     // Returns: "****-**-15"
) {}
```

### Audit Logging

```java
@Aspect
@Component
public class AuditAspect {

    @Around("@annotation(Auditable)")
    public Object audit(ProceedingJoinPoint pjp) throws Throwable {
        AuditLog log = AuditLog.builder()
            .userId(SecurityContextHolder.getContext().getUserId())
            .tenantId(TenantContext.getTenantId())
            .action(pjp.getSignature().getName())
            .timestamp(Instant.now())
            .build();

        try {
            Object result = pjp.proceed();
            log.setStatus("SUCCESS");
            return result;
        } catch (Exception e) {
            log.setStatus("FAILURE");
            log.setError(e.getMessage());
            throw e;
        } finally {
            auditRepository.save(log);
        }
    }
}
```

### Input Validation

```java
public record CreateLearnerRequest(
    @NotBlank
    @Size(max = 100)
    @Pattern(regexp = "^[a-zA-Z\\s]+$", message = "Only letters allowed")
    String firstName,

    @NotNull
    @Pattern(regexp = "\\d{12}", message = "LRN must be 12 digits")
    String lrn,

    @NotNull
    @Past(message = "Birth date must be in the past")
    LocalDate birthDate
) {}
```

---

## API Security

### OAuth2 + PKCE Flow

```
1. Client generates code_verifier (random string)
2. Client creates code_challenge = SHA256(code_verifier)
3. Client redirects to /authorize?code_challenge=...
4. User authenticates
5. Server returns authorization_code
6. Client exchanges code + code_verifier for tokens
7. Server validates SHA256(code_verifier) == code_challenge
```

### Rate Limiting

```java
@Configuration
public class RateLimitConfig {

    @Bean
    public RateLimiter rateLimiter() {
        return RateLimiter.of("api", RateLimiterConfig.custom()
            .limitForPeriod(100)           // 100 requests
            .limitRefreshPeriod(Duration.ofMinutes(1))
            .timeoutDuration(Duration.ofMillis(500))
            .build());
    }
}
```

---

## Data Encryption

### At Rest (Database)

```sql
-- Encrypt sensitive columns
CREATE TABLE learners (
    id UUID PRIMARY KEY,
    lrn TEXT,  -- Stored encrypted
    first_name TEXT,
    birth_date DATE
);

-- Use pgcrypto for column encryption
INSERT INTO learners (id, lrn, first_name, birth_date)
VALUES (
    gen_random_uuid(),
    pgp_sym_encrypt('123456789012', 'encryption-key'),
    'Juan',
    '2010-01-15'
);
```

### In Transit

- TLS 1.3 for all connections
- mTLS between services
- Certificate pinning in mobile apps

---

## Security Headers (CSP)

```java
@Configuration
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) {
        return http
            .headers(headers -> headers
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives("default-src 'self'; script-src 'self'"))
                .frameOptions(frame -> frame.deny())
                .xssProtection(xss -> xss.enable()))
            .build();
    }
}
```

---

## Checklist

### Frontend
- [ ] Tokens stored in memory only
- [ ] XSS prevention (Angular sanitization + CSP)
- [ ] 15-minute inactivity auto-logout
- [ ] Clear all PII on logout
- [ ] Certificate pinning (mobile)

### Backend
- [ ] `@PreAuthorize` on all controller methods
- [ ] PII masking in responses
- [ ] Audit logging for all data access
- [ ] Input validation with Bean Validation
- [ ] Rate limiting per tenant

### Infrastructure
- [ ] TLS 1.3 everywhere
- [ ] HSM for key management
- [ ] WAF + DDoS protection
- [ ] SIEM/SOAR integration
