---
name: frontend-ui-tailwind-standards
description: Standardized guidelines and patterns for Frontend Ui Tailwind Standards.
---

# Frontend Ui Tailwind Standards

## When to use this skill
- Using Tailwind CSS
- Establishing Tailwind conventions
- Code reviews for Tailwind usage

## Workflow
- [ ] Use utility classes directly
- [ ] Follow mobile-first responsive design
- [ ] Extract components for repeated patterns
- [ ] Use @apply sparingly
- [ ] Configure design tokens in tailwind.config

## Instructions

### Class Ordering
```typescript
// Recommended order:
// Layout → Display → Positioning → Box Model → Typography → Visual → Misc
<div className="
  flex flex-col
  items-center justify-between
  w-full max-w-md
  px-4 py-2
  text-lg font-bold
  bg-white dark:bg-gray-800
  rounded-lg shadow-md
  hover:shadow-lg transition-shadow
"/>
```

### Responsive Design
```typescript
// Mobile first
<div className="
  text-sm    
  md:text-base
  lg:text-lg
  xl:text-xl
"/>
```

### Dark Mode
```typescript
<div className="bg-white dark:bg-gray-900">
  <h1 className="text-gray-900 dark:text-white">Title</h1>
</div>
```

## Resources
- Mobile-first breakpoints
- Use design tokens from config
- Avoid @apply except for base styles
