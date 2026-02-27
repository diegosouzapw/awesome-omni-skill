---
name: icons
description: SeekSaaS icon system guide covering @workspace/custom-icons package with Icons and AiIcons components
---

# SeekSaaS Icons Guide

## Overview

SeekSaaS provides a comprehensive icon system through the `@workspace/custom-icons` package. This package includes:
- **Icons**: General-purpose icons for frameworks, tools, and platforms
- **AiIcons**: AI-specific icons for AI providers and tools

## Package Structure

```
packages/custom-icons/
├── src/
│   ├── index.tsx              # Main exports
│   ├── custom/                # General icons
│   │   └── index.tsx         # Icons object (100+ icons)
│   └── ai-icons/              # AI-specific icons
│       └── index.tsx         # AiIcons object
├── package.json
└── tsconfig.json
```

## Installation

The package is already available in the monorepo. Import it into your feature or web app:

```bash
# Already available via workspace protocol
pnpm add @workspace/custom-icons
```

## Usage

### Basic Import

```tsx
import { Icons } from "@workspace/custom-icons";
import { AiIcons } from "@workspace/custom-icons/ai-icons";
```

### Using Icons

```tsx
import { Icons } from "@workspace/custom-icons";

export function MyComponent() {
  return (
    <div>
      <Icons.react size={24} />
      <Icons.nextJS size={24} />
      <Icons.gitHub size={24} />
      <Icons.lucide size={24} />
    </div>
  );
}
```

### Using AiIcons

```tsx
import { AiIcons } from "@workspace/custom-icons/ai-icons";

export function AIComponent() {
  return (
    <div>
      <AiIcons.openai size={24} />
      <AiIcons.claude size={24} />
      <AiIcons.cursor size={24} />
    </div>
  );
}
```

### Custom Props

Most icons accept SVG props:

```tsx
<Icons.react 
  size={48}
  className="text-blue-500"
  onClick={() => console.log("clicked")}
/>

<AiIcons.cursor 
  width={64}
  height={64}
  style={{ color: "#000" }}
/>
```

## Icons Collection

### Framework Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.nextJS` | Next.js |
|  | `Icons.nuxt` | Nuxt.js |
|  | `Icons.svelteKit` | SvelteKit |
|  | `Icons.solidStart` | SolidStart |
|  | `Icons.react` | React |
|  | `Icons.vue` | Vue.js |
|  | `Icons.astro` | Astro |
|  | `Icons.remix` | Remix |
|  | `Icons.reactRouter` | React Router |
|  | `Icons.expo` | Expo |
|  | `Icons.hono` | Hono |
|  | `Icons.fastify` | Fastify |
|  | `Icons.express` | Express |
|  | `Icons.elysia` | Elysia |
|  | `Icons.node` | Node.js |

### Database & ORM Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.prisma` | Prisma |
|  | `Icons.drizzle` | Drizzle ORM |
|  | `Icons.tanstack` | TanStack |

### Tooling Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.biomejs` | Biome |
|  | `Icons.turborepo` | Turborepo |
|  | `Icons.pnpm` | pnpm |
|  | `Icons.yarn` | Yarn |
|  | `Icons.npm` | npm |
|  | `Icons.vite` | Vite |
|  | `Icons.vercel` | Vercel |

### UI Framework Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.shadcn` | shadcn/ui |
|  | `Icons.radix` | Radix UI |
|  | `Icons.tailwind` | Tailwind CSS |
|  | `Icons.tailark` | TailArk |
|  | `Icons.magicui` | Magic UI |

### Auth & Payment Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.betterAuth` | Better Auth |
|  | `Icons.stripe` | Stripe |
|  | `Icons.paypal` | PayPal |
|  | `Icons.alipay` | Alipay |
|  | `Icons.wechatpay` | WeChat Pay |

### Cloud & Deployment Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.cloudflare` | Cloudflare |
|  | `Icons.aws` | AWS |
|  | `Icons.docker` | Docker |

### Social Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.gitHub` | GitHub |
|  | `Icons.x` | Twitter/X |
|  | `Icons.linkedIn` | LinkedIn |
|  | `Icons.vk` | VK |
|  | `Icons.discord` | Discord |
|  | `Icons.youtube` | YouTube |
|  | `Icons.google` | Google |
|  | `Icons.apple` | Apple |

### Language Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.ts` | TypeScript |
|  | `Icons.javascript` | JavaScript |
|  | `Icons.css` | CSS |
|  | `Icons.json` | JSON |
|  | `Icons.bash` | Bash |

### Animation Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.motion` | Framer Motion |

### Documentation Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.fumadocs` | Fumadocs |
|  | `Icons.book` | Generic book |
|  | `Icons.aria` | ARIA |

### Other Icons

| Icon | Name | Usage |
|------|------|-------|
|  | `Icons.lucide` | Lucide |
|  | `Icons.v0` | v0 |
|  | `Icons.creem` | Creem |
|  | `Icons.nitro` | Nitro |
|  | `Icons.inlang` | inlang |
|  | `Icons.i18next` | i18next |

## AiIcons Collection

### AI IDEs & Editors

| Icon | Name | Usage |
|------|------|-------|
|  | `AiIcons.cursor` | Cursor IDE |
|  | `AiIcons.vscode` | VS Code |
|  | `AiIcons.copilot` | GitHub Copilot |
|  | `AiIcons.zed` | Zed Editor |
|  | `AiIcons.windsurf` | WindSurf |

### AI Providers

| Icon | Name | Usage |
|------|------|-------|
|  | `AiIcons.openai` | OpenAI (ChatGPT) |
|  | `AiIcons.claude` | Claude (Anthropic) |
|  | `AiIcons.gemini` | Gemini (Google) |
|  | `AiIcons.deepseek` | DeepSeek |
|  | `AiIcons.xai` | xAI (Grok) |

### AI Concepts

| Icon | Name | Usage |
|------|------|-------|
|  | `AiIcons.vibe` | Vibe coding |
|  | `AiIcons.mcp` | MCP (Model Context Protocol) |

### OpenCode

| Icon | Name | Usage |
|------|------|-------|
|  | `AiIcons.opencode` | OpenCode |

## Using with Tailwind CSS

Combine icons with Tailwind CSS classes:

```tsx
import { Icons } from "@workspace/custom-icons";

export function IconButton({ icon, label }: { icon: keyof typeof Icons; label: string }) {
  const IconComponent = Icons[icon];

  return (
    <button className="flex items-center gap-2 px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition-colors">
      <IconComponent size={20} />
      <span>{label}</span>
    </button>
  );
}
```

### Color Customization

```tsx
// Using className for colors
<Icons.react className="text-blue-500" size={32} />
<Icons.gitHub className="text-gray-900 dark:text-white" size={24} />

// Using style prop
<Icons.nextJS style={{ color: "#000000" }} size={32} />
```

### Size Customization

```tsx
<Icons.lucide size={16} />  // Small
<Icons.lucide size={24} />  // Medium (default)
<Icons.lucide size={32} />  // Large
<Icons.lucide size={48} />  // Extra large
```

## Creating New Icons

### Adding to Icons Object

Edit `packages/custom-icons/src/custom/index.tsx`:

```tsx
import { SiNewTool } from "react-icons/si";

export const Icons = {
  // ... existing icons
  newTool: SiNewTool,
};
```

### Adding to AiIcons Object

Edit `packages/custom-icons/src/ai-icons/index.tsx`:

```tsx
import { SiNewAI } from "react-icons/si";

export const AiIcons = {
  // ... existing icons
  newAI: (props: SVGProps<SVGSVGElement>) => (
    <svg {...props}>
      <path d="..." fill="currentColor" />
    </svg>
  ),
};
```

### Creating Custom SVG Icons

```tsx
import { cn } from "@workspace/ui/lib/utils";
import type { SVGProps } from "react";

export const Icons = {
  customIcon: ({ className, ...props }: SVGProps<SVGSVGElement>) => (
    <svg
      viewBox="0 0 24 24"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      className={cn("size-4", className)}
      {...props}
    >
      <path d="M12 2L2 7l10 5 10-5-10-5z" fill="currentColor" />
      <path d="M2 17l10 5 10-5" stroke="currentColor" strokeWidth={2} />
      <path d="M2 12l10 5 10-5" stroke="currentColor" strokeWidth={2} />
    </svg>
  ),
};
```

## Icon Size Reference

Use consistent sizes across the application:

| Size | Usage | Example |
|------|-------|---------|
| 16 | Small (labels, badges) | `<Icons.icon size={16} />` |
| 20 | Buttons, small UI | `<Icons.icon size={20} />` |
| 24 | Standard (most common) | `<Icons.icon size={24} />` |
| 32 | Cards, sections | `<Icons.icon size={32} />` |
| 48 | Hero sections | `<Icons.icon size={48} />` |
| 64 | Feature highlights | `<Icons.icon size={64} />` |

## Accessibility

Icons should be accessible when used meaningfully:

```tsx
// Decorative icons (screen reader can skip)
<Icons.gitHub aria-hidden="true" />

// Interactive icons (with label)
<button aria-label="GitHub">
  <Icons.gitHub size={24} />
</button>

// Informational icons (with description)
<div role="img" aria-label="Status: Online" aria-description="Green checkmark indicating system is operational">
  <Icons.checkCircle className="text-green-500" size={24} />
</div>
```

## Performance Tips

### Tree Shaking

The icons are exported as an object, so importing the entire module is efficient. For best results:

```tsx
// Good - import what you need
import { Icons } from "@workspace/custom-icons";
const { react, vue } = Icons;

// Avoid - this still works but imports more
import * as AllIcons from "@workspace/custom-icons";
```

### Memoization for Frequently Used Icons

```tsx
import { memo } from "react";
import { Icons } from "@workspace/custom-icons";

const ReactIcon = memo(Icons.react);
const VueIcon = memo(Icons.vue);

export function IconGrid() {
  return (
    <div className="grid grid-cols-4 gap-4">
      <ReactIcon size={32} />
      <VueIcon size={32} />
    </div>
  );
}
```

## Common Patterns

### Icon Button

```tsx
import { Icons } from "@workspace/custom-icons";
import { Button } from "@workspace/ui/components/ui/button";
import { cn } from "@workspace/ui/lib/utils";

interface IconButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  icon: keyof typeof Icons;
  variant?: "default" | "ghost" | "outline";
  size?: "sm" | "md" | "lg";
}

export function IconButton({ icon, variant = "default", size = "md", className, ...props }: IconButtonProps) {
  const IconComponent = Icons[icon];
  const sizeMap = { sm: 16, md: 20, lg: 24 };

  return (
    <Button variant={variant} size="icon" className={cn("h-9 w-9", className)} {...props}>
      <IconComponent size={sizeMap[size]} />
    </Button>
  );
}
```

### Icon Badge

```tsx
import { Icons } from "@workspace/custom-icons";

export function IconBadge({ icon, label, color }: { icon: keyof typeof Icons; label: string; color: string }) {
  const IconComponent = Icons[icon];

  return (
    <span className={cn("inline-flex items-center gap-1 px-2 py-1 rounded-full text-xs font-medium", color)}>
      <IconComponent size={14} />
      {label}
    </span>
  );
}
```

### Icon Card

```tsx
import { Icons } from "@workspace/custom-icons";

export function TechCard({ name, icon, description }: { name: string; icon: keyof typeof Icons; description: string }) {
  const IconComponent = Icons[icon];

  return (
    <div className="p-4 border rounded-lg hover:shadow-lg transition-shadow">
      <IconComponent size={48} className="mb-3 text-blue-500" />
      <h3 className="font-semibold">{name}</h3>
      <p className="text-sm text-muted-foreground">{description}</p>
    </div>
  );
}
```

## TypeScript Support

Full TypeScript support with type inference:

```tsx
import { Icons, AiIcons } from "@workspace/custom-icons";

// Type-safe icon access
const iconName: keyof typeof Icons = "react";
const aiIconName: keyof typeof AiIcons = "claude";

// These will be typed correctly
const Icon = Icons[iconName];
const AiIcon = AiIcons[aiIconName];
```

## Dependencies

```json
{
  "dependencies": {
    "@workspace/ui": "workspace:*",
    "class-variance-authority": "^0.7.1",
    "react-icons": "^5.5.0"
  }
}
```

- **react-icons**: Provides the underlying icon components (Feather, Font Awesome, etc.)
- **@workspace/ui**: For utility functions like `cn()` for class name merging

## Resources

- [react-icons](https://react-icons.github.io/react-icons/)
- [SVG Props](https://react.dev/reference/react/Component)
- [Tailwind CSS](https://tailwindcss.com)
