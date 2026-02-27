---
name: building-ui
description: Complete guide for building beautiful apps with Expo Router. Covers fundamentals, styling, components, navigation, animations, patterns, and native tabs.
version: 1.0.0
license: MIT
---

# Expo UI Guidelines

## References

Consult these resources as needed:

- ./references/route-structure.md -- Route file conventions, dynamic routes, query parameters, groups, and folder organization
- ./references/tabs.md -- Native tab bar with NativeTabs, migration from JS tabs, iOS 26 features
- ./references/icons.md -- SF Symbols with expo-symbols, common icon names, animations, and weights
- ./references/controls.md -- Native iOS controls: Switch, Slider, SegmentedControl, DateTimePicker, Picker
- ./references/visual-effects.md -- Blur effects with expo-blur and liquid glass with expo-glass-effect
- ./references/animations.md -- Reanimated animations: entering, exiting, layout, scroll-driven, and gestures
- ./references/search.md -- Search bar integration with headers, useSearch hook, and filtering patterns
- ./references/gradients.md -- CSS gradients using experimental_backgroundImage (New Architecture only)
- ./references/media.md -- Media handling for Expo Router including camera, audio, video, and file saving
- ./references/storage.md -- Data storage patterns including SQLite, AsyncStorage, and SecureStore
- ./references/webgpu-three.md -- 3D graphics, games, and GPU-powered visualizations with WebGPU and Three.js

## Running the App

**CRITICAL: Always try Expo Go first before creating custom builds.**

Most Expo apps work in Expo Go without any custom native code. Before running `npx expo run:ios` or `npx expo run:android`:

1. **Start with Expo Go**: Run `npx expo start` and scan the QR code with Expo Go
2. **Check if features work**: Test your app thoroughly in Expo Go
3. **Only create custom builds when required** - see below

### When Custom Builds Are Required

- Local Expo modules (custom native code in `modules/` directory)
- Apple targets (widgets, app clips, extensions)
- Third-party native modules not included in Expo Go
- Push notifications with custom configuration

### When Expo Go Works

- Standard Expo SDK modules
- Expo Router navigation
- React Native Reanimated animations
- Most UI components and styling

## Code Style

- Be cautious of unterminated strings. Ensure nested backticks are escaped; never forget to escape quotes correctly.
- Always use import statements at the top of the file.
- Always use kebab-case for file names, e.g. `comment-card.tsx`
- Always remove old route files when moving or restructuring navigation
- Never use special characters in file names
- Configure tsconfig.json with path aliases, and prefer aliases over relative imports for refactors.

## Routes

See `./references/route-structure.md` for detailed route conventions.

- Routes belong in the `app` directory.
- Never co-locate components, types, or utilities in the app directory. This is an anti-pattern.
- Ensure the app always has a route that matches "/", it may be inside a group route.

## Library Preferences

- Never use modules removed from React Native such as Picker, WebView, SafeAreaView, or AsyncStorage
- Never use legacy expo-permissions
- `expo-audio` not `expo-av`
- `expo-video` not `expo-av`
- `expo-symbols` not `@expo/vector-icons`
- `react-native-safe-area-context` not react-native SafeAreaView
- `process.env.EXPO_OS` not `Platform.OS`
- `React.use` not `React.useContext`
- `expo-image` Image component instead of intrinsic element `img`
- `expo-glass-effect` for liquid glass backdrops

## Responsiveness

- Always wrap root component in a scroll view for responsiveness
- Use `<ScrollView contentInsetAdjustmentBehavior="automatic" />` instead of `<SafeAreaView>` for smarter safe area insets
- `contentInsetAdjustmentBehavior="automatic"` should be applied to FlatList and SectionList as well
- Use flexbox instead of Dimensions API
- ALWAYS prefer `useWindowDimensions` over `Dimensions.get()` to measure screen size

## Behavior

- Use expo-haptics conditionally on iOS to make more delightful experiences
- Use views with built-in haptics like `<Switch />` from React Native and `@react-native-community/datetimepicker`
- When a route belongs to a Stack, its first child should almost always be a ScrollView with `contentInsetAdjustmentBehavior="automatic"` set
- Prefer `headerSearchBarOptions` in Stack.Screen options to add a search bar
- Use the `<Text selectable />` prop on text containing data that could be copied
- Consider formatting large numbers like 1.4M or 38k
- Never use intrinsic elements like 'img' or 'div' unless in a webview or Expo DOM component

---

# Styling

## General Styling Rules

- Use StyleSheet.create() for all styles
- Prefer named colors from a theme/design system
- Group related styles together
- Use semantic naming for style objects

## Text Styling

```tsx
import { Text, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    fontWeight: '700',
    color: '#1a1a1a',
  },
  body: {
    fontSize: 16,
    lineHeight: 24,
    color: '#4a4a4a',
  },
});
```

## Shadows

iOS and Android handle shadows differently:

```tsx
const styles = StyleSheet.create({
  card: {
    // iOS shadows
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    // Android shadows
    elevation: 4,
    // Common
    backgroundColor: '#fff',
    borderRadius: 12,
  },
});
```

---

# Navigation

## Link

Use `<Link href="/path" />` from 'expo-router' for navigation between routes.

```tsx
import { Link } from 'expo-router';

// Basic link
<Link href="/path" />

// Wrapping custom components
<Link href="/path" asChild>
  <Pressable>...</Pressable>
</Link>
```

Whenever possible, include a `<Link.Preview>` to follow iOS conventions. Add context menus and previews frequently to enhance navigation.

## Stack

- ALWAYS use `_layout.tsx` files to define stacks
- Use Stack from 'expo-router/stack' for native navigation stacks

### Page Title

Set the page title using Stack.Screen options:

```tsx
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack>
      <Stack.Screen 
        name="index" 
        options={{ title: 'Home' }} 
      />
      <Stack.Screen 
        name="details" 
        options={{ title: 'Details' }} 
      />
    </Stack>
  );
}
```

## Context Menus

Add context menus for enhanced interactions:

```tsx
import { ContextMenu } from 'zeego/context-menu';

<ContextMenu.Root>
  <ContextMenu.Trigger>
    <Pressable>{/* content */}</Pressable>
  </ContextMenu.Trigger>
  <ContextMenu.Content>
    <ContextMenu.Item key="share" onSelect={handleShare}>
      <ContextMenu.ItemTitle>Share</ContextMenu.ItemTitle>
    </ContextMenu.Item>
  </ContextMenu.Content>
</ContextMenu.Root>
```

## Modal

Present modals using the `presentation` option:

```tsx
// In _layout.tsx
<Stack.Screen 
  name="modal" 
  options={{ 
    presentation: 'modal',
    headerShown: false,
  }} 
/>
```

## Sheet

For bottom sheets, use:

```tsx
<Stack.Screen 
  name="sheet" 
  options={{ 
    presentation: 'formSheet',
    sheetGrabberVisible: true,
    sheetInitialDetentIndex: 0,
    sheetAllowedDetents: [0.5, 1],
  }} 
/>
```

## Common route structure

```
app/
├── _layout.tsx        # Root layout (Stack or Tabs)
├── index.tsx          # Home screen "/"
├── (tabs)/
│   ├── _layout.tsx    # Tab navigator
│   ├── index.tsx      # First tab
│   └── settings.tsx   # Settings tab
├── [id].tsx           # Dynamic route "/123"
└── modal.tsx          # Modal screen
```
