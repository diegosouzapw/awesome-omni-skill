---
name: ui-designer
description: Generate and serve live HTML/CSS/JS UI designs from natural language prompts. Use when the user asks to design, create, build, or prototype a website, landing page, UI, dashboard, web page, or frontend mockup. Also triggers on requests to update, tweak, or iterate on a previously generated design. Replaces traditional UI design + frontend dev workflow.
---

# UI Designer

Generate production-quality React pages from natural language, serve live, iterate until satisfied.

## Setup

Check TOOLS.md for `### UI Designer` config. If missing, run first-time setup:
1. Ask user which port (default: 5174)
2. Run: `bash scripts/setup.sh <port>`
3. Save config to TOOLS.md

## Project Structure

```
<serve_dir>/
├── project-a/
│   ├── project.json       (config: name, preferences, design system, pages)
│   ├── assets/             (images, converted to .webp)
│   ├── landing/index.html  (React page via CDN)
│   └── about/index.html
```

### React Page Template (CDN, no build step)
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Page Title</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    function App() { return <div>...</div>; }
    ReactDOM.createRoot(document.getElementById('root')).render(<App />);
  </script>
</body>
</html>
```

## Workflow

**Critical: Message the user at EVERY action — not just milestones. If you're reading a file, say "Reading project config...". If you're writing HTML, say "Writing bounty page...". If you're taking a screenshot, say "Taking screenshot...". The user should never wonder what you're doing. Treat it like a live build log.**

### Step 1: Project Name
Ask: "What's the project name?"
- If `<serve_dir>/<project>/` exists: read `project.json`, show current setup and existing pages, ask if amending or adding
- If new: create directory + `project.json`
- **→ Message user: "Project created / loaded ✓"**

### Step 2: Preferences (new or amending)
Ask about design preferences (style, font, colors, brand assets).
Save to `project.json`.
- **→ Message user: "Preferences saved ✓"**

### Step 3: Page Slug
Ask: "What slug for this page?"
- Check if exists → ask to overwrite or new
- **→ Message user: "Slug confirmed: /<slug> ✓"**

### Step 4: Design Details & Design System
Ask about page content + design system (see Design Principles below).
Update `project.json` with design system details.
- **→ Message user: "Got it, generating now..."**

### Step 5: Generate
Generate the React page. Apply Design Principles strictly.
- **→ Message user: "HTML generated, running visual review..."**

### Step 6: Screenshot Review Loop
```bash
bash scripts/screenshot.sh "http://localhost:<port>/<project>/<slug>/" /tmp/<slug>-review.png 1400 900
```
Analyze with `image` tool. Fix issues. Re-screenshot. Also check mobile (width=390).
- **→ Message user: "Review done, sending preview..."**

**Do at least one desktop + one mobile review pass before sharing.**

### Step 7: Share and Iterate
Send live URL + screenshot to user.
Ask for feedback. Apply changes → re-screenshot → share. Repeat.
- **→ Keep user informed at every iteration step**

### Step 8: Media Assets
If user provides images:
1. Save to `<project>/assets/`
2. Convert to .webp: `bash scripts/convert-image.sh <input> <output.webp> [quality]`
3. Reference in HTML as `../assets/filename.webp`
- **→ Message user: "Image converted: 1.2MB → 340KB (72% smaller) ✓"**

### Step 9: Export
Zip the project folder and send:
```bash
cd <serve_dir> && zip -r /tmp/<project>.zip <project>/
```
Send zip via message tool with `filePath`. The CDN-based React pages work standalone — just open `index.html` or serve with any static server.

## Image Handling

Convert all user-provided images to `.webp` for performance:
```bash
bash scripts/convert-image.sh input.png output.webp 80
```
- Default quality: 80 (good balance of quality/size)
- For hero/banner images: quality 85
- For thumbnails/icons: quality 70
- Always report compression savings to user

For placeholder images during prototyping:
- Photos: `https://picsum.photos/seed/<name>/<width>/<height>`
- Solid placeholders: `https://placehold.co/<width>x<height>/<bg>/<text>`

## Design Principles

Apply these consistently to every generated page. These are non-negotiable quality standards.

### Layout & Spacing
- **Use consistent spacing scale** — stick to Tailwind's scale (4, 6, 8, 12, 16, 20, 24). Don't mix random values.
- **Max content width** — always constrain content (max-w-5xl or max-w-6xl). Never let text run full-width.
- **Vertical rhythm** — consistent gaps between sections (py-16 for sections, py-8 for subsections).
- **Padding on mobile** — minimum px-4 on all containers. Text must never touch screen edges.

### Typography
- **Clear hierarchy** — h1 largest, then h2, h3. Max 3-4 font sizes per page.
- **Line length** — max 65-75 characters per line for readability. Use max-w-prose or max-w-2xl on text blocks.
- **Font weight contrast** — bold for headings (font-bold/font-semibold), regular for body.
- **Text color hierarchy** — white for headings, gray-300 for body, gray-500 for muted/secondary.

### Color & Contrast
- **WCAG AA minimum** — text must have 4.5:1 contrast against background.
- **Limit palette** — max 1 primary color + 1 accent + neutrals. Don't rainbow.
- **Consistent accent usage** — primary color for CTAs, links, active states only. Don't overuse.
- **Dark backgrounds** — use layered darkness (bg-900 → bg-800 → bg-700) for depth, not flat black.

### Responsive Design
- **Mobile-first** — design for 390px first, enhance for larger screens.
- **Breakpoints** — use sm: (640px), md: (768px), lg: (1024px). Test all three.
- **Touch targets** — buttons/links min 44x44px on mobile.
- **Stack on mobile** — grids collapse to single column. Never horizontal scroll for content.
- **Navigation** — hamburger menu on mobile with slide-down panel. Always include.

### Components & Interactions
- **Icons** — always use SVG, never emoji (emoji break in headless browsers and render inconsistently).
- **Buttons** — clear hover states (color shift + slight scale or shadow). Consistent border-radius.
- **Cards** — subtle border (border-white/5 or border-dark-600), slight bg difference from page bg.
- **Transitions** — add `transition` class to all interactive elements. Duration 150-200ms.
- **Focus states** — all interactive elements must have visible focus rings for accessibility.

### Images & Performance
- **All images in .webp** — convert user images with scripts/convert-image.sh.
- **Lazy loading** — add `loading="lazy"` to images below the fold.
- **Aspect ratios** — use `aspect-video` or `aspect-square` classes to prevent layout shift.
- **Alt text** — every image needs descriptive alt text.

### Code Quality
- **Semantic HTML** — use header, main, section, footer, nav. Not div soup.
- **No Lorem ipsum** — always use contextually relevant placeholder text.
- **React components** — break UI into logical components (Navbar, Hero, Features, etc.).
- **State management** — use useState for interactive elements (tabs, modals, dropdowns).

### Common Mistakes to Avoid
- ❌ Text touching screen edges on mobile
- ❌ Emoji for icons (use SVG)
- ❌ Flat black backgrounds (use layered darks)
- ❌ No hover states on clickable elements
- ❌ Inconsistent border-radius across elements
- ❌ Giant font sizes that overflow on mobile
- ❌ Missing meta viewport tag
- ❌ Forgetting hamburger menu on mobile
