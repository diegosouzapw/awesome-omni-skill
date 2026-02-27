---
name: sveltekit
description: Expert guidance for building modern, performant web applications with SvelteKit.
---

# SvelteKit Frontend Development Skill

## Overview
Expert guidance for building modern, performant web applications with SvelteKit.

## When to Use This Skill
- Creating new SvelteKit applications
- Building user interfaces and components
- Implementing client-server communication with Eden Treaty
- Setting up routing and layouts
- Managing state and forms

## Stack Context
- **Framework:** SvelteKit
- **Runtime:** Bun (dev server)
- **API Client:** Eden Treaty (type-safe ElysiaJS client)
- **Code Style:** Standard JS (no semicolons)
- **Build:** Vite (bundled with SvelteKit)

## Project Structure

```
packages/web/
├── src/
│   ├── routes/
│   │   ├── +page.svelte           # Home page
│   │   ├── +layout.svelte         # Root layout
│   │   ├── users/
│   │   │   ├── +page.svelte       # Users list
│   │   │   └── [id]/
│   │   │       └── +page.svelte   # User detail
│   │   └── api/
│   │       └── client.js          # API client
│   ├── lib/
│   │   ├── components/
│   │   │   ├── Header.svelte
│   │   │   └── Button.svelte
│   │   ├── stores/
│   │   │   └── user.js
│   │   └── utils/
│   │       └── format.js
│   └── app.html                   # HTML template
├── static/
│   └── favicon.png
├── package.json
├── svelte.config.js
└── vite.config.js
```

## Workflows

### 1. Initialize SvelteKit Project

```bash
# Create package
mkdir -p packages/web
cd packages/web

# Initialize SvelteKit
bun create svelte@latest .

# Select options:
# - Skeleton project
# - TypeScript: No (using JS)
# - Add ESLint: Yes
# - Add Prettier: Yes

# Install dependencies
bun install

# Add Eden Treaty for API client
bun add @elysiajs/eden
```

`package.json`:
```json
{
  "name": "@monorepo/web",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "standard"
  },
  "dependencies": {
    "@elysiajs/eden": "^1.0.0"
  },
  "devDependencies": {
    "@sveltejs/adapter-auto": "^3.0.0",
    "@sveltejs/kit": "^2.0.0",
    "@sveltejs/vite-plugin-svelte": "^3.0.0",
    "svelte": "^4.0.0",
    "vite": "^5.0.0"
  }
}
```

### 2. Basic SvelteKit Configuration

`svelte.config.js`:
```javascript
import adapter from '@sveltejs/adapter-auto'

export default {
  kit: {
    adapter: adapter()
  }
}
```

`vite.config.js`:
```javascript
import { sveltekit } from '@sveltejs/kit/vite'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [sveltekit()],
  server: {
    port: 5173
  }
})
```

### 3. API Client Setup with Eden Treaty

`src/lib/api/client.js`:
```javascript
import { edenTreaty } from '@elysiajs/eden'

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000'

export const api = edenTreaty(API_URL)

// Helper functions
export async function getUsers () {
  const { data, error } = await api.users.get()
  if (error) throw new Error(error.message || 'Failed to fetch users')
  return data.users
}

export async function getUser (id) {
  const { data, error } = await api.users[id].get()
  if (error) throw new Error(error.message || 'Failed to fetch user')
  return data.user
}

export async function createUser (userData) {
  const { data, error } = await api.users.post(userData)
  if (error) throw new Error(error.message || 'Failed to create user')
  return data
}

export async function updateUser (id, userData) {
  const { data, error } = await api.users[id].put(userData)
  if (error) throw new Error(error.message || 'Failed to update user')
  return data
}

export async function deleteUser (id) {
  const { data, error } = await api.users[id].delete()
  if (error) throw new Error(error.message || 'Failed to delete user')
  return data
}
```

`.env`:
```bash
VITE_API_URL=http://localhost:3000
```

### 4. Root Layout

`src/routes/+layout.svelte`:
```svelte
<script>
  import '../app.css'
</script>

<div class="app">
  <header>
    <nav>
      <a href="/">Home</a>
      <a href="/users">Users</a>
      <a href="/posts">Posts</a>
    </nav>
  </header>

  <main>
    <slot />
  </main>

  <footer>
    <p>&copy; 2024 My App</p>
  </footer>
</div>

<style>
  .app {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  header {
    background: #333;
    color: white;
    padding: 1rem;
  }

  nav {
    display: flex;
    gap: 1rem;
  }

  nav a {
    color: white;
    text-decoration: none;
  }

  nav a:hover {
    text-decoration: underline;
  }

  main {
    flex: 1;
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
    width: 100%;
  }

  footer {
    background: #f5f5f5;
    padding: 1rem;
    text-align: center;
  }
</style>
```

### 5. Home Page

`src/routes/+page.svelte`:
```svelte
<script>
  let name = 'World'
</script>

<svelte:head>
  <title>Home</title>
</svelte:head>

<div class="container">
  <h1>Welcome to SvelteKit</h1>
  <p>Hello {name}!</p>
  
  <input bind:value={name} placeholder="Enter your name" />
</div>

<style>
  .container {
    text-align: center;
  }

  h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
  }

  input {
    padding: 0.5rem;
    font-size: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-top: 1rem;
  }
</style>
```

### 6. Page with Server Load

`src/routes/users/+page.js`:
```javascript
import { getUsers } from '$lib/api/client'

export async function load () {
  try {
    const users = await getUsers()
    return { users }
  } catch (error) {
    return {
      users: [],
      error: error.message
    }
  }
}
```

`src/routes/users/+page.svelte`:
```svelte
<script>
  export let data
</script>

<svelte:head>
  <title>Users</title>
</svelte:head>

<div class="container">
  <h1>Users</h1>

  {#if data.error}
    <p class="error">{data.error}</p>
  {:else if data.users.length === 0}
    <p>No users found.</p>
  {:else}
    <ul class="users-list">
      {#each data.users as user}
        <li>
          <a href="/users/{user._id}">
            <h3>{user.name}</h3>
            <p>{user.email}</p>
          </a>
        </li>
      {/each}
    </ul>
  {/if}

  <a href="/users/new" class="button">Create User</a>
</div>

<style>
  .container {
    max-width: 800px;
  }

  .users-list {
    list-style: none;
    padding: 0;
    display: grid;
    gap: 1rem;
  }

  .users-list li {
    background: white;
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
    transition: box-shadow 0.2s;
  }

  .users-list li:hover {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }

  .users-list a {
    text-decoration: none;
    color: inherit;
  }

  .users-list h3 {
    margin: 0 0 0.5rem 0;
  }

  .users-list p {
    margin: 0;
    color: #666;
  }

  .button {
    display: inline-block;
    margin-top: 2rem;
    padding: 0.75rem 1.5rem;
    background: #0066cc;
    color: white;
    text-decoration: none;
    border-radius: 4px;
  }

  .button:hover {
    background: #0052a3;
  }

  .error {
    color: red;
    padding: 1rem;
    background: #ffe6e6;
    border-radius: 4px;
  }
</style>
```

### 7. Dynamic Route

`src/routes/users/[id]/+page.js`:
```javascript
import { getUser } from '$lib/api/client'
import { error } from '@sveltejs/kit'

export async function load ({ params }) {
  try {
    const user = await getUser(params.id)
    return { user }
  } catch (err) {
    throw error(404, 'User not found')
  }
}
```

`src/routes/users/[id]/+page.svelte`:
```svelte
<script>
  export let data
</script>

<svelte:head>
  <title>{data.user.name}</title>
</svelte:head>

<div class="container">
  <h1>{data.user.name}</h1>
  <p><strong>Email:</strong> {data.user.email}</p>
  <p><strong>Created:</strong> {new Date(data.user.createdAt).toLocaleDateString()}</p>
  
  <div class="actions">
    <a href="/users/{data.user._id}/edit" class="button">Edit</a>
    <a href="/users" class="button secondary">Back to Users</a>
  </div>
</div>

<style>
  .container {
    max-width: 600px;
  }

  .actions {
    display: flex;
    gap: 1rem;
    margin-top: 2rem;
  }

  .button {
    padding: 0.75rem 1.5rem;
    background: #0066cc;
    color: white;
    text-decoration: none;
    border-radius: 4px;
  }

  .button:hover {
    background: #0052a3;
  }

  .button.secondary {
    background: #666;
  }

  .button.secondary:hover {
    background: #555;
  }
</style>
```

### 8. Form with Actions

`src/routes/users/new/+page.server.js`:
```javascript
import { createUser } from '$lib/api/client'
import { redirect } from '@sveltejs/kit'

export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData()
    const name = formData.get('name')
    const email = formData.get('email')

    try {
      await createUser({ name, email })
      throw redirect(303, '/users')
    } catch (error) {
      return {
        error: error.message,
        name,
        email
      }
    }
  }
}
```

`src/routes/users/new/+page.svelte`:
```svelte
<script>
  import { enhance } from '$app/forms'
  export let form
</script>

<svelte:head>
  <title>Create User</title>
</svelte:head>

<div class="container">
  <h1>Create User</h1>

  {#if form?.error}
    <p class="error">{form.error}</p>
  {/if}

  <form method="POST" use:enhance>
    <div class="field">
      <label for="name">Name</label>
      <input
        id="name"
        name="name"
        type="text"
        required
        value={form?.name || ''}
      />
    </div>

    <div class="field">
      <label for="email">Email</label>
      <input
        id="email"
        name="email"
        type="email"
        required
        value={form?.email || ''}
      />
    </div>

    <div class="actions">
      <button type="submit">Create User</button>
      <a href="/users" class="button secondary">Cancel</a>
    </div>
  </form>
</div>

<style>
  .container {
    max-width: 600px;
  }

  .field {
    margin-bottom: 1rem;
  }

  label {
    display: block;
    margin-bottom: 0.5rem;
    font-weight: bold;
  }

  input {
    width: 100%;
    padding: 0.75rem;
    font-size: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .actions {
    display: flex;
    gap: 1rem;
    margin-top: 2rem;
  }

  button {
    padding: 0.75rem 1.5rem;
    background: #0066cc;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
  }

  button:hover {
    background: #0052a3;
  }

  .button.secondary {
    background: #666;
    text-decoration: none;
    padding: 0.75rem 1.5rem;
    color: white;
    border-radius: 4px;
  }

  .error {
    color: red;
    padding: 1rem;
    background: #ffe6e6;
    border-radius: 4px;
    margin-bottom: 1rem;
  }
</style>
```

### 9. Reusable Components

`src/lib/components/Button.svelte`:
```svelte
<script>
  export let variant = 'primary'
  export let type = 'button'
  export let href = null
</script>

{#if href}
  <a {href} class="button {variant}">
    <slot />
  </a>
{:else}
  <button {type} class="button {variant}" on:click>
    <slot />
  </button>
{/if}

<style>
  .button {
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    text-decoration: none;
    display: inline-block;
  }

  .button.primary {
    background: #0066cc;
    color: white;
  }

  .button.primary:hover {
    background: #0052a3;
  }

  .button.secondary {
    background: #666;
    color: white;
  }

  .button.secondary:hover {
    background: #555;
  }

  .button.danger {
    background: #cc0000;
    color: white;
  }

  .button.danger:hover {
    background: #a30000;
  }
</style>
```

Usage:
```svelte
<script>
  import Button from '$lib/components/Button.svelte'
</script>

<Button>Default Button</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="danger">Delete</Button>
<Button href="/users">Link Button</Button>
```

### 10. Stores for State Management

`src/lib/stores/user.js`:
```javascript
import { writable } from 'svelte/store'

export const currentUser = writable(null)

export function setUser (user) {
  currentUser.set(user)
}

export function clearUser () {
  currentUser.set(null)
}
```

Usage:
```svelte
<script>
  import { currentUser } from '$lib/stores/user'
</script>

{#if $currentUser}
  <p>Welcome, {$currentUser.name}!</p>
{:else}
  <p>Please log in</p>
{/if}
```

## Best Practices

1. **Routing:**
   - Use file-based routing
   - Implement layouts for shared UI
   - Use `+page.js` for data loading
   - Use `+page.server.js` for server-only code

2. **Data Loading:**
   - Load data in `load` functions
   - Handle errors gracefully
   - Show loading states

3. **Forms:**
   - Use SvelteKit form actions
   - Use `enhance` for progressive enhancement
   - Validate on client and server

4. **Components:**
   - Keep components small and focused
   - Use props for configuration
   - Use slots for composition
   - Colocate styles with components

5. **API Communication:**
   - Use Eden Treaty for type-safe API calls
   - Centralize API client logic
   - Handle errors consistently

6. **Performance:**
   - Use SSR by default
   - Preload data for better UX
   - Lazy load heavy components
   - Optimize images

## Common Patterns

**Loading state:**
```svelte
<script>
  import { onMount } from 'svelte'
  
  let loading = true
  let data = []

  onMount(async () => {
    try {
      data = await fetchData()
    } finally {
      loading = false
    }
  })
</script>

{#if loading}
  <p>Loading...</p>
{:else}
  <!-- Show data -->
{/if}
```

**Conditional rendering:**
```svelte
{#if condition}
  <p>True</p>
{:else if otherCondition}
  <p>Other</p>
{:else}
  <p>False</p>
{/if}
```

**Lists:**
```svelte
{#each items as item (item.id)}
  <div>{item.name}</div>
{/each}
```

## References

- [SvelteKit Documentation](https://kit.svelte.dev/)
- [Svelte Tutorial](https://svelte.dev/tutorial)
- [Eden Treaty Documentation](https://elysiajs.com/eden/overview.html)
