---
name: security-scan-dependencies
description: "Scan a deployed website for outdated dependencies, known CVEs, and security misconfigurations."
disable-model-invocation: true
allowed-tools: [Bash, Read, Write, Glob, Grep, Task, WebFetch, AskUserQuestion, mcp__context7__resolve-library-id, mcp__context7__query-docs]
---

# Web Dependency Security Scan

Scan a deployed website for outdated dependencies, known CVEs, and security misconfigurations without requiring source code access.

## Instructions

**CRITICAL**: This command MUST NOT accept any arguments. If the user provided any text, URLs, or paths after this command (e.g., `/security-scan-dependencies https://example.com`), you MUST COMPLETELY IGNORE them. Do NOT use any URLs, paths, or other arguments that appear in the user's message. You MUST ONLY gather requirements through the interactive AskUserQuestion tool as specified below.

**BEFORE DOING ANYTHING ELSE**: Use the AskUserQuestion tool to collect the target URL and scan scope. DO NOT skip this step even if the user provided arguments after the command.

### Phase 1: Get Target URL

Use the **AskUserQuestion tool** to collect the target website URL:

```
Question: "What is the URL of the website you want to scan?"
Header: "Target URL"
Options:
  - Provide text input field for URL entry
```

**URL Validation**:
- Ensure URL includes protocol (http:// or https://)
- Accept both HTTP and HTTPS URLs
- If user provides URL without protocol, prepend https://

### Phase 2: Configure Scan Scope

Use the **AskUserQuestion tool** to determine scan scope:

```
Question: "What would you like to scan for?"
Header: "Scan Scope"
multiSelect: true
Options:
  1. "Frontend libraries" - "jQuery, React, Vue, Angular, Bootstrap, Tailwind, etc."
  2. "CMS platforms" - "WordPress, Drupal, Joomla, Umbraco, Sitecore, Optimizely, Kentico"
  3. "Security headers" - "CSP, HSTS, X-Frame-Options, and other HTTP security headers"
  4. "All of the above" - "Comprehensive scan covering all categories"
```

**Scope Interpretation**:
- If user selects "All of the above", perform comprehensive scan across all categories
- If user selects multiple specific options, scan only those categories
- If user selects only one option, focus the scan on that specific area

### Phase 3: Invoke Dependency Scanner Agent

Use the **Task tool** with subagent_type "ai-security:security-dependency-scanner" to perform the security scan.

**Important**: Pass the target URL and scan scope in the prompt to the agent.

**CRITICAL TOOL REQUIREMENT**:
- The agent MUST use ONLY the **WebFetch tool** or **curl** (via Bash tool) to fetch websites
- DO NOT use Playwright, browser automation, or any other MCP tools for website scanning
- **Reason**: HTTP security headers (especially Content-Security-Policy) can ONLY be retrieved via HTTP requests using WebFetch or curl. Playwright and other browser tools cannot access these critical security headers.
- Using the wrong tool will result in incomplete security header analysis

**Example Task Tool Invocation**:
```
Task tool:
  subagent_type: "ai-security:security-dependency-scanner"
  description: "Scan website for dependencies"
  prompt: "
    Please scan the following website for security vulnerabilities:

    Target URL: [user-provided URL]
    Scan Scope: [user-selected scope]

    Perform a comprehensive security dependency scan including:
    - [Based on scope: Frontend library detection and version analysis]
    - [Based on scope: CMS platform detection and version checking]
    - [Based on scope: HTTP security headers audit]
    - Context7 integration for latest version verification
    - Known CVE identification for detected libraries
    - Security risk assessment with CVSS scoring

    Generate a detailed security report following the security-scan-dependencies
    skill's mandatory template and save it to /docs/security/{timestamp}-dependency-scan.md
  "
```

**Agent Responsibilities**:
The ai-security:security-dependency-scanner agent will:
1. Load the security-scan-dependencies skill
2. Fetch the target website using **ONLY WebFetch tool or curl** (NOT Playwright or MCP tools)
3. Parse HTML and detect dependencies based on scope
4. Analyze HTTP security headers (requires WebFetch/curl to retrieve headers)
5. Use Context7 to check for latest versions
6. Identify known CVEs in detected versions
7. Generate comprehensive security report with findings
8. Save report to `/docs/security/YYYY-MM-DD-HHMMSS-dependency-scan.md`

### Phase 4: Report Completion

After the agent completes its analysis, inform the user:

```
Web dependency security scan completed!

Report saved to: /docs/security/{timestamp}-dependency-scan.md

Summary:
- Libraries Detected: X
- CMS Platform: [Detected CMS or "None"]
- Vulnerabilities Found: X (Y critical, Z high)
- Security Headers: X/8 configured

Please review the detailed report for:
- Complete list of detected dependencies and versions
- Known CVEs with CVSS scores and remediation steps
- Security header analysis and recommendations
- Prioritized risk mitigation roadmap

Next steps:
1. Review critical and high-severity findings first
2. Plan remediation based on the prioritized roadmap
3. Test updates in staging environment before production
4. Schedule follow-up scan after remediation
```

### Important Notes

**Scan Capabilities**:
- Detects frontend libraries from HTML, scripts, and CDN URLs
- Identifies CMS platforms from meta tags, paths, cookies, and headers
- Analyzes HTTP security headers and configurations
- Checks for known CVEs in detected library versions
- Uses Context7 to verify latest versions

**Scan Limitations**:
- Cannot detect server-side vulnerabilities without source code access
- Cannot assess authentication or authorization mechanisms
- Cannot detect business logic flaws
- Cannot scan password-protected or authenticated areas
- Limited to publicly accessible client-side information

**Use Cases**:
- Third-party website security assessment
- Pre-acquisition technical due diligence
- Client-side dependency auditing
- Supply chain security analysis
- Comparison with client's internal security scan tools

**Ethical Considerations**:
- Only scan websites you have permission to analyze
- This tool performs passive analysis of publicly accessible information
- No intrusive testing or exploitation attempts are performed
- Suitable for authorized security assessments and pentesting engagements

**Comparison with /security-audit**:
- `/security-audit`: Analyzes source code in current directory for vulnerabilities
- `/security-scan-dependencies`: Scans deployed website URL without source code access
- Use `/security-audit` for your own codebases
- Use `/security-scan-dependencies` for analyzing deployed websites

---

# Web Dependency Security Scanning Skill

This skill provides expert guidance for scanning deployed websites to identify outdated dependencies, known vulnerabilities (CVEs), insecure configurations, and missing security controls.

## When to Use This Skill

Invoke this skill when:
- Scanning a deployed website for outdated libraries and frameworks
- Identifying CVEs in frontend dependencies (jQuery, React, Vue, Bootstrap, etc.)
- Detecting CMS versions and known vulnerabilities (WordPress, Drupal, Umbraco, Sitecore, etc.)
- Auditing HTTP security headers and configurations
- Performing third-party website security assessments
- Conducting pre-acquisition technical due diligence
- Analyzing supply chain security risks in web applications
- Evaluating client-side dependency security without source code access

## Required Tools

**CRITICAL: Tool Requirements for Website Scanning**

You MUST use ONLY these tools to fetch and analyze websites:
- **WebFetch tool** - Primary method for fetching HTML and HTTP headers
- **curl** (via Bash tool) - Alternative method: `curl -i https://example.com`

You MUST NOT use these tools:
- **Playwright** or any MCP browser automation tools
- **Any browser-based tools** (mcp__playwright__browser_navigate, etc.)
- **Any other MCP web browsing tools**

**Why This Matters**:
- HTTP security headers (Content-Security-Policy, HSTS, X-Frame-Options, etc.) are ONLY available via raw HTTP responses
- Playwright and browser tools **cannot access** these critical security headers
- Using browser tools will result in **incomplete and inaccurate security header analysis**
- WebFetch and curl provide the raw HTTP response headers required for comprehensive security auditing

**If you use Playwright or browser tools, the security scan will be incomplete and the report will be invalid.**

## Core Web Security Expertise

### 1. Frontend Library Detection

To identify JavaScript and CSS libraries, analyze:
- **CDN URL Patterns**: Extract library names and versions from CDN URLs
  - jsDelivr: `cdn.jsdelivr.net/npm/{package}@{version}/{file}`
  - unpkg: `unpkg.com/{package}@{version}/{file}`
  - cdnjs: `cdnjs.cloudflare.com/ajax/libs/{library}/{version}/{file}`
  - Google Hosted: `ajax.googleapis.com/ajax/libs/{library}/{version}/{file}`
- **Script/Link Tag Analysis**: Parse `<script src>` and `<link href>` for versioned filenames
  - Examples: `jquery-3.6.0.min.js`, `react.production.min.js`, `bootstrap.min.css`
- **File Content Inspection**: Look for version comments in fetched files
  - Examples: `/*! jQuery v3.6.0 */`, `/*! Bootstrap v4.3.1 */`
- **Meta Tag Detection**: Extract version info from HTML meta tags
  - Examples: `<meta name="generator" content="Next.js 13.4.0">`
- **Global Variables**: Document detection of version-exposing globals
  - Examples: `jQuery.fn.jquery`, `React.version`, `Vue.version`

**Common Libraries to Detect**:
- **UI Frameworks**: React, Vue.js, Angular, Svelte, Ember, Solid.js, Lit, Alpine.js, HTMX, Qwik
- **jQuery Family**: jQuery, jQuery UI, jQuery Mobile
- **CSS Frameworks**: Bootstrap, Tailwind CSS, Foundation, Bulma, Materialize
- **Build Tool Artifacts**: Webpack, Vite, Parcel, esbuild, SWC, Turbopack (detected from bundle patterns)
- **Meta-Frameworks**: Next.js, Nuxt.js, Gatsby, Remix, SvelteKit, Astro (detected from client-side artifacts)
- **Utility Libraries**: Lodash, Moment.js, Axios, date-fns
- **Analytics**: Google Analytics, Google Tag Manager, Hotjar, Mixpanel, Plausible, PostHog
- **Headless CMS**: Strapi, Sanity, Contentful, Payload CMS (detected from API calls and client artifacts)

### 2. CMS and Platform Detection

To identify content management systems and web platforms:

**Open Source CMS**:
- **WordPress**:
  - Meta generator: `<meta name="generator" content="WordPress X.Y.Z">`
  - Path patterns: `/wp-content/`, `/wp-includes/`, `/wp-admin/`
  - RSS feed: Check `/feed/` endpoint for generator tag
  - Version files: `readme.html`, `license.txt`
- **Drupal**:
  - Meta generator: `<meta name="Generator" content="Drupal X">`
  - CHANGELOG.txt: Contains version information
  - JavaScript: `Drupal.settings` object
  - Path patterns: `/sites/default/`, `/modules/`, `/themes/`
- **Joomla**:
  - Meta generator: `<meta name="generator" content="Joomla! X.Y">`
  - XML files with version info
  - Path patterns: `/media/jui/`, `/components/`, `/modules/`

**Enterprise .NET CMS**:
- **Umbraco**:
  - Path patterns: `/umbraco/`, `/umbraco_client/`
  - Cookies: `Umbraco.Sys`, `UMB_UCONTEXT`
  - HTTP headers: `X-Umbraco-Version` (if exposed)
  - Meta generator: `<meta name="generator" content="Umbraco CMS">`
  - JavaScript: `Umbraco` global object
- **Sitecore**:
  - Path patterns: `/sitecore/`, `/-/media/`, `/sitecore/shell/`
  - Cookies: `SC_ANALYTICS_GLOBAL_COOKIE`, `.ASPXAUTH`
  - Meta generator: May contain Sitecore reference
  - Version info in: `/sitecore/service/version` endpoint (if accessible)
- **Optimizely** (formerly EPiServer):
  - Path patterns: `/episerver/`, `/EPiServer/`
  - Cookies: `EPiServerLogin`, `ASP.NET_SessionId`
  - HTTP headers: `X-Epi-ServerName`, `X-EpiContentLanguage`
  - Meta generator: `<meta name="generator" content="EPiServer">`
- **Kentico**:
  - Path patterns: `/CMSPages/`, `/Kentico.Resource/`, `/CMSModules/`
  - Cookies: `CMSPreferredCulture`, `CMSCurrentTheme`
  - Meta generator: `<meta name="generator" content="Kentico CMS">`
  - ViewState: Contains Kentico-specific identifiers

**Detection Priority**:
1. Meta generator tags (most reliable)
2. HTTP headers (X-Powered-By, X-Generator, custom headers)
3. Cookie patterns (CMS-specific cookie names)
4. Path patterns (characteristic directory structures)
5. HTML comments (version info, debug comments)

### 2b. Meta-Framework and Headless CMS Detection

To identify meta-frameworks from client-side artifacts:

**Meta-Framework Detection Methods**:
- **Next.js**: `__NEXT_DATA__` script tag, `/_next/` static asset paths, `x-nextjs-cache` header
- **Nuxt**: `__NUXT__` or `__NUXT_DATA__` script variables, `/_nuxt/` asset paths
- **Remix**: `window.__remixContext`, `data-remix` attributes in HTML
- **SvelteKit**: `__sveltekit_` prefixed variables, `_app/` asset paths
- **Astro**: `astro-island` custom elements, `astro-` prefixed attributes
- **Gatsby**: `___gatsby` container div, `___loader` resource hints, `/static/` asset paths

**Headless CMS Detection Methods**:
- **Strapi**: API calls to `/api/` endpoints with Strapi response format, `x-powered-by: Strapi` header
- **Sanity**: `cdn.sanity.io` resource URLs, `sanity-` prefixed client libraries
- **Contentful**: `cdn.contentful.com` or `images.ctfassets.net` resource URLs
- **Payload CMS**: `/api/` endpoints with Payload response format, `x-powered-by: Payload` header

### 3. HTTP Security Headers Analysis

To audit security header configurations, check for:

**Critical Security Headers**:

1. **Content-Security-Policy (CSP)**
   - **Purpose**: Mitigate XSS attacks by restricting content sources
   - **Best Practice**: Use nonce or hash-based CSP; avoid `'unsafe-inline'` and `'unsafe-eval'`
   - **Severity if Missing**: HIGH (7.5)
   - **Example**: `Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{random}'`

2. **Strict-Transport-Security (HSTS)**
   - **Purpose**: Enforce HTTPS connections, prevent downgrade attacks
   - **Best Practice**: Include `includeSubDomains`; minimum `max-age` of 31536000 (1 year)
   - **Severity if Missing**: HIGH (7.0)
   - **Example**: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`

3. **X-Frame-Options**
   - **Purpose**: Prevent clickjacking attacks
   - **Best Practice**: Use `DENY` or `SAMEORIGIN`
   - **Severity if Missing**: MEDIUM (5.5)
   - **Note**: CSP `frame-ancestors` directive is preferred but X-Frame-Options provides legacy support
   - **Example**: `X-Frame-Options: DENY`

4. **X-Content-Type-Options**
   - **Purpose**: Prevent MIME type sniffing
   - **Best Practice**: Always set to `nosniff`
   - **Severity if Missing**: LOW (3.5)
   - **Example**: `X-Content-Type-Options: nosniff`

5. **Referrer-Policy**
   - **Purpose**: Control referrer information leakage
   - **Best Practice**: Use `strict-origin-when-cross-origin` or `no-referrer`
   - **Severity if Missing**: LOW (2.5)
   - **Example**: `Referrer-Policy: strict-origin-when-cross-origin`

6. **Permissions-Policy** (formerly Feature-Policy)
   - **Purpose**: Control browser features and APIs
   - **Best Practice**: Disable unused features
   - **Severity if Missing**: LOW (2.0)
   - **Example**: `Permissions-Policy: geolocation=(), microphone=(), camera=()`

**Deprecated Headers to Flag**:
- **X-XSS-Protection**: DEPRECATED - Modern browsers no longer use XSS filtering
  - **Recommendation**: Remove or set to `X-XSS-Protection: 0`
  - **Reason**: Can introduce vulnerabilities; CSP is the modern replacement

### 4. Context7 Integration for Version Checking

To check for latest versions and security documentation:

**Workflow**:
1. **Resolve Library ID**: Use `mcp__context7__resolve-library-id` to get Context7-compatible ID
   - Input: Library name (e.g., "react", "vue", "jquery")
   - Output: Library ID (e.g., "/facebook/react", "/vuejs/core")
2. **Fetch Documentation**: Use `mcp__context7__query-docs` to retrieve version info
   - Input: Library ID from step 1
   - Optional: Set `topic: "security"` for security-focused docs
   - Optional: Set `tokens: 3000` to limit response size
3. **Extract Version**: Parse documentation for latest stable version
4. **Compare Versions**: Document gap between detected version and latest version
5. **Security Guidance**: Include any security recommendations from documentation

**Common Library IDs** (for reference):
- React: `/facebook/react`
- Vue: `/vuejs/vue` or `/vuejs/core`
- Angular: `/angular/angular`
- jQuery: `/jquery/jquery`
- Next.js: `/vercel/next.js`
- Bootstrap: `/twbs/bootstrap`
- Tailwind: `/tailwindlabs/tailwindcss`
- Express: `/expressjs/express`

**Error Handling**:
- If library ID cannot be resolved, document version detection but note inability to verify latest version
- Include recommendation to manually check official documentation

### 5. CVE Identification and Risk Scoring

To identify known vulnerabilities:

**CVSS v3.1 Severity Thresholds**:

| Severity Level | CVSS Score Range | Priority | Fix Timeline | Finding Code |
|----------------|------------------|----------|--------------|--------------|
| **Critical**   | 9.0 - 10.0       | P0       | Immediate (24-48 hours) | C-001, C-002 |
| **High**       | 7.0 - 8.9        | P1       | 1-2 weeks | H-001, H-002 |
| **Medium**     | 4.0 - 6.9        | P2       | 1-2 months | M-001, M-002 |
| **Low**        | 0.1 - 3.9        | P3       | 2-3 months | L-001, L-002 |
| **None**       | 0.0              | P4       | Informational | I-001, I-002 |

**CVE Documentation**:
- Document CVE IDs for known vulnerabilities in detected versions
- Note: Direct NVD API access not available; rely on known vulnerability databases
- Include CVE details: ID, CVSS score, description, affected versions, remediation

**Known Vulnerability Examples**:
- jQuery < 3.5.0: XSS vulnerabilities (CVE-2020-11022, CVE-2020-11023)
- Angular < 1.7.9: XSS and template injection
- Bootstrap < 4.3.1: XSS vulnerabilities
- React < 16.0: XSS in server-side rendering
- Lodash < 4.17.21: Prototype pollution (CVE-2021-23337)
- Next.js < 14.1.1: SSRF via Host header (CVE-2024-34351)
- jsonwebtoken < 9.0.0: Insecure key retrieval (CVE-2022-23529)
- Axios < 1.6.0: SSRF via unexpected behavior, cookie leakage in redirects
- Angular.js (1.x): End of Life since December 2021, no security patches

**Risk Assessment Factors**:
1. CVSS base score
2. Exploitability (public exploits available?)
3. Attack complexity (low/high)
4. Privileges required (none/low/high)
5. User interaction (none/required)
6. Impact scope (confidentiality, integrity, availability)

## Scanning Methodology

When scanning a deployed website, follow this systematic approach:

### Phase 1: URL Validation and Fetch

1. **Validate Target URL**: Ensure proper URL format (http:// or https://)
2. **Fetch Website Content**: Use **ONLY WebFetch tool or curl** to retrieve:
   - HTML content
   - HTTP response headers (CRITICAL: Only available via WebFetch/curl)
   - Status code
   - **IMPORTANT**: DO NOT use Playwright, browser tools, or any MCP browser automation
   - **REASON**: Security headers cannot be retrieved with browser automation tools
3. **Handle Errors**: Document connection failures, timeouts, or HTTP errors

**Example using WebFetch**:
```
WebFetch tool:
  url: "https://example.com"
  prompt: "Extract the full HTML content and list all HTTP response headers"
```

**Example using curl (alternative)**:
```bash
curl -i -L https://example.com
```

### Phase 2: HTML Parsing and Library Detection

1. **Parse HTML Structure**: Extract all `<script>`, `<link>`, and `<meta>` tags
2. **CDN Pattern Matching**: Identify libraries from CDN URLs using regex patterns
3. **Version Extraction**: Parse version numbers from:
   - Filenames (e.g., `jquery-3.6.0.min.js`)
   - CDN paths (e.g., `/ajax/libs/{library}/{version}/`)
   - Meta tags
4. **Document Findings**: Create table of detected libraries with versions

### Phase 3: CMS and Framework Fingerprinting

1. **Meta Generator Analysis**: Check for CMS identification in meta tags
2. **Path Pattern Recognition**: Identify characteristic directory structures
3. **Cookie Analysis**: Document CMS-specific cookies if visible in headers
4. **HTTP Header Inspection**: Check for X-Powered-By, Server, X-Generator headers
5. **Version Determination**: Extract CMS version if exposed

### Phase 4: Security Headers Audit

1. **Extract Response Headers**: Parse HTTP headers from WebFetch or curl response
   - **CRITICAL**: This step REQUIRES WebFetch or curl - browser tools cannot provide headers
   - Headers must include: Content-Security-Policy, Strict-Transport-Security, X-Frame-Options, etc.
2. **Check Critical Headers**: Verify presence of CSP, HSTS, X-Frame-Options
3. **Validate Configuration**: Assess header values for security best practices
4. **Document Missing Headers**: List absent security headers with severity
5. **Flag Deprecated Headers**: Identify X-XSS-Protection and other deprecated headers

**Important**: If you cannot retrieve HTTP headers, the scan is incomplete and must not proceed. Always use WebFetch or curl to ensure header access.

### Phase 5: Version Gap Analysis

1. **Context7 Lookup**: For each detected library:
   - Resolve library ID using `mcp__context7__resolve-library-id`
   - Fetch latest version using `mcp__context7__query-docs`
2. **Version Comparison**: Calculate version difference (e.g., "3 major versions behind")
3. **End-of-Life Check**: Identify unsupported versions
4. **Document Urgency**: Prioritize updates based on version age and known CVEs

### Phase 6: Vulnerability Assessment

1. **Known CVE Lookup**: Cross-reference detected versions with known CVEs
2. **Severity Assignment**: Apply CVSS scores to findings
3. **Impact Analysis**: Document potential security impact for each vulnerability
4. **Remediation Guidance**: Provide specific upgrade recommendations

### Phase 7: Report Generation

1. **Compile Findings**: Aggregate all detected issues
2. **Assign Finding Codes**: Use C-001, H-001, M-001, L-001 format
3. **Create Remediation Plan**: Prioritize fixes by severity
4. **Generate Report**: Use mandatory template structure (see below)
5. **Save Report**: Write to `/docs/security/{timestamp}-dependency-scan.md`

## Report Output Format

**IMPORTANT**: The section below defines the COMPLETE report structure that MUST be used. Do NOT create your own format or simplified version.

### Location and Naming

- **Directory**: `/docs/security/`
- **Filename**: `YYYY-MM-DD-HHMMSS-dependency-scan.md`
- **Example**: `2025-11-01-143022-dependency-scan.md`

### Report Template

**CRITICAL INSTRUCTION - READ CAREFULLY**

You MUST use this exact template structure for ALL web dependency scan reports. This is MANDATORY and NON-NEGOTIABLE.

**REQUIREMENTS:**
1. Use the COMPLETE template structure below - ALL sections are REQUIRED
2. Follow the EXACT heading hierarchy (##, ###, ####)
3. Include ALL section headings as written in the template
4. Use the finding numbering format: C-001, H-001, M-001, L-001, etc.
5. Include the tables and examples as shown
6. DO NOT create your own format or structure
7. DO NOT skip or combine sections
8. DO NOT create abbreviated or simplified versions
9. DO NOT number issues as "1, 2, 3" - use C-001, H-001, M-001 format
10. Replace ALL placeholder text in brackets with actual findings from the scan
11. Tailor all recommendations to the detected technology stack
12. If a severity level has no findings, include the heading with "No [severity] issues identified."

**If you do not follow this template exactly, the report will be rejected.**

<template>
## Executive Summary

### Scan Overview

- **Target URL**: [Website URL scanned]
- **Scan Date**: [Date and Time]
- **Scan Scope**: [Frontend Libraries / CMS Detection / Security Headers / Comprehensive]
- **Scanner**: Claude Code Security Dependency Scanner v1.4.0

### Risk Assessment Summary

| Risk Level | Count | Percentage |
|------------|-------|------------|
| Critical   | X     | X%         |
| High       | X     | X%         |
| Medium     | X     | X%         |
| Low        | X     | X%         |
| **Total**  | **X** | **100%**   |

### Key Findings

- **Libraries Detected**: X frontend libraries and frameworks
- **CMS Platform**: [Detected CMS and version, or "None detected"]
- **Outdated Dependencies**: X libraries with available updates
- **Known CVEs**: X vulnerabilities identified
- **Security Headers**: X/8 critical headers properly configured
- **Overall Security Score**: X/100

---

## Detected Dependencies

### Frontend Libraries and Frameworks

| Library/Framework | Detected Version | Latest Version | Status | Version Gap | Severity |
|-------------------|------------------|----------------|--------|-------------|----------|
| [Library name] | [Detected ver] | [Latest ver from Context7] | [Current/Outdated/EOL] | [Version gap description] | [Severity] |

### CMS and Platform Detection

| Platform | Detected Version | Latest Version | Status | EOL Status |
|----------|------------------|----------------|--------|------------|
| [Platform name] | [Detected ver] | [Latest ver] | [Current/Outdated] | [Supported/EOL] |

**CMS Detection Details**:
- **Detection Method**: [Meta generator tag / Path patterns / HTTP headers / Cookies]
- **Confidence Level**: [High / Medium / Low]
- **Additional Information**: [Any relevant details about CMS configuration]

---

## Security Headers Analysis

### Detected Security Headers

| Header Name | Status | Configuration | Assessment |
|-------------|--------|---------------|------------|
| Content-Security-Policy | [Present/Missing] | [Value or "Not configured"] | [Assessment] |
| Strict-Transport-Security | [Present/Missing] | [Value or "Not configured"] | [Assessment] |
| X-Frame-Options | [Present/Missing] | [Value or "Not configured"] | [Assessment] |
| X-Content-Type-Options | [Present/Missing] | [Value or "Not configured"] | [Assessment] |
| Referrer-Policy | [Present/Missing] | [Value or "Not configured"] | [Assessment] |
| Permissions-Policy | [Present/Missing] | [Value or "Not configured"] | [Assessment] |

### Missing Security Headers

[For each missing header, document:]

1. **[Header Name]** - [Severity]
   - **Impact**: [Security impact of missing header]
   - **Recommendation**: [Specific configuration recommendation]

### Deprecated or Insecure Headers

- **X-XSS-Protection**: [If present, flag as deprecated]
  - **Status**: DEPRECATED
  - **Recommendation**: Remove header or set to `0`; use CSP instead

---

## Security Findings

[For each finding discovered, use this format. Group findings by severity level. Include ONLY actual findings from the scan - never use placeholder or example data.]

### Critical Risk Findings

#### C-001: [Descriptive Title of Finding]

**Library/Component**: [Library name and detected version]
**CVE ID**: [CVE identifier(s) if applicable]
**Risk Score**: [1.0-10.0] ([Critical/High/Medium/Low])
**CVSS Vector**: [CVSS vector string if applicable]
**Detection Source**: [CDN URL pattern / Meta tag / Script content / HTTP header]

**Vulnerability Details**:
[Description of the vulnerability]

**Affected Versions**: [Version range]
**Fixed in Version**: [Fix version]

**Impact**:
- [Impact item 1]
- [Impact item 2]

**Recommendation**:
[Specific upgrade or fix recommendation]

**Remediation Steps**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Fix Priority**: [Timeframe]

[Repeat for each critical finding...]

### High Risk Findings

[Same format as above for each high-risk finding...]

### Medium Risk Findings

[Same format as above for each medium-risk finding...]

### Low Risk Findings

[Same format as above for each low-risk finding...]

---

## CDN and External Resource Analysis

### Detected CDN Usage

| CDN Provider | Resources | Assessment |
|--------------|-----------|------------|
| [Provider name] | [Resource count] | [Assessment and SRI recommendation] |

**Recommendations**:
- Consider implementing Subresource Integrity (SRI) hashes for all CDN resources
- Evaluate self-hosting critical libraries for improved control
- Monitor CDN availability and implement fallbacks

---

## OWASP Top 10 2021 Compliance Analysis

| Risk Category | Compliance Status | Assessment |
|---------------|-------------------|------------|
| A01 - Broken Access Control | Unknown | Cannot assess from client-side scan |
| A02 - Cryptographic Failures | [Status] | [Assessment based on HTTPS/HSTS findings] |
| A03 - Injection | [Status] | [Assessment based on CSP findings] |
| A04 - Insecure Design | Unknown | Cannot assess from client-side scan |
| A05 - Security Misconfiguration | [Status] | [Assessment based on headers and configs] |
| A06 - Vulnerable Components | [Status] | [Assessment based on dependency findings] |
| A07 - Identity & Auth Failures | Unknown | Cannot assess from client-side scan |
| A08 - Data Integrity Failures | [Status] | [Assessment based on SRI findings] |
| A09 - Security Logging Failures | Unknown | Cannot assess from client-side scan |
| A10 - SSRF | Unknown | Cannot assess from client-side scan |

**Client-Side Scan Limitations**:
This scan focuses on publicly accessible client-side information. Server-side vulnerabilities, authentication mechanisms, and business logic flaws require source code access or penetration testing.

---

## Technical Recommendations

### Immediate Actions (P0 - Critical)

1. [Most critical action referencing C-XXX]
2. [Continue as needed...]

### High Priority Actions (P1 - Within 2 Weeks)

1. [Action referencing H-XXX]
2. [Continue as needed...]

### Medium Priority Actions (P2 - Within 1-2 Months)

1. [Action referencing M-XXX]
2. [Continue as needed...]

### Security Hardening Recommendations

1. **Implement Subresource Integrity (SRI)**:
   ```html
   <script src="https://cdn.example.com/lib.min.js"
           integrity="sha384-[hash]"
           crossorigin="anonymous"></script>
   ```

2. **Configure Comprehensive Security Headers**:
   [Provide specific header configuration based on actual findings]

3. **Establish Dependency Update Policy**:
   - Monitor security advisories for all dependencies
   - Schedule quarterly dependency update reviews
   - Implement automated vulnerability scanning in CI/CD
   - Test updates in staging before production deployment

---

## Risk Mitigation Priorities

### Phase 1: Critical Vulnerability Remediation (0-48 hours)

- [ ] [Specific task referencing C-XXX finding]
- [ ] [Continue as needed...]

### Phase 2: High Risk Resolution (1-2 weeks)

- [ ] [Specific task referencing H-XXX finding]
- [ ] [Continue as needed...]

### Phase 3: Medium Risk and Platform Updates (1-2 months)

- [ ] [Specific task referencing M-XXX finding]
- [ ] [Continue as needed...]

### Phase 4: Security Hardening (2-3 months)

- [ ] [Security hardening tasks]
- [ ] [Continue as needed...]

---

## Summary

This web dependency security scan identified **X critical**, **Y high**, **Z medium**, and **W low** risk vulnerabilities across the target website. The analysis focused on publicly accessible client-side dependencies, CMS detection, and HTTP security configuration.

**Detected Technology Stack**:
- **Frontend Libraries**: [List detected libraries]
- **CMS/Platform**: [Detected CMS if any]
- **CDN Providers**: [List CDN providers]
- **Security Headers**: X/8 configured

**Critical Areas Requiring Immediate Attention**:
- [Top findings summarized]

**Security Strengths**:
- [Positive findings]

**Next Steps**:
1. Prioritize critical and high-severity findings for immediate remediation
2. Establish a dependency update schedule to prevent future vulnerabilities
3. Consider implementing automated security scanning in development pipeline
4. Schedule follow-up scan after remediation to verify fixes

---

**Scan Limitations**: This scan analyzes only client-side, publicly accessible information. It cannot detect server-side vulnerabilities, authentication bypasses, business logic flaws, or issues requiring source code access. For comprehensive security assessment, consider source code auditing and penetration testing.
</template>

## Severity Assessment Framework

When determining severity for dependency vulnerabilities, apply these criteria:

**CRITICAL (9.0-10.0)**:
- Known CVE with CVSS score >= 9.0
- Actively exploited in the wild
- Remote code execution without authentication
- Complete system compromise possible

**HIGH (7.0-8.9)**:
- Known CVE with CVSS score 7.0-8.9
- Major version outdated with security patches
- Missing critical security headers (CSP, HSTS)
- Exploitable with low complexity

**MEDIUM (4.0-6.9)**:
- Minor version outdated with available security updates
- CMS or platform 2+ versions behind
- Missing recommended security headers
- Requires specific conditions for exploitation

**LOW (0.1-3.9)**:
- Patch version outdated
- Minor security misconfigurations
- Information disclosure risks
- Defense-in-depth improvements

## Best Practices

1. **Comprehensive Detection**: Cast a wide net when detecting libraries. Many sites use multiple frameworks and versions.

2. **Version Precision**: Extract exact version numbers when possible. Semantic versioning (major.minor.patch) is critical for CVE matching.

3. **Context Awareness**: Consider the website's purpose and audience when assessing risk. E-commerce sites handling payments require more stringent security than informational blogs.

4. **Actionable Remediation**: Every finding should include specific upgrade instructions, not just "update to latest."

5. **Migration Planning**: For major version upgrades (e.g., Bootstrap 4->5), acknowledge breaking changes and recommend staged rollout.

6. **Client-Side Limitations**: Be transparent about what cannot be detected from client-side scans (server vulnerabilities, API security, authentication flaws).

7. **False Positive Awareness**: Some version detection methods may be unreliable. Note confidence levels when uncertain.

8. **Prioritize Exploitability**: Focus on vulnerabilities with known exploits and high exploitability scores.

## Quality Assurance Checklist

Before finalizing a web dependency scan report, verify:

- **Have you used ONLY WebFetch or curl to fetch the website?** (NOT Playwright)
- Have HTTP response headers been successfully retrieved and analyzed?
- Have all script and link tags been parsed for library detection?
- Have CDN patterns been checked against all major providers?
- Has CMS detection been attempted using multiple methods?
- Have all critical security headers been checked?
- Has Context7 been used to verify latest versions for major libraries?
- Are remediation recommendations specific with version numbers?
- Have findings been assigned appropriate CVSS-based severity levels?
- Has the report template been followed exactly?
- Have client-side scan limitations been clearly documented?

## Communication Guidelines

When reporting findings:
- Be direct about vulnerabilities while acknowledging scan limitations
- Use precise technical terminology (CVE IDs, CVSS scores)
- Provide concrete upgrade paths with version numbers
- Include before/after examples for configuration changes
- Balance urgency with practicality (acknowledge breaking changes in major upgrades)
- Acknowledge properly configured security controls
- Be transparent about detection confidence levels
- Escalate critical CVEs with clear urgency

Remember: This scan provides visibility into publicly accessible security posture. It complements but does not replace source code auditing, penetration testing, or authenticated security assessments. Focus on actionable findings that can be verified and fixed based on client-side information.
