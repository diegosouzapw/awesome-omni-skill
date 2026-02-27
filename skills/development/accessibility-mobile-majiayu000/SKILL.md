---
name: accessibility-mobile
description: React Native accessibility patterns for iOS and Android. Use when implementing a11y features.
---

# Accessibility Mobile Skill

This skill covers accessibility (a11y) best practices for React Native apps.

## When to Use

Use this skill when:
- Building accessible components
- Implementing screen reader support
- Adding accessibility labels
- Testing accessibility

## Core Principle

**INCLUSIVE BY DEFAULT** - Accessibility is not optional. Build for all users.

## Basic Accessibility Props

```typescript
import { TouchableOpacity, Text, View } from 'react-native';

// Accessible button
<TouchableOpacity
  accessible={true}
  accessibilityRole="button"
  accessibilityLabel="Submit form"
  accessibilityHint="Double tap to submit your information"
  onPress={handleSubmit}
>
  <Text>Submit</Text>
</TouchableOpacity>

// Accessible image
<Image
  source={require('./profile.png')}
  accessible={true}
  accessibilityLabel="Profile picture of John Doe"
/>

// Decorative image (hidden from screen readers)
<Image
  source={require('./decoration.png')}
  accessible={false}
  accessibilityElementsHidden={true}
  importantForAccessibility="no-hide-descendants"
/>
```

## Accessibility Roles

```typescript
// Common roles
<TouchableOpacity accessibilityRole="button">
<TouchableOpacity accessibilityRole="link">
<TextInput accessibilityRole="search">
<Switch accessibilityRole="switch">
<Image accessibilityRole="image">
<Text accessibilityRole="header">
<Text accessibilityRole="text">
<View accessibilityRole="alert">
<View accessibilityRole="checkbox">
<View accessibilityRole="radio">
<View accessibilityRole="tab">
<View accessibilityRole="tablist">
<View accessibilityRole="progressbar">
<View accessibilityRole="slider">
```

## Accessible Forms

```typescript
function AccessibleForm(): React.ReactElement {
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');

  return (
    <View>
      {/* Label association */}
      <Text nativeID="emailLabel">Email Address</Text>
      <TextInput
        value={email}
        onChangeText={setEmail}
        accessibilityLabel="Email Address"
        accessibilityLabelledBy="emailLabel"
        accessibilityRole="none"
        keyboardType="email-address"
        autoComplete="email"
        textContentType="emailAddress"
        // Error state
        accessibilityInvalid={!!emailError}
        accessibilityErrorMessage={emailError}
      />
      {emailError && (
        <Text
          accessibilityRole="alert"
          accessibilityLiveRegion="polite"
          className="text-red-500"
        >
          {emailError}
        </Text>
      )}

      <TouchableOpacity
        accessibilityRole="button"
        accessibilityLabel="Submit registration form"
        accessibilityState={{ disabled: !email }}
        disabled={!email}
        onPress={handleSubmit}
      >
        <Text>Submit</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Accessibility State

```typescript
// Toggle state
<TouchableOpacity
  accessibilityRole="checkbox"
  accessibilityState={{
    checked: isChecked,
  }}
  onPress={() => setIsChecked(!isChecked)}
>
  <Text>{isChecked ? '☑' : '☐'} Accept terms</Text>
</TouchableOpacity>

// Expanded state
<TouchableOpacity
  accessibilityRole="button"
  accessibilityState={{
    expanded: isExpanded,
  }}
  onPress={() => setIsExpanded(!isExpanded)}
>
  <Text>Show details</Text>
</TouchableOpacity>

// Selected state
<TouchableOpacity
  accessibilityRole="tab"
  accessibilityState={{
    selected: isSelected,
  }}
>
  <Text>Tab 1</Text>
</TouchableOpacity>

// Busy state
<View
  accessibilityRole="progressbar"
  accessibilityState={{
    busy: isLoading,
  }}
>
  <ActivityIndicator />
</View>
```

## Accessibility Value

```typescript
// Progress bar
<View
  accessibilityRole="progressbar"
  accessibilityValue={{
    min: 0,
    max: 100,
    now: progress,
    text: `${progress}% complete`,
  }}
>
  <View style={{ width: `${progress}%`, height: 4, backgroundColor: 'blue' }} />
</View>

// Slider
<Slider
  accessibilityRole="adjustable"
  accessibilityValue={{
    min: 0,
    max: 100,
    now: volume,
    text: `Volume ${volume}%`,
  }}
  accessibilityLabel="Volume control"
  value={volume}
  onValueChange={setVolume}
/>
```

## Live Regions

```typescript
// Announce changes to screen readers
<View
  accessibilityLiveRegion="polite" // or "assertive"
  accessibilityRole="alert"
>
  <Text>{statusMessage}</Text>
</View>

// Toast/notification
function Toast({ message, visible }: ToastProps): React.ReactElement | null {
  if (!visible) return null;

  return (
    <View
      accessibilityRole="alert"
      accessibilityLiveRegion="assertive"
      className="bg-black p-4 rounded-lg"
    >
      <Text className="text-white">{message}</Text>
    </View>
  );
}
```

## Grouping Elements

```typescript
// Group related elements
<View
  accessible={true}
  accessibilityLabel="Product: iPhone 15 Pro, Price: $999"
>
  <Text>iPhone 15 Pro</Text>
  <Text>$999</Text>
</View>

// Prevent grouping
<View accessible={false}>
  <TouchableOpacity accessibilityLabel="Edit">
    <Icon name="edit" />
  </TouchableOpacity>
  <TouchableOpacity accessibilityLabel="Delete">
    <Icon name="delete" />
  </TouchableOpacity>
</View>
```

## Focus Management

```typescript
import { useRef } from 'react';
import { AccessibilityInfo, findNodeHandle } from 'react-native';

function FocusExample(): React.ReactElement {
  const headerRef = useRef<Text>(null);

  const focusOnHeader = () => {
    const node = findNodeHandle(headerRef.current);
    if (node) {
      AccessibilityInfo.setAccessibilityFocus(node);
    }
  };

  return (
    <View>
      <Text ref={headerRef} accessibilityRole="header">
        Welcome
      </Text>
      <TouchableOpacity onPress={focusOnHeader}>
        <Text>Focus header</Text>
      </TouchableOpacity>
    </View>
  );
}
```

## Screen Reader Detection

```typescript
import { useEffect, useState } from 'react';
import { AccessibilityInfo } from 'react-native';

function useScreenReader() {
  const [isEnabled, setIsEnabled] = useState(false);

  useEffect(() => {
    AccessibilityInfo.isScreenReaderEnabled().then(setIsEnabled);

    const subscription = AccessibilityInfo.addEventListener(
      'screenReaderChanged',
      setIsEnabled
    );

    return () => subscription.remove();
  }, []);

  return isEnabled;
}

// Usage
function MyComponent(): React.ReactElement {
  const isScreenReaderEnabled = useScreenReader();

  return (
    <View>
      {isScreenReaderEnabled ? (
        <Text>Detailed description for screen reader users</Text>
      ) : (
        <Icon name="info" />
      )}
    </View>
  );
}
```

## Reduce Motion

```typescript
import { useEffect, useState } from 'react';
import { AccessibilityInfo } from 'react-native';

function useReduceMotion() {
  const [reduceMotion, setReduceMotion] = useState(false);

  useEffect(() => {
    AccessibilityInfo.isReduceMotionEnabled().then(setReduceMotion);

    const subscription = AccessibilityInfo.addEventListener(
      'reduceMotionChanged',
      setReduceMotion
    );

    return () => subscription.remove();
  }, []);

  return reduceMotion;
}

// Usage with animations
function AnimatedComponent(): React.ReactElement {
  const reduceMotion = useReduceMotion();

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [
      {
        scale: reduceMotion
          ? 1
          : withSpring(scale.value),
      },
    ],
  }));

  return <Animated.View style={animatedStyle} />;
}
```

## Accessible Navigation

```typescript
// Tab bar with proper accessibility
function TabBar({ tabs, activeTab, onTabPress }) {
  return (
    <View accessibilityRole="tablist">
      {tabs.map((tab, index) => (
        <TouchableOpacity
          key={tab.id}
          accessibilityRole="tab"
          accessibilityState={{ selected: activeTab === index }}
          accessibilityLabel={`${tab.label}, tab ${index + 1} of ${tabs.length}`}
          onPress={() => onTabPress(index)}
        >
          <Text>{tab.label}</Text>
        </TouchableOpacity>
      ))}
    </View>
  );
}
```

## Accessible Lists

```typescript
function AccessibleList({ items }) {
  return (
    <FlashList
      data={items}
      renderItem={({ item, index }) => (
        <View
          accessible={true}
          accessibilityLabel={`Item ${index + 1} of ${items.length}: ${item.title}`}
          accessibilityHint="Double tap to view details"
        >
          <Text>{item.title}</Text>
        </View>
      )}
      accessibilityRole="list"
    />
  );
}
```

## Testing Accessibility

```typescript
import { render, screen } from '@testing-library/react-native';

describe('Accessibility', () => {
  it('has correct accessibility role', () => {
    render(<SubmitButton />);
    expect(screen.getByRole('button')).toBeOnTheScreen();
  });

  it('has accessibility label', () => {
    render(<IconButton icon="heart" label="Add to favorites" />);
    expect(screen.getByLabelText('Add to favorites')).toBeOnTheScreen();
  });

  it('announces state changes', () => {
    render(<Toggle checked={true} label="Notifications" />);
    expect(screen.getByRole('switch')).toHaveAccessibilityState({
      checked: true,
    });
  });
});
```

## Checklist

- [ ] All interactive elements have `accessibilityRole`
- [ ] All images have `accessibilityLabel` or are hidden
- [ ] Form inputs have labels and error messages
- [ ] Touch targets are at least 44x44 points
- [ ] Color is not the only way to convey information
- [ ] Text has sufficient contrast ratio (4.5:1)
- [ ] Animations respect reduce motion setting
- [ ] Focus order is logical
- [ ] Dynamic content uses live regions
- [ ] Screen reader testing on iOS and Android

## Notes

- Test with VoiceOver (iOS) and TalkBack (Android)
- Use Accessibility Inspector in Xcode
- Enable accessibility testing in development
- Consider users with motor impairments
- Provide alternatives for gestures
