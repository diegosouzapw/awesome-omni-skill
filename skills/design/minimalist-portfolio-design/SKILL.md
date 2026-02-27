---
name: minimalist-portfolio-design
description: Minimalist, typography-focused portfolio design system inspired by neo-brutalist and Swiss design principles. Emphasizes bold typography, generous whitespace, monochromatic color schemes, and elegant simplicity for developer/designer portfolios.
---

# Minimalist Portfolio Design Skill

This skill guides the creation of clean, typography-focused portfolio websites with a minimalist aesthetic. Based on neo-brutalist and Swiss design principles, this approach prioritizes readability, visual hierarchy, and professional presentation over decorative elements.

## Core Design Philosophy

**Design Principles:**
- **Typography as the hero**: Let type do the heavy lifting
- **Radical simplicity**: Remove everything that isn't essential
- **Generous whitespace**: Space is a design element, not emptiness
- **Monochromatic palette**: Black, white, and shades of gray
- **Grid-based precision**: Everything aligns to an invisible grid
- **No decoration**: Form follows function
- **Swiss design influence**: Clean, asymmetric layouts with clear hierarchy

**What This Style IS:**
- Bold, confident, professional
- Clean and uncluttered
- Typography-driven
- Timeless and elegant
- Easy to scan and navigate

**What This Style IS NOT:**
- Colorful or playful (no bright colors)
- Heavily animated (subtle interactions only)
- Cluttered or busy
- Decorative or ornamental
- Trendy or flashy


## Typography System

### Font Selection
**Primary Typeface Options** (Neo-Grotesk/Geometric Sans):
1. **Aeonik** (as shown in reference) - Neo-Grotesk, geometric, modern
2. **Inter** - Versatile, excellent screen readability, free
3. **Satoshi** - Geometric, clean, contemporary feel
4. **Supreme** - Bold, geometric, great for large display text
5. **Space Grotesk** - Modern geometric with personality
6. **Neue Montreal** - Swiss-inspired, clean lines
7. **General Sans** - Balanced, professional, versatile

**System Font Stack (fallback):**
```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', sans-serif;
```

### Type Scale & Hierarchy

**Display/Hero Text (Large greeting like "Hello"):**
- Desktop: 8rem-12rem (128px-192px)
- Tablet: 6rem-8rem (96px-128px)
- Mobile: 4rem-6rem (64px-96px)
- Weight: 400-500 (Regular to Medium)
- Line height: 0.9-1.0 (tight)
- Letter spacing: -0.02em to -0.04em (tight)

**Section Headers (Like "Colors", "Typography"):**
- Desktop: 4rem-6rem (64px-96px)
- Tablet: 3rem-4rem (48px-64px)
- Mobile: 2.5rem-3rem (40px-48px)
- Weight: 400-500
- Line height: 1.1
- Letter spacing: -0.02em

**Subheadings/Labels (Like "Primary", "Secondary"):**
- Size: 1.25rem-1.5rem (20px-24px)
- Weight: 400-500
- Line height: 1.4
- Letter spacing: 0

**Body Text (Like the description under "Colors"):**
- Size: 1rem-1.125rem (16px-18px)
- Weight: 400
- Line height: 1.6-1.7
- Letter spacing: 0
- Max width: 65ch (characters) for readability

**Small Text/Labels (Like "Scroll down ↓"):**
- Size: 0.875rem-1rem (14px-16px)
- Weight: 400
- Line height: 1.5
- Letter spacing: 0.01em

**Navigation Links:**
- Size: 1rem-1.125rem (16px-18px)
- Weight: 400-500
- Letter spacing: 0

### Typography Rules
- **One font family** for the entire site (consistency over variety)
- **Use font weight** to create hierarchy, not multiple fonts
- **Tight letter spacing** for large display text
- **Standard spacing** for body text
- **Generous line height** (1.6-1.7) for readable body text
- **Tight line height** (0.9-1.1) for display text
- **Left-aligned** text (not centered, except for specific hero elements)
- **No all-caps** for body text (only for small labels if needed)


## Color System

### Monochromatic Palette (Primary Approach)

This is the CORE aesthetic - black, white, and grays only.

**Color Palette:**
```css
--color-black: #222222;        /* Primary - rich black, not pure black */
--color-gray-dark: #7B7B7B;    /* Secondary - medium gray */
--color-gray-light: #F8F8F8;   /* Tertiary - off-white background */
--color-white: #FFFFFF;        /* Pure white */
--color-gray-50: #FAFAFA;      /* Lightest gray */
--color-gray-100: #F5F5F5;     /* Very light gray */
--color-gray-200: #EEEEEE;     /* Light gray */
--color-gray-300: #E0E0E0;     /* Border gray */
--color-gray-400: #BDBDBD;     /* Disabled state */
--color-gray-500: #9E9E9E;     /* Mid gray */
--color-gray-600: #757575;     /* Text gray */
--color-gray-700: #616161;     /* Dark text gray */
--color-gray-800: #424242;     /* Almost black */
--color-gray-900: #212121;     /* Darkest gray */
```

**Color Usage:**
- **Background**: #F8F8F8 or #FFFFFF (light mode) / #0A0A0A (dark mode)
- **Text primary**: #222222 (light mode) / #F8F8F8 (dark mode)
- **Text secondary**: #7B7B7B (both modes)
- **Borders**: #E0E0E0 (light mode) / #2A2A2A (dark mode)
- **Cards/Surfaces**: #FFFFFF (light mode) / #1A1A1A (dark mode)
- **Hover states**: Slight darkening/lightening (10-15% opacity change)

### Accent Color Option (Minimal Use)

**Only if absolutely necessary** - use ONE accent color sparingly:
- For CTAs only (Book A Call button)
- For hover states on key elements
- For active link indicators
- **Never** for backgrounds or large areas

**Suggested Accent Colors** (choose ONE):
- **Tech Blue**: #0066FF (IBM blue, professional)
- **Success Green**: #00B386 (subtle, trustworthy)
- **Warm Red**: #FF3B30 (Apple red, attention-grabbing)
- **Keep it black**: No accent color at all (purest approach)

### Color Application Rules

1. **Light Mode Default**: White/off-white background (#F8F8F8 to #FFFFFF)
2. **Dark Mode Option**: Dark gray/black background (#0A0A0A to #121212)
3. **Text contrast**: Minimum 4.5:1 ratio for body text, 3:1 for large text
4. **No gradients**: Solid colors only
5. **No shadows** (or very subtle 1-2px shadows): Use borders instead
6. **Hover states**: Opacity changes (0.7-0.8) or background color shifts
7. **Photography**: Black and white or desaturated (like the reference portrait)

### Dark Mode Considerations

If implementing dark mode:
- Background: #0A0A0A to #121212 (not pure black)
- Surfaces: #1A1A1A to #222222
- Text: #F8F8F8 to #FFFFFF
- Borders: #2A2A2A to #333333
- Keep the same monochromatic approach
- Avoid pure white on pure black (causes eye strain)


## Layout & Spacing System

### Grid System

**Container Widths:**
- Max content width: 1440px-1600px
- Content padding: 5-10% on sides (responsive)
- Mobile padding: 24px-32px on sides

**Grid Structure:**
- 12-column grid for flexibility
- For hero: Asymmetric 60/40 or 50/50 split (text/image)
- Generous gutters: 32px-48px between columns

### Spacing Scale (8pt Grid)

All spacing in multiples of 8:
```
4px (0.25rem)   - Minimal gap
8px (0.5rem)    - Tight spacing
16px (1rem)     - Small spacing
24px (1.5rem)   - Medium spacing
32px (2rem)     - Standard spacing
48px (3rem)     - Large spacing
64px (4rem)     - XL spacing
96px (6rem)     - Section spacing (mobile)
128px (8rem)    - Section spacing (tablet)
160px (10rem)   - Section spacing (desktop)
192px (12rem)   - Hero spacing
```

### Vertical Rhythm

**Between sections:**
- Mobile: 96px-128px
- Tablet: 128px-160px
- Desktop: 160px-192px

**Within sections:**
- Between heading and content: 32px-48px
- Between paragraphs: 24px-32px
- Between list items: 16px-24px

**Component spacing:**
- Navigation padding: 24px-32px vertical
- Card padding: 32px-48px internal
- Button padding: 16px-32px horizontal, 12px-16px vertical

### Responsive Breakpoints

```css
/* Mobile-first approach */
mobile: 320px-640px
tablet: 641px-1024px
desktop: 1025px-1440px
xl: 1441px+

/* Tailwind equivalent */
sm: 640px
md: 768px
lg: 1024px
xl: 1280px
2xl: 1536px
```

### Layout Patterns for Portfolio

#### Hero Section (Full Viewport)
```
Layout: 50/50 split (text left, image right)
Height: 100vh (full viewport)
Vertical alignment: center

Left side:
- Large greeting text: "Hello"
- Subtitle: "— It's [Name] a [role]"
- Stats (optional): "+200 Project completed"
- Scroll indicator: "Scroll down ↓"
- Vertical text (optional): "Product designer"

Right side:
- Large portrait photo
- Black and white or desaturated
- Professional, confident pose
- No background (or subtle gradient)
```

**Navigation:**
- Position: Absolute top or fixed top
- Height: 80px-100px
- Logo: Top left (24px-32px from edges)
- Links: Top center or right
- CTA: Top right ("Book A Call" with arrow)
- Background: Transparent or subtle backdrop-blur on scroll
- Border: None or 1px bottom border (#E0E0E0)

#### About/Info Sections
```
Layout: Single column, centered content
Max width: 800px-900px for readability
Heading: Left-aligned, large
Body: Left-aligned, comfortable line length
```

#### Portfolio Grid
```
Desktop: 3 columns
Tablet: 2 columns  
Mobile: 1 column
Gap: 32px-48px
Card aspect ratio: 4:3 or 16:9
```

### Whitespace Philosophy

**More space = More premium**

- Empty space is a design element
- Don't fill every pixel
- Let content breathe
- Use whitespace to guide the eye
- Balance is key: Not too sparse, not too crowded

**Whitespace Checklist:**
- [ ] Is there breathing room around all elements?
- [ ] Can the eye easily identify sections?
- [ ] Does the layout feel calm and organized?
- [ ] Are related items grouped with less space?
- [ ] Are unrelated items separated with more space?


## Component Design Patterns

### Navigation Bar

**Structure:**
```
[Logo]                    [About Me] [Portfolio] [Services] [Blog]                    [Book A Call →]
```

**Specifications:**
- Height: 80px-100px
- Position: Fixed top with backdrop-filter blur OR absolute
- Background: Transparent → Semi-transparent on scroll
- Logo: Simple wordmark or icon (24px-32px size)
- Links: 16px-18px, Medium weight
- Link spacing: 32px-48px apart
- CTA button: Outlined or filled, right-aligned
- Hover: Opacity 0.7 or subtle underline (1-2px)

**Mobile Navigation:**
- Hamburger menu: Simple 3-line icon (24px)
- Menu: Full-screen overlay OR slide-in drawer
- Links: Larger text (24px-32px), vertical stack
- Animation: Smooth 200-300ms ease

### Buttons

**Primary Button (CTA):**
```css
padding: 14px-16px vertical, 28px-32px horizontal
border-radius: 4px-8px (subtle, not pill)
font-size: 16px-18px
font-weight: 500
background: #222222 (or accent color)
color: #FFFFFF
border: none OR 1-2px solid
hover: opacity: 0.9 or transform: translateY(-2px)
```

**Secondary Button:**
```css
Same sizing as primary
background: transparent
color: #222222
border: 1-2px solid #222222
hover: background: #222222, color: #FFFFFF
```

**Text Link:**
```css
No background, no border
Underline: 1px solid (only on hover) OR always present
Hover: opacity: 0.7
```

### Cards (Project Cards)

**Structure:**
```
┌──────────────────────────┐
│                          │
│      Project Image       │
│       (16:9 ratio)       │
│                          │
├──────────────────────────┤
│  Project Title           │
│  Brief description...    │
│  [Tag] [Tag] [Tag]       │
└──────────────────────────┘
```

**Specifications:**
- Background: #FFFFFF or #FAFAFA
- Border: 1px solid #E0E0E0 OR subtle shadow
- Border-radius: 8px-12px (subtle)
- Padding: 0 (image edge-to-edge) + 24px-32px bottom
- Image: Fill width, fixed aspect ratio
- Hover: Scale 1.02 OR lift with shadow OR border color change
- Transition: 200-300ms ease

**Card Content:**
- Title: 20px-24px, Medium weight
- Description: 16px, Regular, 2-3 lines max
- Tags: 12px-14px, padding 6px-12px, rounded-full, border 1px

### Stats Display ("+200 Project completed")

**Style:**
```
Number: 3rem-4rem (48px-64px), Medium weight
Label: 1rem (16px), Regular weight, gray color
Layout: Vertical stack
Spacing: 8px between number and label
```

**Positioning:**
- Near hero text
- Subtle, not competing with main message
- Can be multiple stats side-by-side (32px-48px gap)

### Photography/Images

**Portrait Photo (Hero):**
- Black and white OR heavily desaturated
- Professional, direct gaze
- Clean background (solid color or subtle gradient)
- High quality: Sharp, well-lit
- Aspect ratio: Portrait (3:4) or square (1:1)
- Size: Large, fills 40-50% of viewport

**Project Images:**
- High quality: Mockups or screenshots
- Consistent aspect ratio across all projects
- Consider: 16:9 (landscape) or 4:3
- Hover: Slight zoom (scale 1.05) or no effect
- Alt text: Always include for accessibility

### Iconography

**Icon Style:**
- Minimal: Line icons only (no filled icons)
- Stroke weight: 1.5px-2px
- Size: 20px-24px standard, 32px-40px for featured
- Color: Match text color (#222222 or #7B7B7B)
- Examples: Arrow (→), Menu (≡), Close (×), Social icons

**Icon Usage:**
- Navigation: Hamburger menu, arrow for CTA
- Social links: GitHub, LinkedIn, Twitter/X, Email
- Arrows: For links, buttons, scroll indicators
- Keep it minimal: Don't overuse icons

### Forms (Contact Forms)

**Input Fields:**
```css
padding: 16px
border: 1px solid #E0E0E0
border-radius: 4px-8px
background: #FFFFFF or #FAFAFA
font-size: 16px
focus: border-color: #222222, outline: none
```

**Label:**
```css
font-size: 14px-16px
font-weight: 500
margin-bottom: 8px
color: #222222
```

**Layout:**
- Single column, full width
- Fields: Name, Email, Message (3 fields max)
- Spacing: 24px between fields
- Submit button: Full width or right-aligned

### Scroll Indicator

**Text Style:**
```
"Scroll down ↓"
font-size: 14px-16px
color: #7B7B7B
position: bottom center or bottom left
animation: subtle bounce (optional)
```

**Alternative:**
- Animated line/arrow
- Mouse icon with wheel animation
- Keep it subtle and minimal


## Portfolio Page Structure

### 1. Hero Section (Above the Fold)

**Purpose**: Immediate impact, clear identity

**Content:**
- Large greeting: "Hello" (or your variation)
- Name and role: "— It's [Name] a [role]"
- Optional stats: "+200 Project completed" "+50 Startup raised"
- Scroll indicator: "Scroll down ↓"
- Portrait photo: Right side, B&W, professional

**Layout:**
- Full viewport height (100vh)
- 50/50 split (text left, photo right) OR 60/40
- Vertically centered content
- White or off-white background
- Minimalist navigation at top

**Typography:**
- "Hello": 10rem-12rem desktop, 6rem mobile
- Subtitle: 1.25rem-1.5rem, gray color (#7B7B7B)
- Stats: 3rem-4rem numbers, 1rem labels

**Key Principles:**
- Less is more: Don't overcrowd
- Immediate clarity: Who you are, what you do
- Professional confidence: Bold but not aggressive
- Clear next action: Scroll or CTA button

### 2. About Section

**Purpose**: Personal connection, your story

**Content:**
- Section heading: "About" or "Who I Am"
- Your story: 2-3 paragraphs max
- What you do currently
- What drives you
- Optional: Skills/tools you use

**Layout:**
- Single column, centered
- Max width: 800px for readability
- Large section heading: 4rem-6rem
- Body text: 1.125rem, line-height 1.7
- Optional: Photo or no photo

**Tone:**
- Professional but human
- Authentic, not generic
- Confident without ego
- Clear and concise

### 3. Portfolio/Work Section

**Purpose**: Showcase your best work

**Content:**
- Section heading: "Work", "Projects", or "Selected Work"
- 4-8 projects (quality over quantity)
- Each project: Image, title, description, tech stack

**Layout:**
- Grid: 3 columns desktop, 2 tablet, 1 mobile
- OR: 2 columns (larger cards)
- OR: Bento grid (varied sizes for featured work)
- Generous gaps: 32px-48px

**Project Card:**
```
┌─────────────────────────┐
│                         │
│    Project Image        │
│    (Full width, 16:9)   │
│                         │
│  Project Title          │
│  Brief description of   │
│  what you built and why │
│  [React] [Next.js] [TS] │
│                         │
│  [View Project →]       │
└─────────────────────────┘
```

**Interaction:**
- Hover: Subtle scale or border change
- Click: Link to case study or live site
- Clean, minimal hover states

### 4. Services Section (Optional)

**Purpose**: What you offer clients/employers

**Content:**
- Section heading: "Services" or "What I Do"
- 3-5 service offerings
- Brief description for each

**Layout:**
- Grid: 2-3 columns OR single column list
- Each service: Icon (optional), title, description
- Clean, scannable

**Examples:**
- Web Development
- Mobile App Development
- UI/UX Design
- Consulting
- Custom Solutions

### 5. Skills/Tech Stack Section

**Purpose**: Quick scan of your technical abilities

**Content:**
- Section heading: "Skills" or "Tech Stack"
- Technologies you use
- Grouped by category (optional)

**Layout Options:**

**Option A: Simple List**
```
Frontend: React, Next.js, TypeScript, Tailwind
Backend: Node.js, Supabase, PostgreSQL
Mobile: React Native, Expo
Tools: Git, Figma, VS Code
```

**Option B: Icon Grid**
```
[React]  [Next.js]  [TypeScript]  [Tailwind]
[Node]   [Supabase] [PostgreSQL]  [Git]
[Figma]  [React Native]  [Expo]
```

**Option C: Categorized Cards**
```
┌──────────────────┐ ┌──────────────────┐
│ Frontend         │ │ Backend          │
│ • React          │ │ • Node.js        │
│ • Next.js        │ │ • Supabase       │
│ • TypeScript     │ │ • PostgreSQL     │
└──────────────────┘ └──────────────────┘
```

**Styling:**
- Monochromatic: Black text on white/gray background
- OR: Subtle gray tags with borders
- Consistent sizing and spacing
- No skill bars or percentage indicators

### 6. Contact Section

**Purpose**: Easy way to get in touch

**Content:**
- Section heading: "Get In Touch" or "Contact"
- Email link OR contact form (prefer simplicity)
- Social links: GitHub, LinkedIn, Twitter
- Optional: Calendar booking link

**Layout Options:**

**Option A: Minimal (Preferred)**
```
Let's work together

email@example.com
[GitHub] [LinkedIn] [Twitter]
```

**Option B: Simple Form**
```
Name: [________________]
Email: [________________]
Message: [____________]
         [____________]
         
[Send Message]
```

**Styling:**
- Centered content
- Large, clear email link
- Social icons: Line style, 24px-32px
- Button: Same style as primary CTA
- Generous spacing

### 7. Footer

**Purpose**: Supporting information, links

**Content:**
- Copyright: © 2024 Your Name
- Links: Privacy, Terms (if needed)
- Social links (if not in contact section)
- Optional: Back to top link

**Layout:**
- Single row, horizontal
- OR: Minimal vertical stack on mobile
- Small text: 14px
- Color: Gray (#7B7B7B)
- Padding: 32px-48px vertical

**Styling:**
- Clean and minimal
- No background color (or subtle gray)
- Border top: 1px solid #E0E0E0 (optional)

## Page Architecture Summary

```
┌────────────────────────────────────┐
│ Navigation (Fixed/Absolute Top)    │
├────────────────────────────────────┤
│                                    │
│ Hero Section (100vh)               │
│ • Large greeting                   │
│ • Your name + role                 │
│ • Portrait photo                   │
│                                    │
├────────────────────────────────────┤
│                                    │
│ About Section                      │
│ • Who you are                      │
│ • Your story                       │
│                                    │
├────────────────────────────────────┤
│                                    │
│ Work Section                       │
│ • Project grid                     │
│ • 4-8 projects                     │
│                                    │
├────────────────────────────────────┤
│                                    │
│ Skills Section                     │
│ • Tech stack                       │
│ • Tools                            │
│                                    │
├────────────────────────────────────┤
│                                    │
│ Contact Section                    │
│ • Email or form                    │
│ • Social links                     │
│                                    │
├────────────────────────────────────┤
│ Footer                             │
│ • Copyright + links                │
└────────────────────────────────────┘
```

**Spacing Between Sections:**
- Mobile: 96px-128px
- Desktop: 160px-192px


## Technical Implementation

### Next.js + TypeScript + Tailwind Setup

**Recommended Stack:**
- Next.js 14+ (App Router)
- TypeScript
- Tailwind CSS
- Framer Motion (for animations)
- next/font (for font optimization)

### Tailwind Configuration

```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        black: '#222222',
        'gray-dark': '#7B7B7B',
        'gray-light': '#F8F8F8',
        white: '#FFFFFF',
      },
      fontFamily: {
        sans: ['var(--font-inter)', 'system-ui', 'sans-serif'],
      },
      fontSize: {
        'display': ['10rem', { lineHeight: '0.9', letterSpacing: '-0.02em' }],
        'hero': ['6rem', { lineHeight: '1.0', letterSpacing: '-0.02em' }],
      },
      spacing: {
        '128': '32rem',
        '144': '36rem',
        '160': '40rem',
      },
    },
  },
  plugins: [],
}
export default config
```

### Font Setup (next/font)

```typescript
// app/layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
  display: 'swap',
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="font-sans bg-gray-light text-black antialiased">
        {children}
      </body>
    </html>
  )
}
```

### Hero Section Component Example

```typescript
// components/Hero.tsx
'use client'

import Image from 'next/image'
import { motion } from 'framer-motion'

export default function Hero() {
  return (
    <section className="h-screen flex items-center px-8 md:px-16 lg:px-20">
      <div className="max-w-[1440px] mx-auto w-full">
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-16 items-center">
          {/* Left: Text Content */}
          <motion.div
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            transition={{ duration: 0.6 }}
            className="space-y-8"
          >
            {/* Stats */}
            <div className="flex gap-12 text-sm">
              <div>
                <div className="text-4xl font-medium">+200</div>
                <div className="text-gray-dark mt-1">Project completed</div>
              </div>
              <div>
                <div className="text-4xl font-medium">+50</div>
                <div className="text-gray-dark mt-1">Startup raised</div>
              </div>
            </div>

            {/* Main Greeting */}
            <h1 className="text-hero md:text-display font-medium leading-none tracking-tight">
              Hello
            </h1>

            {/* Subtitle */}
            <p className="text-xl text-gray-dark">
              — It's Kanji, a web developer
            </p>

            {/* Scroll Indicator */}
            <div className="pt-16">
              <p className="text-sm text-gray-dark">Scroll down ↓</p>
            </div>
          </motion.div>

          {/* Right: Portrait Photo */}
          <motion.div
            initial={{ opacity: 0, scale: 0.95 }}
            animate={{ opacity: 1, scale: 1 }}
            transition={{ duration: 0.6, delay: 0.2 }}
            className="relative aspect-[3/4] max-w-xl mx-auto"
          >
            <Image
              src="/portrait.jpg"
              alt="Kanji - Web Developer"
              fill
              className="object-cover grayscale"
              priority
            />
          </motion.div>
        </div>
      </div>
    </section>
  )
}
```

### Navigation Component Example

```typescript
// components/Navigation.tsx
'use client'

import Link from 'next/link'
import { useState, useEffect } from 'react'

export default function Navigation() {
  const [scrolled, setScrolled] = useState(false)

  useEffect(() => {
    const handleScroll = () => {
      setScrolled(window.scrollY > 50)
    }
    window.addEventListener('scroll', handleScroll)
    return () => window.removeEventListener('scroll', handleScroll)
  }, [])

  return (
    <nav
      className={`fixed top-0 left-0 right-0 z-50 transition-all duration-300 ${
        scrolled
          ? 'bg-white/80 backdrop-blur-md border-b border-gray-light'
          : 'bg-transparent'
      }`}
    >
      <div className="max-w-[1440px] mx-auto px-8 md:px-16 lg:px-20">
        <div className="flex items-center justify-between h-20">
          {/* Logo */}
          <Link href="/" className="text-xl font-medium hover:opacity-70 transition-opacity">
            Kanji
          </Link>

          {/* Navigation Links */}
          <div className="hidden md:flex items-center gap-8">
            <Link href="#about" className="text-base hover:opacity-70 transition-opacity">
              About Me
            </Link>
            <Link href="#portfolio" className="text-base hover:opacity-70 transition-opacity">
              Portfolio
            </Link>
            <Link href="#services" className="text-base hover:opacity-70 transition-opacity">
              Services
            </Link>
            <Link href="#blog" className="text-base hover:opacity-70 transition-opacity">
              Blog
            </Link>
          </div>

          {/* CTA Button */}
          <Link
            href="#contact"
            className="border border-black px-6 py-2.5 rounded-lg hover:bg-black hover:text-white transition-all duration-300"
          >
            Book A Call →
          </Link>
        </div>
      </div>
    </nav>
  )
}
```

### Project Card Component

```typescript
// components/ProjectCard.tsx
import Image from 'next/image'
import Link from 'next/link'

interface ProjectCardProps {
  title: string
  description: string
  image: string
  tags: string[]
  link: string
}

export default function ProjectCard({ title, description, image, tags, link }: ProjectCardProps) {
  return (
    <Link href={link} className="group block">
      <div className="border border-gray-300 rounded-xl overflow-hidden transition-all duration-300 hover:border-black">
        {/* Image */}
        <div className="relative aspect-video bg-gray-light">
          <Image
            src={image}
            alt={title}
            fill
            className="object-cover transition-transform duration-300 group-hover:scale-105"
          />
        </div>

        {/* Content */}
        <div className="p-6 space-y-4">
          <h3 className="text-2xl font-medium">{title}</h3>
          <p className="text-gray-dark line-clamp-2">{description}</p>

          {/* Tags */}
          <div className="flex flex-wrap gap-2">
            {tags.map((tag) => (
              <span
                key={tag}
                className="text-sm px-3 py-1 border border-gray-300 rounded-full"
              >
                {tag}
              </span>
            ))}
          </div>
        </div>
      </div>
    </Link>
  )
}
```

### Performance Optimizations

**Image Optimization:**
```typescript
// Always use Next.js Image component
import Image from 'next/image'

// For external images
<Image
  src="/path-to-image.jpg"
  alt="Description"
  width={1200}
  height={800}
  quality={85}
  priority={false} // true for above-fold images
/>

// For dynamic images
<Image
  src={imageUrl}
  alt={altText}
  fill
  className="object-cover"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

**Font Optimization:**
```typescript
// Use next/font for automatic font optimization
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  preload: true,
})
```

**Code Splitting:**
```typescript
// Dynamic imports for heavy components
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
  ssr: false, // if not needed on server
})
```


## Animations & Interactions

### Animation Philosophy

**Principles:**
- Subtle over flashy
- Purposeful, not decorative
- Fast and smooth (200-400ms)
- Respect prefers-reduced-motion
- Enhance, don't distract

### Recommended Animations

**Page Load:**
```typescript
// Fade in + slight upward movement
initial={{ opacity: 0, y: 20 }}
animate={{ opacity: 1, y: 0 }}
transition={{ duration: 0.6, ease: 'easeOut' }}
```

**Hover States:**
```css
/* Subtle opacity change */
transition: opacity 200ms ease;
hover:opacity-70

/* Slight lift */
transition: transform 200ms ease;
hover:translate-y-[-2px]

/* Border color change */
transition: border-color 200ms ease;
hover:border-black
```

**Scroll Animations:**
```typescript
// Use Framer Motion's scroll-triggered animations
import { motion, useScroll, useTransform } from 'framer-motion'

const { scrollYProgress } = useScroll()
const opacity = useTransform(scrollYProgress, [0, 0.5], [1, 0])

<motion.div style={{ opacity }}>
  Content
</motion.div>
```

**Navigation Transitions:**
```css
/* Backdrop blur on scroll */
backdrop-filter: blur(8px);
transition: background-color 300ms ease, backdrop-filter 300ms ease;
```

### What NOT to Animate

- Don't animate on every scroll
- No auto-playing carousels
- No excessive parallax (causes motion sickness)
- No animations that delay content visibility
- No animations longer than 600ms

### Framer Motion Best Practices

```typescript
// Stagger children animations
<motion.div
  initial="hidden"
  animate="visible"
  variants={{
    hidden: { opacity: 0 },
    visible: {
      opacity: 1,
      transition: {
        staggerChildren: 0.1
      }
    }
  }}
>
  {items.map((item) => (
    <motion.div
      key={item.id}
      variants={{
        hidden: { opacity: 0, y: 20 },
        visible: { opacity: 1, y: 0 }
      }}
    >
      {item.content}
    </motion.div>
  ))}
</motion.div>
```

### Respecting User Preferences

```typescript
// Check for reduced motion preference
import { useReducedMotion } from 'framer-motion'

const shouldReduceMotion = useReducedMotion()

const variants = shouldReduceMotion
  ? { hidden: { opacity: 0 }, visible: { opacity: 1 } }
  : { hidden: { opacity: 0, y: 20 }, visible: { opacity: 1, y: 0 } }
```

## Accessibility Guidelines

### WCAG 2.1 AA Standards (Minimum)

**Color Contrast:**
- Body text: 4.5:1 minimum contrast ratio
- Large text (24px+): 3:1 minimum
- Use contrast checker tools: WebAIM, Stark

**Keyboard Navigation:**
```typescript
// Ensure all interactive elements are keyboard accessible
<button
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
  className="focus:outline-none focus:ring-2 focus:ring-black focus:ring-offset-2"
>
  Click Me
</button>
```

**Focus States:**
```css
/* Never remove focus outlines without replacement */
focus:outline-none /* Only if you provide alternative */
focus:ring-2 
focus:ring-black 
focus:ring-offset-2

/* Or use default browser focus */
/* Don't add: outline: none */
```

**Semantic HTML:**
```html
<!-- Use proper heading hierarchy -->
<h1>Main Title</h1>
  <h2>Section Title</h2>
    <h3>Subsection</h3>

<!-- Use semantic elements -->
<nav>...</nav>
<main>...</main>
<article>...</article>
<footer>...</footer>

<!-- Not divs for everything -->
```

**Alt Text for Images:**
```typescript
// Descriptive alt text
<Image
  src="/project-screenshot.jpg"
  alt="RCVR inventory management dashboard showing real-time stock levels"
  width={1200}
  height={800}
/>

// Decorative images
<Image
  src="/decorative-pattern.jpg"
  alt=""
  width={100}
  height={100}
  aria-hidden="true"
/>
```

**ARIA Labels:**
```typescript
// For icon-only buttons
<button aria-label="Open menu">
  <MenuIcon />
</button>

// For links with icons
<a href="https://github.com" aria-label="Visit my GitHub profile">
  <GitHubIcon />
</a>
```

**Skip to Content Link:**
```typescript
// Allow keyboard users to skip navigation
<a
  href="#main-content"
  className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 bg-black text-white px-4 py-2 rounded"
>
  Skip to main content
</a>

<main id="main-content">
  {/* Main content */}
</main>
```

### Responsive Design Considerations

**Mobile-First Approach:**
```css
/* Base styles (mobile) */
.heading {
  font-size: 2rem;
}

/* Tablet and up */
@media (min-width: 768px) {
  .heading {
    font-size: 3rem;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .heading {
    font-size: 4rem;
  }
}
```

**Touch Targets:**
- Minimum size: 44x44px
- Adequate spacing between interactive elements
- Easy to tap on mobile devices

**Tailwind Responsive Classes:**
```html
<div class="text-4xl md:text-6xl lg:text-8xl">
  Responsive Text
</div>

<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
  Cards
</div>
```

## Design Checklist

### Before Launching

**Visual Design:**
- [ ] Typography hierarchy is clear and consistent
- [ ] Color contrast meets WCAG AA standards (4.5:1 for text)
- [ ] Spacing follows 8pt grid system
- [ ] All elements are properly aligned
- [ ] Whitespace is generous and balanced
- [ ] Photography is high quality and professional
- [ ] No pixelated or low-res images
- [ ] Consistent border radius (8px-12px)
- [ ] Consistent hover states on all interactive elements

**Content:**
- [ ] Typos are fixed (use Grammarly or similar)
- [ ] All links work and open correctly
- [ ] Contact information is accurate
- [ ] Project descriptions are clear and concise
- [ ] About section tells your story authentically
- [ ] Skills/tech stack is current and accurate
- [ ] No lorem ipsum or placeholder text

**Technical:**
- [ ] Images are optimized (WebP format, proper sizing)
- [ ] Fonts are properly loaded (using next/font)
- [ ] Page loads in under 3 seconds
- [ ] All pages are responsive (mobile, tablet, desktop)
- [ ] Navigation works on mobile (hamburger menu)
- [ ] Forms validate inputs properly
- [ ] External links open in new tab (_blank + rel="noopener")
- [ ] Console has no errors
- [ ] Lighthouse score: 90+ on all metrics

**Accessibility:**
- [ ] All images have alt text
- [ ] Focus states are visible
- [ ] Keyboard navigation works throughout site
- [ ] Color contrast meets standards
- [ ] Heading hierarchy is semantic (h1 → h2 → h3)
- [ ] Skip to content link exists
- [ ] ARIA labels on icon buttons
- [ ] Forms have proper labels

**SEO:**
- [ ] Page titles are descriptive and unique
- [ ] Meta descriptions are compelling (150-160 chars)
- [ ] Open Graph tags for social sharing
- [ ] Favicon is present
- [ ] sitemap.xml exists
- [ ] robots.txt is configured

**Performance:**
- [ ] Images lazy load (except above-fold)
- [ ] No layout shift (CLS score low)
- [ ] First Contentful Paint under 1.5s
- [ ] Largest Contentful Paint under 2.5s
- [ ] Animations don't cause jank
- [ ] Bundle size is optimized

## Common Mistakes to Avoid

**Design Mistakes:**
1. Too much color (stick to monochrome + 1 accent max)
2. Small text (minimum 16px for body)
3. Poor contrast (especially in dark mode)
4. Cluttered layouts (embrace whitespace)
5. Inconsistent spacing (use 8pt grid)
6. Over-animation (subtle is better)
7. Generic stock photos (use real project screenshots)
8. Unclear navigation (keep it simple)
9. No clear CTA (every section should have purpose)
10. Trying too many trends at once

**Technical Mistakes:**
1. Unoptimized images (use Next.js Image)
2. No loading states (users see blank screens)
3. Forgetting mobile users (test on real devices)
4. No error handling (forms, API calls)
5. Slow page loads (optimize everything)
6. No focus states (accessibility fail)
7. Broken links (test all links)
8. Missing alt text (bad for SEO and a11y)
9. Console errors (shows lack of polish)
10. No analytics (can't measure success)

**Content Mistakes:**
1. Too much text (be concise)
2. No personality (let your voice show)
3. Listing every project ever (quality > quantity)
4. Generic descriptions ("Built a website...")
5. No context for projects (why you built it)
6. Outdated tech stack (remove old skills)
7. No contact info (how will they reach you?)
8. Broken English (if not native, get help)
9. No about section (they want to know you)
10. Claiming expertise you don't have

## Design Inspiration & References

**Portfolio Examples:**
- Behance.net - Browse portfolios
- Awwwards.com - Award-winning designs
- Dribbble.com - UI/UX inspiration
- SiteInspire.com - Curated web design

**Typography Resources:**
- Typewolf.com - Font pairing examples
- Fonts.google.com - Free fonts
- Fontsource - NPM fonts for Next.js

**Color Tools:**
- Coolors.co - Palette generator
- Contrast-ratio.com - Check WCAG compliance
- Realtime Colors - Visualize palettes on UI

**This Reference Style:**
- Clean, minimal, typography-focused
- Neo-brutalist influence
- Swiss design principles
- Professional yet approachable

## When Redesigning an Existing Portfolio

### Assessment Questions:
1. What feels outdated specifically?
   - Colors? (Too many colors, outdated palette)
   - Typography? (Small text, poor hierarchy)
   - Layout? (Cluttered, old grid systems)
   - Content? (Outdated projects, old tech stack)

2. Who is your target audience?
   - Recruiters/hiring managers
   - Potential clients
   - Fellow developers/designers

3. What's your primary goal?
   - Get hired (full-time or freelance)
   - Attract clients
   - Build personal brand

4. What do you want to keep?
   - Existing branding/logo
   - Color scheme
   - Specific sections
   - Content/copy

### Redesign Approach:

**Phase 1: Simplify**
- Remove unnecessary elements
- Reduce color palette to monochrome
- Improve typography (larger, clearer)
- Increase whitespace

**Phase 2: Restructure**
- Optimize page flow
- Improve navigation
- Better section hierarchy
- Mobile-first layout

**Phase 3: Polish**
- Add subtle animations
- Optimize images
- Improve copy
- Test accessibility

**Phase 4: Launch**
- Test on devices
- Check performance
- Get feedback
- Deploy

Remember: The goal is to showcase your work in the clearest, most professional way possible. Let your projects speak for themselves with minimal interference from the design.
