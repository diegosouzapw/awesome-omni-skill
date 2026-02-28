---
name: vibe-security
description: Write secure web applications following security best practices. Use when working on any web application to ensure OWASP compliance, input validation, authentication, and protection against XSS, CSRF, SSRF, and injection attacks.
license: MIT
compatibility: opencode
metadata:
  category: development
  version: "1.0"
---

# Secure Coding Guide for Web Applications

## Overview

This guide provides comprehensive secure coding practices for web applications. As an AI assistant, your role is to approach code from a **bug hunter's perspective** and make applications **as secure as possible** without breaking functionality.

```sudolang
SecurityPrinciples {
  Constraints {
    Defense in depth: Never rely on a single security control.
    Fail securely: When something fails, fail closed (deny access).
    Least privilege: Grant minimum permissions necessary.
    Input validation: Never trust user input, validate everything server-side.
    Output encoding: Encode data appropriately for the context it's rendered in.
  }
}
```

---

## Access Control Issues

Access control vulnerabilities occur when users can access resources or perform actions beyond their intended permissions.

### Core Requirements

```sudolang
AccessControl {
  User-Level Authorization
    require Each user must only access/modify their own data.
    require No user should access data from other users or organizations.
    require Always verify ownership at data layer, not just route level.

  Use UUIDs Instead of Sequential IDs
    require Use UUIDv4 or similar non-guessable identifiers.
    warn "Sequential IDs allowed only if explicitly requested by user"

  Account Lifecycle Handling
    require When user removed from organization: immediately revoke all access tokens and sessions.
    require When account deleted/deactivated: invalidate all active sessions and API keys.
    require Implement token revocation lists or short-lived tokens with refresh mechanisms.

  Constraints {
    Verify user owns the resource on every request (don't trust client-side data).
    Check organization membership for multi-tenant apps.
    Validate role permissions for role-based actions.
    Re-validate permissions after any privilege change.
    Check parent resource ownership (e.g., if accessing comment, verify user owns parent post).
  }
}
```

### Common Pitfalls to Avoid

- **IDOR (Insecure Direct Object Reference)**: Always verify the requesting user has permission to access the requested resource ID
- **Privilege Escalation**: Validate role changes server-side; never trust role info from client
- **Horizontal Access**: User A accessing User B's resources with the same privilege level
- **Vertical Access**: Regular user accessing admin functionality
- **Mass Assignment**: Filter which fields users can update; don't blindly accept all request body fields

### Implementation Pattern

```
# Pseudocode for secure resource access
function getResource(resourceId, currentUser):
    resource = database.find(resourceId)

    if resource is null:
        return 404  # Don't reveal if resource exists

    if resource.ownerId != currentUser.id:
        if not currentUser.hasOrgAccess(resource.orgId):
            return 404  # Return 404, not 403, to prevent enumeration

    return resource
```

---

## Client-Side Bugs

### Cross-Site Scripting (XSS)

Every input controllable by the user—whether directly or indirectly—must be sanitized against XSS.

#### Input Sources to Protect

**Direct Inputs:**

- Form fields (email, name, bio, comments, etc.)
- Search queries
- File names during upload
- Rich text editors / WYSIWYG content

**Indirect Inputs:**

- URL parameters and query strings
- URL fragments (hash values)
- HTTP headers used in the application (Referer, User-Agent if displayed)
- Data from third-party APIs displayed to users
- WebSocket messages
- postMessage data from iframes
- LocalStorage/SessionStorage values if rendered

**Often Overlooked:**

- Error messages that reflect user input
- PDF/document generators that accept HTML
- Email templates with user data
- Log viewers in admin panels
- JSON responses rendered as HTML
- SVG file uploads (can contain JavaScript)
- Markdown rendering (if allowing HTML)

#### Protection Strategies

```sudolang
XSSProtection {
  Output Encoding (Context-Specific)
    require HTML context: HTML entity encode (`<` to `&lt;`).
    require JavaScript context: JavaScript escape.
    require URL context: URL encode.
    require CSS context: CSS escape.
    require Use framework's built-in escaping (React's JSX, Vue's {{ }}, etc.).

  Input Sanitization
    require Use established libraries (DOMPurify for HTML).
    require Whitelist allowed tags/attributes for rich text.
    require Strip or encode dangerous patterns.

  CSP {
    template: """
      Content-Security-Policy:
        default-src 'self';
        script-src 'self';
        style-src 'self' 'unsafe-inline';
        img-src 'self' data: https:;
        font-src 'self';
        connect-src 'self' https://api.yourdomain.com;
        frame-ancestors 'none';
        base-uri 'self';
        form-action 'self';
    """

    Constraints {
      Avoid `'unsafe-inline'` and `'unsafe-eval'` for scripts.
      Use nonces or hashes for inline scripts when necessary.
      Report violations: `report-uri /csp-report`.
    }
  }

  AdditionalHeaders {
    require X-Content-Type-Options: nosniff.
    require X-Frame-Options: DENY (or use CSP frame-ancestors).
  }
}
```

---

### Cross-Site Request Forgery (CSRF)

Every state-changing endpoint must be protected against CSRF attacks.

#### Endpoints Requiring CSRF Protection

**Authenticated Actions:**

- All POST, PUT, PATCH, DELETE requests
- Any GET request that changes state (fix these to use proper HTTP methods)
- File uploads
- Settings changes
- Payment/transaction endpoints

**Pre-Authentication Actions:**

- Login endpoints (prevent login CSRF)
- Signup endpoints
- Password reset request endpoints
- Password change endpoints
- Email/phone verification endpoints
- OAuth callback endpoints

#### Protection Mechanisms

```sudolang
CSRFProtection {
  TokenRequirements {
    require Token is cryptographically random (use secure random generator).
    require Token is tied to user session.
    require Token is validated server-side on all state-changing requests.
    require Missing token = rejected request.
    require Token regenerated on authentication state change.
  }

  CookieSettings {
    require SameSite cookie attribute is set.
    require Secure flag on session cookies.
    require HttpOnly flag on session cookies.

    template: """
      Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly
    """

    SameSiteOptions {
      Strict: Cookie never sent cross-site (best security).
      Lax: Cookie sent on top-level navigations (good balance).
      warn "Always combine with CSRF tokens for defense in depth"
    }
  }

  DoubleSubmitPattern {
    Send CSRF token in both cookie and request body/header.
    Server validates they match.
  }
}
```

#### Edge Cases and Common Mistakes

- **Token presence check**: CSRF validation must NOT depend on whether the token is present, always require it
- **Token per form**: Consider unique tokens per form for sensitive operations
- **JSON APIs**: Don't assume JSON content-type prevents CSRF; validate Origin/Referer headers AND use tokens
- **CORS misconfiguration**: Overly permissive CORS can bypass SameSite cookies
- **Subdomains**: CSRF tokens should be scoped because subdomain takeover can lead to CSRF
- **Flash/PDF uploads**: Legacy browser plugins could bypass SameSite
- **GET requests with side effects**: Never perform state changes on GET
- **Token leakage**: Don't include CSRF tokens in URLs
- **Token in URL vs Header**: Prefer custom headers (X-CSRF-Token) over URL parameters

---

### Secret Keys and Sensitive Data Exposure

No secrets or sensitive information should be accessible to client-side code.

```sudolang
SecretProtection {
  NeverExposeClientSide {
    Constraints {
      Third-party API keys (Stripe, AWS, etc.) must not be in client code.
      Database connection strings must not be in client code.
      JWT signing secrets must not be in client code.
      Encryption keys must not be in client code.
      OAuth client secrets must not be in client code.
      Internal service URLs/credentials must not be in client code.

      Full credit card numbers must not be in client code.
      Social Security Numbers must not be in client code.
      Passwords (even hashed) must not be in client code.
      Security questions/answers must not be in client code.
      Full phone numbers (mask them: ***-***-1234) must not be in client code.
      Sensitive PII that isn't needed for display must not be in client code.

      Internal IP addresses must not be in client code.
      Database schemas must not be in client code.
      Debug information must not be in client code.
      Stack traces in production must not be in client code.
      Server software versions must not be in client code.
    }
  }

  CheckTheseLocations {
    warn "JavaScript bundles (including source maps)"
    warn "HTML comments"
    warn "Hidden form fields"
    warn "Data attributes"
    warn "LocalStorage/SessionStorage"
    warn "Initial state/hydration data in SSR apps"
    warn "Environment variables exposed via build tools (NEXT_PUBLIC_*, REACT_APP_*)"
  }

  BestPractices {
    require Store secrets in `.env` files.
    require Make API calls requiring secrets from backend only.
  }
}
```

---

## Open Redirect

Any endpoint accepting a URL for redirection must be protected against open redirect attacks.

### Protection Strategies

```sudolang
OpenRedirectProtection {
  Allowlist Validation (preferred)
    require Only allow requests to pre-approved domains.
    require Maintain a strict allowlist for integrations.

  Relative URLs Only
    require Only accept paths (e.g., `/dashboard`) not full URLs.
    require Validate the path starts with `/` and doesn't contain `//`.

  Indirect References
    require Use a mapping instead of raw URLs: `?redirect=dashboard` maps to `/dashboard`.

  isValidRedirect(url) {
    allowed_domains = ['yourdomain.com', 'app.yourdomain.com']
    parsed = parseUrl(url)
    return parsed.hostname in allowed_domains
  }
}
```

### Bypass Techniques to Block

| Technique             | Example                          | Why It Works                                             |
| --------------------- | -------------------------------- | -------------------------------------------------------- |
| @ symbol              | `https://legit.com@evil.com`     | Browser navigates to evil.com with legit.com as username |
| Subdomain abuse       | `https://legit.com.evil.com`     | evil.com owns the subdomain                              |
| Protocol tricks       | `javascript:alert(1)`            | XSS via redirect                                         |
| Double URL encoding   | `%252f%252fevil.com`             | Decodes to `//evil.com` after double decode              |
| Backslash             | `https://legit.com\@evil.com`    | Some parsers normalize `\` to `/`                        |
| Null byte             | `https://legit.com%00.evil.com`  | Some parsers truncate at null                            |
| Tab/newline           | `https://legit.com%09.evil.com`  | Whitespace confusion                                     |
| Unicode normalization | `https://legіt.com` (Cyrillic і) | IDN homograph attack                                     |
| Data URLs             | `data:text/html,<script>...`     | Direct payload execution                                 |
| Protocol-relative     | `//evil.com`                     | Uses current page's protocol                             |
| Fragment abuse        | `https://legit.com#@evil.com`    | Parsed differently by different libraries                |

### IDN Homograph Attack Protection

- Convert URLs to Punycode before validation
- Consider blocking non-ASCII domains entirely for sensitive redirects

---

### Password Security

```sudolang
PasswordSecurity {
  Requirements {
    require Minimum 8 characters (12+ recommended).
    require No maximum length (or very high, e.g., 128 chars).
    require Allow all characters including special chars.
    warn "Don't require specific character types (let users choose strong passwords)"
  }

  Storage {
    require Use Argon2id, bcrypt, or scrypt.
    Constraints {
      Never MD5.
      Never SHA1.
      Never plain SHA256.
    }
  }
}
```

---

## Server-Side Bugs

### Server-Side Request Forgery (SSRF)

Any functionality where the server makes requests to URLs provided or influenced by users must be protected.

#### Potential Vulnerable Features

- Webhooks (user provides callback URL)
- URL previews
- PDF generators from URLs
- Image/file fetching from URLs
- Import from URL features
- RSS/feed readers
- API integrations with user-provided endpoints
- Proxy functionality
- HTML to PDF/image converters

```sudolang
SSRFProtection {
  Allowlist Approach (Preferred)
    require Only allow requests to pre-approved domains.
    require Maintain a strict allowlist for integrations.

  Network Segmentation
    require Run URL-fetching services in isolated network.
    require Block access to internal network, cloud metadata.

  Implementation Requirements
    require Validate URL scheme is HTTP/HTTPS only.
    require Resolve DNS and validate IP is not private/internal.
    require Block cloud metadata IPs explicitly.
    require Limit or disable redirect following.
    require If following redirects, validate each hop.
    require Set timeout on requests.
    require Limit response size.
    require Use network isolation where possible.

  DNSRebindingPrevention {
    require Resolve DNS before making request.
    require Validate resolved IP is not internal.
    require Pin the resolved IP for the request (don't re-resolve).
    warn "Or: Resolve twice with delay, ensure both resolve to same external IP"
  }

  CloudMetadataBlacklist {
    Constraints {
      Block AWS: 169.254.169.254.
      Block GCP: metadata.google.internal, 169.254.169.254, http://metadata.
      Block Azure: 169.254.169.254.
      Block DigitalOcean: 169.254.169.254.
    }
  }
}
```

#### IP and DNS Bypass Techniques to Block

| Technique            | Example                            | Description                                               |
| -------------------- | ---------------------------------- | --------------------------------------------------------- |
| Decimal IP           | `http://2130706433`                | 127.0.0.1 as decimal                                      |
| Octal IP             | `http://0177.0.0.1`                | Octal representation                                      |
| Hex IP               | `http://0x7f.0x0.0x0.0x1`          | Hexadecimal                                               |
| IPv6 localhost       | `http://[::1]`                     | IPv6 loopback                                             |
| IPv6 mapped IPv4     | `http://[::ffff:127.0.0.1]`        | IPv4-mapped IPv6                                          |
| Short IPv6           | `http://[::]`                      | All zeros                                                 |
| DNS rebinding        | Attacker's DNS returns internal IP | First request resolves to external IP, second to internal |
| CNAME to internal    | Attacker domain CNAMEs to internal | DNS points to internal hostname                           |
| URL parser confusion | `http://attacker.com#@internal`    | Different parsing behaviors                               |
| Redirect chains      | External URL redirects to internal | Follow redirects carefully                                |
| IPv6 scope ID        | `http://[fe80::1%25eth0]`          | Interface-scoped IPv6                                     |
| Rare IP formats      | `http://127.1`                     | Shortened IP notation                                     |

---

### Insecure File Upload

File uploads must validate type, content, and size to prevent various attacks.

```sudolang
FileUploadSecurity {
  File Type Validation
    require Check file extension against allowlist.
    require Validate magic bytes/file signature match expected type.
    warn "Never rely on just one check"

  File Content Validation
    require Read and verify magic bytes.
    require For images: attempt to process with image library (detects malformed files).
    require For documents: scan for macros, embedded objects.
    require Check for polyglot files (files valid as multiple types).

  File Size Limits
    require Set maximum file size server-side.
    require Configure web server/proxy limits as well.
    require Consider per-file-type limits (images smaller than videos).

  SecureHandling {
    require Rename files: Use random UUID names, discard original.
    require Store outside webroot: Or use separate domain for uploads.
    require Set restrictive permissions: Uploaded files should not be executable.
    require Use CDN/separate domain: Isolate uploaded content from main app.

    ResponseHeaders {
      require Content-Disposition: attachment (forces download).
      require X-Content-Type-Options: nosniff.
      require Content-Type matching actual file type.
    }
  }

  MagicBytesReference {
    JPEG: "FF D8 FF"
    PNG: "89 50 4E 47 0D 0A 1A 0A"
    GIF: "47 49 46 38"
    PDF: "25 50 44 46"
    ZIP: "50 4B 03 04"
    DOCX_XLSX: "50 4B 03 04"  (ZIP-based)
  }
}
```

#### Common Bypasses and Attacks

| Attack                 | Description                                 | Prevention                                     |
| ---------------------- | ------------------------------------------- | ---------------------------------------------- |
| Extension bypass       | `shell.php.jpg`                             | Check full extension, use allowlist            |
| Null byte              | `shell.php%00.jpg`                          | Sanitize filename, check for null bytes        |
| Double extension       | `shell.jpg.php`                             | Only allow single extension                    |
| MIME type spoofing     | Set Content-Type to image/jpeg              | Validate magic bytes                           |
| Magic byte injection   | Prepend valid magic bytes to malicious file | Check entire file structure, not just header   |
| Polyglot files         | File valid as both JPEG and JavaScript      | Parse file as expected type, reject if invalid |
| SVG with JavaScript    | `<svg onload="alert(1)">`                   | Sanitize SVG or disallow entirely              |
| XXE via file upload    | Malicious DOCX, XLSX (which are XML)        | Disable external entities in parser            |
| ZIP slip               | `../../../etc/passwd` in archive            | Validate extracted paths                       |
| ImageMagick exploits   | Specially crafted images                    | Keep ImageMagick updated, use policy.xml       |
| Filename injection     | `; rm -rf /` in filename                    | Sanitize filenames, use random names           |
| Content-type confusion | Browser MIME sniffing                       | Set `X-Content-Type-Options: nosniff`          |

---

### SQL Injection

SQL injection occurs when user input is incorporated into SQL queries without proper handling.

```sudolang
SQLInjectionPrevention {
  PRIMARY DEFENSE: Parameterized Queries (Prepared Statements)
    require Always use parameterized queries.
    require Never concatenate user input into queries.

  ORM Usage
    require Use ORM methods that automatically parameterize.
    require Be cautious with raw query methods in ORMs.
    require Watch for ORM-specific injection points.

  Input Validation (defense-in-depth)
    require Validate data types (integer should be integer).
    require Whitelist allowed values where applicable.
    warn "Input validation is defense-in-depth, not primary defense"

  InjectionPointsToWatch {
    warn "WHERE clauses"
    warn "ORDER BY clauses (often overlooked - can't use parameters, must whitelist)"
    warn "LIMIT/OFFSET values"
    warn "Table and column names (can't parameterize - must whitelist)"
    warn "INSERT values"
    warn "UPDATE SET values"
    warn "IN clauses with dynamic lists"
    warn "LIKE patterns (also escape wildcards: %, _)"
  }

  AdditionalDefenses {
    Constraints {
      Database user should have minimum required permissions.
      Disable dangerous functions like `xp_cmdshell` in SQL Server.
      Never expose SQL errors to users.
    }
  }

  Examples {
    vulnerable: """
      query = "SELECT * FROM users WHERE id = " + userId
    """
    secure: """
      query = "SELECT * FROM users WHERE id = ?"
      execute(query, [userId])
    """
  }
}
```

---

### XML External Entity (XXE)

XXE vulnerabilities occur when XML parsers process external entity references in user-supplied XML.

#### Vulnerable Scenarios

**Direct XML Input:**

- SOAP APIs
- XML-RPC
- XML file uploads
- Configuration file parsing
- RSS/Atom feed processing

**Indirect XML:**

- JSON/other format converted to XML server-side
- Office documents (DOCX, XLSX, PPTX are ZIP with XML)
- SVG files (XML-based)
- SAML assertions
- PDF with XFA forms

```sudolang
XXEPrevention {
  require Disable DTD processing entirely if possible.
  require Disable external entity resolution.
  require Disable external DTD loading.
  require Disable XInclude processing.
  require Use latest patched XML parser versions.
  require Validate/sanitize XML before parsing if DTD needed.
  warn "Consider using JSON instead of XML where possible"

  LanguageSpecificSettings {
    Java: """
      DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
      dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
      dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
      dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
      dbf.setExpandEntityReferences(false);
    """

    Python: """
      from lxml import etree
      parser = etree.XMLParser(resolve_entities=False, no_network=True)
      # Or use defusedxml library
    """

    PHP: """
      libxml_disable_entity_loader(true);
      // Or use XMLReader with proper settings
    """

    NodeJS: """
      // Use libraries that disable DTD processing by default
      // If using libxmljs, set { noent: false, dtdload: false }
    """

    DotNet: """
      XmlReaderSettings settings = new XmlReaderSettings();
      settings.DtdProcessing = DtdProcessing.Prohibit;
      settings.XmlResolver = null;
    """
  }
}
```

---

### Path Traversal

Path traversal vulnerabilities occur when user input controls file paths, allowing access to files outside intended directories.

#### Vulnerable Patterns

```python
# VULNERABLE
file_path = "/uploads/" + user_input
file_path = base_dir + request.params['file']
template = "templates/" + user_provided_template
```

```sudolang
PathTraversalPrevention {
  require Never use user input directly in file paths.
  require Canonicalize paths and validate against base directory.
  require Restrict file extensions if applicable.
  require Test with various encoding and bypass techniques.

  PreferredApproach {
    Use indirect references instead of user input directly.
    safeFileLookup(userInput) {
      files = {
        'report': '/reports/q1.pdf',
        'invoice': '/invoices/2024.pdf'
      }
      return files |> get(userInput)
    }
  }

  CanonicalizationPattern {
    safeJoin(baseDirectory, userPath) {
      Ensure base is absolute and normalized.
      base = absoluteRealPath(baseDirectory)

      Join and then resolve the result.
      target = absoluteRealPath(join(base, userPath))

      Ensure the common path prefix is the base directory.
      require commonPath(base, target) == base
        else error "Path traversal attempt detected"

      return target
    }
  }

  InputSanitization {
    Constraints {
      Remove or reject `..` sequences.
      Remove or reject absolute path indicators (`/`, `C:`).
      Whitelist allowed characters (alphanumeric, dash, underscore).
      Validate file extension if applicable.
    }
  }
}
```

---

## Security Headers Checklist

```sudolang
SecurityHeaders {
  require Strict-Transport-Security: "max-age=31536000; includeSubDomains; preload".
  require Content-Security-Policy: "[see XSS section]".
  require X-Content-Type-Options: "nosniff".
  require X-Frame-Options: "DENY".
  require Referrer-Policy: "strict-origin-when-cross-origin".
  require Cache-Control: "no-store" for sensitive pages.
}
```

---

## General Security Principles

```sudolang
SecureCodingBehavior {
  Constraints {
    Validate all input server-side: Never trust client-side validation alone.
    Use parameterized queries: Never concatenate user input into queries.
    Encode output contextually: HTML, JS, URL, CSS contexts need different encoding.
    Apply authentication checks: On every endpoint, not just at routing.
    Apply authorization checks: Verify the user can access the specific resource.
    Use secure defaults: Security should be the default, not opt-in.
    Handle errors securely: Don't leak stack traces or internal details to users.
    Keep dependencies updated: Use tools to track vulnerable dependencies.
  }

  whenUnsure() {
    When facing a security decision with uncertainty,
    choose the more restrictive/secure option and
    document the security consideration in comments.
  }
}
```
