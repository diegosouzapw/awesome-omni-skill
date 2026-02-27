---
name: secure-storage-patterns
description: expo-secure-store patterns for sensitive data. Use when storing tokens and credentials.
---

# Secure Storage Patterns Skill

This skill covers secure data storage for React Native with Expo.

## When to Use

Use this skill when:
- Storing authentication tokens
- Saving sensitive user data
- Managing API keys
- Storing encryption keys

## Core Principle

**SECURE BY DEFAULT** - Always use SecureStore for sensitive data, never AsyncStorage.

## Installation

```bash
npx expo install expo-secure-store
```

## Basic Usage

```typescript
import * as SecureStore from 'expo-secure-store';

// Save value
await SecureStore.setItemAsync('authToken', 'your-token-here');

// Get value
const token = await SecureStore.getItemAsync('authToken');

// Delete value
await SecureStore.deleteItemAsync('authToken');
```

## Authentication Token Storage

```typescript
// lib/secureStorage.ts
import * as SecureStore from 'expo-secure-store';

const TOKEN_KEY = 'authToken';
const REFRESH_TOKEN_KEY = 'refreshToken';
const USER_KEY = 'user';

export const secureStorage = {
  // Token management
  async saveToken(token: string): Promise<void> {
    await SecureStore.setItemAsync(TOKEN_KEY, token);
  },

  async getToken(): Promise<string | null> {
    return SecureStore.getItemAsync(TOKEN_KEY);
  },

  async deleteToken(): Promise<void> {
    await SecureStore.deleteItemAsync(TOKEN_KEY);
  },

  // Refresh token
  async saveRefreshToken(token: string): Promise<void> {
    await SecureStore.setItemAsync(REFRESH_TOKEN_KEY, token);
  },

  async getRefreshToken(): Promise<string | null> {
    return SecureStore.getItemAsync(REFRESH_TOKEN_KEY);
  },

  async deleteRefreshToken(): Promise<void> {
    await SecureStore.deleteItemAsync(REFRESH_TOKEN_KEY);
  },

  // User data
  async saveUser(user: User): Promise<void> {
    await SecureStore.setItemAsync(USER_KEY, JSON.stringify(user));
  },

  async getUser(): Promise<User | null> {
    const userStr = await SecureStore.getItemAsync(USER_KEY);
    return userStr ? JSON.parse(userStr) : null;
  },

  async deleteUser(): Promise<void> {
    await SecureStore.deleteItemAsync(USER_KEY);
  },

  // Clear all auth data
  async clearAuth(): Promise<void> {
    await Promise.all([
      SecureStore.deleteItemAsync(TOKEN_KEY),
      SecureStore.deleteItemAsync(REFRESH_TOKEN_KEY),
      SecureStore.deleteItemAsync(USER_KEY),
    ]);
  },
};
```

## Auth Store with Secure Storage

```typescript
import { create } from 'zustand';
import { secureStorage } from '@/lib/secureStorage';

interface User {
  id: string;
  email: string;
  name: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  initialize: () => Promise<void>;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  token: null,
  isAuthenticated: false,
  isLoading: true,

  initialize: async () => {
    try {
      const [token, user] = await Promise.all([
        secureStorage.getToken(),
        secureStorage.getUser(),
      ]);

      if (token && user) {
        set({
          token,
          user,
          isAuthenticated: true,
          isLoading: false,
        });
      } else {
        set({ isLoading: false });
      }
    } catch {
      set({ isLoading: false });
    }
  },

  login: async (email, password) => {
    set({ isLoading: true });

    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) {
      set({ isLoading: false });
      throw new Error('Login failed');
    }

    const { user, token, refreshToken } = await response.json();

    await Promise.all([
      secureStorage.saveToken(token),
      secureStorage.saveRefreshToken(refreshToken),
      secureStorage.saveUser(user),
    ]);

    set({
      user,
      token,
      isAuthenticated: true,
      isLoading: false,
    });
  },

  logout: async () => {
    await secureStorage.clearAuth();
    set({
      user: null,
      token: null,
      isAuthenticated: false,
    });
  },
}));
```

## Initialize Auth on App Start

```typescript
// app/_layout.tsx
import { useEffect } from 'react';
import { useAuthStore } from '@/store/authStore';

export default function RootLayout(): React.ReactElement {
  const initialize = useAuthStore((state) => state.initialize);
  const isLoading = useAuthStore((state) => state.isLoading);

  useEffect(() => {
    initialize();
  }, [initialize]);

  if (isLoading) {
    return <LoadingScreen />;
  }

  return <Stack />;
}
```

## Token Refresh Pattern

```typescript
// lib/api.ts
import axios from 'axios';
import { secureStorage } from './secureStorage';

const api = axios.create({
  baseURL: process.env.EXPO_PUBLIC_API_URL,
});

// Request interceptor - add token
api.interceptors.request.use(async (config) => {
  const token = await secureStorage.getToken();
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor - refresh token on 401
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const refreshToken = await secureStorage.getRefreshToken();

        if (!refreshToken) {
          throw new Error('No refresh token');
        }

        const response = await axios.post('/api/auth/refresh', {
          refreshToken,
        });

        const { token: newToken } = response.data;
        await secureStorage.saveToken(newToken);

        originalRequest.headers.Authorization = `Bearer ${newToken}`;
        return api(originalRequest);
      } catch {
        await secureStorage.clearAuth();
        // Navigate to login
        return Promise.reject(error);
      }
    }

    return Promise.reject(error);
  }
);

export default api;
```

## Biometric Authentication

```typescript
import * as LocalAuthentication from 'expo-local-authentication';

export const biometricAuth = {
  async isAvailable(): Promise<boolean> {
    const compatible = await LocalAuthentication.hasHardwareAsync();
    const enrolled = await LocalAuthentication.isEnrolledAsync();
    return compatible && enrolled;
  },

  async authenticate(): Promise<boolean> {
    const result = await LocalAuthentication.authenticateAsync({
      promptMessage: 'Authenticate to continue',
      fallbackLabel: 'Use passcode',
    });

    return result.success;
  },
};

// Usage with secure storage
async function getSecureData(): Promise<string | null> {
  const authenticated = await biometricAuth.authenticate();

  if (!authenticated) {
    throw new Error('Authentication failed');
  }

  return secureStorage.getToken();
}
```

## Storage Options

```typescript
// SecureStore options
await SecureStore.setItemAsync('key', 'value', {
  keychainAccessible: SecureStore.AFTER_FIRST_UNLOCK,
});

// Available options:
// AFTER_FIRST_UNLOCK - accessible after device unlocked once
// AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY - same, but not synced
// ALWAYS - always accessible (less secure)
// ALWAYS_THIS_DEVICE_ONLY - always accessible, not synced
// WHEN_UNLOCKED - only when device is unlocked (default)
// WHEN_UNLOCKED_THIS_DEVICE_ONLY - same, but not synced
```

## When to Use What

```typescript
// ✅ SecureStore - Sensitive data
await SecureStore.setItemAsync('authToken', token);
await SecureStore.setItemAsync('apiKey', key);
await SecureStore.setItemAsync('encryptionKey', key);

// ❌ AsyncStorage - Non-sensitive data only
await AsyncStorage.setItem('theme', 'dark');
await AsyncStorage.setItem('onboardingComplete', 'true');
await AsyncStorage.setItem('lastViewedProduct', productId);
```

## Notes

- Always use SecureStore for tokens and credentials
- SecureStore has ~2KB limit per item
- Data is encrypted using device keychain/keystore
- Consider biometric authentication for extra security
- Clear sensitive data on logout
- Handle storage errors gracefully
