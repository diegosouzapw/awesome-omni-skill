---
name: react-synapse
description: A React state management library using Preact Signals with fine-grained reactivity. Use it when you need global state without providers, minimal re-renders, or immutable updates via draft mutations. Works with React 18+.
license: ISC
compatibility: Requires React 18+, works with any React framework
metadata:
  author: slashnot84@gmail.com
  version: "0.1.0"
---

## Overview

React-synapse provides fine-grained reactive state management for React using Preact Signals. It offers a global singleton store pattern that requires no providers, supports draft mutations via Mutative, and minimizes re-renders by tracking only accessed values.

## When to use

- Building React applications needing global state without Context/Providers
- Fine-grained reactivity where components re-render only when accessed values change
- Complex nested state updates requiring immutable patterns with mutable syntax
- Sharing state between components without prop drilling

## Core API patterns

### String key access: `[value, setter]`

Use when you need both the value and a setter function for updates.

```jsx
import { createSignalStore } from 'react-synapse';

const { useStore } = createSignalStore({ count: 0, user: null });

function Counter() {
  const [count, setCount] = useStore('count');

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

### Selector for derived state: `value`

Use when you only need to read values, especially computed/derived values.

```jsx
function CartSummary() {
  const total = useStore(s => s.items.reduce((sum, item) => sum + item.price, 0));
  const itemCount = useStore(s => s.items.length);

  return <div>{itemCount} items - ${total}</div>;
}
```

### Draft mutations for updates

Use draft mutations for nested object/array updates. Mutative handles immutability internally.

```jsx
function UserProfile() {
  const [user, setUser] = useStore('user');

  const updateName = (name) => {
    setUser(draft => {
      draft.profile.name = name;
    });
  };

  const addTodo = (text) => {
    setUser(draft => {
      draft.todos.push({ id: Date.now(), text, completed: false });
    });
  };

  return (
    <input
      value={user.profile.name}
      onChange={e => updateName(e.target.value)}
    />
  );
}
```

## Rules

### ✅ DO

- **Use string keys for local component state**
  ```jsx
  const [count, setCount] = useStore('count');
  ```

- **Use selector functions for read-only/derived values**
  ```jsx
  const total = useStore(s => s.price * s.qty);
  ```

- **Use draft mutations for nested updates**
  ```jsx
  setUser(draft => { draft.name = 'Jane'; });
  ```

- **Create store at module level (not in components)**
  ```javascript
  // store.js - create once at module level
  export const { useStore } = createSignalStore({ ... });
  ```

- **Clear store between tests**
  ```javascript
  import { globalStore } from 'react-synapse';
  beforeEach(() => globalStore.clearStore());
  ```

### ❌ DON'T

- **Don't use React Context or Providers** - not needed with react-synapse
- **Don't destructure the store object** - loses reactivity
  ```jsx
  const [store] = useStore('store');
  const { user } = store; // ❌ Loses reactivity
  ```

- **Don't call useStore without arguments** - throws error
  ```jsx
  const state = useStore(); // ❌ Error
  ```

- **Don't mutate state directly** - always use setters
  ```jsx
  const [user, setUser] = useStore('user');
  user.name = 'Jane'; // ❌ Direct mutation
  ```

- **Don't create stores inside components** - creates new store on each render

## Examples

### Example 1: Todo List with Filter

```jsx
// store.js
import { createSignalStore } from 'react-synapse';

export const { useStore } = createSignalStore({
  todos: [],
  filter: 'all' // 'all' | 'active' | 'completed'
});

// TodoList.jsx
import { useStore } from './store';

function TodoList() {
  const [todos, setTodos] = useStore('todos');
  const [filter, setFilter] = useStore('filter');

  const addTodo = (text) => {
    setTodos(draft => {
      draft.push({ id: Date.now(), text, completed: false });
    });
  };

  const toggleTodo = (id) => {
    setTodos(draft => {
      const todo = draft.find(t => t.id === id);
      if (todo) todo.completed = !todo.completed;
    });
  };

  const filtered = todos.filter(t => {
    if (filter === 'active') return !t.completed;
    if (filter === 'completed') return t.completed;
    return true;
  });

  return (
    <div>
      <div>
        {['all', 'active', 'completed'].map(f => (
          <button key={f} onClick={() => setFilter(f)}>{f}</button>
        ))}
      </div>
      <ul>
        {filtered.map(todo => (
          <li
            key={todo.id}
            onClick={() => toggleTodo(todo.id)}
            style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Example 2: Shopping Cart with Derived State

```jsx
// store.js
import { createSignalStore } from 'react-synapse';

export const { store, useStore } = createSignalStore({
  items: [],
  coupon: null
});

// Cart.jsx
import { useStore } from './store';

function Cart() {
  const [items, setItems] = useStore('items');
  const [coupon, setCoupon] = useStore('coupon');

  // Derived values via selectors
  const subtotal = useStore(s => s.items.reduce((sum, i) => sum + i.price * i.qty, 0));
  const discount = useStore(s => s.coupon ? s.subtotal * s.coupon.discount : 0);
  const total = subtotal - discount;

  const addItem = (item) => {
    setItems(draft => {
      const existing = draft.find(i => i.id === item.id);
      if (existing) {
        existing.qty += 1;
      } else {
        draft.push({ ...item, qty: 1 });
      }
    });
  };

  return (
    <div>
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.name} x {item.qty} - ${item.price * item.qty}</li>
        ))}
      </ul>
      <p>Subtotal: ${subtotal.toFixed(2)}</p>
      {coupon && <p>Discount: -${discount.toFixed(2)}</p>}
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

### Example 3: Multi-Component State Sharing

```jsx
// store.js
export const { useStore } = createSignalStore({
  messages: [],
  currentUser: { name: 'Guest' }
});

// ChatInput.jsx
import { useStore } from './store';

export function ChatInput() {
  const [messages, setMessages] = useStore('messages');
  const [currentUser] = useStore('currentUser');

  const sendMessage = (text) => {
    setMessages(draft => {
      draft.push({ id: Date.now(), text, sender: currentUser.name });
    });
  };

  return (
    <input
      placeholder="Type a message..."
      onKeyDown={(e) => {
        if (e.key === 'Enter' && e.target.value.trim()) {
          sendMessage(e.target.value);
          e.target.value = '';
        }
      }}
    />
  );
}

// MessageList.jsx
import { useStore } from './store';

export function MessageList() {
  const [messages] = useStore('messages');
  const [currentUser] = useStore('currentUser');

  return (
    <div>
      {messages.map(msg => (
        <div key={msg.id} className={msg.sender === currentUser.name ? 'own' : 'other'}>
          <strong>{msg.sender}:</strong> {msg.text}
        </div>
      ))}
    </div>
  );
}

// App.jsx - No providers needed!
import { ChatInput } from './ChatInput';
import { MessageList } from './MessageList';

function App() {
  return (
    <div>
      <MessageList />
      <ChatInput />
    </div>
  );
}
```

## Quick reference

| Pattern | Returns | Use When |
|---------|---------|----------|
| `useStore('key')` | `[value, setter]` | Need to read and update state |
| `useStore(s => s.prop)` | `value` | Read-only or derived values |
| `useStore(s => [s.a, s.b])` | `[a, b]` | Multiple related reads |
| `useStore(s => ({a: s.a}))` | `{a}` | Named destructuring |
| `useStore('key', {unwrap: false})` | Signal | Fine-grained control |
| `useReactive(value)` | `[value, setter]` | Local component state |
| `useReactiveSignal($sig)` | `value` | Subscribe to existing signal |

## Module exports

```javascript
// Main - everything
import { createSignalStore, useReactive, computed, effect } from 'react-synapse';

// Store only
import { createSignalStore, globalStore } from 'react-synapse/store';

// Signals only
import { useReactive, computed, effect, signal } from 'react-synapse/signals';

// Mutative (draft mutations)
import { create } from 'react-synapse/mutative';
```
