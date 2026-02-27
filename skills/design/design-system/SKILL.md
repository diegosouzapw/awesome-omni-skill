---
name: design-system
description: design system with Tailwind v4.0, accessibility patterns, and project-specific UI/UX rules. Use for all KKOOKK frontend development.
---

# Design System

## When to Use

- Styling KKOOKK components with Tailwind CSS v4.0
- Implementing KKOOKK-specific layouts and interactions
- Adding accessibility (a11y) features
- Building mobile-optimized web UI

---

## Core Design Principles

1. **Tactile Certainty (물리적 확신)**: Mimic physical stamp sensation through visual feedback
2. **Paperless Freedom (지갑의 자유)**: Light, mobile-first, no-install PWA experience
3. **Human Bridge (연결의 가치)**: Warm digital connection between owners and customers

---

## UI Component Library
- **Base**: [shadcn/ui](https://ui.shadcn.com/) (Radix UI primitives + Tailwind)
- Install components as needed: `npx shadcn@latest add button dialog ...`
- Components copied to `src/components/ui/` — customize freely
- **Do NOT** install other UI libraries without approval

### When to Use shadcn/ui

| Use shadcn/ui | Build from scratch |
|---------------|-------------------|
| Dialog, Modal, Sheet | Simple presentational components |
| Dropdown, Select, Combobox | KKOOKK-specific stamp card UI |
| Toast, Alert | Custom animations with Framer Motion |
| Form inputs (with validation) | Layout components |

---

## Tailwind CSS v4.0

### Configuration

**All custom colors defined in `frontend/src/assets/styles/index.css` using `@theme`**

Quick reference:
- Primary CTA: `bg-kkookk-orange-500` (#FF4D00)
- Owner: `bg-kkookk-indigo`, `text-kkookk-steel`
- Customer: `bg-kkookk-sand`, `text-kkookk-yellow`
- Background: `bg-kkookk-paper` (#FAF9F6)
- Text: `text-kkookk-navy` (#1A1C1E)

### shadcn/ui vs KKOOKK Colors

| Context | Use | Example |
|---------|-----|---------|
| shadcn components | `bg-primary`, `bg-destructive` | Dialog, Alert from shadcn |
| KKOOKK custom | `bg-kkookk-orange-500` | StampCard, custom buttons |
| Buttons | `.btn-primary` from `index.css` | CTA buttons |

**Why**: shadcn uses CSS variables (`--primary`), custom components use direct color classes.

### Class Organization Order

1. Layout (flex/grid, position)
2. Spacing (p/m/gap)
3. Size (w/h)
4. Typography (text-*, font-*)
5. Colors (bg-*, text-*, border-*)
6. Effects (shadow, rounded, opacity)
7. States (hover:, focus:, active:, disabled:)

### Conditional Classes

```tsx
import { twMerge } from 'tailwind-merge';

<div className={twMerge(
    "flex items-center p-4 rounded-xl",
    isActive && "bg-kkookk-orange-500 text-white",
    isDisabled && "opacity-50 cursor-not-allowed"
)}>
```

### Pattern Extraction

If a class combination appears **3+ times**, extract it to:
- A reusable component, OR
- A utility in `@layer utilities` (see `.btn-primary` in `index.css`)

---

## Color System

### Color Usage

- **Primary CTA**: `bg-kkookk-orange-500` (#FF4D00)
- **Owner UI**: Indigo (#2E58FF) + Steel Gray (#64748B) - professional tone
- **Customer UI**: Sand (#F5F5F0) + Yellow (#FFD600) - warm, friendly
- **Error**: `text-kkookk-red` (#DC2626)
- **Warning**: `text-kkookk-amber` (#F59E0B)

### WCAG AAA Compliance

- Minimum 7:1 contrast ratio
- Navy Black on Paper White: 14.8:1 ✅
- Test under 400+ lux (outdoor)

---

## Typography

### Type Scale

| Category | Class | Size | Weight | Usage |
|----------|-------|------|--------|-------|
| Display Hero | `text-6xl` | 60px | ExtraBold (800) | Stamp count |
| Display Sub | `text-4xl` | 36px | ExtraBold (800) | Rewards |
| Heading 1 | `text-2xl` | 24px | SemiBold (600) | Page title |
| Heading 2 | `text-xl` | 20px | SemiBold (600) | Sections |
| Body 1 | `text-base` | 16px | Medium (500) | Main text |
| Body 2 | `text-sm` | 14px | Regular (400) | Supporting |
| Caption | `text-xs` | 12px | Regular (400) | Micro-copy |

Font: Pretendard Variable

---

## Components

### KKOOKK Stamp Card

```tsx
<div className="p-4 rounded-2xl shadow-lg bg-white">
```

### Buttons

Use utility classes from `index.css`:
- `.btn-primary` - Orange CTA
- `.btn-secondary` - Indigo
- `.btn-outline`, `.btn-ghost`

Requirements:
- Height: `h-14` (56px) - prevents accidental taps
- Radius: `rounded-2xl` (16px)
- Active: `active:scale-95` - tactile feedback
- **Anti-double-tap**: Disable 300ms after click

```tsx
const [isProcessing, setIsProcessing] = useState(false);

const handleClick = async () => {
  if (isProcessing) return;
  setIsProcessing(true);
  try {
    await action();
  } finally {
    setTimeout(() => setIsProcessing(false), 300);
  }
};
```

---

## Accessibility

### Minimum Requirements

| Requirement | Implementation | KKOOKK Standard |
|-------------|----------------|-----------------|
| Labels | `<label htmlFor="id">` | All inputs |
| Focus rings | `focus:ring-4 focus:ring-kkookk-orange-500/50` | All interactive |
| ARIA | `aria-label`, `aria-modal` | Dialogs, icons |
| Touch targets | WCAG 44px minimum | **56px (`h-14`)** |
| Contrast | WCAG AAA 7:1 | All text |

### Focus States

```tsx
<button className="focus:outline-none focus:ring-4 focus:ring-kkookk-orange-500/50 focus:ring-offset-2">
```

### Form Labels

```tsx
<label htmlFor="store-name" className="text-sm font-medium">매장 이름</label>
<input id="store-name" aria-required="true" aria-invalid={!!errors.storeName} />
{errors.storeName && <p id="store-name-error" className="text-sm text-kkookk-red">{errors.storeName.message}</p>}
```

### Icon Buttons

```tsx
<button aria-label="닫기"><CloseIcon /></button>
```

### Keyboard Navigation

- Tab order must be logical
- Escape closes modals
- Enter/Space activates buttons

---

## Animations

### Signature: Stamp Impact

```tsx
<motion.div animate={{ scale: [0.8, 1.1, 1.0] }} transition={{ duration: 0.2 }}>
```

### Page Transitions

```tsx
<motion.div initial={{ x: '100%' }} animate={{ x: 0 }} exit={{ x: '100%' }}>
```

### Modal Slide-up

```tsx
<motion.div initial={{ y: '100%' }} animate={{ y: 0 }} className="fixed inset-x-0 bottom-0 rounded-t-3xl">
```

### Polling Pulse

```tsx
<motion.div animate={{ scale: [1, 1.05, 1], opacity: [1, 0.8, 1] }} transition={{ duration: 1.5, repeat: Infinity }}>
```

### State Matrix

| State | Classes | Usage |
|-------|---------|-------|
| Default | `shadow-md rounded-2xl` | Idle |
| Hover | `hover:shadow-lg` | Desktop |
| Active | `active:scale-95` | Touch feedback |
| Loading | `animate-pulse opacity-70` | Processing |
| Disabled | `opacity-50 cursor-not-allowed` | Unmet conditions |

---

## Page Type Guidelines

### Owner Pages (백오피스)

- **Tone**: Professional, efficient
- **Colors**: Indigo, Steel Gray
- **Layout**: Desktop-first, 2-3 column grid
- **CTA**: Top-right (+ New)

```tsx
<div className="flex items-center justify-between mb-6">
  <h1 className="text-2xl font-semibold">Title</h1>
  <button className="btn-secondary">+ New</button>
</div>
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
```

### Customer Pages (지갑)

- **Tone**: Warm, encouraging
- **Colors**: Orange, Sand, Yellow
- **Layout**: Mobile-first, single column
- **CTA**: Fixed bottom, full width

```tsx
<div className="px-4 pt-8 pb-6 text-center">
  <p className="text-6xl font-extrabold text-kkookk-orange-500">12</p>
  <p className="text-base text-kkookk-steel mt-2">개 적립</p>
</div>
<div className="fixed bottom-0 inset-x-0 p-4 bg-white border-t">
  <button className="btn-primary w-full">Action</button>
</div>
```

### Terminal Pages (단말)

- **Tone**: Clear, fast decisions
- **Colors**: White background, Orange accent
- **Layout**: Center-aligned, large elements
- **CTA**: Approval/Reject side-by-side

```tsx
<div className="min-h-screen bg-white flex flex-col items-center justify-center p-6">
  <div className="flex gap-4">
    <button className="h-14 px-8 bg-kkookk-steel text-white">거절</button>
    <button className="h-14 px-8 bg-kkookk-orange-500 text-white">승인</button>
  </div>
</div>
```

---

## Edge Cases

### Empty State

```tsx
<div className="flex flex-col items-center gap-4 py-8 text-center">
  <EmptyIcon className="w-16 h-16 text-slate-400" />
  <p className="text-base">아직 스탬프가 없어요</p>
  <button className="btn-primary">첫 도장 모으러 가기</button>
</div>
```

### Error State

```tsx
<div className="flex flex-col items-center gap-4 py-8">
  <ErrorIcon className="w-16 h-16 text-kkookk-red" />
  <p className="text-base">네트워크 연결을 확인해주세요</p>
  <button className="btn-primary" onClick={retry}>다시 시도</button>
</div>
```

---

## Mobile Optimization

- **Outdoor visibility**: High contrast (Navy on Paper = 14.8:1)
- **Touch targets**: Minimum 56px (`h-14`)
- **Spacing**: Minimum 8px between targets (`gap-2`)
- **Safe areas**: `px-4` horizontal margin
- **Mobile-first**: Design for 375px, enhance for larger

---

## Instructions for Claude

1. **Always check `index.css`** for colors and utilities
2. Apply class organization order for every component
3. Use `twMerge` for conditional classes
4. Extract patterns after 3+ repetitions
5. **Every CTA needs anti-double-tap** (300ms disable)
6. **All interactive elements need focus rings**
7. **Mobile-first**: 375px base, enhance with `md:`, `lg:`
8. If violates accessibility, suggest compliant alternative
9. **When creating pages**: Match page type (Owner/Customer/Terminal) guidelines
10. **When adding animations**: Use Framer Motion for complex, CSS classes for simple