---
name: flutter-app-builder
description: Complete Flutter mobile app development from initial setup through App Store deployment. Use when building Flutter apps, adding features (authentication, databases, APIs), implementing security, or preparing apps for production release. Includes project templates, architecture patterns, and deployment guidance.
---

# Flutter App Builder

Build production-ready Flutter apps with authentication, databases, and security features ready for App Store deployment.

## Development workflow

1. **Initialize project** - Create Flutter project structure with proper architecture
2. **Implement core features** - Add authentication, database, state management
3. **Add security** - Implement secure storage, API security, data encryption
4. **Prepare for deployment** - Configure app signing, store listings, build releases
5. **Deploy** - Submit to App Store and Google Play

## Project initialization

Start by copying the base template from `assets/flutter-template/` to create a well-structured Flutter project:

```bash
cp -r assets/flutter-template/ <project-name>
cd <project-name>
flutter pub get
```

The template includes:
- Clean architecture with feature-based organization
- State management setup (Provider/Riverpod)
- Navigation structure
- Theme configuration
- Common utilities

## Core feature implementation

### Authentication

For authentication patterns and implementation guides, see `references/authentication.md`.

Common patterns:
- **Firebase Auth**: Email/password, Google Sign-In, Apple Sign-In
- **Supabase Auth**: Similar to Firebase with PostgreSQL backend
- **Custom backend**: JWT tokens with secure storage

### Database integration

For database setup and patterns, see `references/database.md`.

Options:
- **Firebase Firestore**: Real-time NoSQL, good for most apps
- **Supabase**: PostgreSQL with real-time features
- **SQLite**: Local-first apps with offline support
- **Hive/Isar**: Fast local storage for Flutter

### State management

The template uses Provider by default. For complex apps, consider:
- **Riverpod**: Modern, compile-safe Provider
- **Bloc**: Structured state management with events
- **GetX**: All-in-one solution (state + routing + dependencies)

See `references/state-management.md` for implementation patterns.

## Security implementation

CRITICAL security practices:

1. **Secure storage**: Never store sensitive data in SharedPreferences
   - Use `flutter_secure_storage` for tokens, API keys
   - See `references/security.md` for patterns

2. **API security**: 
   - Use HTTPS only
   - Implement certificate pinning for sensitive apps
   - Store API keys in environment variables, not code

3. **Data encryption**:
   - Encrypt sensitive local data
   - Use encrypted databases for sensitive information

4. **Input validation**:
   - Validate all user inputs
   - Sanitize data before database operations

Full security checklist in `references/security.md`.

## App Store deployment

### Pre-deployment checklist

1. **Update app metadata**:
   - App name, version, build number
   - Bundle identifier (iOS) / Application ID (Android)
   - Icons and splash screens

2. **Configure signing**:
   - iOS: Generate certificates, provisioning profiles
   - Android: Generate signing keystore
   - See `references/deployment.md` for step-by-step

3. **Test thoroughly**:
   - Run on real devices (iOS and Android)
   - Test all features, especially authentication flows
   - Check performance and memory usage

4. **Build release versions**:
   ```bash
   flutter build ios --release
   flutter build appbundle --release
   ```

### Store submission

Complete submission guides in `references/deployment.md`:
- App Store Connect configuration
- Google Play Console setup
- Screenshots, descriptions, privacy policies
- Review guidelines compliance

## Testing and debugging

Use these scripts for common development tasks:

```bash
# Run with hot reload
flutter run

# Run tests
flutter test

# Analyze code quality
flutter analyze

# Check for outdated packages
flutter pub outdated
```

## Common patterns and solutions

For frequently needed implementations:
- **Push notifications**: See `references/features.md`
- **In-app purchases**: See `references/features.md`
- **Location services**: See `references/features.md`
- **Camera and image handling**: See `references/features.md`
- **API integration**: See `references/api-integration.md`

## Troubleshooting

Common issues and solutions in `references/troubleshooting.md`.
