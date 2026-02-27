---
name: setup-tanstack-start
description: Bootstrap a new web project with TanStack Start, React, Tailwind CSS v4, and shadcn/ui on top of the base tooling stack. Consult this skill whenever creating a web app, setting up a frontend project, starting a React application, or initializing anything involving TanStack Start, TanStack Router, TanStack Query, Tailwind, shadcn, or Vite.
disable-model-invocation: true
---

# Setup

Bootstrap a new web project on top of the base tooling stack.

Adds: TanStack Start + Router + Query + Devtools, Tailwind CSS v4, React with Vite, shadcn/ui

## Why This Stack

- **TanStack Start** — Full-stack React framework with SSR, built on Vite and Nitro. Provides file-based routing, server functions, and streaming out of the box.
- **TanStack Router** — Type-safe routing with built-in search param validation, code splitting, and preloading.
- **TanStack Query** — Async server-state management with automatic caching, deduplication, and background refetching. SSR integration with Router streams prefetched queries to the client.
- **Tailwind CSS v4** — Utility-first CSS with a new engine that's faster and uses standard CSS syntax for configuration.
- **shadcn/ui** — Copy-paste component library that gives full ownership of the code. Components are customized to project conventions after installation.

## Steps

### 1. Update package.json

```json
{
  "scripts": {
    "build": "vite build",
    "dev": "vite dev",
    "preview": "vite preview",
    "test:e2e": "playwright test",
    "validate": "bun run build && bun run lint && bun run types && bun run unused && bun run test"
  },
  "knip": {
    "ignore": ["src/components/ui/**"]
  }
}
```

### 2. Install dependencies

```bash
bun add @base-ui/react @tailwindcss/vite @tanstack/react-devtools @tanstack/react-form @tanstack/react-form-devtools @tanstack/react-query @tanstack/react-query-devtools @tanstack/react-router @tanstack/react-router-devtools @tanstack/react-router-ssr-query @tanstack/react-start react react-dom tailwindcss vite-tsconfig-paths
```

```bash
bun add -d @playwright/test @tanstack/devtools-vite @types/react @types/react-dom @vitejs/plugin-react @vitest/browser-playwright nitro vite vitest-browser-react
```

### 2.1. Install Playwright browsers

```bash
bunx playwright install chromium
```

### 3. Update tsconfig.json

```json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "lib": ["dom", "dom.iterable", "esnext"],
    "paths": {
      "@/*": ["./src/*"]
    },
    "types": ["vite/client"]
  },
  "include": ["**/*.ts", "**/*.tsx"]
}
```

The `paths` mapping is required because shadcn's CLI resolves `@/` literally when generating import paths. Without it, components end up in a physical `@/` directory instead of `src/`.

### 4. Update biome.jsonc

```jsonc
{
  "extends": ["ultracite/core", "ultracite/react"],
  "files": {
    "includes": ["**", "!src/components/ui/**"]
  },
  "overrides": [
    {
      "includes": ["**/$*.ts", "**/$*.tsx"],
      "linter": {
        "rules": {
          "style": {
            "useFilenamingConvention": "off"
          }
        }
      }
    }
  ]
}
```

### 5. Update .gitignore

```
# tanstack
.nitro
.output
.tanstack
.vercel
.vinxi
dist

# playwright
playwright-report
test-results
```

### 6. vite.config.ts

```ts
import tailwindcss from "@tailwindcss/vite";
import { devtools } from "@tanstack/devtools-vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import { nitro } from "nitro/vite";
import { defineConfig } from "vite";
import viteTsConfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [devtools(), viteTsConfigPaths(), tailwindcss(), tanstackStart(), nitro(), viteReact()],
});
```

### 7. Create vitest.config.ts

vitest.config.ts is separate from vite.config.ts because TanStack Start plugins (nitro, devtools, tanstackStart) should NOT run during tests.

Uses two projects to separate environments by file extension:
- `*.test.ts` → node (unit tests — has access to `process`, no DOM needed)
- `*.test.tsx` → browser mode (component tests — real Chromium via Playwright)

```ts
import { playwright } from "@vitest/browser-playwright";
import tailwindcss from "@tailwindcss/vite";
import viteReact from "@vitejs/plugin-react";
import viteTsConfigPaths from "vite-tsconfig-paths";
import { defineConfig } from "vitest/config";

export default defineConfig({
  plugins: [viteReact(), tailwindcss(), viteTsConfigPaths()],
  test: {
    projects: [
      {
        extends: true,
        test: {
          name: "unit",
          include: ["src/**/*.test.ts"],
          environment: "node",
        },
      },
      {
        extends: true,
        optimizeDeps: {
          exclude: [
            "@tanstack/react-start",
            "@tanstack/start-server-core",
            "@tanstack/start-client-core",
          ],
        },
        test: {
          name: "browser",
          include: ["src/**/*.test.tsx"],
          browser: {
            provider: playwright(),
            enabled: true,
            instances: [{ browser: "chromium" }],
          },
        },
      },
    ],
  },
});
```

### 8. Create playwright.config.ts

```ts
import { defineConfig } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  use: {
    baseURL: "http://localhost:3000",
  },
  webServer: {
    command: "bun run dev",
    port: 3000,
    reuseExistingServer: true,
  },
});
```

### 9. Create e2e/ directory

Create an empty `e2e/` directory at the project root. E2E tests live here, separate from unit and component tests.

### 10. Update src/lib/env.ts

```ts
import { createEnv } from "@t3-oss/env-core";
import { z } from "zod";

export const env = createEnv({
  client: {},
  clientPrefix: "VITE_",
  emptyStringAsUndefined: true,
  runtimeEnv: { ...process.env, ...import.meta.env },
  server: {},
  shared: {
    DEV: z.boolean(),
  },
});
```

### 10.1. Update src/lib/env.test.ts

Update the test from setup-base to verify the web-specific env configuration:

```ts
import { expect, test } from "vitest";
import { env } from "./env";

test("env initializes without error", () => {
  expect(env).toBeDefined();
});

test("DEV is a boolean", () => {
  expect(typeof env.DEV).toBe("boolean");
});
```

### 11. Set up shadcn/ui

The user configures their preset at `ui.shadcn.com/create` and copies the preset URL. Launch a **general-purpose subagent** (Task tool with `subagent_type: "general-purpose"`) to scaffold a temp project and extract the config files. Provide the preset URL and the project's CSS file path in the prompt.

The temp project approach is necessary because shadcn's `create` command generates a full project scaffold, but only a few config files are needed. Extracting them avoids polluting the existing project structure.

The subagent must:

1. **Create temp project:**
   ```bash
   rm -rf .tmp && mkdir .tmp && bunx --bun shadcn@latest create tmp -p "<preset-url>" -t vite -c .tmp
   ```
   > Always pass `-t vite -c .tmp` at the end. The CLI needs the explicit template flag and working directory even though the preset URL already encodes the template.

2. **Extract files** from `.tmp/tmp/`:
   | Source (`.tmp/tmp/...`) | Destination | Action |
   |-------------------------|-------------|--------|
   | `components.json` | `./components.json` | Copy to project root. Update `tailwind.css` path to `src/styles/app.css` |
   | `src/index.css` | `src/styles/app.css` | Copy as-is to `src/styles/app.css` |
   | `src/lib/utils.ts` | `src/lib/utils.ts` | Copy if `cn()` utility doesn't exist yet |
   | `package.json` | — | Read to identify new dependencies to install with `bun add` |

3. **Install dependencies** identified from the temp `package.json`.

4. **Clean up:**
   ```bash
   rm -rf .tmp
   ```

### 12. Install base shadcn/ui components

Install `button`, `empty`, and `label`.

### 13. Create src/components/form.tsx

App-level form abstraction using TanStack Form and Base UI Field. Provides a `useAppForm` hook with pre-configured form and field components that integrate with shadcn's `Button` and `Label`.

```tsx
import { Field } from "@base-ui/react/field";
import { type AnyFormApi, createFormHook, createFormHookContexts } from "@tanstack/react-form";
import { Button } from "@/components/ui/button";
import { Label as BaseLabel } from "@/components/ui/label";

const { fieldContext, formContext, useFieldContext, useFormContext } = createFormHookContexts();

export const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  formComponents: {
    Root: FormRoot,
    Submit: FormSubmit,
  },
  fieldComponents: {
    Root: FieldRoot,
    Label: FieldLabel,
    Control: FieldControl,
    ErrorMessage: FieldErrorMessage,
  },
});

function FormRoot({
  form,
  ...props
}: React.ComponentProps<"form"> & {
  form: AnyFormApi & {
    AppForm: React.ComponentType<React.PropsWithChildren>;
  };
}) {
  return (
    <form.AppForm>
      <form
        noValidate
        onSubmit={(e) => {
          e.preventDefault();
          e.stopPropagation();
          form.handleSubmit();
        }}
        {...props}
      />
    </form.AppForm>
  );
}

function FormSubmit(props: Omit<React.ComponentProps<typeof Button>, "disabled" | "type">) {
  const form = useFormContext();

  return (
    <form.Subscribe selector={(state) => [state.isPristine, state.canSubmit, state.isSubmitting]}>
      {([isPristine, canSubmit, isSubmitting]) => (
        <Button {...props} disabled={isPristine || !canSubmit || isSubmitting} type="submit" />
      )}
    </form.Subscribe>
  );
}

function FieldRoot({ className, ...props }: Field.Root.Props) {
  const field = useFieldContext();

  return (
    <Field.Root
      className={className}
      invalid={!field.state.meta.isValid}
      name={field.name}
      {...props}
    />
  );
}

function FieldLabel(props: Field.Label.Props) {
  return <Field.Label render={<BaseLabel />} {...props} />;
}

function FieldControl(props: Field.Control.Props) {
  const form = useFormContext();
  const field = useFieldContext<string>();

  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => (
        <Field.Control
          disabled={isSubmitting}
          onValueChange={field.handleChange}
          value={field.state.value}
          {...props}
        />
      )}
    </form.Subscribe>
  );
}

function FieldErrorMessage(props: Field.Error.Props) {
  const field = useFieldContext();
  const { errors } = field.state.meta;

  if (errors.length === 0) {
    return null;
  }

  return (
    <Field.Error className="text-left font-medium text-destructive text-sm" match {...props}>
      {errors.map((error) => (
        <div key={error?.message ?? String(error)}>{error?.message ?? String(error)}</div>
      ))}
    </Field.Error>
  );
}
```

### 13.1. Create src/components/form.test.tsx

```tsx
import { page } from "vitest/browser";
import { render } from "vitest-browser-react";
import { expect, test, vi } from "vitest";
import { z } from "zod";
import { useAppForm } from "./form";

function TestForm({ onSubmit = vi.fn() }: { onSubmit?: (data: { email: string }) => void }) {
  const form = useAppForm({
    defaultValues: { email: "" },
    validators: {
      onChange: z.object({ email: z.string().email("Invalid email") }),
    },
    onSubmit: ({ value }) => onSubmit(value),
  });

  return (
    <form.Root form={form}>
      <form.AppField name="email">
        {(field) => (
          <field.Root>
            <field.Label>Email</field.Label>
            <field.Control />
            <field.ErrorMessage />
          </field.Root>
        )}
      </form.AppField>
      <form.Submit>Submit</form.Submit>
    </form.Root>
  );
}

test("submit button is disabled when form is pristine", async () => {
  render(<TestForm />);
  await expect.element(page.getByRole("button", { name: "Submit" })).toBeDisabled();
});

test("submit button is enabled after valid input", async () => {
  render(<TestForm />);
  await page.getByLabelText("Email").fill("alice@test.com");
  await expect.element(page.getByRole("button", { name: "Submit" })).toBeEnabled();
});

test("shows error message for invalid input", async () => {
  render(<TestForm />);
  await page.getByLabelText("Email").fill("not-an-email");
  await expect.element(page.getByText("Invalid email")).toBeInTheDocument();
});

test("calls onSubmit with form data", async () => {
  const onSubmit = vi.fn();
  render(<TestForm onSubmit={onSubmit} />);
  await page.getByLabelText("Email").fill("alice@test.com");
  await page.getByRole("button", { name: "Submit" }).click();
  expect(onSubmit).toHaveBeenCalledWith({ email: "alice@test.com" });
});
```

### 14. src/router.tsx

```tsx
import { QueryClient } from "@tanstack/react-query";
import { createRouter, type ErrorComponentProps, Link } from "@tanstack/react-router";
import { setupRouterSsrQueryIntegration } from "@tanstack/react-router-ssr-query";
import { Button } from "./components/ui/button";
import { Empty, EmptyDescription, EmptyHeader, EmptyMedia, EmptyTitle } from "./components/ui/empty";
import { env } from "./lib/env";
import { routeTree } from "./routeTree.gen";

export function getRouter() {
  const queryClient = new QueryClient();

  const router = createRouter({
    routeTree,
    scrollRestoration: true,
    defaultPreload: "intent",
    defaultErrorComponent: DefaultErrorComponent,
    defaultNotFoundComponent: DefaultNotFoundComponent,
    context: { queryClient },
  });

  setupRouterSsrQueryIntegration({
    router,
    queryClient,
  });

  return router;
}

function DefaultErrorComponent({ error }: ErrorComponentProps) {
  return (
    <ErrorLayout
      description={env.DEV ? error.message : "An unexpected error occurred"}
      title="Something went wrong"
    />
  );
}

function DefaultNotFoundComponent() {
  return (
    <ErrorLayout
      description="The page you are looking for does not exist."
      title="Page not found"
    />
  );
}

function ErrorLayout({
  title,
  description,
}: {
  title: React.ReactNode;
  description: React.ReactNode;
}) {
  return (
    <div className="grid min-h-svh place-items-center px-4">
      <Empty className="max-w-lg border">
        <EmptyHeader>
          <EmptyMedia variant="icon">
            {/* Import and render a warning/error icon from the project's icon library */}
          </EmptyMedia>
          <EmptyTitle>{title}</EmptyTitle>
          <EmptyDescription>{description}</EmptyDescription>
        </EmptyHeader>
        <Button nativeButton={false} render={<Link to="/" />}>
          Go home
        </Button>
      </Empty>
    </div>
  );
}

declare module "@tanstack/react-router" {
  interface Register {
    router: ReturnType<typeof getRouter>;
  }
}
```

### 15. src/routes/__root.tsx

```tsx
import type { QueryClient } from "@tanstack/react-query";
import { createRootRouteWithContext, HeadContent, Outlet, Scripts } from "@tanstack/react-router";
import { lazy, Suspense } from "react";
import { env } from "../lib/env";
import appCss from "../styles/app.css?url";

export const Route = createRootRouteWithContext<{ queryClient: QueryClient }>()({
  head: () => ({
    meta: [
      {
        charSet: "utf-8",
      },
      {
        name: "viewport",
        content: "width=device-width, initial-scale=1",
      },
      {
        title: "<AppName>",
      },
    ],
    links: [
      {
        rel: "stylesheet",
        href: appCss,
      },
    ],
  }),
  component: RootComponent,
  shellComponent: RootDocument,
});

function RootComponent() {
  return <Outlet />;
}

const Devtools = lazy(() =>
  import("../components/devtools").then((mod) => ({ default: mod.Devtools })),
);

function RootDocument({ children }: { children: React.ReactNode }) {
  return (
    <html className="dark" lang="en">
      <head>
        <HeadContent />
      </head>
      <body className="bg-background font-sans text-foreground antialiased">
        {children}
        {env.DEV && (
          <Suspense>
            <Devtools />
          </Suspense>
        )}
        <Scripts />
      </body>
    </html>
  );
}
```

### 16. src/components/devtools.tsx

```tsx
import { TanStackDevtools } from "@tanstack/react-devtools";
import { formDevtoolsPlugin } from "@tanstack/react-form-devtools";
import { ReactQueryDevtoolsPanel } from "@tanstack/react-query-devtools";
import { TanStackRouterDevtoolsPanel } from "@tanstack/react-router-devtools";

export function Devtools() {
  return (
    <TanStackDevtools
      config={{
        position: "bottom-right",
      }}
      plugins={[
        {
          name: "TanStack Router",
          render: <TanStackRouterDevtoolsPanel />,
        },
        {
          name: "TanStack Query",
          render: <ReactQueryDevtoolsPanel />,
        },
        formDevtoolsPlugin(),
      ]}
    />
  );
}
```

### 17. src/routes/index.tsx

```tsx
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/")({
  component: RouteComponent,
});

function RouteComponent() {
  return (
    <main className="grid min-h-svh place-items-center">
      <h1 className="text-4xl font-bold"><AppName></h1>
    </main>
  );
}
```

### 18. Set up dark mode in src/styles/app.css

shadcn generates a `@custom-variant dark` line and separate `:root` / `.dark` blocks. Replace them to support three modes: light, dark, and system (auto).

#### 18.1 Replace the `@custom-variant dark` line

```css
/* Before (generated by shadcn) */
@custom-variant dark (&:is(.dark *));

/* After */
@custom-variant dark {
  &:is(.dark *) {
    @slot;
  }

  @media (prefers-color-scheme: dark) {
    &:is(.auto *) {
      @slot;
    }
  }
}
```

This makes Tailwind's `dark:` utilities work for both `.dark` (forced) and `.auto` (system preference).

#### 18.2 Add `color-scheme` classes

Add these right after the `@custom-variant` block:

```css
.light {
  color-scheme: light;
}

.dark {
  color-scheme: dark;
}

.auto {
  color-scheme: light dark;
}
```

#### 18.3 Merge `:root` and `.dark` blocks using `light-dark()`

Instead of separate `:root` (light) and `.dark` blocks, define every variable once using `light-dark(lightValue, darkValue)`. The browser picks the correct value based on the `color-scheme` property set by the classes above.

```css
/* Before (two blocks with duplicated variables) */
:root {
  --background: oklch(1 0 0);
  /* ... */
}

.dark {
  --background: oklch(0.145 0 0);
  /* ... */
}

/* After (single block, zero duplication) */
:root {
  --background: light-dark(oklch(1 0 0), oklch(0.145 0 0));
  --foreground: light-dark(oklch(0.145 0 0), oklch(0.985 0 0));
  /* ... */
}
```

Variables that share the same value in both modes don't need `light-dark()`. Delete the `.dark { ... }` block entirely.

### 19. Create src/lib/theme.ts

Server functions for cookie-based theme persistence, extracted into a utility module so the component doesn't import `@tanstack/react-start` directly (which has virtual module imports that break vitest browser mode pre-transforms):

```ts
import { createServerFn } from "@tanstack/react-start";
import { getCookie, setCookie } from "@tanstack/react-start/server";
import { z } from "zod";

const STORAGE_KEY = "app-theme";

export const THEME_VALUES = ["light", "dark", "auto"] as const;
export const themeSchema = z.enum(THEME_VALUES);

export const getTheme = createServerFn().handler(() => {
  return themeSchema.parse(getCookie(STORAGE_KEY) ?? "auto");
});

export const setTheme = createServerFn()
  .inputValidator(themeSchema)
  .handler(({ data }) => setCookie(STORAGE_KEY, data));
```

### 19.1. Create src/components/theme-toggle.tsx

Toggle component that cycles through light → dark → system:

Import sun, moon, and monitor icons from the icon library configured in `components.json`.

```tsx
import { useRouteContext, useRouter } from "@tanstack/react-router";
import { Monitor, Moon, Sun } from "<icon-library>"; // use the project's icon library
import { setTheme, THEME_VALUES, themeSchema } from "@/lib/theme";
import { Button } from "./ui/button";

export function ThemeToggle(props: React.ComponentProps<typeof Button>) {
  const { theme } = useRouteContext({ from: "__root__" });
  const router = useRouter();

  function toggleTheme() {
    const next =
      THEME_VALUES[(THEME_VALUES.indexOf(theme) + 1) % THEME_VALUES.length];
    setTheme({ data: themeSchema.parse(next) }).then(() =>
      router.invalidate(),
    );
  }

  const THEME_LABELS = {
    light: "Light",
    dark: "Dark",
    auto: "System",
  } as const;

  let Icon = Monitor;
  if (theme === "dark") {
    Icon = Moon;
  } else if (theme === "light") {
    Icon = Sun;
  }

  return (
    <Button
      aria-label="Toggle theme"
      onClick={toggleTheme}
      size="sm"
      variant="outline"
      {...props}
    >
      <Icon />
      {THEME_LABELS[theme]}
    </Button>
  );
}
```

### 19.2. Create src/components/theme-toggle.test.tsx

```tsx
import { page } from "vitest/browser";
import { render } from "vitest-browser-react";
import { expect, test, vi } from "vitest";

const { mockSetTheme, mockInvalidate } = vi.hoisted(() => ({
  mockSetTheme: vi.fn(() => Promise.resolve()),
  mockInvalidate: vi.fn(),
}));

vi.mock("@tanstack/react-router", () => ({
  useRouteContext: vi.fn(() => ({ theme: "auto" })),
  useRouter: vi.fn(() => ({ invalidate: mockInvalidate })),
}));

vi.mock("@/lib/theme", () => ({
  setTheme: mockSetTheme,
  THEME_VALUES: ["light", "dark", "auto"] as const,
  themeSchema: { parse: (v: string) => v },
}));

import { useRouteContext } from "@tanstack/react-router";
import { ThemeToggle } from "./theme-toggle";

test("displays System label for auto theme", async () => {
  render(<ThemeToggle />);
  await expect.element(page.getByRole("button", { name: "Toggle theme" })).toHaveTextContent("System");
});

test("displays Light label for light theme", async () => {
  vi.mocked(useRouteContext).mockReturnValue({ theme: "light" });
  render(<ThemeToggle />);
  await expect.element(page.getByRole("button", { name: "Toggle theme" })).toHaveTextContent("Light");
});

test("displays Dark label for dark theme", async () => {
  vi.mocked(useRouteContext).mockReturnValue({ theme: "dark" });
  render(<ThemeToggle />);
  await expect.element(page.getByRole("button", { name: "Toggle theme" })).toHaveTextContent("Dark");
});

test("calls setTheme on click", async () => {
  vi.mocked(useRouteContext).mockReturnValue({ theme: "auto" });
  render(<ThemeToggle />);
  await page.getByRole("button", { name: "Toggle theme" }).click();
  expect(mockSetTheme).toHaveBeenCalled();
});
```

### 20. Update src/routes/__root.tsx for theme support

Import `getTheme` and add `beforeLoad` to read the theme cookie on every navigation:

```tsx
import { getTheme } from "../lib/theme";

export const Route = createRootRoute({
  beforeLoad: async () => ({ theme: await getTheme() }),
  // ... rest of config
});
```

Apply the theme class to `<html>`:

```tsx
function RootDocument({ children }: { children: React.ReactNode }) {
  const { theme } = Route.useRouteContext();

  return (
    <html className={theme} lang="en">
      {/* ... */}
    </html>
  );
}
```

### 21. Add ThemeToggle to the index route

```tsx
import { ThemeToggle } from "../components/theme-toggle";

function RouteComponent() {
  return (
    <main className="grid min-h-svh place-items-center">
      <ThemeToggle />
      {/* ... */}
    </main>
  );
}
```

### 22. Update .github/workflows/ci.yml

Add an `e2e` job alongside the existing `check` job. It installs Playwright browsers, builds the app, and runs E2E tests against the production build:

```yaml
  e2e:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Bun
        uses: oven-sh/setup-bun@v1
      - name: Cache Bun dependencies
        uses: actions/cache@v4
        with:
          path: ~/.bun/install/cache
          key: ${{ runner.os }}-bun-${{ hashFiles('**/bun.lock') }}
          restore-keys: |
            ${{ runner.os }}-bun-
      - name: Install dependencies
        run: bun install
      - name: Install Playwright browsers
        run: bunx playwright install --with-deps chromium
      - name: Build
        run: bun run build
      - name: Run E2E tests
        run: bun run test:e2e
```

### 23. Create e2e/app.spec.ts

A smoke test that verifies the app boots and the theme toggle works end-to-end:

```ts
import { expect, test } from "@playwright/test";

test("homepage loads", async ({ page }) => {
  await page.goto("/");
  await expect(page.getByRole("heading")).toBeVisible();
});

test("theme toggle cycles through modes", async ({ page }) => {
  await page.goto("/");

  const toggle = page.getByRole("button", { name: "Toggle theme" });

  // Default is auto (System)
  await expect(toggle).toContainText("System");

  // First click needs retry — SSR renders the button before React hydrates the onClick handler
  await expect(async () => {
    await toggle.click();
    await expect(toggle).toContainText("Light", { timeout: 1000 });
  }).toPass({ timeout: 10_000 });

  // Subsequent clicks work immediately — app is already hydrated
  await toggle.click();
  await expect(toggle).toContainText("Dark");

  await toggle.click();
  await expect(toggle).toContainText("System");
});
```
