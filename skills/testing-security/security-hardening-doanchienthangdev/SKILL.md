---
name: security-hardening
description: Advanced security patterns including zero-trust architecture, secret management, CSP, and compliance frameworks
category: security
triggers:
  - security hardening
  - zero trust
  - secret management
  - csp
  - security headers
  - compliance
  - penetration testing
---

# Security Hardening

Implement **advanced security patterns** beyond basic OWASP guidelines. This skill covers zero-trust architecture, secret management, security headers, and compliance frameworks.

## Purpose

Build defense-in-depth for production systems:

- Implement zero-trust security model
- Manage secrets securely with Vault
- Configure comprehensive security headers
- Design Content Security Policy (CSP)
- Prepare for security audits
- Meet compliance requirements (SOC2, GDPR)

## Features

### 1. Zero-Trust Architecture

```typescript
// Zero-trust middleware chain
import { Request, Response, NextFunction } from 'express';

// 1. Verify identity on every request
async function verifyIdentity(req: Request, res: Response, next: NextFunction) {
  const token = extractToken(req);

  if (!token) {
    return res.status(401).json({ error: 'Authentication required' });
  }

  try {
    // Verify token signature and expiration
    const decoded = await verifyToken(token);

    // Check if token is revoked
    if (await isTokenRevoked(decoded.jti)) {
      return res.status(401).json({ error: 'Token revoked' });
    }

    // Verify user still exists and is active
    const user = await getUserById(decoded.sub);
    if (!user || user.status !== 'active') {
      return res.status(401).json({ error: 'User not found or inactive' });
    }

    req.user = user;
    req.tokenClaims = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// 2. Verify authorization for every action
async function verifyAuthorization(resource: string, action: string) {
  return (req: Request, res: Response, next: NextFunction) => {
    const allowed = checkPermission(req.user, resource, action);

    if (!allowed) {
      auditLog({
        event: 'authorization_denied',
        userId: req.user.id,
        resource,
        action,
        ip: req.ip,
      });

      return res.status(403).json({ error: 'Access denied' });
    }

    next();
  };
}

// 3. Verify device/context
async function verifyContext(req: Request, res: Response, next: NextFunction) {
  const deviceFingerprint = req.headers['x-device-fingerprint'];
  const geoLocation = await getGeoLocation(req.ip);

  // Check for unusual patterns
  const risk = await assessRisk({
    userId: req.user.id,
    deviceFingerprint,
    geoLocation,
    userAgent: req.headers['user-agent'],
    timestamp: Date.now(),
  });

  if (risk.score > 0.8) {
    // Require step-up authentication
    return res.status(403).json({
      error: 'Additional verification required',
      challenge: risk.challengeType,
    });
  }

  if (risk.score > 0.5) {
    // Log suspicious activity
    auditLog({
      event: 'suspicious_activity',
      userId: req.user.id,
      riskScore: risk.score,
      factors: risk.factors,
    });
  }

  next();
}

// Apply zero-trust chain
app.use('/api', verifyIdentity, verifyContext);
app.get('/api/users/:id', verifyAuthorization('users', 'read'), getUser);
app.put('/api/users/:id', verifyAuthorization('users', 'write'), updateUser);
```

### 2. Secret Management with Vault

```typescript
import Vault from 'node-vault';

class SecretManager {
  private vault: Vault.client;
  private cache: Map<string, { value: any; expires: number }> = new Map();
  private cacheTTL = 300000; // 5 minutes

  constructor() {
    this.vault = Vault({
      apiVersion: 'v1',
      endpoint: process.env.VAULT_ADDR,
      token: process.env.VAULT_TOKEN,
    });
  }

  async getSecret(path: string): Promise<any> {
    // Check cache
    const cached = this.cache.get(path);
    if (cached && cached.expires > Date.now()) {
      return cached.value;
    }

    // Fetch from Vault
    const response = await this.vault.read(`secret/data/${path}`);
    const secret = response.data.data;

    // Cache the secret
    this.cache.set(path, {
      value: secret,
      expires: Date.now() + this.cacheTTL,
    });

    return secret;
  }

  async getDatabaseCredentials(): Promise<{ username: string; password: string }> {
    // Dynamic database credentials
    const response = await this.vault.read('database/creds/my-role');
    return {
      username: response.data.username,
      password: response.data.password,
    };
  }

  async rotateSecret(path: string, newValue: any): Promise<void> {
    await this.vault.write(`secret/data/${path}`, {
      data: newValue,
    });

    // Invalidate cache
    this.cache.delete(path);
  }

  // AWS IAM credentials via Vault
  async getAWSCredentials(): Promise<AWS.Credentials> {
    const response = await this.vault.read('aws/creds/my-role');
    return {
      accessKeyId: response.data.access_key,
      secretAccessKey: response.data.secret_key,
      sessionToken: response.data.security_token,
    };
  }
}

// Environment variable encryption
import { createCipheriv, createDecipheriv, randomBytes, scrypt } from 'crypto';

async function encryptEnvFile(envContent: string, password: string): Promise<string> {
  const salt = randomBytes(16);
  const key = await new Promise<Buffer>((resolve, reject) => {
    scrypt(password, salt, 32, (err, key) => {
      if (err) reject(err);
      else resolve(key);
    });
  });

  const iv = randomBytes(16);
  const cipher = createCipheriv('aes-256-gcm', key, iv);

  let encrypted = cipher.update(envContent, 'utf8', 'hex');
  encrypted += cipher.final('hex');

  const authTag = cipher.getAuthTag();

  return JSON.stringify({
    salt: salt.toString('hex'),
    iv: iv.toString('hex'),
    authTag: authTag.toString('hex'),
    encrypted,
  });
}
```

### 3. Security Headers

```typescript
import helmet from 'helmet';

// Comprehensive security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        "'strict-dynamic'",
        (req, res) => `'nonce-${res.locals.nonce}'`,
      ],
      styleSrc: ["'self'", "'unsafe-inline'"], // Consider using nonces for styles too
      imgSrc: ["'self'", "data:", "https:"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      connectSrc: ["'self'", "https://api.example.com"],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
      upgradeInsecureRequests: [],
      blockAllMixedContent: [],
    },
    reportOnly: false,
  },
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: { policy: 'same-origin' },
  crossOriginResourcePolicy: { policy: 'same-origin' },
  dnsPrefetchControl: { allow: false },
  frameguard: { action: 'deny' },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  ieNoOpen: true,
  noSniff: true,
  originAgentCluster: true,
  permittedCrossDomainPolicies: { permittedPolicies: 'none' },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  xssFilter: true,
}));

// CSP nonce generation
app.use((req, res, next) => {
  res.locals.nonce = crypto.randomBytes(16).toString('base64');
  next();
});

// Custom security headers
app.use((req, res, next) => {
  // Prevent caching of sensitive responses
  if (req.path.startsWith('/api/')) {
    res.set('Cache-Control', 'no-store, no-cache, must-revalidate, private');
    res.set('Pragma', 'no-cache');
    res.set('Expires', '0');
  }

  // Feature policy / Permissions policy
  res.set('Permissions-Policy',
    'accelerometer=(), camera=(), geolocation=(), gyroscope=(), ' +
    'magnetometer=(), microphone=(), payment=(), usb=()'
  );

  // Clear site data on logout
  if (req.path === '/logout') {
    res.set('Clear-Site-Data', '"cache", "cookies", "storage"');
  }

  next();
});

// CSP violation reporting
app.post('/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
  const report = req.body['csp-report'];

  logger.warn({
    type: 'csp_violation',
    violatedDirective: report['violated-directive'],
    blockedUri: report['blocked-uri'],
    documentUri: report['document-uri'],
    sourceFile: report['source-file'],
    lineNumber: report['line-number'],
  });

  res.status(204).end();
});
```

### 4. Input Validation & Sanitization

```typescript
import { z } from 'zod';
import DOMPurify from 'isomorphic-dompurify';
import sqlstring from 'sqlstring';

// Strict input validation schemas
const UserInputSchema = z.object({
  email: z.string().email().max(255).toLowerCase(),
  name: z.string()
    .min(2)
    .max(100)
    .regex(/^[a-zA-Z\s'-]+$/, 'Invalid characters in name'),
  phone: z.string()
    .regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number')
    .optional(),
  bio: z.string().max(1000).optional(),
});

// HTML sanitization for rich text
function sanitizeHTML(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
    ALLOW_DATA_ATTR: false,
  });
}

// SQL injection prevention (for raw queries)
function sanitizeSQL(input: string): string {
  return sqlstring.escape(input);
}

// Path traversal prevention
function sanitizePath(userPath: string, basePath: string): string {
  const resolvedPath = path.resolve(basePath, userPath);

  if (!resolvedPath.startsWith(path.resolve(basePath))) {
    throw new Error('Path traversal detected');
  }

  return resolvedPath;
}

// File upload validation
const ALLOWED_MIME_TYPES = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

async function validateFileUpload(file: Express.Multer.File): Promise<void> {
  // Check file size
  if (file.size > MAX_FILE_SIZE) {
    throw new Error('File too large');
  }

  // Verify MIME type using magic bytes
  const fileType = await import('file-type');
  const type = await fileType.fromBuffer(file.buffer);

  if (!type || !ALLOWED_MIME_TYPES.includes(type.mime)) {
    throw new Error('Invalid file type');
  }

  // Check for malicious content in images
  if (type.mime.startsWith('image/')) {
    await scanImageForMalware(file.buffer);
  }
}
```

### 5. Audit Logging

```typescript
interface AuditEvent {
  timestamp: Date;
  eventType: string;
  userId?: string;
  resourceType?: string;
  resourceId?: string;
  action: string;
  outcome: 'success' | 'failure';
  ipAddress: string;
  userAgent?: string;
  details?: Record<string, any>;
  requestId?: string;
}

class AuditLogger {
  async log(event: AuditEvent): Promise<void> {
    // Store in append-only audit log
    await db.auditLog.create({
      data: {
        ...event,
        timestamp: event.timestamp || new Date(),
        hash: this.calculateHash(event), // Tamper detection
      },
    });

    // Send to SIEM for real-time analysis
    await this.sendToSIEM(event);
  }

  private calculateHash(event: AuditEvent): string {
    const content = JSON.stringify({
      ...event,
      timestamp: event.timestamp.toISOString(),
    });

    return crypto
      .createHmac('sha256', process.env.AUDIT_SECRET!)
      .update(content)
      .digest('hex');
  }

  private async sendToSIEM(event: AuditEvent): Promise<void> {
    // Send to security information and event management system
    await fetch(process.env.SIEM_ENDPOINT!, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(event),
    });
  }
}

// Audit middleware
function auditMiddleware(action: string, resourceType: string) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const startTime = Date.now();

    res.on('finish', async () => {
      await auditLogger.log({
        timestamp: new Date(),
        eventType: 'api_call',
        userId: req.user?.id,
        resourceType,
        resourceId: req.params.id,
        action,
        outcome: res.statusCode < 400 ? 'success' : 'failure',
        ipAddress: req.ip,
        userAgent: req.headers['user-agent'],
        requestId: req.headers['x-request-id'] as string,
        details: {
          method: req.method,
          path: req.path,
          statusCode: res.statusCode,
          duration: Date.now() - startTime,
        },
      });
    });

    next();
  };
}

// Security-sensitive actions require audit
app.delete('/api/users/:id',
  auditMiddleware('delete', 'user'),
  deleteUser
);

app.post('/api/admin/roles',
  auditMiddleware('create', 'role'),
  createRole
);
```

### 6. Compliance Frameworks

```typescript
// GDPR compliance helpers
class GDPRCompliance {
  // Right to access - export user data
  async exportUserData(userId: string): Promise<UserDataExport> {
    const user = await db.user.findUnique({
      where: { id: userId },
      include: {
        orders: true,
        addresses: true,
        preferences: true,
        activityLog: true,
      },
    });

    return {
      personalData: {
        name: user.name,
        email: user.email,
        phone: user.phone,
        createdAt: user.createdAt,
      },
      orders: user.orders,
      addresses: user.addresses,
      preferences: user.preferences,
      activityLog: user.activityLog,
      exportedAt: new Date(),
    };
  }

  // Right to erasure - delete user data
  async eraseUserData(userId: string): Promise<void> {
    await db.$transaction([
      db.activityLog.deleteMany({ where: { userId } }),
      db.address.deleteMany({ where: { userId } }),
      db.preference.deleteMany({ where: { userId } }),
      // Anonymize orders (keep for accounting)
      db.order.updateMany({
        where: { userId },
        data: { userId: null, customerName: 'DELETED' },
      }),
      db.user.delete({ where: { id: userId } }),
    ]);

    await auditLogger.log({
      timestamp: new Date(),
      eventType: 'gdpr_erasure',
      userId,
      action: 'data_erasure',
      outcome: 'success',
    });
  }

  // Data retention enforcement
  async enforceRetention(): Promise<void> {
    const retentionPolicies = [
      { table: 'activityLog', retentionDays: 90 },
      { table: 'sessions', retentionDays: 30 },
      { table: 'auditLog', retentionDays: 365 * 7 }, // 7 years for compliance
    ];

    for (const policy of retentionPolicies) {
      const cutoff = new Date();
      cutoff.setDate(cutoff.getDate() - policy.retentionDays);

      await db[policy.table].deleteMany({
        where: { createdAt: { lt: cutoff } },
      });
    }
  }
}

// PCI-DSS compliance
const pciCompliance = {
  // Never store CVV
  sanitizePaymentData(data: PaymentData): SanitizedPaymentData {
    return {
      cardLastFour: data.cardNumber.slice(-4),
      cardBrand: detectCardBrand(data.cardNumber),
      expiryMonth: data.expiryMonth,
      expiryYear: data.expiryYear,
      // CVV is NEVER stored
    };
  },

  // Mask PAN in logs
  maskPAN(pan: string): string {
    return pan.slice(0, 6) + '******' + pan.slice(-4);
  },
};
```

## Use Cases

### 1. API Security Hardening

```typescript
// Complete API security setup
app.use(helmet());
app.use(rateLimiter);
app.use(verifyIdentity);
app.use(auditMiddleware);
```

### 2. Secure File Handling

```typescript
// Secure file upload endpoint
app.post('/upload', authenticate, async (req, res) => {
  await validateFileUpload(req.file);
  const safePath = sanitizePath(req.body.path, UPLOAD_DIR);
  // ... process file
});
```

## Best Practices

### Do's

- **Defense in depth** - Multiple layers of security
- **Principle of least privilege** - Minimal permissions
- **Regular security audits** - Penetration testing
- **Automated vulnerability scanning** - CI/CD integration
- **Incident response planning** - Document procedures
- **Security training** - Educate team members

### Don'ts

- Don't store secrets in code
- Don't trust client input
- Don't expose stack traces
- Don't use deprecated crypto
- Don't skip security headers
- Don't ignore security alerts

## Related Skills

- **owasp** - Security fundamentals
- **oauth** - Authentication patterns
- **defense-in-depth** - Layered security

## Reference Resources

- [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [HashiCorp Vault](https://www.vaultproject.io/docs)
- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [SOC 2 Compliance](https://www.aicpa.org/soc2)
