---
name: security-engineer
description: Application security, vulnerability assessment ve secure coding practices için kullanılır. Authentication, authorization, OWASP Top 10 ve security audit konularında uzman.
---

# Security Engineer Skill

Application security, vulnerability assessment ve secure coding practices.

## When to Use

- Security code review yaparken
- Vulnerability scanning sonuçlarını değerlendirirken
- Security best practices implementasyonu yaparken
- Authentication ve authorization geliştirirken
- Data encryption implementasyonu yaparken
- Security audit compliance kontrolleri yaparken
- OWASP Top 10 güvenlik açıklarını önlerken
- Input validation ve output encoding yaparken

## Instructions

### Görevler
- Security code reviews
- Vulnerability scanning
- Security best practices implementation
- Authentication ve authorization
- Data encryption
- Security audit compliance
- OWASP Top 10 prevention

### Kurallar
- Input validation all user input
- Output encoding (prevent XSS)
- SQL injection prevention
- CSRF protection
- Secure headers (CSP, HSTS, X-Frame-Options)
- Secret management
- Regular security audits

### OWASP Top 10 Prevention

```markdown
1. Injection (SQL, NoSQL, Command)
   - Use parameterized queries
   - Validate and sanitize input
   - Use ORMs where possible

2. Broken Authentication
   - Implement proper session management
   - Use secure password hashing (bcrypt)
   - Implement MFA where possible

3. Sensitive Data Exposure
   - Encrypt data at rest and in transit
   - Use HTTPS everywhere
   - Don't store sensitive data unnecessarily

4. XML External Entities (XXE)
   - Disable XML external entity processing
   - Use JSON instead of XML

5. Broken Access Control
   - Implement proper authorization checks
   - Use role-based access control (RBAC)
   - Deny by default

6. Security Misconfiguration
   - Disable debug mode in production
   - Remove default credentials
   - Keep dependencies updated

7. Cross-Site Scripting (XSS)
   - Encode output
   - Use Content Security Policy
   - Sanitize user input

8. Insecure Deserialization
   - Don't deserialize untrusted data
   - Use type checking
   - Implement integrity checks

9. Using Components with Known Vulnerabilities
   - Keep dependencies updated
   - Monitor security advisories
   - Use npm audit / Snyk

10. Insufficient Logging & Monitoring
    - Log security events
    - Implement alerting
    - Regular log review
```

### Secure Headers Configuration

```typescript
// next.config.ts
const securityHeaders = [
  {
    key: "X-DNS-Prefetch-Control",
    value: "on",
  },
  {
    key: "Strict-Transport-Security",
    value: "max-age=63072000; includeSubDomains; preload",
  },
  {
    key: "X-Frame-Options",
    value: "DENY",
  },
  {
    key: "X-Content-Type-Options",
    value: "nosniff",
  },
  {
    key: "X-XSS-Protection",
    value: "1; mode=block",
  },
  {
    key: "Referrer-Policy",
    value: "strict-origin-when-cross-origin",
  },
  {
    key: "Permissions-Policy",
    value: "camera=(), microphone=(), geolocation=()",
  },
  {
    key: "Content-Security-Policy",
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://*.supabase.co;
      frame-ancestors 'none';
    `.replace(/\s+/g, " ").trim(),
  },
];

export default {
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: securityHeaders,
      },
    ];
  },
};
```

### Input Validation Example

```typescript
import { z } from "zod";
import DOMPurify from "dompurify";

// Schema with strict validation
const userInputSchema = z.object({
  name: z
    .string()
    .min(2, "İsim en az 2 karakter olmalı")
    .max(100, "İsim en fazla 100 karakter olabilir")
    .regex(/^[a-zA-ZğüşıöçĞÜŞİÖÇ\s]+$/, "Sadece harf ve boşluk kullanılabilir"),
  email: z
    .string()
    .email("Geçerli bir e-posta adresi girin")
    .max(255),
  phone: z
    .string()
    .regex(/^\+?[0-9]{10,15}$/, "Geçerli bir telefon numarası girin")
    .optional(),
});

// Sanitize HTML content
function sanitizeHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ["b", "i", "em", "strong", "p", "br"],
    ALLOWED_ATTR: [],
  });
}

// SQL injection prevention - use parameterized queries
async function getMemberById(id: number) {
  // GOOD: Parameterized query
  const { data, error } = await supabase
    .from("members")
    .select("*")
    .eq("id", id)
    .single();

  // BAD: String concatenation (NEVER DO THIS)
  // const query = `SELECT * FROM members WHERE id = ${id}`;
}
```

### Authentication Security

```typescript
import { createServerClient } from "@supabase/ssr";
import { cookies } from "next/headers";

// Secure session validation
export async function getSession() {
  const cookieStore = await cookies();
  
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll();
        },
      },
    }
  );

  const { data: { session }, error } = await supabase.auth.getSession();
  
  if (error || !session) {
    return null;
  }

  // Verify session is not expired
  const now = Math.floor(Date.now() / 1000);
  if (session.expires_at && session.expires_at < now) {
    return null;
  }

  return session;
}

// Rate limiting for auth endpoints
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, "1 m"), // 5 attempts per minute
});

export async function rateLimitCheck(identifier: string) {
  const { success, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error("Çok fazla deneme yapıldı. Lütfen bekleyin.");
  }
  
  return { remaining };
}
```

### Security Audit Checklist

```markdown
- [ ] All user inputs are validated
- [ ] SQL injection prevention verified
- [ ] XSS prevention implemented
- [ ] CSRF tokens used for state-changing operations
- [ ] Authentication properly implemented
- [ ] Authorization checks on all endpoints
- [ ] Sensitive data encrypted
- [ ] Security headers configured
- [ ] Dependencies are up-to-date
- [ ] No secrets in code or logs
- [ ] Error messages don't expose sensitive info
- [ ] Rate limiting implemented
- [ ] Audit logging enabled
```
