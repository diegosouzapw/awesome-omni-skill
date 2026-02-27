---
name: proqio-ui
description: Proqio UI component library patterns and best practices. Trigger when building UI across the project, importing from proqio-ui, choosing between custom vs library components, or using Button/Dialog/Form/Select/Table/Sidebar and any other Proqio UI component.
license: Apache-2.0
metadata:
    author: Infinitus
    version: '1.0'
    scope: [frontend, ui]
    auto_invoke: 'Using proqio-ui components, building UI components, importing from proqio-ui'
    allowed-tools: Read, Edit, Write, Glob, Grep, Bash, WebFetch, WebSearch, Task
---

## Critical Rules

- **ALWAYS** prefer proqio-ui components over building custom ones
- **NEVER** build a custom button, input, dialog, table, badge, tooltip, dropdown or any component that already exists in proqio-ui
- **ALWAYS** import from `proqio-ui` (single named import)
- CSS is already imported globally in `src/main.tsx` — do NOT re-import `proqio-ui/style.css` in components

## Import Pattern

```typescript
// ✅ Named imports from single entry point
import { Button, TextField, Dialog, DialogTrigger, DialogContent } from 'proqio-ui';

// ✅ Import variant functions separately when needed outside JSX
import { buttonVariants } from 'proqio-ui';

// ✅ Import hooks
import { useSidebar, useFormField } from 'proqio-ui';

// ❌ NEVER: deep imports
import { Button } from 'proqio-ui/dist/button';
```

## Component Decision Tree

```
Need a clickable action?           → Button / IconButton / LinkButton
Need a text input?                 → TextField / Textarea / NumberField
Need a dropdown selection?         → Select (compound)
Need a checkbox/radio/switch?      → Checkbox / RadioGroup / Switch
Need a modal/dialog?               → Dialog / AlertDialog
Need contextual menu?              → DropdownMenu / ContextMenu
Need a hover tooltip?              → Tooltip (wrap with TooltipProvider)
Need an inline popup?              → Popover
Need a status label?               → Badge
Need a nav path?                   → Breadcrumb
Need tabbed content?               → Tabs
Need expandable content?           → Accordion / Collapsible
Need date input?                   → Datepicker / Calendar
Need a form?                       → Form + FormField + FormItem + FormLabel + FormControl + FormMessage
Need a data table?                 → Table / SubTable
Need a side navigation?            → Sidebar (compound)
Need a progress indicator?         → LinearProgress / Stepper
Need empty content placeholder?    → EmptyState
Need loading placeholder?          → Skeleton
Need an alert/notification box?    → Callout
Need grouped buttons?              → ButtonGroup
```

## All Available Components

### Actions

| Component                | Key Props                                          | Variants                                                                                                                   |
| ------------------------ | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `Button`                 | `variant`, `intent`, `size`, `loading`, `inverted` | variant: `text\|primary\|outline`; intent: `default\|danger\|warning\|success\|info\|subtle`; size: `xs\|sm\|base\|lg\|xl` |
| `IconButton`             | Same as Button                                     | Same as Button                                                                                                             |
| `LinkButton`             | `asChild`                                          | Unstyled, polymorphic                                                                                                      |
| `ButtonGroup`            | `orientation`                                      | + `ButtonGroupText`, `ButtonGroupSeparator`                                                                                |
| `Toggle` / `ToggleGroup` | —                                                  | —                                                                                                                          |

### Inputs

| Component                       | Key Props                                  | Notes                                            |
| ------------------------------- | ------------------------------------------ | ------------------------------------------------ |
| `TextField`                     | `leftSlot`, `rightSlot`, `variant`, `size` | Wraps `<input>`                                  |
| `Textarea`                      | `maxLength`                                | Use with `TextareaCounter`                       |
| `NumberField`                   | `variant`, `size`                          | variant: `outline\|ghost`; size: `base\|compact` |
| `Checkbox`                      | `checked`, `onCheckedChange`               | Radix primitive                                  |
| `RadioGroup` / `RadioGroupItem` | —                                          | Radix primitive                                  |
| `Switch`                        | `size`                                     | size: `base\|md\|lg`                             |

### Display

| Component        | Key Props               | Variants                                                                                                                     |
| ---------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `Badge`          | `type`, `theme`, `size` | type: `badge\|tag`; theme: `gray\|light-gray\|orange\|red\|yellow\|green\|blue\|indigo\|purple\|pink\|black`; size: `sm\|lg` |
| `Callout`        | `variant`               | `danger\|warning\|success\|info\|subtle`                                                                                     |
| `Skeleton`       | —                       | Loading placeholder                                                                                                          |
| `LinearProgress` | `size`                  | size: `default\|compact\|slim`; + `LinearProgressBar`, `LinearProgressValue`                                                 |

### Navigation

| Component    | Notes                                                                                                                 |
| ------------ | --------------------------------------------------------------------------------------------------------------------- |
| `Breadcrumb` | + `BreadcrumbList`, `BreadcrumbItem`, `BreadcrumbLink`, `BreadcrumbPage`, `BreadcrumbSeparator`, `BreadcrumbEllipsis` |
| `Tabs`       | + `TabsList`, `TabsTrigger`, `TabsContent`; supports `collapsible` and tooltip                                        |
| `Sidebar`    | Full compound system — see Sidebar section below                                                                      |
| `Stepper`    | status: `default\|error\|warning\|success\|in-progress\|not-needed`                                                   |

### Overlay

| Component      | Notes                                                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `Dialog`       | + `DialogTrigger`, `DialogContent`, `DialogHeader`, `DialogBody`, `DialogFooter`, `DialogTitle`, `DialogCloseXButton`          |
| `AlertDialog`  | For destructive confirmations. + `AlertDialogAction`, `AlertDialogCancel`                                                      |
| `Popover`      | + `PopoverTrigger`, `PopoverContent`, `PopoverArrow`                                                                           |
| `Tooltip`      | Must wrap app in `<TooltipProvider>`. + `TooltipTrigger`, `TooltipContent`                                                     |
| `DropdownMenu` | + `DropdownMenuTrigger`, `DropdownMenuContent`, `DropdownMenuItem`, `DropdownMenuCheckboxItem`, `DropdownMenuSwitchItem`, etc. |
| `ContextMenu`  | Right-click triggered version of DropdownMenu                                                                                  |

### Data

| Component    | Notes                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------- |
| `Table`      | + `TableWrapper`, `TableHeader`, `TableBody`, `TableRow`, `TableHead`, `TableCell`                      |
| `Select`     | Compound with cmdk search. + `SelectTrigger`, `SelectContent`, `SelectItem`, `SelectCheckboxItem`, etc. |
| `Calendar`   | DayPicker wrapper                                                                                       |
| `Datepicker` | Full compound. Needs `DatepickerProvider`                                                               |

### Layout

| Component     | Notes                                                                                                        |
| ------------- | ------------------------------------------------------------------------------------------------------------ |
| `Accordion`   | + `AccordionItem`, `AccordionTrigger`, `AccordionContent`                                                    |
| `Collapsible` | + `CollapsibleTrigger`, `CollapsibleContent`                                                                 |
| `EmptyState`  | + `EmptyStateIcon`, `EmptyStateTitle`, `EmptyStateDescription`, `EmptyStateAction`; variant: `dashed\|solid` |

## Compound Component Pattern

Most complex components use compound composition. Always use all required sub-components:

```typescript
// ✅ Dialog
<Dialog>
  <DialogTrigger asChild>
    <Button variant="primary">Open</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Confirm Action</DialogTitle>
      <DialogCloseXButton />
    </DialogHeader>
    <DialogBody>Are you sure?</DialogBody>
    <DialogFooter>
      <Button variant="outline">Cancel</Button>
      <Button variant="primary" intent="danger">Delete</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>

// ✅ AlertDialog (for destructive actions)
<AlertDialog>
  <AlertDialogTrigger asChild>
    <Button intent="danger">Delete</Button>
  </AlertDialogTrigger>
  <AlertDialogContent>
    <AlertDialogHeader>
      <AlertDialogTitle>Delete resource?</AlertDialogTitle>
      <AlertDialogDescription>This action cannot be undone.</AlertDialogDescription>
    </AlertDialogHeader>
    <AlertDialogFooter>
      <AlertDialogCancel>Cancel</AlertDialogCancel>
      <AlertDialogAction>Delete</AlertDialogAction>
    </AlertDialogFooter>
  </AlertDialogContent>
</AlertDialog>
```

## Form Pattern (React Hook Form)

proqio-ui Form components integrate with react-hook-form:

```typescript
import { useForm } from 'react-hook-form';
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage, TextField, Button } from 'proqio-ui';

function LoginForm() {
  const form = useForm<{ email: string; password: string }>();

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="flex flex-col gap-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <TextField {...field} type="email" placeholder="you@example.com" />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" variant="primary">Login</Button>
      </form>
    </Form>
  );
}
```

## Select Pattern

Select uses compound composition with optional search (cmdk):

```typescript
import { Select, SelectTrigger, SelectValue, SelectContent, SelectList, SelectItem } from 'proqio-ui';

<Select value={value} onValueChange={setValue}>
  <SelectTrigger>
    <SelectValue placeholder="Choose option" />
  </SelectTrigger>
  <SelectContent>
    <SelectList>
      <SelectItem value="aws">AWS</SelectItem>
      <SelectItem value="azure">Azure</SelectItem>
      <SelectItem value="gcp">GCP</SelectItem>
    </SelectList>
  </SelectContent>
</Select>
```

## Badge Usage

```typescript
import { Badge, BadgeIcon, BadgeRemove } from 'proqio-ui';

// Status badges
<Badge theme="green">Active</Badge>
<Badge theme="red" type="tag">Critical</Badge>
<Badge theme="yellow" size="sm">Warning</Badge>

// With icon
<Badge theme="blue"><BadgeIcon><CloudIcon /></BadgeIcon>AWS</Badge>

// Removable
<Badge theme="indigo"><BadgeRemove onClick={handleRemove} />Tag</Badge>
```

## Button Intent Guide

```typescript
// Default actions
<Button variant="primary">Save</Button>
<Button variant="outline">Cancel</Button>
<Button variant="text">Learn more</Button>

// Semantic intents
<Button variant="primary" intent="danger">Delete</Button>
<Button variant="primary" intent="success">Approve</Button>
<Button variant="primary" intent="warning">Proceed with caution</Button>
<Button variant="outline" intent="info">View details</Button>

// Loading state
<Button variant="primary" loading={isSubmitting}>Saving...</Button>
```

## Tooltip Pattern

```typescript
// ✅ TooltipProvider must wrap the app or section (already in app root if configured)
import { TooltipProvider, Tooltip, TooltipTrigger, TooltipContent } from 'proqio-ui';

<TooltipProvider>
  <Tooltip>
    <TooltipTrigger asChild>
      <IconButton aria-label="Settings"><CogIcon /></IconButton>
    </TooltipTrigger>
    <TooltipContent>Settings</TooltipContent>
  </Tooltip>
</TooltipProvider>
```

## Sidebar System

```typescript
import { SidebarProvider, Sidebar, SidebarContent, SidebarMenu, SidebarMenuItem,
  SidebarMenuButton, SidebarHeader, SidebarFooter, useSidebar } from 'proqio-ui';

// SidebarProvider wraps the layout
<SidebarProvider>
  <Sidebar>
    <SidebarHeader>Logo</SidebarHeader>
    <SidebarContent>
      <SidebarMenu>
        <SidebarMenuItem>
          <SidebarMenuButton asChild>
            <a href="/dashboard">Dashboard</a>
          </SidebarMenuButton>
        </SidebarMenuItem>
      </SidebarMenu>
    </SidebarContent>
    <SidebarFooter>User info</SidebarFooter>
  </Sidebar>
  <main>{children}</main>
</SidebarProvider>
```

## Stepper for Multi-Step Flows

```typescript
import { Stepper, StepperItem, StepperTrigger, StepperIndicator,
  StepperTitle, StepperSeparator } from 'proqio-ui';

<Stepper value={currentStep} onValueChange={setCurrentStep}>
  <StepperItem value={1} status={step1Status}>
    <StepperTrigger>
      <StepperIndicator />
      <StepperTitle>Connect account</StepperTitle>
    </StepperTrigger>
    <StepperSeparator />
  </StepperItem>
  <StepperItem value={2} status={step2Status}>
    <StepperTrigger>
      <StepperIndicator />
      <StepperTitle>Configure scan</StepperTitle>
    </StepperTrigger>
  </StepperItem>
</Stepper>
```

## EmptyState Pattern

```typescript
import { EmptyState, EmptyStateIcon, EmptyStateTitle, EmptyStateDescription,
  EmptyStateAction, Button } from 'proqio-ui';

<EmptyState variant="dashed">
  <EmptyStateIcon><CloudIcon /></EmptyStateIcon>
  <EmptyStateTitle>No findings</EmptyStateTitle>
  <EmptyStateDescription>Run a scan to see security findings.</EmptyStateDescription>
  <EmptyStateAction>
    <Button variant="primary">Run scan</Button>
  </EmptyStateAction>
</EmptyState>
```

## Callout for Inline Alerts

```typescript
import { Callout } from 'proqio-ui';

<Callout variant="warning">
  MFA is not enabled for this account.
</Callout>

<Callout variant="danger">
  Critical vulnerability detected.
</Callout>
```

## asChild Pattern

Many components accept `asChild` to render as a child element instead of the default element:

```typescript
// ✅ DialogTrigger renders as Button (no nested button)
<DialogTrigger asChild>
  <Button variant="outline">Open dialog</Button>
</DialogTrigger>

// ✅ SidebarMenuButton renders as link
<SidebarMenuButton asChild>
  <a href="/settings">Settings</a>
</SidebarMenuButton>
```

## CSS / Theming Setup

proqio-ui with Tailwind 4 requires **two things in `src/index.css`** and nothing else:

```css
/* src/index.css */
@import 'tailwindcss';
@import 'proqio-ui/theme.css'; /* CSS custom properties / design tokens */

@source '../node_modules/proqio-ui/dist'; /* Tailwind scans proqio-ui classes */
```

**Why each line matters:**

| Line                                       | Purpose                                                                         | What breaks without it                                          |
| ------------------------------------------ | ------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `@import 'proqio-ui/theme.css'`            | Loads all CSS variables (`--tag-brand-*`, `--light-surface-*`, semantic tokens) | Components render without colors/spacing                        |
| `@source '../node_modules/proqio-ui/dist'` | Tells Tailwind 4 to include classes used inside proqio-ui                       | Tailwind purges proqio-ui classes → broken styles in production |

**Valid CSS exports from proqio-ui:**

```
proqio-ui/theme.css  → dist/theme.css  ✅ (design tokens, 88KB)
proqio-ui/           → dist/proqio-ui.js  ✅ (JS bundle with components)
proqio-ui/style.css  → ❌ DOES NOT EXIST (no such export)
```

**DO NOT** import `proqio-ui/style.css` — that file does not exist in the package exports. Component styles come from Tailwind generating CSS based on the classes found via `@source`.

### Dark Mode

proqio-ui uses the `.dark` class for dark mode. Use Tailwind's `dark:` variant normally — they are compatible:

```typescript
<div className="bg-white dark:bg-slate-900">
  <Button variant="primary">Works in both modes</Button>
</div>
```
