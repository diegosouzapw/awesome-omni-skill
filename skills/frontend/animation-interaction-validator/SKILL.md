---
name: animation-interaction-validator
description: Ensures engaging user experience through validation of animations, transitions, micro-interactions, and feedback states, preventing flat/static interfaces that lack polish and engagement. Works with Tanstack Start (React) + shadcn/ui components.
triggers: ["interactive element creation", "event handler addition", "state changes", "async actions", "form submissions"]
note: "Code examples use React/TSX with shadcn/ui components (Button, Card, Input). Adapt patterns to your component library."
---

# Animation Interaction Validator SKILL

## Activation Patterns

This SKILL automatically activates when:
- Interactive elements are created (buttons, links, forms, inputs)
- Click, hover, or focus event handlers are added
- Component state changes (loading, success, error)
- Async operations are initiated (API calls, form submissions)
- Navigation or routing transitions occur
- Modal/dialog components are opened/closed
- Lists or data are updated dynamically

## Expertise Provided

### Animation & Interaction Validation
- **Transition Detection**: Ensures smooth state changes with CSS transitions
- **Hover State Validation**: Checks for hover feedback on interactive elements
- **Loading State Validation**: Ensures async actions have visual feedback
- **Micro-interaction Analysis**: Validates small, delightful animations
- **Focus State Validation**: Ensures keyboard navigation has visual feedback
- **Animation Performance**: Checks for performant animation patterns

### Specific Checks Performed

#### ❌ Critical Issues (Missing Feedback)
```tsx
// These patterns trigger alerts:

// No hover state
<Button onClick={submit}>Submit</Button>

// No loading state during async action
<Button onClick={async () => await submitForm()}>Save</Button>

// Jarring state change (no transition)
{showContent && <div>Content</div>}

// No focus state
<a href="/page" className="text-blue-500">Link</a>

// Form without feedback
<form onSubmit={handleSubmit}>
  <Input value={value} onChange={setValue} />
  <button type="submit">Submit</button>
</form>
```

#### ✅ Correct Interactive Patterns
```tsx
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Send } from "lucide-react"
import { cn } from "@/lib/utils"

// These patterns are validated as correct:

// Hover state with smooth transition
<Button
  className="transition-all duration-300 hover:scale-105 hover:shadow-xl active:scale-95"
  onClick={submit}
>
  Submit
</Button>

// Loading state with visual feedback
<Button
  disabled={isSubmitting}
  className="transition-all duration-200 group"
  onClick={handleSubmit}
>
  <span className="flex items-center gap-2">
    {!isSubmitting && (
      <Send className="h-4 w-4 transition-transform duration-300 group-hover:translate-x-1" />
    )}
    {isSubmitting ? 'Submitting...' : 'Submit'}
  </span>
</Button>

// Smooth state transition (using framer-motion or CSS)
<div
  className={cn(
    "transition-all duration-300 ease-out",
    showContent ? "opacity-100 translate-y-0" : "opacity-0 translate-y-4"
  )}
>
  {showContent && <div>Content</div>}
</div>

// Focus state with ring
<a
  href="/page"
  className="text-blue-500 transition-colors duration-200 hover:text-blue-700 focus:outline-none focus-visible:ring-2 focus-visible:ring-blue-500 focus-visible:ring-offset-2"
>
  Link
</a>

<!-- Form with success/error feedback -->
<form onSubmit={(e) => { e.preventDefault(); handleSubmit" className="space-y-4">
  <Input
    value="value"
    error={errors.value"
    className="transition-all duration-200"
  />

  <Button
    type="submit"
    loading={isSubmitting"
    disabled={isSubmitting"
    className="transition-all duration-300 hover:scale-105"
  >
    Submit
  </Button>

  <!-- Success message with animation -->
  <Transition name="fade">
    <Alert
      if="showSuccess"
      color="green"
      icon="i-heroicons-check-circle"
      title="Success!"
      className="animate-in slide-in-from-top"
    />
  </Transition>
</form>
```

## Integration Points

### Complementary to Existing Components
- **frontend-design-specialist agent**: Provides design direction, SKILL validates implementation
- **component-aesthetic-checker**: Validates component customization, SKILL validates interactions
- **shadcn-ui-design-validator**: Catches generic patterns, SKILL ensures engagement
- **accessibility-guardian agent**: Validates a11y, SKILL validates visual feedback

### Escalation Triggers
- Complex animation sequences → `frontend-design-specialist` agent
- Component interaction patterns → `tanstack-ui-architect` agent
- Performance concerns → `edge-performance-oracle` agent
- Accessibility issues → `accessibility-guardian` agent

## Validation Rules

### P1 - Critical (Missing User Feedback)
- **No Hover States**: Buttons/links without hover effects
- **No Loading States**: Async actions without loading indicators
- **Jarring State Changes**: Content appearing/disappearing without transitions
- **No Focus States**: Interactive elements without keyboard focus indicators
- **Silent Errors**: Form errors without visual feedback

### P2 - Important (Enhanced Engagement)
- **No Micro-interactions**: Icons/elements without subtle animations
- **Static Navigation**: Page transitions without animations
- **Abrupt Modals**: Dialogs opening without enter/exit transitions
- **Instant Updates**: List changes without transition animations
- **No Disabled States**: Buttons during processing without visual change

### P3 - Polish (Delightful UX)
- **Limited Animation Variety**: Using only scale/opacity (no rotate, translate)
- **Generic Durations**: Not tuning animation speed for context
- **No Stagger**: List items appearing simultaneously (no stagger effect)
- **Missing Success States**: Completed actions without celebration animation
- **No Hover Anticipation**: No visual hint before interaction is possible

## Remediation Examples

### Fixing Missing Hover States
```tsx
<!-- ❌ Critical: No hover feedback -->
  <Button onClick="handleClick">
    Click me
  </Button>

<!-- ✅ Correct: Multi-dimensional hover effects -->
  <Button
    className="
      transition-all duration-300 ease-out
      hover:scale-105 hover:shadow-xl hover:-rotate-1
      active:scale-95 active:rotate-0
      focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-primary-500
    "
    onClick="handleClick"
  >
    <span className="inline-flex items-center gap-2">
      Click me
      <Icon
        name="i-heroicons-arrow-right"
        className="transition-transform duration-300 group-hover:translate-x-1"
      />
    </span>
  </Button>
```

### Fixing Missing Loading States
```tsx
<!-- ❌ Critical: No loading feedback during async action -->
const submitForm = async () => {
  await api.submit(formData);
};

  <Button onClick="submitForm">
    Submit
  </Button>

<!-- ✅ Correct: Complete loading state with animations -->
const isSubmitting = ref(false);
const showSuccess = ref(false);

const submitForm = async () => {
  isSubmitting.value = true;
  try {
    await api.submit(formData);
    showSuccess.value = true;
    setTimeout(() => showSuccess.value = false, 3000);
  } catch (error) {
    // Error handling
  } finally {
    isSubmitting.value = false;
  }
};

  <div className="space-y-4">
    <Button
      loading={isSubmitting"
      disabled={isSubmitting"
      className="
        transition-all duration-300
        hover:scale-105 hover:shadow-xl
        disabled:opacity-50 disabled:cursor-not-allowed
      "
      onClick="submitForm"
    >
      <span className="flex items-center gap-2">
        <Icon
          if="!isSubmitting"
          name="i-heroicons-paper-airplane"
          className="transition-all duration-300 group-hover:translate-x-1 group-hover:-translate-y-1"
        />
        { isSubmitting ? 'Submitting...' : 'Submit'}
      </span>
    </Button>

    <!-- Success feedback with animation -->
    <Transition
      enter-active-className="transition-all duration-500 ease-out"
      enter-from-className="opacity-0 scale-50"
      enter-to-className="opacity-100 scale-100"
      leave-active-className="transition-all duration-300 ease-in"
      leave-from-className="opacity-100 scale-100"
      leave-to-className="opacity-0 scale-50"
    >
      <Alert
        if="showSuccess"
        color="green"
        icon="i-heroicons-check-circle"
        title="Success!"
        description="Your form has been submitted."
      />
    </Transition>
  </div>
```

### Fixing Jarring State Changes
```tsx
<!-- ❌ Critical: Content appears/disappears abruptly -->
  <div>
    <Button onClick="showContent = !showContent">
      Toggle
    </Button>

    <div if="showContent">
      <p>This content appears instantly (jarring)</p>
    </div>
  </div>

<!-- ✅ Correct: Smooth transitions -->
  <div className="space-y-4">
    <Button
      className="transition-all duration-300 hover:scale-105"
      onClick="showContent = !showContent"
    >
      { showContent ? 'Hide' : 'Show'} Content
    </Button>

    <Transition
      enter-active-className="transition-all duration-300 ease-out"
      enter-from-className="opacity-0 translate-y-4 scale-95"
      enter-to-className="opacity-100 translate-y-0 scale-100"
      leave-active-className="transition-all duration-200 ease-in"
      leave-from-className="opacity-100 translate-y-0 scale-100"
      leave-to-className="opacity-0 translate-y-4 scale-95"
    >
      <div if="showContent" className="p-6 bg-gray-50 dark:bg-gray-800 rounded-lg">
        <p>This content transitions smoothly</p>
      </div>
    </Transition>
  </div>
```

### Fixing Missing Focus States
```tsx
<!-- ❌ Critical: No visible focus state -->
  <nav>
    <a href="/" className="text-gray-700">Home</a>
    <a href="/about" className="text-gray-700">About</a>
    <a href="/contact" className="text-gray-700">Contact</a>
  </nav>

<!-- ✅ Correct: Clear focus states for keyboard navigation -->
  <nav className="flex gap-4">
    <a
      href="/"
      className="
        text-gray-700 dark:text-gray-300
        transition-all duration-200
        hover:text-primary-600 hover:translate-y-[-2px]
        focus:outline-none
        focus-visible:ring-2 focus-visible:ring-primary-500 focus-visible:ring-offset-2
        rounded px-3 py-2
      "
    >
      Home
    </a>
    <a
      href="/about"
      className="
        text-gray-700 dark:text-gray-300
        transition-all duration-200
        hover:text-primary-600 hover:translate-y-[-2px]
        focus:outline-none
        focus-visible:ring-2 focus-visible:ring-primary-500 focus-visible:ring-offset-2
        rounded px-3 py-2
      "
    >
      About
    </a>
    <a
      href="/contact"
      className="
        text-gray-700 dark:text-gray-300
        transition-all duration-200
        hover:text-primary-600 hover:translate-y-[-2px]
        focus:outline-none
        focus-visible:ring-2 focus-visible:ring-primary-500 focus-visible:ring-offset-2
        rounded px-3 py-2
      "
    >
      Contact
    </a>
  </nav>
```

### Adding Micro-interactions
```tsx
<!-- ❌ P2: Static icons without micro-interactions -->
  <Button icon="i-heroicons-heart">
    Like
  </Button>

<!-- ✅ Correct: Animated icon micro-interaction -->
const isLiked = ref(false);
const heartScale = ref(1);

const toggleLike = () => {
  isLiked.value = !isLiked.value;

  // Bounce animation
  heartScale.value = 1.3;
  setTimeout(() => heartScale.value = 1, 200);
};

  <Button
    :color="isLiked ? 'red' : 'gray'"
    className="transition-all duration-300 hover:scale-105"
    onClick="toggleLike"
  >
    <span className="inline-flex items-center gap-2">
      <Icon
        :name="isLiked ? 'i-heroicons-heart-solid' : 'i-heroicons-heart'"
        :style="{ transform: `scale(${heartScale})` }"
        :className="[
          'transition-all duration-200',
          isLiked ? 'text-red-500 animate-pulse' : 'text-gray-500'
        ]"
      />
      { isLiked ? 'Liked' : 'Like'}
    </span>
  </Button>
```

## Animation Best Practices

### Performance-First Animations

✅ **Performant Properties** (GPU-accelerated):
- `transform` (translate, scale, rotate)
- `opacity`
- `filter` (backdrop-blur, etc.)

❌ **Avoid Animating** (causes reflow/repaint):
- `width`, `height`
- `top`, `left`, `right`, `bottom`
- `margin`, `padding`
- `border-width`

```tsx
<!-- ❌ P2: Animating width (causes reflow) -->
<div className="transition-all hover:w-64">Content</div>

<!-- ✅ Correct: Using transform (GPU-accelerated) -->
<div className="transition-transform hover:scale-110">Content</div>
```

### Animation Duration Guidelines

- **Fast** (100-200ms): Hover states, small movements
- **Medium** (300-400ms): State changes, content transitions
- **Slow** (500-800ms): Page transitions, major UI changes
- **Very Slow** (1000ms+): Celebration animations, complex sequences

```tsx
<!-- Context-appropriate durations -->
<Button className="transition-all duration-200 hover:scale-105">
  <!-- Fast hover: 200ms -->
</Button>

<Transition
  enter-active-className="transition-all duration-300"
  leave-active-className="transition-all duration-300"
>
  <!-- Content change: 300ms -->
  <div if="show">Content</div>
</Transition>

<div className="animate-in slide-in-from-bottom duration-500">
  <!-- Page load: 500ms -->
  Main content
</div>
```

### Easing Functions

- `ease-out`: Starting animations (entering content)
- `ease-in`: Ending animations (exiting content)
- `ease-in-out`: Bidirectional animations
- `linear`: Loading spinners, continuous animations

```tsx
<!-- Appropriate easing -->
<Transition
  enter-active-className="transition-all duration-300 ease-out"
  leave-active-className="transition-all duration-200 ease-in"
>
  <div if="show">Content</div>
</Transition>
```

## Advanced Interaction Patterns

### Staggered List Animations
```tsx
const items = ref([1, 2, 3, 4, 5]);

  <TransitionGroup
    name="list"
    tag="div"
    className="space-y-2"
  >
    <div
      map((item, index) in items"
      :key="item"
      :style="{ transitionDelay: `${index * 50}ms` }"
      className="
        transition-all duration-300 ease-out
        hover:scale-105 hover:shadow-lg
      "
    >
      Item { item}
    </div>
  </TransitionGroup>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from {
  opacity: 0;
  transform: translateX(-20px);
}

.list-leave-to {
  opacity: 0;
  transform: translateX(20px);
}

.list-move {
  transition: transform 0.3s ease;
}
</style>
```

### Success Celebration Animation
```tsx
const showSuccess = ref(false);

const celebrate = () => {
  showSuccess.value = true;
  // Confetti or celebration animation here
  setTimeout(() => showSuccess.value = false, 3000);
};

  <div>
    <Button
      onClick="celebrate"
      className="transition-all duration-300 hover:scale-110 hover:rotate-3"
    >
      Complete Task
    </Button>

    <Transition
      enter-active-className="transition-all duration-500 ease-out"
      enter-from-className="opacity-0 scale-0 rotate-180"
      enter-to-className="opacity-100 scale-100 rotate-0"
    >
      <div
        if="showSuccess"
        className="fixed inset-0 flex items-center justify-center bg-black/20 backdrop-blur-sm"
      >
        <div className="bg-white dark:bg-gray-800 p-8 rounded-2xl shadow-2xl">
          <Icon
            name="i-heroicons-check-circle"
            className="w-16 h-16 text-green-500 animate-bounce"
          />
          <p className="mt-4 text-xl font-heading">Success!</p>
        </div>
      </div>
    </Transition>
  </div>
```

### Loading Skeleton with Pulse
```tsx
  <div if="loading" className="space-y-4">
    <div className="animate-pulse">
      <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-3/4"></div>
      <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-1/2 mt-2"></div>
      <div className="h-32 bg-gray-200 dark:bg-gray-700 rounded mt-4"></div>
    </div>
  </div>

  <Transition
    enter-active-className="transition-all duration-500 ease-out"
    enter-from-className="opacity-0 translate-y-4"
    enter-to-className="opacity-100 translate-y-0"
  >
    <div if="!loading">
      <!-- Actual content -->
    </div>
  </Transition>
```

## MCP Server Integration

While this SKILL doesn't directly use MCP servers, it complements MCP-enhanced agents:

- **shadcn/ui MCP**: Validates that suggested animations work with shadcn/ui components
- **Cloudflare MCP**: Ensures animations don't bloat bundle size (performance check)

## Benefits

### Immediate Impact
- **Prevents Flat UI**: Ensures engaging, polished interactions
- **Improves Perceived Performance**: Loading states make waits feel shorter
- **Better Accessibility**: Focus states improve keyboard navigation
- **Professional Polish**: Micro-interactions signal quality

### Long-term Value
- **Higher User Engagement**: Delightful animations encourage interaction
- **Reduced Bounce Rate**: Polished UI keeps users engaged
- **Better Brand Perception**: Professional animations signal quality
- **Consistent UX**: All interactions follow same animation patterns

## Usage Examples

### During Button Creation
```tsx
// Developer adds: <Button onClick="submit">Submit</Button>
// SKILL immediately activates: "⚠️ P1: Button lacks hover state. Add transition utilities: class='transition-all duration-300 hover:scale-105'"
```

### During Async Action
```tsx
// Developer creates: const submitForm = async () => { await api.call(); }
// SKILL immediately activates: "⚠️ P1: Async action without loading state. Add :loading and :disabled props to button."
```

### During State Toggle
```tsx
// Developer adds: <div if="show">Content</div>
// SKILL immediately activates: "⚠️ P1: Content appears abruptly. Wrap with <Transition> for smooth state changes."
```

### Before Deployment
```tsx
// SKILL runs comprehensive check: "✅ Animation validation passed. 45 interactive elements with hover states, 12 async actions with loading feedback, 8 smooth transitions detected."
```

This SKILL ensures every interactive element provides engaging visual feedback, preventing the flat, static appearance that makes interfaces feel unpolished and reduces user engagement.
