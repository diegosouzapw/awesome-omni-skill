---
name: vue-writer
description: Skill for creating and editing Vue.js components following Prowi conventions. Use when writing Vue files, creating components, or refactoring frontend code. Enforces modern patterns like defineModel(), TypeScript, and Composition API.
---

# Vue Component Writer Skill

You are an expert Vue.js developer for the Prowi application. Your role is to create modern, maintainable Vue components that follow established conventions and avoid legacy patterns.

## 🚨 CRITICAL: Use defineModel(), NOT modelValue!

**This is the #1 mistake to avoid!** Many existing components use the old pattern, but ALL new code must use `defineModel()`.

### ❌ WRONG - Old Pattern (Do NOT use)

```vue
<script setup>
// BAD - Using modelValue prop + manual emit
const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  }
});

const emit = defineEmits(['update:modelValue']);

// Manual handling with watch or methods
const updateValue = (value) => {
  emit('update:modelValue', value);
};
</script>
```

### ✅ CORRECT - Modern Pattern (Always use)

```vue
<script setup lang="ts">
// GOOD - Using defineModel()
const model = defineModel<string>({ default: '' });

// That's it! No manual emit, no watch, just use it directly
</script>

<template>
  <input v-model="model" />
</template>
```

**Why defineModel() is better:**
- Cleaner, less boilerplate code
- No manual emit or watch logic needed
- Type-safe with TypeScript
- Automatic two-way binding
- Official Vue 3.4+ pattern

## Component Structure

**STRICT ORDER** - Always follow this structure:

```vue
<script setup lang="ts">
// 1. Imports
import { ref, computed } from 'vue';
import Button from '@/Components/App/Forms/Button.vue';

// 2. Props
const props = defineProps({
  // props here
});

// 3. Models (for two-way binding)
const model = defineModel<string>();

// 4. Emits
const emit = defineEmits(['save', 'cancel']);

// 5. Reactive state
const loading = ref(false);

// 6. Computed properties
const isValid = computed(() => model.value?.length > 0);

// 7. Methods
function handleSubmit() {
  emit('save', model.value);
}
</script>

<template>
  <!-- HTML here -->
</template>

<style scoped>
/* Scoped styles here */
</style>
```

**Key Requirements:**
- ✅ `<script setup lang="ts">` for TypeScript (REQUIRED for new components)
- ✅ Script at top, template middle, style at bottom
- ✅ Use `<style scoped>` to prevent style leakage
- ✅ Follow the 7-step structure order inside script

## Two-Way Binding with defineModel()

### Basic Usage

```vue
<script setup lang="ts">
// Simple model
const model = defineModel<string>();

// With default value
const checked = defineModel<boolean>({ default: false });

// With required
const required = defineModel<number>({ required: true });
</script>

<template>
  <input v-model="model" />
  <input type="checkbox" v-model="checked" />
</template>
```

### Multiple Models

```vue
<script setup lang="ts">
// Named models for multiple v-model bindings
const firstName = defineModel<string>('firstName');
const lastName = defineModel<string>('lastName');
</script>

<template>
  <input v-model="firstName" placeholder="First name" />
  <input v-model="lastName" placeholder="Last name" />
</template>
```

### Usage in Parent Component

```vue
<template>
  <!-- Single model -->
  <MyInput v-model="userName" />

  <!-- Multiple models -->
  <MyForm
    v-model:firstName="user.firstName"
    v-model:lastName="user.lastName"
  />
</template>
```

## Props Definition

**Always use full object syntax** with type, required, and default:

```vue
<script setup lang="ts">
import type { PropType } from 'vue';
import type { User } from '@/types/generated';

const props = defineProps({
  // Simple types
  title: {
    type: String,
    required: true,
  },

  // With default
  isActive: {
    type: Boolean,
    default: false,
  },

  // Multiple types
  value: {
    type: [String, Number],
    default: '',
  },

  // Complex types with PropType
  user: {
    type: Object as PropType<User>,
    required: true,
  },

  // Array of specific type
  items: {
    type: Array as PropType<User[]>,
    default: () => [],
  },

  // Object with specific shape
  config: {
    type: Object as PropType<{ enabled: boolean; count: number }>,
    default: () => ({ enabled: true, count: 0 }),
  },
});
</script>
```

**Type Locations:**
- Generated types: `resources/js/types/generated.d.ts` (models, enums, data classes)
- Enums: `resources/js/types/enums.ts`
- Routes: `resources/js/types/ziggy.ts`

## TypeScript Requirements

### New Components - MUST Use TypeScript

```vue
<script setup lang="ts">
import { ref, computed } from 'vue';
import type { PropType } from 'vue';
import type { User, CustomerUser } from '@/types/generated';

const props = defineProps({
  user: {
    type: Object as PropType<User>,
    required: true,
  },
});

const model = defineModel<string | null>({ default: null });
const count = ref<number>(0);
const users = ref<CustomerUser[]>([]);

const formattedName = computed<string>(() => {
  return props.user.name.toUpperCase();
});

function updateCount(value: number): void {
  count.value = value;
}
</script>
```

### Type Imports

```typescript
// Vue types
import type { PropType, ComputedRef, Ref } from 'vue';

// Generated backend types
import type {
  User,
  Customer,
  CustomerUser,
  // ... other models
} from '@/types/generated';

// Enums
import {
  UserStatusEnum,
  PaymentStatusEnum
} from '@/types/enums';
```

## Inertia.js (v0.11.1) - IMPORTANT!

**This project uses OLD Inertia version** - use these imports:

```typescript
// ✅ CORRECT - Old imports
import { Inertia } from '@inertiajs/inertia';
import { usePage } from '@inertiajs/inertia-vue3';
import { useForm } from '@inertiajs/inertia-vue3';
import { Link } from '@inertiajs/inertia-vue3';

// ❌ WRONG - New imports (don't exist in v0.11)
import { router } from '@inertiajs/vue3'; // DON'T USE
import { usePage } from '@inertiajs/vue3'; // DON'T USE
```

### Using Props vs usePage()

```vue
<script setup lang="ts">
import { usePage } from '@inertiajs/inertia-vue3';

// ✅ GOOD - Use defineProps for component-specific data
const props = defineProps({
  user: {
    type: Object,
    required: true,
  },
  items: {
    type: Array,
    required: true,
  },
});

// ✅ GOOD - Use usePage() ONLY for global Inertia properties
const page = usePage();
const currentUser = page.props.value.auth.user;
const flashMessage = page.props.value.flash.message;

// ❌ BAD - Don't use usePage() for component props
// const { user, items } = usePage().props.value; // WRONG!
</script>
```

### Form Handling

```vue
<script setup lang="ts">
import { useForm } from '@inertiajs/inertia-vue3';

const props = defineProps({
  user: {
    type: Object,
    required: true,
  },
});

const form = useForm({
  name: props.user.name,
  email: props.user.email,
});

function submit() {
  form.put(route('users.update', props.user.id), {
    onSuccess: () => {
      // Handle success
    },
    onError: () => {
      // Handle errors
    },
  });
}
</script>

<template>
  <form @submit.prevent="submit">
    <input v-model="form.name" />
    <div v-if="form.errors.name" class="text-red-500">
      {{ form.errors.name }}
    </div>

    <button type="submit" :disabled="form.processing">
      Save
    </button>
  </form>
</template>
```

## Styling with Tailwind CSS

### Use Scoped Styles

```vue
<template>
  <div class="container">
    <button class="btn-primary">Click me</button>
  </div>
</template>

<style scoped>
/* Scoped to this component only */
.container {
  @apply max-w-4xl mx-auto p-4;
}

.btn-primary {
  @apply bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded;
}

/* Deep selector for child components */
:deep(.child-class) {
  @apply text-gray-700;
}
</style>
```

### Group Related Classes

```vue
<template>
  <!-- ✅ GOOD - Grouped by purpose -->
  <div class="flex items-center justify-between gap-4 p-4 bg-white rounded-lg shadow-md">
    <span class="text-lg font-bold text-gray-900">Title</span>
    <button class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded">
      Action
    </button>
  </div>

  <!-- ❌ BAD - Random order -->
  <div class="p-4 rounded-lg flex bg-white items-center shadow-md justify-between gap-4">
    <span class="font-bold text-gray-900 text-lg">Title</span>
  </div>
</template>
```

## Component Communication

### Props Down, Events Up

```vue
<!-- Parent.vue -->
<script setup lang="ts">
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const selectedItem = ref<string | null>(null);

function handleSelection(item: string) {
  selectedItem.value = item;
}
</script>

<template>
  <ChildComponent
    :items="['A', 'B', 'C']"
    @itemSelected="handleSelection"
  />
</template>

<!-- ChildComponent.vue -->
<script setup lang="ts">
const props = defineProps({
  items: {
    type: Array as PropType<string[]>,
    required: true,
  },
});

const emit = defineEmits<{
  itemSelected: [item: string]
}>();

function selectItem(item: string) {
  emit('itemSelected', item);
}
</script>

<template>
  <ul>
    <li
      v-for="item in items"
      :key="item"
      @click="selectItem(item)"
    >
      {{ item }}
    </li>
  </ul>
</template>
```

## Early Exit Pattern

Reduce nesting with early returns:

```vue
<script setup lang="ts">
// ✅ GOOD - Early exit
function processUser(user: User | null) {
  if (!user) {
    return;
  }

  if (!user.isActive) {
    return;
  }

  // Main logic here
  updateUserData(user);
}

// ❌ BAD - Deep nesting
function processUser(user: User | null) {
  if (user) {
    if (user.isActive) {
      // Main logic deeply nested
      updateUserData(user);
    }
  }
}
</script>
```

## Legacy Components (Options API)

### When to Migrate to Composition API

**Migrate when:**
- Making significant changes (50%+ of component logic)
- Adding complex reactive state
- Component will need future modifications
- Refactoring for maintainability

**Keep Options API when:**
- Making minor tweaks (< 20% of component)
- Time-constrained quick fixes
- Component is stable and rarely changed
- Would require extensive testing to migrate

### Example Migration

```vue
<!-- BEFORE - Options API -->
<script>
export default {
  props: ['modelValue', 'placeholder'],
  emits: ['update:modelValue'],
  data() {
    return {
      internalValue: this.modelValue
    };
  },
  watch: {
    modelValue(newValue) {
      this.internalValue = newValue;
    }
  },
  methods: {
    updateValue(value) {
      this.internalValue = value;
      this.$emit('update:modelValue', value);
    }
  }
};
</script>

<!-- AFTER - Composition API with defineModel -->
<script setup lang="ts">
const props = defineProps({
  placeholder: {
    type: String,
    default: '',
  },
});

const model = defineModel<string>({ default: '' });
</script>

<template>
  <input
    v-model="model"
    :placeholder="placeholder"
  />
</template>
```

## Complete Example: Modern Component

```vue
<script setup lang="ts">
import { ref, computed } from 'vue';
import type { PropType } from 'vue';
import type { User } from '@/types/generated';
import { UserStatusEnum } from '@/types/enums';
import { useForm } from '@inertiajs/inertia-vue3';
import Button from '@/Components/App/Forms/Button.vue';
import Input from '@/Components/App/Forms/Input.vue';

// Props
const props = defineProps({
  user: {
    type: Object as PropType<User>,
    required: true,
  },
  isEditable: {
    type: Boolean,
    default: false,
  },
});

// Models
const isModalOpen = defineModel<boolean>('isModalOpen', { default: false });

// Emits
const emit = defineEmits<{
  saved: [user: User],
  cancelled: []
}>();

// State
const form = useForm({
  name: props.user.name,
  email: props.user.email,
  status: props.user.status,
});

// Computed
const isActive = computed(() => {
  return props.user.status === UserStatusEnum.ACTIVE;
});

const canSubmit = computed(() => {
  return form.name.length > 0 &&
         form.email.length > 0 &&
         !form.processing;
});

// Methods
function handleSubmit(): void {
  form.put(route('users.update', props.user.id), {
    onSuccess: () => {
      isModalOpen.value = false;
      emit('saved', form.data());
    },
    onError: () => {
      console.error('Failed to save user');
    },
  });
}

function handleCancel(): void {
  form.reset();
  isModalOpen.value = false;
  emit('cancelled');
}
</script>

<template>
  <div class="max-w-2xl mx-auto p-6">
    <!-- Header -->
    <div class="flex items-center justify-between mb-6">
      <h2 class="text-2xl font-bold text-gray-900">
        Edit User
      </h2>
      <span
        class="px-3 py-1 text-sm font-medium rounded-full"
        :class="{
          'bg-green-100 text-green-800': isActive,
          'bg-gray-100 text-gray-800': !isActive,
        }"
      >
        {{ isActive ? 'Active' : 'Inactive' }}
      </span>
    </div>

    <!-- Form -->
    <form @submit.prevent="handleSubmit" class="space-y-4">
      <!-- Name Input -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Name
        </label>
        <Input
          v-model="form.name"
          type="text"
          :disabled="!isEditable"
          :error="form.errors.name"
        />
        <div v-if="form.errors.name" class="mt-1 text-sm text-red-600">
          {{ form.errors.name }}
        </div>
      </div>

      <!-- Email Input -->
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-1">
          Email
        </label>
        <Input
          v-model="form.email"
          type="email"
          :disabled="!isEditable"
          :error="form.errors.email"
        />
        <div v-if="form.errors.email" class="mt-1 text-sm text-red-600">
          {{ form.errors.email }}
        </div>
      </div>

      <!-- Actions -->
      <div class="flex items-center justify-end gap-3 pt-4">
        <Button
          type="button"
          variant="secondary"
          @click="handleCancel"
        >
          Cancel
        </Button>
        <Button
          type="submit"
          variant="primary"
          :disabled="!canSubmit"
          :loading="form.processing"
        >
          Save Changes
        </Button>
      </div>
    </form>
  </div>
</template>

<style scoped>
/* Component-specific styles if needed */
</style>
```

## Anti-Patterns Summary

### ❌ Don't Do This

```vue
<!-- 1. Using modelValue prop instead of defineModel -->
<script setup>
const props = defineProps(['modelValue']);
const emit = defineEmits(['update:modelValue']);
</script>

<!-- 2. Missing TypeScript -->
<script setup>  <!-- No lang="ts" -->
const props = defineProps({
  user: Object,  // No PropType
});
</script>

<!-- 3. Wrong Inertia imports -->
<script setup lang="ts">
import { router } from '@inertiajs/vue3'; // Wrong version!
</script>

<!-- 4. Using usePage() for component props -->
<script setup lang="ts">
const { user, items } = usePage().props.value; // Wrong!
</script>

<!-- 5. No scoped styles -->
<style>  <!-- Not scoped! -->
.my-class { }
</style>
```

### ✅ Do This Instead

```vue
<!-- 1. Use defineModel -->
<script setup lang="ts">
const model = defineModel<string>();
</script>

<!-- 2. Include TypeScript -->
<script setup lang="ts">
import type { PropType } from 'vue';
const props = defineProps({
  user: {
    type: Object as PropType<User>,
    required: true,
  },
});
</script>

<!-- 3. Correct Inertia imports -->
<script setup lang="ts">
import { Inertia } from '@inertiajs/inertia';
import { usePage } from '@inertiajs/inertia-vue3';
</script>

<!-- 4. Use defineProps for component data -->
<script setup lang="ts">
const props = defineProps({
  user: Object as PropType<User>,
  items: Array as PropType<Item[]>,
});
</script>

<!-- 5. Use scoped styles -->
<style scoped>
.my-class { }
</style>
```

## Checklist for New Components

Before considering a component complete, verify:

- ✅ Uses `<script setup lang="ts">` with TypeScript
- ✅ Uses `defineModel()` for two-way binding (NOT modelValue prop)
- ✅ Follows script → template → style order
- ✅ Props use full object syntax with PropType for complex types
- ✅ Uses correct Inertia v0 imports
- ✅ Has `<style scoped>` if styles are needed
- ✅ Groups related Tailwind classes logically
- ✅ Uses early exit patterns to reduce nesting
- ✅ Emits are properly typed (TypeScript)
- ✅ Component communicates via props down, events up
- ✅ Uses generated types from `resources/js/types/generated.d.ts`

## Final Reminder

**The #1 mistake to avoid:** Using `modelValue` prop pattern instead of `defineModel()`.

If you see this in new code:
```vue
const props = defineProps(['modelValue']);
const emit = defineEmits(['update:modelValue']);
```

**STOP** and use this instead:
```vue
const model = defineModel<YourType>();
```

Your goal is to create clean, type-safe, modern Vue components that will be easy for future developers to maintain and extend.
