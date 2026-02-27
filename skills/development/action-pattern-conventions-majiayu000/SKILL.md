---
name: Action Pattern Conventions
description: This skill should be used when the user asks about "Laravel action pattern", "action class naming", "how to structure actions", "React component patterns", "Node.js service structure", "framework-specific conventions", or discusses creating reusable, focused classes following action pattern conventions in Laravel, Symfony, React, Vue, or Node.js projects.
version: 0.1.0
---

# Action Pattern Conventions

## Purpose

This skill provides language and framework-specific guidance for implementing the action pattern across different technology stacks. It explains conventions for creating focused, reusable action classes, components, services, and modules that encapsulate specific business operations.

## When to Use

Use this skill when refactoring code into actions, understanding how to structure operations for a specific framework, or ensuring extracted code follows project conventions. It covers Laravel, Symfony, React, Vue, and Node.js.

## Universal Action Pattern

The action pattern applies universally across frameworks:

**Core concept:** One action = one operation with a clear entry point

**Characteristics:**
- **Single responsibility**: One reason to exist
- **Clear interface**: Single entry method (`handle()`, `execute()`, call method)
- **Dependency injection**: Dependencies injected, not global
- **Reusability**: Can be called from multiple contexts (controllers, jobs, CLI, API, webhooks)
- **Testability**: Testable in isolation
- **Naming**: Describes the operation clearly

**Pattern:**
```
public function handle($input) {
    // 1. Validate/prepare input
    // 2. Execute operation (business logic)
    // 3. Return result or side effect
}
```

## Laravel Action Pattern

### File Structure

```
app/
├── Actions/
│   ├── Users/
│   │   ├── CreateUserAction.php
│   │   ├── UpdateUserAction.php
│   │   └── DeleteUserAction.php
│   ├── Orders/
│   │   ├── CreateOrderAction.php
│   │   └── ProcessPaymentAction.php
│   └── Notifications/
│       ├── SendWelcomeEmailAction.php
│       └── SendOrderConfirmationAction.php
```

### Basic Action Class

```php
<?php

namespace App\Actions\Users;

final readonly class CreateUserAction {
    public function __construct(private UserRepository $users) {}

    public function handle(array $data): User {
        // Validate (optional, can use Form Request instead)
        $validated = $this->validate($data);

        // Create user
        $user = $this->users->create($validated);

        // Side effects (notifications, etc.)
        // Only if tightly coupled to creation
        // Otherwise use jobs or separate actions

        return $user;
    }

    private function validate(array $data): array {
        // Custom validation if needed
        return $data;
    }
}
```

### Usage in Controllers

```php
class UserController extends Controller {
    public function store(CreateUserRequest $request, CreateUserAction $createUser) {
        // Constructor injection of action
        $user = $createUser->handle($request->validated());

        return response()->json(['user' => $user], 201);
    }
}
```

### Naming Conventions

**Action class names:**
- Operation + "Action" suffix: `CreateUserAction`, `SendEmailAction`
- Verb-noun format: Clear what it does
- Namespace by domain: `Users/`, `Orders/`, `Payments/`

**Method names:**
- `handle()` - Primary method for the action
- Specific methods for complex operations: `validateUser()`, `persistToDatabase()`

**File structure:**
- One action per file
- Directory per domain/entity type
- `app/Actions/` root directory

### Advanced Patterns

**Action with Transaction:**
```php
final readonly class CreateOrderAction {
    public function __construct(private OrderRepository $orders) {}

    public function handle(array $data): Order {
        return DB::transaction(function () use ($data) {
            $order = $this->orders->create($data);
            $this->orders->attachItems($order->id, $data['items']);
            return $order;
        });
    }
}
```

**Action with Events:**
```php
final readonly class ProcessPaymentAction {
    public function __construct(private PaymentGateway $gateway) {}

    public function handle(Order $order): Payment {
        $payment = $this->gateway->process($order->total);

        // Dispatch event instead of tightly coupling logic
        event(new PaymentProcessed($order, $payment));

        return $payment;
    }
}
```

**Action Composition:**
```php
final readonly class CompleteOrderAction {
    public function __construct(
        private ProcessPaymentAction $processPayment,
        private SendConfirmationAction $sendConfirmation,
    ) {}

    public function handle(Order $order): Order {
        $payment = $this->processPayment->handle($order);
        $this->sendConfirmation->handle($order);

        return $order->markComplete();
    }
}
```

## React Component & Hook Pattern

### Component Structure

```
src/
├── components/          # Reusable UI components
│   ├── button.tsx
│   ├── card.tsx
│   └── form-field.tsx
├── sections/            # Composite sections (headers, forms, features)
│   ├── user-profile-form.tsx
│   └── order-summary.tsx
├── layouts/             # Page layouts
│   ├── dashboard-layout.tsx
│   └── auth-layout.tsx
└── pages/               # Route pages
    ├── users/
    │   ├── index.tsx
    │   └── show.tsx
    └── orders/
        ├── index.tsx
        └── create.tsx
```

### Component Convention

**Small, focused components (<100 lines):**
```tsx
interface ButtonProps {
    label: string;
    onClick: () => void;
    variant?: 'primary' | 'secondary';
}

export function Button({ label, onClick, variant = 'primary' }: ButtonProps) {
    return (
        <button
            className={`btn btn-${variant}`}
            onClick={onClick}
        >
            {label}
        </button>
    );
}
```

**Composite sections (reusable blocks):**
```tsx
interface UserFormProps {
    user?: User;
    onSubmit: (data: UserData) => Promise<void>;
}

export function UserProfileForm({ user, onSubmit }: UserFormProps) {
    const form = useUserForm(user);

    const handleSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        await onSubmit(form.data);
    };

    return (
        <form onSubmit={handleSubmit}>
            <FormField label="Name" value={form.data.name} />
            <FormField label="Email" value={form.data.email} />
            <button type="submit">Save</button>
        </form>
    );
}
```

### Custom Hook Convention

**Naming:**
- `use` + PascalCase: `useUserForm`, `useFetchUsers`, `useAuthContext`
- Describe what it does: `useFormValidation`, `useLocalStorage`, `usePaginatedData`

**Pattern:**
```tsx
function useUserForm(initialUser?: User) {
    const [data, setData] = useState(initialUser || {});
    const [errors, setErrors] = useState({});

    const validate = () => {
        // Validation logic
    };

    const submit = async () => {
        // Submit logic
    };

    return { data, setData, errors, validate, submit };
}

// Usage
function MyComponent() {
    const form = useUserForm(user);
    return <form onSubmit={form.submit}>...</form>;
}
```

### Composition Pattern

**Extract child components:**
```tsx
function UserDashboard({ userId }) {
    const user = useUserData(userId);

    return (
        <div className="dashboard">
            <UserHeader user={user} />
            <UserStats user={user} />
            <UserActivity user={user} />
        </div>
    );
}

// Separate components
function UserHeader({ user }) {
    return <header>{user.name}</header>;
}

function UserStats({ user }) {
    return <div>Stats content</div>;
}

function UserActivity({ user }) {
    return <div>Activity content</div>;
}
```

## Vue Composition & Pattern

### Component Structure

```
src/
├── components/          # Reusable UI components
├── views/              # Route views/pages
├── composables/        # Reusable composition functions
│   ├── useUserForm.ts
│   └── useFetchData.ts
└── services/           # API client services
    └── userService.ts
```

### Composable Convention

```typescript
// composables/useUserForm.ts
import { ref, computed } from 'vue';

export function useUserForm(initialUser = null) {
    const data = ref(initialUser || {});
    const errors = ref({});

    const validate = () => {
        // Validation logic
    };

    const submit = async () => {
        // Submit logic
    };

    return { data, errors, validate, submit };
}

// Usage in component
<script setup lang="ts">
import { useUserForm } from '@/composables/useUserForm';

const form = useUserForm(props.user);
</script>
```

## Node.js / TypeScript Pattern

### Service Structure

```
src/
├── services/
│   ├── user.service.ts
│   ├── order.service.ts
│   ├── email.service.ts
│   └── payment.service.ts
├── repositories/       # Data access
│   ├── user.repository.ts
│   └── order.repository.ts
├── actions/           # Complex operations
│   ├── create-order.action.ts
│   └── process-payment.action.ts
└── utils/             # Helper functions
    ├── validation.ts
    └── formatting.ts
```

### Service Class Convention

```typescript
export class UserService {
    constructor(private userRepository: UserRepository) {}

    async create(data: CreateUserDTO): Promise<User> {
        // Operation logic
        return user;
    }

    async update(id: string, data: UpdateUserDTO): Promise<User> {
        // Operation logic
        return user;
    }

    async delete(id: string): Promise<void> {
        // Operation logic
    }
}
```

### Action Class Convention (Node.js)

```typescript
export class CreateOrderAction {
    constructor(
        private orderService: OrderService,
        private paymentService: PaymentService,
    ) {}

    async execute(data: CreateOrderDTO): Promise<Order> {
        // Complex multi-step operation
        const order = await this.orderService.create(data);

        if (data.paymentMethod) {
            await this.paymentService.process(order.id, data.paymentMethod);
        }

        return order;
    }
}
```

### Helper Function Convention

```typescript
// utils/validation.ts
export function validateEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

export function validatePassword(password: string): boolean {
    return password.length >= 8;
}

// Usage
if (!validateEmail(email)) {
    throw new Error('Invalid email');
}
```

## Framework Auto-Detection

The plugin auto-detects your project type based on:

**Laravel:**
- Presence of `composer.json` with laravel/framework
- Directory structure with `app/`, `routes/`, `config/`

**React:**
- Presence of `package.json` with react dependency
- `.tsx` or `.jsx` files in `src/`

**Vue:**
- Presence of `package.json` with vue dependency
- `.vue` files in `src/`

**Node.js / Symfony:**
- Presence of `package.json` or `composer.json`
- Service-based file structure

## Customization per Project

Override defaults in `.claude/code-splitter.local.md`:

```yaml
---
# Laravel
laravel_actions_path: app/Actions
laravel_action_namespace: "App\\Actions"

# React
react_components_path: src/components
react_hooks_path: src/hooks

# Vue
vue_composables_path: src/composables
vue_components_path: src/components

# Node.js
node_services_path: src/services
node_actions_path: src/actions

# General
action_method_name: execute      # Instead of handle
max_lines_per_file: 100
---
```

## Additional Resources

For detailed examples and advanced patterns, see:
- **`references/framework-patterns.md`** - Comprehensive framework-specific patterns
- **`examples/`** - Real working examples for each framework

## Key Principles

1. **One operation per action/service**: Clear, focused responsibility
2. **Dependency injection**: Inject dependencies, don't rely on globals
3. **Reusability**: Can be called from multiple contexts
4. **Testability**: Testable in isolation with mocked dependencies
5. **Naming clarity**: Names describe what the action does
6. **Framework conventions**: Follow established patterns for your framework

Next steps: Use `/scan-code` to identify refactoring candidates, or `/split-code <file>` to apply these patterns to your code.
