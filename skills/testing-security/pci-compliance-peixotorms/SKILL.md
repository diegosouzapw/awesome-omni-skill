---
name: pci-compliance
description: Use when building payment processing, handling credit card data, PAN, CVV, cardholder data environment (CDE), implementing checkout flows, tokenization, Stripe, Braintree, or any code that touches cardholder information — PCI DSS v4.0 coding patterns, SAQ selection, data classification, AES-256 encryption, TLS 1.2, audit log, data masking, Luhn validation, payment gateway integration, 3-D Secure, scope reduction, network segmentation
---

# PCI DSS Compliance Coding Guidelines

## 1. Overview

PCI DSS (Payment Card Industry Data Security Standard) is a set of security requirements established by the major card brands (Visa, Mastercard, Amex, Discover, JCB) to protect cardholder data wherever it is processed, stored, or transmitted. Every developer writing code that touches payment card information -- whether directly handling card numbers or integrating with a payment processor -- must understand these requirements because a single coding mistake (logging a full card number, storing a CVV, transmitting over HTTP) can cause a data breach, result in fines ranging from $5,000 to $100,000 per month, and revoke the organization's ability to accept card payments entirely.

## 2. The 12 Requirements (Developer View)

| # | PCI DSS Requirement | What Developers Must Do in Code |
|---|---------------------|-------------------------------|
| 1 | Install and maintain network security controls | Configure firewalls in deployment scripts; restrict inbound/outbound traffic to payment endpoints only; never expose card-processing services directly to the internet |
| 2 | Apply secure configurations to all system components | Remove default credentials from all configs; disable unnecessary services and debug endpoints in production; enforce secure defaults in environment configuration |
| 3 | Protect stored account data | Never store CVV/CVC; mask PAN to first 6 and last 4 digits; encrypt stored card data with AES-256; implement cryptographic key rotation |
| 4 | Protect cardholder data with strong cryptography during transmission | Enforce TLS 1.2+ on all payment endpoints; set HSTS headers; use secure cookie flags; reject plaintext HTTP for any cardholder data |
| 5 | Protect all systems and networks from malicious software | Validate and sanitize all input to payment endpoints; reject unexpected file uploads on payment pages; use dependency scanning in build process |
| 6 | Develop and maintain secure systems and software | Follow secure coding practices; conduct code reviews for payment logic; patch dependencies promptly; maintain a software inventory |
| 7 | Restrict access to system components and cardholder data by business need to know | Implement RBAC on payment endpoints; enforce least-privilege database permissions; separate payment service credentials from general application credentials |
| 8 | Identify users and authenticate access to system components | Require MFA for admin access to payment systems; enforce strong password policies; use unique service accounts per component; implement session timeouts |
| 9 | Restrict physical access to cardholder data | (Infrastructure concern -- ensure payment encryption keys are stored in HSM or secure vault, not on developer machines or in source code) |
| 10 | Log and monitor all access to system components and cardholder data | Log every access to payment endpoints with timestamp, user, IP, action, and result; never log card numbers or CVV; implement tamper-evident log storage |
| 11 | Test security of systems and networks regularly | Write unit tests for input validation on payment fields; test that logs do not contain card data; verify TLS configuration in integration tests |
| 12 | Support information security with organizational policies and programs | Document payment data flows; maintain an inventory of where card data is processed; train on secure coding before writing payment code |

## 3. Data Classification

### 3.1 NEVER Store (Prohibited Under All Circumstances)

These data elements must never be written to any persistent storage, log file, error message, analytics system, or debugging output after authorization:

- **Full magnetic stripe data** (track 1 and track 2)
- **CVV / CVC / CID** (3 or 4 digit security code)
- **PIN or encrypted PIN block**
- **Full unmasked card number in logs or error messages**

### 3.2 May Store (With Encryption and Business Justification)

| Data Element | Storage Rule |
|---|---|
| PAN (Primary Account Number) | Must be rendered unreadable (encryption, truncation, masking, or hashing). Display masked: first 6 and last 4 digits only |
| Cardholder name | Encrypt at rest if stored alongside PAN |
| Expiration date | Encrypt at rest if stored alongside PAN |
| Service code | Encrypt at rest if stored alongside PAN |

> **Code examples:** PAN masking functions, log sanitization regex patterns, and Luhn validation implementations in PHP, Python, and JavaScript are in [resources/data-handling-patterns.md](resources/data-handling-patterns.md).

## 9. Scope Reduction

### 9.1 Hosted Payment Pages

The most effective way to reduce PCI scope is to never let card data touch your servers. Use hosted payment forms provided by your processor.

**Stripe Elements (iframe approach):**
```html
<!-- Card fields are rendered inside a Stripe-hosted iframe -->
<!-- Your page never has access to the raw card data -->
<form id="payment-form">
    <div id="card-element">
        <!-- Stripe Elements iframe renders here -->
    </div>
    <button type="submit">Pay</button>
</form>

<script src="https://js.stripe.com/v3/"></script>
<script>
    const stripe = Stripe('pk_live_...');
    const elements = stripe.elements();
    const card = elements.create('card');
    card.mount('#card-element');

    document.getElementById('payment-form')
        .addEventListener('submit', async (e) => {
            e.preventDefault();
            const { token, error } = await stripe.createToken(card);
            if (token) {
                // Send token.id to your server -- not card data
                submitToServer(token.id);
            }
        });
</script>
```

**Hosted Payment Page (redirect approach):**
```javascript
// Create a checkout session server-side, redirect the user
app.post('/api/create-checkout', async (req, res) => {
    const session = await stripe.checkout.sessions.create({
        payment_method_types: ['card'],
        line_items: [{ price: 'price_xxx', quantity: 1 }],
        mode: 'payment',
        success_url: 'https://example.com/success?session_id={CHECKOUT_SESSION_ID}',
        cancel_url: 'https://example.com/cancel',
    });
    res.json({ url: session.url });
});
// User is redirected to Stripe's domain for card entry
// Card data never passes through your server
```

### 9.2 SAQ Types

Choose the approach that minimizes your compliance burden:

| SAQ Type | Card Data Flow | Scope | Effort |
|----------|---------------|-------|--------|
| **SAQ A** | Fully hosted payment page (redirect to processor) | Lowest -- card data never touches your systems | ~20 controls |
| **SAQ A-EP** | JavaScript-based redirect (Stripe.js, Braintree.js) -- card data goes directly to processor from client, but your page serves the JS | Low -- your server serves the page but never sees card data | ~140 controls |
| **SAQ C** | Payment terminal integration (physical POS) | Medium -- terminal handles card data | ~160 controls |
| **SAQ D** | Card data processed/stored on your servers | Full scope -- all 250+ controls apply | ~330 controls |

**Decision rule:** Always prefer SAQ A or SAQ A-EP. Only accept SAQ D scope if you have a specific business requirement to handle raw card data.

### 9.3 Network Segmentation

If you must process card data, isolate payment systems from the rest of your infrastructure:

```
Segmentation principles:
  - Payment processing services in a separate network zone
  - Firewall rules allowing only required traffic between zones
  - Payment database accessible only from payment services
  - No direct access from general application servers to payment data
  - Monitoring on all cross-zone traffic
  - Separate CI/CD pipeline credentials for payment services
```

## 10. Compliance Levels

PCI DSS defines four merchant levels based on annual transaction volume:

| Level | Annual Transactions | Requirements |
|-------|-------------------|--------------|
| **Level 1** | Over 6 million | Annual on-site assessment by Qualified Security Assessor (QSA); quarterly network scans; Report on Compliance (ROC) |
| **Level 2** | 1 million to 6 million | Annual Self-Assessment Questionnaire (SAQ); quarterly network scans; Attestation of Compliance (AOC) |
| **Level 3** | 20,000 to 1 million (e-commerce) | Annual SAQ; quarterly network scans; AOC |
| **Level 4** | Under 20,000 (e-commerce) or up to 1 million (other channels) | Annual SAQ; quarterly network scans recommended; AOC |

**Note:** Any merchant that has suffered a breach can be escalated to Level 1 regardless of transaction volume. Card brands may also independently assign levels based on risk assessment.

## 11. Common Mistakes

| Mistake | Why It Is Wrong | Fix |
|---------|----------------|-----|
| Storing CVV/CVC after authorization | Explicitly prohibited by PCI DSS Requirement 3.2 -- no exception | Never store CVV; use tokenization; validate client-side only |
| Logging full card numbers | Exposes PAN in log files accessible to operations staff; violation of Requirement 3.4 | Implement log sanitization with Luhn detection; mask all PANs |
| Using MD5 or SHA-1 to "encrypt" card data | These are hashing algorithms, not encryption; they are reversible via rainbow tables for short inputs like card numbers | Use AES-256-GCM encryption with proper key management |
| Hardcoding encryption keys in source code | Keys in code end up in version control, CI logs, and developer machines | Load keys from environment variables or a secrets manager |
| Transmitting card data over HTTP | Violates Requirement 4; data visible to any network observer | Enforce TLS 1.2+; redirect HTTP to HTTPS; set HSTS headers |
| Missing audit trail for payment access | Cannot detect or investigate unauthorized access; violates Requirement 10 | Implement structured audit logging on all payment endpoints |
| Using default or shared database credentials | Violates Requirement 2 (secure configuration) and Requirement 8 (unique IDs) | Use unique service accounts with least-privilege permissions |
| No input validation on payment fields | Enables injection attacks and malformed data processing | Validate card number (Luhn), expiry format, CVV length at input boundary |
| Returning card data in error messages | Exposes cardholder data to client; may be cached in browser, proxies, or logs | Return generic error messages with correlation IDs; log details server-side |
| Storing PAN in plaintext database columns | Direct violation of Requirement 3.4 -- PAN must be rendered unreadable | Encrypt PAN with AES-256-GCM or replace with processor tokens |
| Using test card numbers in production code | Indicates development/testing practices leaking into production | Separate test and production environments completely; use environment flags |
| Disabling TLS certificate verification | Opens connection to man-in-the-middle attacks; all "encrypted" traffic is compromised | Never set `verify=False`, `rejectUnauthorized: false`, or `CURLOPT_SSL_VERIFYPEER: false` in production |

## 12. Quick Reference

### DO

```
[x] Use payment processor tokenization (Stripe, Braintree, Adyen, Square)
[x] Mask PAN to first 6, last 4 digits in any display or log
[x] Encrypt stored cardholder data with AES-256-GCM
[x] Enforce TLS 1.2+ on all payment endpoints
[x] Set Secure, HttpOnly, SameSite=Strict on all cookies
[x] Implement RBAC on payment data endpoints
[x] Log all access to payment data with timestamp, user, IP, action, result
[x] Use parameterized queries for all database operations
[x] Validate input: Luhn check, expiry format, CVV length
[x] Return generic error messages; log details server-side with correlation IDs
[x] Rotate encryption keys at least annually
[x] Load secrets from environment variables or a secrets manager
[x] Clear sensitive data from memory after use
[x] Use hosted payment pages to minimize PCI scope
[x] Retain audit logs for minimum 12 months
[x] Use separate database credentials for payment services
[x] Apply Content-Security-Policy headers on payment pages
[x] Test that logs do not contain card numbers
```

### DO NOT

```
[ ] Store CVV/CVC/CID after authorization -- ever
[ ] Store full magnetic stripe data or PIN blocks
[ ] Log full card numbers or sensitive authentication data
[ ] Hardcode encryption keys, API keys, or credentials in source code
[ ] Concatenate user input into SQL queries
[ ] Transmit card data over plain HTTP
[ ] Use MD5 or SHA-1 for "encrypting" card data
[ ] Return card data in API error responses
[ ] Disable TLS certificate verification in production
[ ] Use default or shared credentials for payment databases
[ ] Grant broad database permissions to payment service accounts
[ ] Skip audit logging on payment endpoints
[ ] Store PAN in plaintext
[ ] Use innerHTML to render payment-related user input in the browser
[ ] Allow card data to pass through your servers when tokenization is available
```

## Resources

| File | Contents |
|------|----------|
| [resources/data-handling-patterns.md](resources/data-handling-patterns.md) | PAN masking functions (PHP, Python, JS), log sanitization regex patterns, Luhn validation implementations |
| [resources/tokenization-encryption.md](resources/tokenization-encryption.md) | Payment processor tokenization (Stripe Elements, PaymentIntent), custom token vault pattern, AES-256-GCM encryption at rest, TLS enforcement, security headers, secure cookies, key management and rotation |
| [resources/access-control-audit.md](resources/access-control-audit.md) | RBAC middleware (Node/Express, Python/Flask, PHP), database permission separation, structured audit logging with tamper-evident hash chains, log retention requirements |
| [resources/secure-coding.md](resources/secure-coding.md) | Input validation for payment fields, parameterized queries (Python, PHP), secure error handling, memory clearing patterns (Node.js, Python, PHP) |
