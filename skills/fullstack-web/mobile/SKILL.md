---
name: mobile
description: "React Native, Expo, mobile apps. Auto-use for mobile work."
version: 2.0.0
allowed-tools:
  - Read
  - Edit
  - Write
  - Bash
  - Grep
  - Glob
---

# Mobile

**Auto-use when:** Expo, React Native, mobile, app, NativeWind, iOS, Android

**Works with:** `backend` for API, `quality` for testing

---

## Quick Setup

### File Structure
```
app/
├── _layout.tsx        # Root (providers, fonts)
├── index.tsx          # Home
├── (auth)/            # Auth group
│   ├── _layout.tsx
│   └── login.tsx
├── (tabs)/            # Tab group (authenticated)
│   ├── _layout.tsx
│   ├── index.tsx
│   └── profile.tsx
└── [id].tsx           # Dynamic route
```

### Root Layout
```typescript
// app/_layout.tsx
import { Stack } from 'expo-router'
import { QueryClientProvider } from '@tanstack/react-query'
import '../global.css'

export default function RootLayout() {
  return (
    <QueryClientProvider client={queryClient}>
      <AuthProvider>
        <Stack screenOptions={{ headerShown: false }} />
      </AuthProvider>
    </QueryClientProvider>
  )
}
```

### Protected Tabs
```typescript
// app/(tabs)/_layout.tsx
import { Tabs, Redirect } from 'expo-router'
import { useAuth } from '@/providers/auth'

export default function TabLayout() {
  const { session, isLoading } = useAuth()

  if (isLoading) return null
  if (!session) return <Redirect href="/login" />

  return (
    <Tabs>
      <Tabs.Screen name="index" options={{ title: 'Home' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile' }} />
    </Tabs>
  )
}
```

---

## NativeWind

```typescript
// Usage (Tailwind syntax)
<View className="flex-1 bg-white p-4">
  <Text className="text-lg font-bold text-gray-900">Title</Text>
  <Pressable className="bg-blue-500 p-4 rounded-lg active:opacity-70">
    <Text className="text-white text-center">Button</Text>
  </Pressable>
</View>

// Platform-specific
<View className="ios:pt-12 android:pt-8" />

// Dark mode
<Text className="text-gray-900 dark:text-white" />
```

---

## Supabase Mobile

### Client
```typescript
// lib/supabase.ts
import AsyncStorage from '@react-native-async-storage/async-storage'
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(url, key, {
  auth: {
    storage: AsyncStorage,
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: false,  // Important for mobile
  },
})
```

### Realtime with Cleanup (CRITICAL)
```typescript
useEffect(() => {
  let channel: RealtimeChannel

  const setup = async () => {
    const { data } = await supabase.from('items').select('*')
    setData(data || [])

    channel = supabase
      .channel(`room:${roomId}`)
      .on('postgres_changes', { event: '*', schema: 'public', table: 'items' },
        (payload) => setData(prev => [...prev, payload.new])
      )
      .subscribe()
  }

  setup()

  // CRITICAL: Cleanup
  return () => {
    if (channel) supabase.removeChannel(channel)
  }
}, [roomId])
```

---

## Common Patterns

### Navigation
```typescript
import { router, useLocalSearchParams, Link } from 'expo-router'

router.push('/profile')
router.push({ pathname: '/user/[id]', params: { id: '123' } })
router.replace('/login')
router.back()

const { id } = useLocalSearchParams<{ id: string }>()
```

### Platform-Specific
```typescript
import { Platform } from 'react-native'

const padding = Platform.OS === 'ios' ? 50 : 30

const styles = Platform.select({
  ios: { shadowColor: '#000', shadowOpacity: 0.1 },
  android: { elevation: 4 },
})
```

### Safe Area
```typescript
import { useSafeAreaInsets } from 'react-native-safe-area-context'

const insets = useSafeAreaInsets()
<View style={{ paddingTop: insets.top }} />

// Or with NativeWind
<View className="pt-safe pb-safe" />
```

---

## EAS Commands

```bash
# Development build
eas build --profile development --platform ios

# Preview (internal testing)
eas build --profile preview --platform all

# Production
eas build --profile production --platform all

# Submit to stores
eas submit --platform ios
eas submit --platform android

# OTA update
eas update --branch production --message "Bug fixes"
```

---

## Package Installation

```bash
# ALWAYS use npx expo install (not npm install)
npx expo install package-name

# Common packages
npx expo install react-native-reanimated
npx expo install @react-native-async-storage/async-storage
npx expo install expo-image-picker
npx expo install expo-secure-store
```

---

## Checklist

```
[] Uses REAL data (not mock)
[] Loading states
[] Error states
[] Empty states
[] Cleanup subscriptions on unmount
[] Use expo-secure-store for secrets
[] HTTPS for all API calls
[] Platform-specific handling where needed
```
