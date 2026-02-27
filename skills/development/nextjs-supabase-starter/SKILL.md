---
name: nextjs-supabase-starter
description: Rules and best practices for building a Next.js starter app with Supabase (auth, profiles, RLS, declarative schemas, setup script). Use when working on Next.js + Supabase integration, authentication, migrations, or this starter app codebase.
---

# Next.js + Supabase Starter App Rules

Apply these rules when implementing or modifying the starter app. Names for routes, components, and hooks are flexible—use clear, consistent names and document choices. The starter should be **well-documented**, **easy to set up**, and include essential features for a modern web app with authentication.

---

## 1. Next.js

- **Version**: Use the latest Next.js; follow Next.js 13+ App Router conventions.
- **App Router only**: Use the `app/` directory; do not use `pages/`.
- **TypeScript**: Use TypeScript throughout; no `any` without justification. Use explicit semicolons at the end of statements in `.ts` and `.tsx` files.
- **Project structure**: Organize with `app/`, `components/`, `lib/`, etc. Document structure and conventions in README.
- **Styling**: Use Tailwind CSS primarily, and plain CSS if needed.
- **Mobile-first**: All layout and typography must be **mobile-first**. In Tailwind, base (unprefixed) classes define styles for the smallest viewport (mobile). Use `sm:`, `md:`, `lg:` only to add or override for larger screens. Do not use base classes for desktop and then scale down with breakpoints—design for mobile first, then enhance upward (e.g. base `text-sm` / `p-4`, then `md:text-base` / `md:p-6`). Use touch-friendly minimum sizes on mobile (e.g. `min-h-[2.75rem]` for buttons/links) and relax with `sm:min-h-0` or `md:` when appropriate. Page content should use `min-h-[calc(100vh-3.5rem)]` (or similar) to account for the header so vertical centering and scroll behavior are correct.
- **Middleware**: Use Next.js middleware for Supabase token refresh; implement the refresh logic in `proxy.ts` and use it from `middleware.ts` (see Supabase section).
- **Server vs client**: Prefer Server Components by default; add `'use client'` only where needed (e.g. forms, auth state in UI). Colocate route-specific components in route folders when it makes sense.
- **UX**: Use `loading.tsx` and `error.tsx` in route segments for loading and error states where appropriate.

---

## 2. Supabase Integration

- **CLI and client**: Install and configure Supabase CLI and client libraries; use `@supabase/ssr` for SSR-safe usage.
- **Local dev**: Set up Supabase for local development; run via `npx supabase start`; setup script configures and documents this.
- **Two client utilities**: Provide separate helpers for **server** and **client** components (e.g. `createServerClient`, `createBrowserClient` in `lib/supabase/`) using `@supabase/ssr`.
- **Middleware**: Configure Next.js middleware for token refresh using `proxy.ts`: put the session refresh / proxy logic in `proxy.ts` and invoke it from `middleware.ts` so the session is updated on each request.
- **Environment**: Require `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` in `.env.local`; document in README. Never expose the service_role key in client or frontend code; use only the anon key in the app.
- **Storage** (e.g. avatars): If using Supabase Storage, create a bucket and set RLS so users can read/update only their own objects (e.g. by path or user id).

---

## 3. Database and Migrations

- **No manual SQL**: All schema changes go through **migration files** only.
- **Declarative schemas**: Define tables in `supabase/schemas/` (e.g. `supabase/schemas/profiles.sql`). Generate migrations with:
  ```bash
  npx supabase db diff -f <migration_name>
  ```
- **Profiles table**:
  - Primary key: `id` referencing `auth.users(id)`.
  - Include `updated_at` with a trigger that sets it on row update.
  - Define in declarative schema; add trigger and RLS in migrations.
- **Automatic profile creation**: Use a PostgreSQL trigger that:
  - Fires **AFTER INSERT** on `auth.users`.
  - Reads the new user’s email (and id) from the inserted row.
  - Inserts one row into `profiles` with that `id` and email. Trigger and function live in a migration file.

---

## 4. Row Level Security (RLS)

- **Enable RLS** on `profiles`.
- **Policies** (use `auth.uid()`):
  - **SELECT**: User can read only their own profile.
  - **UPDATE**: User can update only their own profile.
  - **INSERT**: User can insert only their own profile (backup to trigger).
- Ensure users cannot read or modify other users’ profiles.

---

## 5. Authentication

- **Implement**: Sign up, sign in, sign out.
- **Protected routes**: Redirect unauthenticated users to login (e.g. from dashboard/profile).
- **State**: Handle auth state correctly in both server and client components.
- **Patterns**: Use consistent patterns—e.g. a `useAuth()` (or similar) hook for client components and server-side helpers (e.g. `getUser()`) for server components. Document these in README.
- **Errors**: Handle auth errors (invalid credentials, network, etc.) and show clear messages.

---

## 6. Example Pages and Routes

Route names are flexible (e.g. `/auth/login`, `/account`); keep them logical and consistent.

- **Home (`/`)**: Welcome, auth status, link to login/signup when logged out, link to dashboard when logged in.
- **Login**: Email/password form, error handling, redirect to dashboard on success.
- **Signup**: Email/password form, error handling, redirect to dashboard on success.
- **Dashboard (protected)**: Require auth; show profile summary and link to profile; sign out.
- **Profile (protected)**: Require auth; show profile; form to update (e.g. `full_name` and any other profile fields); **avatar upload** that allows selecting/uploading an image, stores it (e.g. Supabase Storage), updates `avatar_url` in `profiles`, displays the avatar, and includes a save/update button; handle upload and validation errors.

---

## 7. Setup Script

- **Purpose**: One command to install deps, start Supabase, write env, run migrations. Implement as `setup.sh`, `setup.js`, or use a tool like **zx** for cross-platform support; ensure it works reliably and handles edge cases.
- **Assumptions**: Supabase is already initialized (`supabase/` with migrations and schemas exists). Do not assume a clean machine beyond that.
- **Idempotent**: Safe to run multiple times. If `.env.local` exists, either skip or update it. If Supabase is already running, handle gracefully (e.g. skip start or detect and continue).
- **Steps** (in order):
  1. `npm install`
  2. `npx supabase start` (or detect already running)
  3. Extract Supabase URL and anon key from `supabase start` output
  4. Create or update `.env.local` with `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`
  5. Run migrations: `npx supabase db reset` or `npx supabase migration up`
  6. Print clear output showing what was done and next steps
- **Errors**: Provide helpful error messages if something goes wrong. In README, include instructions on how to run the script and troubleshoot.

---

## 8. Code Organization and Documentation

- **Explicit structure**: Decide where things live (e.g. `components/`, `components/ui/`, `lib/hooks/`, `lib/utils/`) and document in README so anyone using the starter understands structure and conventions.
- **Reusable components**: Store shared UI (buttons, forms, etc.) in a documented location.
- **Hooks and utils**: Keep custom hooks (e.g. auth) and utility functions in consistent locations; document naming and usage.
- **README must include**: Project description and purpose; prerequisites (Node.js version, Docker for Supabase, etc.); quick start (how to run the setup script); manual setup (step-by-step if someone prefers); project structure; how to use this starter for new projects; environment variables; database schema overview; authentication flow; deployment instructions; GitHub Actions setup and configuration; troubleshooting section.

---

## 9. Testing

- **Framework**: Use a unit test framework (e.g. Jest, Vitest).
- **Coverage**: Include at least a few examples for:
  - React components
  - Utility functions
  - Auth-related code
- **README**: How to run tests and how to add new tests.

---

## 10. Deployment and CI/CD

- **Deployment docs**: Document how to deploy to production (e.g. Vercel, Netlify): steps to set up the production Supabase project; how to configure environment variables in the deployment platform; how to link the production database; any platform-specific considerations (e.g. on Vercel: set env vars in project settings, use production Supabase URL/anon key, do not commit `.env.local`).
- **Vercel**: If deploying on Vercel, use the dashboard for env vars, connect the repo, and ensure `NEXT_PUBLIC_*` and Supabase keys are set. Middleware runs on the Edge runtime by default.
- **GitHub Actions**: Create a workflow that triggers on deployment or push to main/production branch; connects to the production Supabase instance; runs pending migrations via Supabase CLI; uses GitHub Secrets for sensitive credentials; handles errors with clear feedback. Document in README how to set up and configure the workflow and secrets.

---

## 11. General

- **Error handling**: Include proper error handling for authentication and database operations; avoid silent failures.
- **Comments**: Add comments in code where logic or intent is non-obvious.
- **Naming**: Use clear, consistent names for routes, components, hooks, and functions; document in README when choices affect how others use the starter.

## 12. Submission Checklist

- Remove `node_modules` before submitting. Zip the project folder; do not delete the `supabase/` directory (migrations and schemas must remain).
- Ensure the setup script is included and executable.
- Verify README is comprehensive and clear.
