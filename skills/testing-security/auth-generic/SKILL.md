---
name: auth-generic
description: "Generic authentication skill for any custom auth system. Use when project.json has authentication.provider: custom or when no specific auth skill exists. Triggers on: custom auth, generic login, form-based auth, custom login flow."
---

# Generic Authentication

Authenticate with any form-based authentication system using configurable selectors. This is the fallback skill when no provider-specific skill exists.

---

## Prerequisites

1. **Project configuration** in `docs/project.json`:
   ```json
   {
     "authentication": {
       "method": "email-password",
       "provider": "custom",
       "testUser": {
         "mode": "fixed",
         "emailVar": "TEST_EMAIL",
         "emailDefault": "test@example.com",
         "passwordVar": "TEST_PASSWORD"
       },
       "routes": {
         "login": "/login",
         "authenticated": "/dashboard"
       },
       "selectors": {
         "emailInput": "input[name='email']",
         "passwordInput": "input[name='password']",
         "submitButton": "button[type='submit']"
       }
     }
   }
   ```

2. **Environment variables** in `.env.local`:
   ```
   TEST_EMAIL=test@example.com
   TEST_PASSWORD=your-test-password
   ```

3. **Test user exists** in your authentication system

---

## The Job

1. Read authentication config from `project.json`
2. Navigate to login page
3. Fill form fields using configured selectors
4. Submit the form
5. Wait for redirect to authenticated page
6. Return authenticated page context

---

## Supported Authentication Methods

### Email/Password

Standard username/password login:

```json
{
  "authentication": {
    "method": "email-password",
    "selectors": {
      "emailInput": "input[type='email'], input[name='email'], input#email",
      "passwordInput": "input[type='password'], input[name='password']",
      "submitButton": "button[type='submit']"
    }
  }
}
```

### Passwordless OTP

Two-step OTP verification:

```json
{
  "authentication": {
    "method": "passwordless-otp",
    "selectors": {
      "emailInput": "input[type='email']",
      "submitEmailButton": "button:has-text('Send code')",
      "otpInputs": "input.otp-digit",
      "submitOtpButton": "button:has-text('Verify')"
    },
    "verification": {
      "source": "api",
      "endpoint": "/api/test/get-otp",
      "method": "POST",
      "bodyTemplate": "{\"email\": \"{{email}}\"}"
    },
    "routes": {
      "login": "/login",
      "verify": "/verify",
      "authenticated": "/dashboard"
    }
  }
}
```

### Magic Link (with test override)

For magic link flows, configure a test bypass:

```json
{
  "authentication": {
    "method": "magic-link",
    "selectors": {
      "emailInput": "input[type='email']",
      "submitButton": "button:has-text('Send link')"
    },
    "verification": {
      "source": "database",
      "query": "SELECT token FROM magic_links WHERE email = ? ORDER BY created_at DESC LIMIT 1"
    },
    "routes": {
      "login": "/login",
      "magicLinkPattern": "/auth/verify?token={{token}}",
      "authenticated": "/dashboard"
    }
  }
}
```

---

## Authentication Flow

### Step 1: Load Configuration

```typescript
import * as fs from 'fs';
import * as path from 'path';

interface Selectors {
  emailInput?: string;
  passwordInput?: string;
  submitButton?: string;
  submitEmailButton?: string;
  otpInputs?: string;
  submitOtpButton?: string;
  usernameInput?: string;
}

interface AuthConfig {
  method: string;
  provider: string;
  testUser: {
    mode: 'fixed' | 'dynamic';
    emailVar?: string;
    emailDefault?: string;
    emailPattern?: string;
    passwordVar?: string;
    passwordDefault?: string;
    usernameVar?: string;
  };
  routes: {
    login: string;
    verify?: string;
    authenticated: string;
    magicLinkPattern?: string;
  };
  selectors: Selectors;
  verification?: {
    source: string;
    endpoint?: string;
    method?: string;
    bodyTemplate?: string;
    query?: string;
  };
  steps?: AuthStep[];
}

interface AuthStep {
  action: 'fill' | 'click' | 'wait' | 'navigate' | 'custom';
  selector?: string;
  value?: string;
  waitFor?: string;
  url?: string;
  timeout?: number;
}

function loadAuthConfig(projectRoot: string): AuthConfig {
  const projectJsonPath = path.join(projectRoot, 'docs', 'project.json');
  const projectJson = JSON.parse(fs.readFileSync(projectJsonPath, 'utf-8'));
  
  if (!projectJson.authentication) {
    throw new Error('No authentication config found in project.json. Run /setup-auth first.');
  }
  
  // Ensure selectors exist with defaults
  const config = projectJson.authentication;
  config.selectors = config.selectors || {};
  
  return config;
}

function loadEnv(projectRoot: string): void {
  const envPath = path.join(projectRoot, '.env.local');
  if (fs.existsSync(envPath)) {
    const content = fs.readFileSync(envPath, 'utf-8');
    content.split('\n').forEach(line => {
      const match = line.trim().match(/^([^=]+)=(.*)$/);
      if (match && !process.env[match[1]]) {
        process.env[match[1]] = match[2];
      }
    });
  }
}

function getTestCredentials(config: AuthConfig): { email: string; password?: string; username?: string } {
  let email: string;
  let password: string | undefined;
  let username: string | undefined;
  
  if (config.testUser.mode === 'dynamic') {
    const uuid = crypto.randomUUID().slice(0, 8);
    const pattern = config.testUser.emailPattern || 'test-{uuid}@example.com';
    email = pattern.replace('{uuid}', uuid);
    password = process.env[config.testUser.passwordVar || 'TEST_PASSWORD'] 
      || config.testUser.passwordDefault;
  } else {
    const emailVar = config.testUser.emailVar || 'TEST_EMAIL';
    email = process.env[emailVar] || config.testUser.emailDefault || 'test@example.com';
    
    if (config.method === 'email-password') {
      const passwordVar = config.testUser.passwordVar || 'TEST_PASSWORD';
      password = process.env[passwordVar] || config.testUser.passwordDefault;
      
      if (!password) {
        throw new Error(`Password not configured. Set ${passwordVar} in .env.local`);
      }
    }
    
    if (config.testUser.usernameVar) {
      username = process.env[config.testUser.usernameVar];
    }
  }
  
  return { email, password, username };
}
```

### Step 2: Default Selectors

```typescript
const DEFAULT_SELECTORS: Selectors = {
  emailInput: 'input[type="email"], input[name="email"], input#email, input[autocomplete="email"]',
  passwordInput: 'input[type="password"], input[name="password"], input#password',
  submitButton: 'button[type="submit"], button:has-text("Sign in"), button:has-text("Log in"), button:has-text("Continue")',
  otpInputs: 'input[maxlength="1"], input.otp-input, input[inputmode="numeric"]',
  usernameInput: 'input[name="username"], input#username, input[autocomplete="username"]'
};

function getSelector(config: AuthConfig, key: keyof Selectors): string {
  return config.selectors[key] || DEFAULT_SELECTORS[key] || '';
}
```

### Step 3: Authenticate with Playwright

```typescript
import { Page } from 'playwright';

interface AuthResult {
  success: boolean;
  email: string;
  error?: string;
}

async function authenticateGeneric(
  page: Page,
  baseUrl: string,
  projectRoot: string
): Promise<AuthResult> {
  loadEnv(projectRoot);
  const config = loadAuthConfig(projectRoot);
  const credentials = getTestCredentials(config);
  
  try {
    // Navigate to login
    await page.goto(`${baseUrl}${config.routes.login}`);
    
    // Handle based on method
    switch (config.method) {
      case 'email-password':
        await handleEmailPassword(page, config, credentials);
        break;
      case 'passwordless-otp':
        await handlePasswordlessOtp(page, config, credentials, baseUrl);
        break;
      case 'magic-link':
        await handleMagicLink(page, config, credentials, baseUrl);
        break;
      default:
        // If custom steps are defined, execute them
        if (config.steps) {
          await executeCustomSteps(page, config.steps, credentials);
        } else {
          throw new Error(`Unknown auth method: ${config.method}. Define custom steps.`);
        }
    }
    
    // Wait for authenticated page
    await page.waitForURL(new RegExp(config.routes.authenticated), { timeout: 15000 });
    console.log(`Authentication successful, landed on ${config.routes.authenticated}`);
    
    return { success: true, email: credentials.email };
    
  } catch (error) {
    const message = error instanceof Error ? error.message : String(error);
    console.error(`Authentication failed: ${message}`);
    return { success: false, email: credentials.email, error: message };
  }
}

async function handleEmailPassword(
  page: Page,
  config: AuthConfig,
  credentials: { email: string; password?: string; username?: string }
): Promise<void> {
  const emailSelector = getSelector(config, 'emailInput');
  const passwordSelector = getSelector(config, 'passwordInput');
  const submitSelector = getSelector(config, 'submitButton');
  
  await page.waitForSelector(emailSelector);
  
  // Fill username if configured, otherwise email
  if (credentials.username && config.selectors.usernameInput) {
    await page.fill(config.selectors.usernameInput, credentials.username);
  } else {
    await page.fill(emailSelector, credentials.email);
  }
  
  await page.fill(passwordSelector, credentials.password!);
  await page.click(submitSelector);
}

async function handlePasswordlessOtp(
  page: Page,
  config: AuthConfig,
  credentials: { email: string },
  baseUrl: string
): Promise<void> {
  const emailSelector = getSelector(config, 'emailInput');
  const submitEmailSelector = config.selectors.submitEmailButton || getSelector(config, 'submitButton');
  
  // Step 1: Enter email
  await page.waitForSelector(emailSelector);
  await page.fill(emailSelector, credentials.email);
  await page.click(submitEmailSelector);
  
  // Step 2: Wait for verify page
  if (config.routes.verify) {
    await page.waitForURL(new RegExp(config.routes.verify));
  }
  await page.waitForTimeout(1000); // Allow OTP to be generated
  
  // Step 3: Get OTP code
  const otp = await fetchOtpCode(config, credentials.email, baseUrl);
  
  // Step 4: Enter OTP
  const otpSelector = getSelector(config, 'otpInputs');
  const otpInputs = page.locator(otpSelector);
  const inputCount = await otpInputs.count();
  
  if (inputCount >= 6) {
    // Multiple single-digit inputs
    for (let i = 0; i < Math.min(otp.length, inputCount); i++) {
      await otpInputs.nth(i).fill(otp[i]);
    }
  } else {
    // Single input for full code
    await otpInputs.first().fill(otp);
  }
  
  // Step 5: Submit OTP
  const submitOtpSelector = config.selectors.submitOtpButton || getSelector(config, 'submitButton');
  await page.click(submitOtpSelector);
}

async function handleMagicLink(
  page: Page,
  config: AuthConfig,
  credentials: { email: string },
  baseUrl: string
): Promise<void> {
  const emailSelector = getSelector(config, 'emailInput');
  const submitSelector = getSelector(config, 'submitButton');
  
  // Enter email
  await page.waitForSelector(emailSelector);
  await page.fill(emailSelector, credentials.email);
  await page.click(submitSelector);
  
  // Wait for magic link to be created
  await page.waitForTimeout(1000);
  
  // Fetch token
  const token = await fetchMagicLinkToken(config, credentials.email);
  
  // Navigate directly to magic link URL
  const magicLinkUrl = config.routes.magicLinkPattern!.replace('{{token}}', token);
  await page.goto(`${baseUrl}${magicLinkUrl}`);
}

async function fetchOtpCode(
  config: AuthConfig,
  email: string,
  baseUrl: string
): Promise<string> {
  if (!config.verification) {
    throw new Error('OTP verification config required for passwordless-otp method');
  }
  
  if (config.verification.source === 'api') {
    const body = config.verification.bodyTemplate?.replace('{{email}}', email) || JSON.stringify({ email });
    const response = await fetch(`${baseUrl}${config.verification.endpoint}`, {
      method: config.verification.method || 'POST',
      headers: { 'Content-Type': 'application/json' },
      body
    });
    const data = await response.json();
    return data.code || data.otp || data.token;
  }
  
  throw new Error(`OTP source "${config.verification.source}" not implemented in generic skill`);
}

async function fetchMagicLinkToken(config: AuthConfig, email: string): Promise<string> {
  // This would need project-specific implementation
  // Generic skill can call an API endpoint if configured
  throw new Error('Magic link token retrieval requires project-specific implementation');
}

async function executeCustomSteps(
  page: Page,
  steps: AuthStep[],
  credentials: { email: string; password?: string }
): Promise<void> {
  for (const step of steps) {
    const value = step.value
      ?.replace('{{email}}', credentials.email)
      ?.replace('{{password}}', credentials.password || '');
    
    switch (step.action) {
      case 'fill':
        await page.fill(step.selector!, value!);
        break;
      case 'click':
        await page.click(step.selector!);
        break;
      case 'wait':
        if (step.selector) {
          await page.waitForSelector(step.selector, { timeout: step.timeout || 5000 });
        } else if (step.url) {
          await page.waitForURL(new RegExp(step.url), { timeout: step.timeout || 10000 });
        } else {
          await page.waitForTimeout(step.timeout || 1000);
        }
        break;
      case 'navigate':
        await page.goto(step.url!);
        break;
    }
  }
}
```

---

## Complete Script Template

```typescript
import { chromium } from 'playwright';
import * as fs from 'fs';
import * as path from 'path';

const PROJECT_ROOT = process.cwd();
const BASE_URL = process.env.PLAYWRIGHT_BASE_URL || 'http://localhost:3000';

// (Include helper functions from above)

async function main() {
  const browser = await chromium.launch();
  const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 }
  });
  const page = await context.newPage();
  
  try {
    const result = await authenticateGeneric(page, BASE_URL, PROJECT_ROOT);
    
    if (!result.success) {
      console.error('Authentication failed:', result.error);
      process.exit(1);
    }
    
    console.log(`Logged in as ${result.email}`);
    
  } finally {
    await browser.close();
  }
}

main().catch(console.error);
```

---

## Custom Steps Example

For complex multi-step authentication flows, define custom steps:

```json
{
  "authentication": {
    "method": "custom",
    "provider": "custom",
    "testUser": {
      "mode": "fixed",
      "emailVar": "TEST_EMAIL",
      "passwordVar": "TEST_PASSWORD"
    },
    "routes": {
      "login": "/login",
      "authenticated": "/app"
    },
    "steps": [
      { "action": "fill", "selector": "input#username", "value": "{{email}}" },
      { "action": "click", "selector": "button:has-text('Next')" },
      { "action": "wait", "selector": "input#password" },
      { "action": "fill", "selector": "input#password", "value": "{{password}}" },
      { "action": "click", "selector": "input#remember-me" },
      { "action": "click", "selector": "button:has-text('Sign in')" }
    ]
  }
}
```

---

## Selector Best Practices

### Prefer Stable Selectors

```json
{
  "selectors": {
    "emailInput": "input[data-testid='email-input']",
    "passwordInput": "input[data-testid='password-input']",
    "submitButton": "button[data-testid='submit-btn']"
  }
}
```

### Fallback Chains

The skill tries selectors in order. Configure multiple fallbacks:

```json
{
  "selectors": {
    "emailInput": "input[data-testid='email'], input[name='email'], input#email"
  }
}
```

### Avoid Fragile Selectors

- ❌ `.login-form > div:nth-child(2) > input`
- ❌ `body > main > form > button`
- ✅ `input[name='email']`
- ✅ `button[type='submit']`
- ✅ `[data-testid='login-button']`

---

## Troubleshooting

### "Selector not found"
- Use browser DevTools to verify the selector
- Check if the element is inside an iframe
- Wait for dynamic content to load

### "Timeout waiting for authenticated page"
- Verify `routes.authenticated` matches the actual redirect URL
- Check for JavaScript errors in browser console
- Increase timeout if the server is slow

### "Unknown auth method"
- Set `method` to one of: `email-password`, `passwordless-otp`, `magic-link`
- Or define custom `steps` for non-standard flows

### Multi-step forms
Use the `steps` array for forms that span multiple pages or have conditional logic.

---

## When to Use Generic vs Provider-Specific Skills

| Scenario | Recommendation |
|----------|----------------|
| Standard Supabase setup | Use `auth-supabase-otp` or `auth-supabase-password` |
| Standard NextAuth setup | Use `auth-nextauth-credentials` |
| Custom auth with standard form | Use `auth-generic` with selectors |
| Complex multi-step auth | Use `auth-generic` with custom steps |
| OAuth flows | Defer to provider-specific skill (future) |

---

## Integration with Other Skills

This skill is the fallback used by:
- `screenshot` - when no provider-specific skill matches
- `e2e-playwright` - for custom auth systems
- `qa-browser-tester` - for any form-based auth

Agents should:
1. Check `project.json` for `authentication` config
2. Try provider-specific skill first (`auth-supabase-*`, `auth-nextauth-*`)
3. Fall back to `auth-generic` if provider is `custom` or unrecognized
4. Call `authenticateGeneric()` before accessing protected pages
