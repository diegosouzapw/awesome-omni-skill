---
name: ensuring-mobile-security
description: Ensure app security. Use when handling sensitive data, configuring networking, or preparing for production.
---

# Security Standards

## When to use this skill
- When handling user credentials or sensitive personal data (PII).
- When configuring API calls and networking.
- When implementing authentication logic.
- When preparing the app for a production release.

## Data Storage
- **Credentials**: NEVER store passwords, tokens, or API keys in `AsyncStorage`, `SharedPreferences`, `UserDefaults` (iOS), or `localstorage` (Web).
    - **Use**: Secure storage wrappers like `expo-secure-store` or `flutter_secure_storage` which leverage the OS Keychain/Keystore.
- **Local Data**: Encrypt sensitive databases (e.g., Realm/SQLite encryption) if they contain user data.
- **Cleanup**: Wipe all sensitive local data and tokens immediately upon logout.

## Network Security
- **Transport**: Enforce **HTTPS** for all connections. Block cleartext traffic in `Info.plist` (iOS) and `AndroidManifest.xml` (Android).
- **Certificate Pinning**: Implement certificate pinning for critical APIs to prevent MitM attacks.
- **Validation**: Strict SSL certificate validation.
- **Logging**: NEVER log sensitive headers (Auth tokens) or request bodies containing PII in production or crash reports.

## Authentication
- **Protocol**: Use industry standards like OAuth 2.0 / OIDC. Avoid custom auth schemes.
- **Biometrics**: Use FaceID/TouchID for quick re-auth where appropriate, but always fallback to PIN/Password.
- **Sessions**:
    - Implement short-lived access tokens and long-lived refresh tokens.
    - Enforce local session timeouts for banking/fintech apps.
- **Privacy**: Obscure the app preview in the recent apps switcher (Task Manager) to prevent screenshotting sensitive screens.

## Code Security
- **Production Builds**:
    - Enable code obfuscation (ProGuard/R8 for Android).
    - Strip debug symbols.
    - Remove all `console.log` / `print` statements using build tools (e.g., `babel-plugin-transform-remove-console`).
- **Validation**: Sanitize and validate ALL user inputs on both client and server to prevent injection attacks.
- **Rate Limiting**: Handle 429 Too Many Requests gracefully.
