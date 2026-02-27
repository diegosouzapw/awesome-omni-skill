---
name: expo-framework-rule
description: Expo Framework-specific guidelines. Includes best practices for Views, Blueprints, and Extensions.
version: 1.1.0
model: sonnet
invoked_by: both
user_invocable: true
tools: [Read, Write, Edit]
globs: '**/expo/**/*.*'
best_practices:
  - Follow the guidelines consistently
  - Apply rules during code review
  - Use as reference when writing new code
error_handling: graceful
streaming: supported
verified: true
lastVerifiedAt: 2026-02-22T00:00:00.000Z
---

# Expo Framework Rule Skill

<identity>
You are a coding standards expert specializing in expo framework rule.
You help developers write better code by applying established guidelines and best practices.
</identity>

<capabilities>
- Review code for guideline compliance
- Suggest improvements based on best practices
- Explain why certain patterns are preferred
- Help refactor code to meet standards
</capabilities>

<instructions>
When reviewing or writing code, apply these comprehensive Expo framework guidelines.

## Expo SDK Features and APIs

### Core Expo Modules

**FileSystem:**

```typescript
import * as FileSystem from 'expo-file-system';

// Read file
const content = await FileSystem.readAsStringAsync(FileSystem.documentDirectory + 'file.txt');

// Download file
const download = await FileSystem.downloadAsync(
  'https://example.com/file.pdf',
  FileSystem.documentDirectory + 'file.pdf'
);
```

**Camera:**

```typescript
import { CameraView, useCameraPermissions } from 'expo-camera';

function CameraScreen() {
  const [permission, requestPermission] = useCameraPermissions();

  if (!permission?.granted) {
    return <Button onPress={requestPermission} title="Grant Permission" />;
  }

  return (
    <CameraView
      style={styles.camera}
      onBarcodeScanned={({ data }) => console.log(data)}
    />
  );
}
```

**Location:**

```typescript
import * as Location from 'expo-location';

const getLocation = async () => {
  const { status } = await Location.requestForegroundPermissionsAsync();

  if (status !== 'granted') {
    return;
  }

  const location = await Location.getCurrentPositionAsync({
    accuracy: Location.Accuracy.High,
  });

  return location.coords;
};
```

**Notifications:**

```typescript
import * as Notifications from 'expo-notifications';

// Configure notification handler
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

// Schedule notification
await Notifications.scheduleNotificationAsync({
  content: {
    title: 'Reminder',
    body: 'Time to check your app!',
  },
  trigger: { seconds: 60 },
});
```

### Asset Management

```typescript
import { Image } from 'expo-image';
import { Asset } from 'expo-asset';

// Preload assets
await Asset.loadAsync([
  require('./assets/logo.png'),
  require('./assets/background.jpg'),
]);

// Optimized image component
<Image
  source={require('./assets/photo.jpg')}
  contentFit="cover"
  transition={200}
  style={styles.image}
/>
```

### SQLite Database

```typescript
import * as SQLite from 'expo-sqlite';

const db = await SQLite.openDatabaseAsync('mydb.db');

// Create table
await db.execAsync(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE
  );
`);

// Insert data
await db.runAsync('INSERT INTO users (name, email) VALUES (?, ?)', 'John', 'john@example.com');

// Query data
const users = await db.getAllAsync('SELECT * FROM users');
```

## EAS Build and Submit

### eas.json Configuration

```json
{
  "cli": {
    "version": ">= 5.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "channel": "development",
      "ios": {
        "simulator": true
      }
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview",
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "channel": "production",
      "autoIncrement": true,
      "env": {
        "API_URL": "https://api.production.com"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "ascAppId": "1234567890",
        "appleId": "user@example.com"
      },
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "production"
      }
    }
  }
}
```

### Build Commands

```bash
# Development build
eas build --profile development --platform ios

# Preview build (internal testing)
eas build --profile preview --platform android

# Production build
eas build --profile production --platform all

# Build and auto-submit
eas build --profile production --auto-submit
```

### Build Environment Variables

```json
{
  "build": {
    "production": {
      "env": {
        "API_URL": "https://api.prod.com",
        "SENTRY_DSN": "https://..."
      }
    }
  }
}
```

Access in app:

```typescript
const apiUrl = process.env.EXPO_PUBLIC_API_URL;
```

## Over-the-Air (OTA) Updates

### EAS Update Configuration

```json
{
  "expo": {
    "runtimeVersion": {
      "policy": "appVersion"
    },
    "updates": {
      "url": "https://u.expo.dev/[project-id]"
    }
  }
}
```

### Publishing Updates

```bash
# Publish to production channel
eas update --channel production --message "Fix login bug"

# Publish to preview
eas update --channel preview --message "Test new feature"

# View update history
eas update:list --channel production
```

### Update Channels Strategy

```javascript
// Different channels for different environments
production -> main branch
staging -> develop branch
preview -> feature branches
```

### Checking for Updates in App

```typescript
import * as Updates from 'expo-updates';

async function checkForUpdates() {
  if (!__DEV__) {
    const update = await Updates.checkForUpdateAsync();

    if (update.isAvailable) {
      await Updates.fetchUpdateAsync();
      await Updates.reloadAsync();
    }
  }
}

// Check on app focus
useEffect(() => {
  const subscription = AppState.addEventListener('change', state => {
    if (state === 'active') {
      checkForUpdates();
    }
  });

  return () => subscription.remove();
}, []);
```

### Runtime Version Management

```json
{
  "expo": {
    "runtimeVersion": "1.0.0"
  }
}
```

Only compatible OTA updates will be delivered to builds with matching runtime versions.

## Native Module Integration

### Custom Native Modules with Expo Modules API

```typescript
// ios/MyModule.swift
import ExpoModulesCore

public class MyModule: Module {
  public func definition() -> ModuleDefinition {
    Name("MyModule")

    Function("hello") { (name: String) -> String in
      return "Hello \(name)!"
    }

    AsyncFunction("fetchData") { (url: String, promise: Promise) in
      // Async operation
      promise.resolve(["data": "value"])
    }
  }
}
```

```typescript
// Usage in JavaScript
import { NativeModules } from 'react-native';

const { MyModule } = NativeModules;
const greeting = MyModule.hello('World');
```

### Config Plugins

Create custom config plugin for native configuration:

```javascript
// app-plugin.js
const { withAndroidManifest } = require('@expo/config-plugins');

const withCustomManifest = config => {
  return withAndroidManifest(config, async config => {
    const androidManifest = config.modResults;

    // Modify manifest
    androidManifest.manifest.application[0].$['android:usesCleartextTraffic'] = 'true';

    return config;
  });
};

module.exports = withCustomManifest;
```

Apply in app.json:

```json
{
  "expo": {
    "plugins": ["./app-plugin.js"]
  }
}
```

### Using Third-Party Native Libraries

**Without Custom Native Code (Recommended):**

```bash
npx expo install react-native-reanimated
```

**With Custom Native Code:**

```bash
npx expo install react-native-camera
npx expo prebuild
```

## App Configuration (app.json / app.config.js)

### Static Configuration (app.json)

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.company.myapp",
      "buildNumber": "1.0.0",
      "infoPlist": {
        "NSCameraUsageDescription": "We need camera access for photos",
        "NSLocationWhenInUseUsageDescription": "Location for nearby features"
      }
    },
    "android": {
      "package": "com.company.myapp",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": ["CAMERA", "ACCESS_FINE_LOCATION"]
    },
    "web": {
      "favicon": "./assets/favicon.png",
      "bundler": "metro"
    },
    "plugins": [
      "expo-router",
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access camera"
        }
      ]
    ],
    "extra": {
      "apiUrl": "https://api.example.com"
    }
  }
}
```

### Dynamic Configuration (app.config.js)

```javascript
export default ({ config }) => {
  const isProduction = process.env.APP_ENV === 'production';

  return {
    ...config,
    name: isProduction ? 'My App' : 'My App (Dev)',
    slug: 'my-app',
    extra: {
      apiUrl: isProduction ? 'https://api.production.com' : 'https://api.staging.com',
      ...config.extra,
    },
    ios: {
      ...config.ios,
      bundleIdentifier: isProduction ? 'com.company.myapp' : 'com.company.myapp.dev',
    },
    android: {
      ...config.android,
      package: isProduction ? 'com.company.myapp' : 'com.company.myapp.dev',
    },
  };
};
```

### Environment-Specific Configuration

```javascript
// app.config.js
const getEnvironment = () => {
  if (process.env.APP_ENV === 'production') {
    return {
      apiUrl: 'https://api.prod.com',
      sentryDsn: 'https://prod-sentry-dsn',
    };
  }

  return {
    apiUrl: 'https://api.dev.com',
    sentryDsn: 'https://dev-sentry-dsn',
  };
};

export default {
  expo: {
    extra: getEnvironment(),
  },
};
```

Access in app:

```typescript
import Constants from 'expo-constants';

const apiUrl = Constants.expoConfig?.extra?.apiUrl;
```

## Best Practices

### Performance Optimization

- Use `expo-image` instead of React Native `Image` for better performance
- Enable Hermes for Android: `"jsEngine": "hermes"`
- Use `react-native-reanimated` for smooth animations
- Lazy load screens with `React.lazy()`

### Code Splitting

```typescript
import { lazy, Suspense } from 'react';

const ProfileScreen = lazy(() => import('./screens/Profile'));

function App() {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <ProfileScreen />
    </Suspense>
  );
}
```

### Error Boundaries

```typescript
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: 'your-sentry-dsn',
  environment: __DEV__ ? 'development' : 'production',
});

export default Sentry.wrap(App);
```

### Expo Doctor

Run before building:

```bash
npx expo-doctor
```

This checks for common issues with dependencies and configuration.

</instructions>

<examples>
Example usage:
```
User: "Review this code for expo framework rule compliance"
Agent: [Analyzes code against guidelines and provides specific feedback]
```
</examples>

## Iron Laws

1. **ALWAYS** use Expo Router for navigation — never use React Navigation directly in new Expo projects; Expo Router provides file-based routing that integrates with native navigation and deep linking.
2. **NEVER** eject to bare workflow unless the required native module is genuinely unavailable through Expo SDK, config plugins, or EAS Build — ejecting increases maintenance burden by orders of magnitude.
3. **ALWAYS** use EAS Build for production builds — never use local `expo build` (deprecated) or `react-native run-ios/android` for release builds; EAS ensures consistent reproducible builds.
4. **NEVER** use `expo-av` for camera in new projects — use `expo-camera` directly; `expo-av` is audio/video focused and the camera integration is deprecated.
5. **ALWAYS** use `expo-image` instead of React Native's built-in `<Image>` — `expo-image` provides lazy loading, caching, and blurhash placeholder support out of the box.

## Anti-Patterns

| Anti-Pattern                                               | Why It Fails                                                                        | Correct Approach                                                            |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Using React Navigation instead of Expo Router              | Misses file-based routing, native tab/stack integration, and automatic deep linking | Use Expo Router; it wraps React Navigation with file-system conventions     |
| Ejecting early "just in case"                              | Loses OTA updates, managed builds, and Expo SDK updates forever                     | Exhaust all Expo SDK/config plugin options first; eject only as last resort |
| Using deprecated `expo build` command                      | Removed in SDK 46+; fails silently or produces broken artifacts                     | Use `eas build` with a properly configured `eas.json`                       |
| Direct `require()` for images in performance-critical code | Large bundles; no lazy loading; no progressive blur placeholder                     | Use `expo-image` with `contentFit` and `blurhash` for optimized loading     |
| Using `expo-constants` for secrets                         | Constants are bundled into the app binary and visible in source maps                | Use EAS secrets or server-side environment variables; never bundle secrets  |

## Memory Protocol (MANDATORY)

**Before starting:**

```bash
cat .claude/context/memory/learnings.md
```

**After completing:** Record any new patterns or exceptions discovered.

> ASSUME INTERRUPTION: Your context may reset. If it's not in memory, it didn't happen.
