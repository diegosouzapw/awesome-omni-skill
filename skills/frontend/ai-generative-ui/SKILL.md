---
name: ai-generative-ui
description: Data-driven generative UI — tool results render as rich React components in chat instead of raw JSON. Uses a registry pattern with _ui field, not createStreamableUI(). Use this skill when the user says "generative ui", "rich tool cards", "custom tool rendering", or "tool components".
author: "@mattwoodco"
version: 1.0.0
created: 2026-02-13
updated: 2026-02-13
dependencies: [ai-tools]
---

# AI Generative UI

Experience layer that renders tool results as interactive React components inside the chat stream instead of raw JSON. Uses a data-driven approach: tool `execute` functions return a `_ui` field that names a registered component, and the client-side renderer looks it up in a registry.

This does **not** use `createStreamableUI()` (that is an older RSC pattern incompatible with the current architecture). Instead, tool results are plain data objects with a `_ui` type hint that the client uses to select the appropriate React component.

## Prerequisites

- Next.js app with `src/` directory and App Router
- `ai-core` skill installed (provides `getModel()`)
- `ai-chat` skill installed (provides chat UI, route pipeline, message renderer)
- `ai-tools` skill installed (provides tool calling framework and `tool-invocation` rendering)

## Installation

No additional packages required. Uses `ai`, `zod`, and `@ai-sdk/react` already installed by prerequisite skills.

## What Gets Created

```
src/
├── lib/
│   └── ai/
│       └── ui-registry.ts             # Component registry — maps tool _ui values to React components
└── components/
    └── ai/
        └── gen-ui/
            ├── weather-card.tsx        # Example: weather display with temperature + conditions
            ├── data-card.tsx           # Example: structured key-value display card
            └── confirmation.tsx        # Example: interactive yes/no confirmation card
```

## What Gets Modified

```
src/
├── app/
│   └── api/
│       └── ai/
│           └── chat/
│               └── route.ts           # Tool execute functions return _ui field
└── components/
    └── ai/
        └── message.tsx                # Check tool-result for _ui field, render registered component
```

## Comment Slots

- **message.tsx**: `// [ai-generative-ui]: check for _ui field` — checks tool results for `_ui` field and renders registered component
- **message.tsx**: `// [ai-generative-ui]: import gen-ui components to trigger registration` — side-effect imports that register components

## Setup Steps

### Step 1: Create `src/lib/ai/ui-registry.ts`

```typescript
import { type ComponentType } from "react";

/**
 * Registry mapping `_ui` field values from tool results to React components.
 *
 * When a tool's `execute` function returns `{ ...data, _ui: "WeatherCard" }`,
 * the message renderer looks up "WeatherCard" in this registry and renders
 * the matched component with the tool result as the `data` prop.
 *
 * If no component is found, the renderer falls back to the default JSON
 * tool result card from ai-tools.
 */

type GenUIProps<T = Record<string, unknown>> = {
  data: T;
};

type GenUIComponent = ComponentType<GenUIProps>;

const registry = new Map<string, GenUIComponent>();

/**
 * Register a component for a given _ui key.
 * Call this at module scope in your component files.
 */
export function registerUIComponent(
  key: string,
  component: GenUIComponent
): void {
  registry.set(key, component);
}

/**
 * Look up a component by _ui key.
 * Returns undefined if no component is registered.
 */
export function getUIComponent(
  key: string
): GenUIComponent | undefined {
  return registry.get(key);
}

/**
 * Check if a tool result has a _ui field that maps to a registered component.
 */
export function hasUIComponent(result: unknown): result is {
  _ui: string;
  [key: string]: unknown;
} {
  return (
    typeof result === "object" &&
    result !== null &&
    "_ui" in result &&
    typeof (result as Record<string, unknown>)._ui === "string" &&
    registry.has((result as Record<string, unknown>)._ui as string)
  );
}

export type { GenUIProps, GenUIComponent };
```

### Step 2: Create `src/components/ai/gen-ui/weather-card.tsx`

```tsx
"use client";

import { memo } from "react";
import { registerUIComponent, type GenUIProps } from "@/lib/ai/ui-registry";

type WeatherData = {
  location: string;
  temperature: number;
  unit: string;
  conditions: string;
  humidity: number;
  windSpeed: number;
  windUnit: string;
  feelsLike: number;
  _ui: string;
};

function getWeatherIcon(conditions: string): string {
  const lower = conditions.toLowerCase();
  if (lower.includes("sun") || lower.includes("clear")) return "sun";
  if (lower.includes("cloud") && lower.includes("part"))
    return "cloud-sun";
  if (lower.includes("cloud")) return "cloud";
  if (lower.includes("rain") || lower.includes("drizzle"))
    return "cloud-rain";
  if (lower.includes("snow")) return "snowflake";
  if (lower.includes("thunder") || lower.includes("storm"))
    return "cloud-lightning";
  if (lower.includes("fog") || lower.includes("mist")) return "cloud-fog";
  return "thermometer";
}

function WeatherIcon({ conditions }: { conditions: string }) {
  const icon = getWeatherIcon(conditions);

  const icons: Record<string, React.ReactNode> = {
    sun: (
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="32"
        height="32"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        strokeLinecap="round"
        strokeLinejoin="round"
        className="text-yellow-500"
      >
        <circle cx="12" cy="12" r="4" />
        <path d="M12 2v2" />
        <path d="M12 20v2" />
        <path d="m4.93 4.93 1.41 1.41" />
        <path d="m17.66 17.66 1.41 1.41" />
        <path d="M2 12h2" />
        <path d="M20 12h2" />
        <path d="m6.34 17.66-1.41 1.41" />
        <path d="m19.07 4.93-1.41 1.41" />
      </svg>
    ),
    cloud: (
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="32"
        height="32"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        strokeLinecap="round"
        strokeLinejoin="round"
        className="text-gray-400"
      >
        <path d="M17.5 19H9a7 7 0 1 1 6.71-9h1.79a4.5 4.5 0 1 1 0 9Z" />
      </svg>
    ),
    "cloud-rain": (
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="32"
        height="32"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        strokeLinecap="round"
        strokeLinejoin="round"
        className="text-blue-400"
      >
        <path d="M4 14.899A7 7 0 1 1 15.71 8h1.79a4.5 4.5 0 0 1 2.5 8.242" />
        <path d="M16 14v6" />
        <path d="M8 14v6" />
        <path d="M12 16v6" />
      </svg>
    ),
    thermometer: (
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="32"
        height="32"
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        strokeWidth="2"
        strokeLinecap="round"
        strokeLinejoin="round"
        className="text-orange-400"
      >
        <path d="M14 4v10.54a4 4 0 1 1-4 0V4a2 2 0 0 1 4 0Z" />
      </svg>
    ),
  };

  return <>{icons[icon] ?? icons.thermometer}</>;
}

const WeatherCard = memo(function WeatherCard({ data }: GenUIProps<WeatherData>) {
  return (
    <div className="overflow-hidden rounded-xl border bg-gradient-to-br from-blue-50 to-sky-50 dark:from-blue-950 dark:to-sky-950">
      <div className="p-4">
        {/* Location + Icon row */}
        <div className="flex items-start justify-between">
          <div>
            <p className="text-sm font-medium text-muted-foreground">
              {data.location}
            </p>
            <p className="mt-1 text-3xl font-bold tracking-tight">
              {Math.round(data.temperature)}{data.unit === "celsius" ? "\u00B0C" : "\u00B0F"}
            </p>
          </div>
          <WeatherIcon conditions={data.conditions} />
        </div>

        {/* Conditions */}
        <p className="mt-1 text-sm capitalize text-muted-foreground">
          {data.conditions}
        </p>

        {/* Details row */}
        <div className="mt-4 grid grid-cols-3 gap-3 border-t pt-3">
          <div>
            <p className="text-xs text-muted-foreground">Feels like</p>
            <p className="text-sm font-medium">
              {Math.round(data.feelsLike)}{data.unit === "celsius" ? "\u00B0" : "\u00B0"}
            </p>
          </div>
          <div>
            <p className="text-xs text-muted-foreground">Humidity</p>
            <p className="text-sm font-medium">{data.humidity}%</p>
          </div>
          <div>
            <p className="text-xs text-muted-foreground">Wind</p>
            <p className="text-sm font-medium">
              {data.windSpeed} {data.windUnit}
            </p>
          </div>
        </div>
      </div>
    </div>
  );
});

// Register the component so the message renderer can find it
registerUIComponent("WeatherCard", WeatherCard as unknown as React.ComponentType<GenUIProps>);

export { WeatherCard };
```

### Step 3: Create `src/components/ai/gen-ui/data-card.tsx`

```tsx
"use client";

import { useId, memo } from "react";
import { registerUIComponent, type GenUIProps } from "@/lib/ai/ui-registry";

type DataCardData = {
  title: string;
  subtitle?: string;
  fields: Array<{
    label: string;
    value: string | number | boolean;
    type?: "text" | "number" | "badge" | "link";
  }>;
  _ui: string;
};

const DataCard = memo(function DataCard({ data }: GenUIProps<DataCardData>) {
  const fieldId = useId();

  return (
    <div className="overflow-hidden rounded-xl border">
      {/* Header */}
      <div className="border-b bg-muted/50 px-4 py-3">
        <h3 className="text-sm font-semibold">{data.title}</h3>
        {data.subtitle && (
          <p className="mt-0.5 text-xs text-muted-foreground">
            {data.subtitle}
          </p>
        )}
      </div>

      {/* Fields */}
      <div className="divide-y">
        {data.fields.map((field) => (
          <div
            key={`${fieldId}-${field.label}`}
            className="flex items-center justify-between px-4 py-2.5"
          >
            <span className="text-sm text-muted-foreground">
              {field.label}
            </span>
            <span className="text-sm font-medium">
              {field.type === "badge" ? (
                <span className="inline-flex items-center rounded-full bg-primary/10 px-2 py-0.5 text-xs font-medium text-primary">
                  {String(field.value)}
                </span>
              ) : field.type === "link" ? (
                <a
                  href={String(field.value)}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary underline-offset-4 hover:underline"
                >
                  {String(field.value)}
                </a>
              ) : (
                String(field.value)
              )}
            </span>
          </div>
        ))}
      </div>
    </div>
  );
});

registerUIComponent("DataCard", DataCard as unknown as React.ComponentType<GenUIProps>);

export { DataCard };
```

### Step 4: Create `src/components/ai/gen-ui/confirmation.tsx`

```tsx
"use client";

import { useState, useCallback, memo } from "react";
import { registerUIComponent, type GenUIProps } from "@/lib/ai/ui-registry";

type ConfirmationData = {
  title: string;
  description: string;
  confirmLabel?: string;
  cancelLabel?: string;
  callbackUrl?: string;
  payload?: Record<string, unknown>;
  _ui: string;
};

type ConfirmationState = "pending" | "confirmed" | "cancelled" | "loading";

const Confirmation = memo(function Confirmation({ data }: GenUIProps<ConfirmationData>) {
  const [state, setState] = useState<ConfirmationState>("pending");

  const handleAction = useCallback(
    async (action: "confirmed" | "cancelled") => {
      if (!data.callbackUrl) {
        setState(action);
        return;
      }

      setState("loading");
      try {
        await fetch(data.callbackUrl, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            action,
            ...(data.payload ?? {}),
          }),
        });
        setState(action);
      } catch {
        setState(action);
      }
    },
    [data.callbackUrl, data.payload]
  );

  if (state === "confirmed") {
    return (
      <div className="flex items-center gap-2 rounded-xl border border-green-200 bg-green-50 px-4 py-3 dark:border-green-800 dark:bg-green-950">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="18"
          height="18"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          strokeWidth="2"
          strokeLinecap="round"
          strokeLinejoin="round"
          className="text-green-600 dark:text-green-400"
        >
          <path d="M22 11.08V12a10 10 0 1 1-5.93-9.14" />
          <polyline points="22 4 12 14.01 9 11.01" />
        </svg>
        <p className="text-sm font-medium text-green-700 dark:text-green-300">
          Confirmed: {data.title}
        </p>
      </div>
    );
  }

  if (state === "cancelled") {
    return (
      <div className="flex items-center gap-2 rounded-xl border border-gray-200 bg-gray-50 px-4 py-3 dark:border-gray-700 dark:bg-gray-900">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="18"
          height="18"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          strokeWidth="2"
          strokeLinecap="round"
          strokeLinejoin="round"
          className="text-gray-400"
        >
          <circle cx="12" cy="12" r="10" />
          <path d="m15 9-6 6" />
          <path d="m9 9 6 6" />
        </svg>
        <p className="text-sm text-muted-foreground">
          Cancelled: {data.title}
        </p>
      </div>
    );
  }

  return (
    <div className="overflow-hidden rounded-xl border">
      <div className="p-4">
        <h3 className="text-sm font-semibold">{data.title}</h3>
        <p className="mt-1 text-sm text-muted-foreground">
          {data.description}
        </p>
      </div>
      <div className="flex gap-2 border-t bg-muted/30 px-4 py-3">
        <button
          type="button"
          onClick={() => handleAction("confirmed")}
          disabled={state === "loading"}
          className="inline-flex items-center justify-center rounded-lg bg-primary px-4 py-2 text-sm font-medium text-primary-foreground transition-colors hover:bg-primary/90 disabled:opacity-50"
        >
          {state === "loading" ? (
            <span className="flex items-center gap-2">
              <span className="h-3.5 w-3.5 animate-spin rounded-full border-2 border-primary-foreground border-t-transparent" />
              Processing...
            </span>
          ) : (
            data.confirmLabel ?? "Confirm"
          )}
        </button>
        <button
          type="button"
          onClick={() => handleAction("cancelled")}
          disabled={state === "loading"}
          className="inline-flex items-center justify-center rounded-lg border px-4 py-2 text-sm font-medium transition-colors hover:bg-muted disabled:opacity-50"
        >
          {data.cancelLabel ?? "Cancel"}
        </button>
      </div>
    </div>
  );
});

registerUIComponent("Confirmation", Confirmation as unknown as React.ComponentType<GenUIProps>);

export { Confirmation };
```

### Step 5: Modify `src/components/ai/message.tsx`

Update the `tool-invocation` case to check for a `_ui` field on tool results **before** falling back to the default tool card. This must be inserted before any existing artifact check or the default `ToolInvocationCard`.

Find this in `src/components/ai/message.tsx`:

```typescript
          case "tool-invocation": {
            return (
              <ToolInvocationCard
                key={`${partId}-${index}`}
                part={part}
              />
            );
          }
```

Replace with:

```typescript
          case "tool-invocation": {
            // [ai-generative-ui]: check for _ui field → render registered component
            if (
              part.state === "output-available" &&
              part.output &&
              hasUIComponent(part.output)
            ) {
              const Component = getUIComponent(part.output._ui);
              if (Component) {
                return (
                  <Component
                    key={`${partId}-${index}`}
                    data={part.output as Record<string, unknown>}
                  />
                );
              }
            }
            return (
              <ToolInvocationCard
                key={`${partId}-${index}`}
                part={part}
              />
            );
          }
```

Add the imports at the top of the file. Find:

```typescript
import { useId } from "react";
```

Replace with:

```typescript
import { useId } from "react";
import { hasUIComponent, getUIComponent } from "@/lib/ai/ui-registry";
// [ai-generative-ui]: import gen-ui components to trigger registration
import "@/components/ai/gen-ui/weather-card";
import "@/components/ai/gen-ui/data-card";
import "@/components/ai/gen-ui/confirmation";
```

**Important:** If `ai-artifacts` is also installed, the generative UI check should come **before** the artifact check, since `_ui` is a more specific match. The combined case would look like:

```typescript
          case "tool-invocation": {
            // [ai-generative-ui]: check for _ui field → render registered component
            if (
              part.state === "output-available" &&
              part.output &&
              hasUIComponent(part.output)
            ) {
              const Component = getUIComponent(part.output._ui);
              if (Component) {
                return (
                  <Component
                    key={`${partId}-${index}`}
                    data={part.output as Record<string, unknown>}
                  />
                );
              }
            }
            // [ai-artifacts]: render artifact refs as clickable cards
            if (
              part.state === "output-available" &&
              part.output &&
              typeof part.output === "object" &&
              "_artifact" in part.output
            ) {
              const artifactResult = part.output as {
                id: string;
                type: string;
                title: string;
                _update?: boolean;
              };
              return (
                <ArtifactCard
                  key={`${partId}-${index}`}
                  id={artifactResult.id}
                  type={artifactResult.type}
                  title={artifactResult.title}
                  isUpdate={!!artifactResult._update}
                  onClick={onArtifactClick}
                />
              );
            }
            return (
              <ToolInvocationCard
                key={`${partId}-${index}`}
                part={part}
              />
            );
          }
```

### Step 6: Modify `src/app/api/ai/chat/route.ts`

Update existing tool definitions to include `_ui` fields in their return values. This example shows how to add `_ui` to the weather tool (from ai-tools). Apply the same pattern to any tool that should render a custom component.

Find this in `src/lib/ai/tools/index.ts` (or wherever the weather tool is defined):

```typescript
export const weather = tool({
  description: "Get current weather for a location",
  parameters: z.object({
    location: z.string().describe("City name or location"),
  }),
  execute: async ({ location }) => {
```

After the existing `execute` return statement, ensure it returns a `_ui` field:

```typescript
  execute: async ({ location }) => {
    // ... existing weather fetch logic ...
    return {
      location,
      temperature: data.temperature,
      unit: "celsius",
      conditions: data.conditions,
      humidity: data.humidity,
      windSpeed: data.windSpeed,
      windUnit: "km/h",
      feelsLike: data.feelsLike,
      _ui: "WeatherCard",
    };
  },
```

For tools that should render a structured data card, return `_ui: "DataCard"`:

```typescript
  execute: async (params) => {
    // ... compute result ...
    return {
      title: "Calculation Result",
      fields: [
        { label: "Expression", value: params.expression },
        { label: "Result", value: result, type: "number" },
      ],
      _ui: "DataCard",
    };
  },
```

For tools that need user confirmation before proceeding, return `_ui: "Confirmation"`:

```typescript
  execute: async (params) => {
    return {
      title: "Delete all completed tasks?",
      description: "This will permanently remove 5 completed tasks.",
      confirmLabel: "Delete",
      cancelLabel: "Keep",
      callbackUrl: "/api/ai/sessions/tasks/bulk-delete",
      payload: { status: "done" },
      _ui: "Confirmation",
    };
  },
```

## Usage

### Adding Custom Generative UI Components

To create a new generative UI component:

1. **Create the component** in `src/components/ai/gen-ui/`:

```tsx
"use client";

import { registerUIComponent, type GenUIProps } from "@/lib/ai/ui-registry";

type StockData = {
  symbol: string;
  price: number;
  change: number;
  changePercent: number;
  _ui: string;
};

function StockCard({ data }: GenUIProps<StockData>) {
  const isPositive = data.change >= 0;
  return (
    <div className="rounded-xl border p-4">
      <div className="flex items-center justify-between">
        <span className="font-bold">{data.symbol}</span>
        <span className={isPositive ? "text-green-600" : "text-red-600"}>
          {isPositive ? "+" : ""}{data.changePercent.toFixed(2)}%
        </span>
      </div>
      <p className="text-2xl font-bold">${data.price.toFixed(2)}</p>
    </div>
  );
}

registerUIComponent("StockCard", StockCard as unknown as React.ComponentType<GenUIProps>);
export { StockCard };
```

1. **Import the component** in `message.tsx` to trigger registration:

```typescript
import "@/components/ai/gen-ui/stock-card";
```

1. **Return `_ui` from the tool** in your tool's `execute` function:

```typescript
return { symbol: "AAPL", price: 182.5, change: 3.2, changePercent: 1.78, _ui: "StockCard" };
```

### Fallback Behavior

When a tool result contains a `_ui` field but no component is registered for that key, or when the result has no `_ui` field at all, the renderer falls back to the default `ToolInvocationCard` from ai-tools (which shows the tool name, parameters, and JSON result in a collapsible card).

This means you can incrementally add generative UI components. Tools without `_ui` fields continue to work exactly as before.

## Acceptance Criteria

- Ask "what's the weather in Tokyo" -- WeatherCard component renders with temperature, conditions, humidity, and wind (not raw JSON)
- Ask a math question that uses the calculator tool -- default tool card renders (calculator has no `_ui` field or no registered component)
- The DataCard component renders structured key-value pairs when a tool returns `_ui: "DataCard"`
- The Confirmation component renders confirm/cancel buttons and transitions to confirmed/cancelled state on click
- Unregistered `_ui` values fall back gracefully to the default JSON tool result card
- Tools without a `_ui` field render the default collapsible tool card
- Adding a new generative UI component requires only: (1) create component file with `registerUIComponent`, (2) import it in message.tsx, (3) return `_ui` from tool
- `tsc` passes with no errors
