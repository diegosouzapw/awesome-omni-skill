---
name: sarvam-style-frontend
description: Build high-confidence, India-first AI product sites with a clean, assertive, enterprise aesthetic using React + Framer Motion.
license: See LICENSE
---

## Design Direction

**Purpose**: Market a serious, production-grade AI platform for enterprises and government with an India-first identity. Convey trust, scale, and clarity while keeping the interface modern and confident.

**Tone**:

- Clean, assertive, and product-focused.

- No “AI slop” gradients or neon overload.

- Strong typographic hierarchy with bold headlines, tight line lengths, and concise copy.

- Subtle motion; everything feels composed, never chaotic.

**Differentiating Moves**:

- Hero section is *sparse but powerful*: a single strong headline, subhead, and one primary CTA, supported by logos and a simple “platform pillars” strip.

- Reusable “section patterns”: three-up cards, badge rows, testimonial modules, research/update cards.

- India-first cues via copy (“India’s Sovereign AI Platform”, “AI for all from India”), not cliché tricolor overload.

## Tech Stack

- **Framework**: React (Vite or Next.js).

- **Animation**: Framer Motion for:

    - Hero text entrance (fade + slight upward motion).

    - Logo/marquee scroll or fade-in.

    - Card hover elevation & shadow transitions.

- **Styling**:

    - CSS Modules or a single global CSS file with CSS variables for colors and typography.

    - Prefer utility-ish classes but handcrafted, not Tailwind.

## Typography

Use a strong sans family with good weight range and a refined secondary face.

Recommended pairing:

- **Display / Headings**: `Satoshi`, `General Sans`, or `Metropolis` (download via Fontshare or local files).

- **Body**: `IBM Plex Sans`, `Source Sans 3`, or `System UI` as fallback.

Example `@font-face` setup:


@font-face {

  font-family: 'Satoshi';

  src: url('/fonts/Satoshi-Regular.woff2') format('woff2');

  font-weight: 400;

  font-style: normal;

  font-display: swap;

}

@font-face {

  font-family: 'Satoshi';

  src: url('/fonts/Satoshi-Medium.woff2') format('woff2');

  font-weight: 500;

  font-style: normal;

  font-display: swap;

}

@font-face {

  font-family: 'Satoshi';

  src: url('/fonts/Satoshi-Bold.woff2') format('woff2');

  font-weight: 700;

  font-style: normal;

  font-display: swap;

}

Global typography::root {

  --font-sans: 'Satoshi', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

}

body {

  font-family: var(--font-sans);

  color: #050816;

  background-color: #f5f5f7;

}

Color System

Extracted-style palette inspired by Sarvam::root {

  /* Neutral surface */

  --bg-body: #f5f5f7;

  --bg-surface: #ffffff;

  --bg-subtle: #f0f2f5;

  /* Primary */

  --primary: #111827;         /* deep ink */

  --primary-soft: #1f2937;

  --primary-contrast: #f97316; /* warm accent (CTA) */

  /* Borders & lines */

  --border-subtle: #e5e7eb;

  --border-strong: #d1d5db;

  /* Text */

  --text-primary: #020617;

  --text-muted: #6b7280;

  --text-soft: #9ca3af;

  /* Badges */

  --badge-bg: #eef2ff;

  --badge-text: #4338ca;

  /* Shadow */

  --shadow-soft: 0 18px 45px rgba(15, 23, 42, 0.08);

}

Layout & Composition

- Max-width container ~1120–1200px, centered with generous top/bottom padding.

- Hero area:

 ▫ Left: text stack (badge, eyebrow, h1, subcopy, CTA + secondary link).

 ▫ Right: could be interactive demo or cards (like “Population-scale Applications / State-of-the-art Models / Infrastructure…”).

- Sections have clear titles + descriptions + content grids (3 columns on desktop, stack on mobile).

.page {

  min-height: 100vh;

  background: radial-gradient(circle at top left, #e5f0ff 0, #f5f5f7 40%, #f5f5f7 100%);

}

.container {

  max-width: 1120px;

  margin: 0 auto;

  padding: 3rem 1.5rem 5rem;

}

Motion Guidelines

Use Framer Motion:

- Hero:

 ▫ Fade in + slight upward movement on load.

 ▫ Stagger text elements (badge → h1 → subcopy → buttons).

- Logos:

 ▫ Either static grid with subtle hover scale, or slow continuous marquee.

- Cards:

 ▫ Hover: scale to ‎⁠1.02⁠, elevate shadow, subtle border-color shift.

 ▫ On scroll: fade in + translateY 16px.

Example Framer Motion pattern:const fadeUp = {

  hidden: { opacity: 0, y: 24 },

  show: (i = 0) => ({

    opacity: 1,

    y: 0,

    transition: {

      delay: 0.15 + i * 0.07,

      duration: 0.5,

      ease: [0.21, 0.47, 0.32, 0.98],

    },

  }),

};

Use this for hero lines and cards via ‎⁠custom={index}⁠.

Components

Build everything as composable React components:

- ‎⁠HeroSection⁠: hero copy + CTA + supporting “pillar cards”

- ‎⁠LogoStrip⁠: partner logos grid/marquee

- ‎⁠PlatformPillarsSection⁠: three cards with icon/eyebrow/title/body

- ‎⁠CapabilitiesTabs⁠: tabs for Conversational Agents / TTS / STT / Vision / Dubbing with a simple demo area

- ‎⁠DeploymentOptions⁠: Sarvam Cloud / Private Cloud / On-Prem cards

- ‎⁠TestimonialSection⁠: quote + person + logo

- ‎⁠UpdatesSection⁠: research & updates cards

- ‎⁠Footer⁠: structured nav + metadata

Each component:

- Accepts data via props (titles, copy, links).

- Encodes Sarvam-style layout and motion internally.

Constraints

- Avoid overusing gradients; when used, they’re soft and minimal.

- Don’t use generic purple/blue “AI gradient blobs”.

- Don’t over-animate; keep motion focused and meaningful.

- Respect accessibility: color contrast, focus styles on buttons/links, semantic HTML.

Developer Workflow

1. Define global tokens (colors, typography, spacing) via CSS variables.

2. Implement ‎⁠<HeroSection />⁠ with Framer Motion animations and test on multiple viewport sizes.

3. Add ‎⁠<LogoStrip />⁠ and ‎⁠<PlatformPillarsSection />⁠ to match the “Powering India’s AI-first future” and “Full-stack platform” narrative.

4. Layer in remaining sections: capabilities tabs, deployment options, testimonial, updates.

5. Iterate on microcopy and motion timing to ensure the page feels confident, not hyperactive.

You can drop this in `SKILL.md` for your /how-to-web flow.

---

## 2. Implementing the hero / top section in React + CSS + Framer Motion

Let’s implement one specific element inspired by the current page:

- The hero headline: “India’s Sovereign AI Platform / AI for all from India”

- Subtext blocks like “Built on sovereign compute…”

- Primary CTA: “Experience Sarvam”

- Partner logos section (“India builds with Sarvam”)

- “Powering India’s AI-first future” strip with 3 pillar cards.

### 2.1. File structure

Something like:

- `src/components/Hero.tsx`

- `src/components/LogoStrip.tsx`

- `src/components/PlatformPillars.tsx`

- `src/styles/global.css` (or `Hero.css` etc.)

### 2.2. Global CSS: fonts + colors

Assuming you’ve downloaded fonts like Satoshi and placed in `public/fonts`.


/* src/styles/global.css */

/* Fonts */

@font-face {

  font-family: 'Satoshi';

  src: url('/fonts/Satoshi-Regular.woff2') format('woff2');

  font-weight: 400;

  font-style: normal;

  font-display: swap;

}

@font-face {

  font-family: 'Satoshi';

  src: url('/fonts/Satoshi-Medium.woff2') format('woff2');

  font-weight: 500;

  font-style: normal;

  font-display: swap;

}

@font-face {

  font-family: 'Satoshi';

  src: url('/fonts/Satoshi-SemiBold.woff2') format('woff2');

  font-weight: 600;

  font-style: normal;

  font-display: swap;

}

:root {

  --bg-body: #f5f5f7;

  --bg-surface: #ffffff;

  --bg-subtle: #f0f2f5;

  --primary: #111827;

  --primary-soft: #1f2937;

  --primary-contrast: #f97316;

  --border-subtle: #e5e7eb;

  --border-strong: #d1d5db;

  --text-primary: #020617;

  --text-muted: #6b7280;

  --text-soft: #9ca3af;

  --badge-bg: #eef2ff;

  --badge-text: #4338ca;

  --shadow-soft: 0 18px 45px rgba(15, 23, 42, 0.08);

}

*,

*::before,

*::after {

  box-sizing: border-box;

}

body {

  margin: 0;

  font-family: 'Satoshi', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;

  background: radial-gradient(circle at top left, #e5f0ff 0, #f5f5f7 40%, #f5f5f7 100%);

  color: var(--text-primary);

}

.page {

  min-height: 100vh;

}

.container {

  max-width: 1120px;

  margin: 0 auto;

  padding: 3rem 1.5rem 5rem;

}

2.3. Hero component (React + Framer Motion)// src/components/Hero.tsx

import { motion } from 'framer-motion';

import './Hero.css';

const fadeUp = {

  hidden: { opacity: 0, y: 24 },

  show: (i: number = 0) => ({

    opacity: 1,

    y: 0,

    transition: {

      delay: 0.15 + i * 0.08,

      duration: 0.55,

      ease: [0.21, 0.47, 0.32, 0.98],

    },

  }),

};

export function Hero() {

  return (

    <section className="hero">

      <div className="hero-inner">

        <div className="hero-text">

          <motion.div

            className="hero-badge"

            initial="hidden"

            animate="show"

            variants={fadeUp}

            custom={0}

          >

            India&apos;s Sovereign AI Platform

          </motion.div>

          <motion.h1

            className="hero-title"

            initial="hidden"

            animate="show"

            variants={fadeUp}

            custom={1}

          >

            AI for all <br /> from&nbsp;India

          </motion.h1>

          <motion.p

            className="hero-subtitle"

            initial="hidden"

            animate="show"

            variants={fadeUp}

            custom={2}

          >

            Built on sovereign compute. Powered by frontier-class models. Delivering population-scale

            impact.

          </motion.p>

          <motion.div

            className="hero-cta-row"

            initial="hidden"

            animate="show"

            variants={fadeUp}

            custom={3}

          >

            <button className="hero-primary-btn">

              Experience Sarvam

            </button>

            <button className="hero-secondary-btn">

              Talk to Sales

            </button>

          </motion.div>

          <motion.p

            className="hero-footnote"

            initial="hidden"

            animate="show"

            variants={fadeUp}

            custom={4}

          >

            Built for Enterprise · Government · Developers

          </motion.p>

        </div>

        <motion.div

          className="hero-side"

          initial={{ opacity: 0, y: 32 }}

          animate={{ opacity: 1, y: 0 }}

          transition={{ delay: 0.4, duration: 0.7, ease: [0.21, 0.47, 0.32, 0.98] }}

        >

          <div className="hero-pillars">

            <div className="pillar-card">

              <div className="pillar-eyebrow">Sovereign by design</div>

              <h3>Full control, entirely in India</h3>

              <p>

                Build, deploy, and run AI with full sovereignty over data, models, and infrastructure.

              </p>

            </div>

            <div className="pillar-card">

              <div className="pillar-eyebrow">State-of-the-art models</div>

              <h3>Built for India&apos;s languages</h3>

              <p>

                Industry-leading performance tuned for Indian languages, culture, and context.

              </p>

            </div>

            <div className="pillar-card">

              <div className="pillar-eyebrow">Human at the core</div>

              <h3>Engineers in the loop</h3>

              <p>

                Forward-deployed teams work with you to ship production-ready AI agents.

              </p>

            </div>

          </div>

        </motion.div>

      </div>

    </section>

  );

}

2.4. Hero-specific CSS/* src/components/Hero.css */

.hero {

  padding-top: 5rem;

  padding-bottom: 3rem;

}

.hero-inner {

  max-width: 1120px;

  margin: 0 auto;

  padding: 0 1.5rem;

  display: grid;

  grid-template-columns: minmax(0, 1.2fr) minmax(0, 1fr);

  gap: 3.5rem;

  align-items: center;

}

.hero-text {

  max-width: 560px;

}

.hero-badge {

  display: inline-flex;

  align-items: center;

  padding: 0.35rem 0.8rem;

  border-radius: 999px;

  background-color: var(--badge-bg);

  color: var(--badge-text);

  font-size: 0.75rem;

  letter-spacing: 0.06em;

  text-transform: uppercase;

  font-weight: 600;

  margin-bottom: 1rem;

}

.hero-title {

  font-size: clamp(2.6rem, 3.5vw, 3.4rem);

  line-height: 1.05;

  letter-spacing: -0.04em;

  margin: 0 0 1.2rem;

  color: var(--text-primary);

}

.hero-subtitle {

  margin: 0 0 1.8rem;

  font-size: 1rem;

  line-height: 1.6;

  color: var(--text-muted);

  max-width: 440px;

}

.hero-cta-row {

  display: flex;

  align-items: center;

  gap: 0.9rem;

  margin-bottom: 1.3rem;

}

.hero-primary-btn {

  border: none;

  cursor: pointer;

  padding: 0.75rem 1.4rem;

  border-radius: 999px;

  background-color: var(--primary-contrast);

  color: #111827;

  font-weight: 600;

  font-size: 0.95rem;

  box-shadow: 0 16px 30px rgba(249, 115, 22, 0.3);

  transition: transform 150ms ease, box-shadow 150ms ease, background-color 150ms ease;

}

.hero-primary-btn:hover {

  transform: translateY(-1px);

  box-shadow: 0 20px 40px rgba(249, 115, 22, 0.35);

  background-color: #ea580c;

}

.hero-secondary-btn {

  border-radius: 999px;

  border: 1px solid var(--border-subtle);

  background-color: rgba(255, 255, 255, 0.85);

  color: var(--text-primary);

  padding: 0.75rem 1.2rem;

  cursor: pointer;

  font-size: 0.9rem;

  font-weight: 500;

  display: inline-flex;

  align-items: center;

  gap: 0.4rem;

  backdrop-filter: blur(8px);

  transition: background-color 150ms ease, border-color 150ms ease, transform 150ms ease;

}

.hero-secondary-btn:hover {

  background-color: #ffffff;

  border-color: var(--border-strong);

  transform: translateY(-1px);

}

.hero-footnote {

  margin: 0;

  font-size: 0.85rem;

  color: var(--text-soft);

}

/* Right side pillars */

.hero-side {

  display: flex;

  justify-content: flex-end;

}

.hero-pillars {

  background-color: var(--bg-surface);

  border-radius: 1.5rem;

  padding: 1.8rem 1.6rem;

  border: 1px solid rgba(209, 213, 219, 0.8);

  box-shadow: var(--shadow-soft);

  display: grid;

  grid-template-columns: 1fr;

  gap: 1.3rem;

}

.pillar-card {

  padding: 0.9rem 0.9rem 1rem;

  border-radius: 1rem;

  border: 1px solid rgba(229, 231, 235, 0.9);

  background: linear-gradient(

    135deg,

    rgba(248, 250, 252, 0.95),

    rgba(241, 245, 249, 0.9)

  );

  transition: border-color 150ms ease, transform 150ms ease, box-shadow 150ms ease;

}

.pillar-card:hover {

  transform: translateY(-3px);

  box-shadow: 0 16px 40px rgba(15, 23, 42, 0.12);

  border-color: rgba(148, 163, 184, 0.9);

}

.pillar-eyebrow {

  font-size: 0.75rem;

  font-weight: 600;

  text-transform: uppercase;

  letter-spacing: 0.08em;

  color: var(--text-soft);

  margin-bottom: 0.3rem;

}

.pillar-card h3 {

  font-size: 0.98rem;

  margin: 0 0 0.35rem;

  color: var(--text-primary);

}

.pillar-card p {

  margin: 0;

  font-size: 0.86rem;

  line-height: 1.5;

  color: var(--text-muted);

}

@media (min-width: 960px) {

  .hero-pillars {

    grid-template-columns: 1fr;

  }

}

@media (max-width: 960px) {

  .hero-inner {

    grid-template-columns: minmax(0, 1fr);

    gap: 2.8rem;

  }

  .hero-side {

    justify-content: flex-start;

  }

}

@media (max-width: 640px) {

  .hero {

    padding-top: 3.5rem;

  }

  .hero-cta-row {

    flex-direction: column;

    align-items: flex-start;

  }

  .hero-pillars {

    padding: 1.4rem 1.2rem;

  }

}

2.5. Partner logo strip

Sarvam has an “India builds with Sarvam” logo grid. You can implement a reusable version:// src/components/LogoStrip.tsx

import { motion } from 'framer-motion';

import './LogoStrip.css';

const logos = [

  'Partner 1',

  'Partner 2',

  'Partner 3',

  'Partner 4',

  'Partner 5',

  'Partner 6',

  'Partner 7',

  'Partner 8',

];

export function LogoStrip() {

  return (

    <section className="logo-strip">

      <div className="logo-strip-inner">

        <div className="logo-strip-heading">

          <span className="logo-strip-eyebrow">India builds with Sarvam</span>

        </div>

        <div className="logo-grid">

          {logos.map((name, idx) => (

            <motion.div

              key={name}

              className="logo-pill"

              initial={{ opacity: 0, y: 10 }}

              whileInView={{ opacity: 1, y: 0 }}

              viewport={{ once: true, margin: '-40px' }}

              transition={{ delay: idx * 0.03, duration: 0.4 }}

            >

              <span>{name}</span>

            </motion.div>

          ))}

        </div>

      </div>

    </section>

  );

}

/* src/components/LogoStrip.css */

.logo-strip {

  padding: 1.8rem 0 2.6rem;

}

.logo-strip-inner {

  max-width: 1120px;

  margin: 0 auto;

  padding: 0 1.5rem;

}

.logo-strip-heading {

  display: flex;

  justify-content: space-between;

  align-items: center;

  margin-bottom: 1.4rem;

}

.logo-strip-eyebrow {

  font-size: 0.8rem;

  text-transform: uppercase;

  letter-spacing: 0.12em;

  color: var(--text-soft);

  font-weight: 600;

}

.logo-grid {

  display: grid;

  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));

  gap: 0.9rem;

}

.logo-pill {

  border-radius: 999px;

  padding: 0.7rem 1.2rem;

  border: 1px solid var(--border-subtle);

  background: rgba(255, 255, 255, 0.88);

  backdrop-filter: blur(8px);

  display: flex;

  align-items: center;

  justify-content: center;

  color: var(--text-muted);

  font-size: 0.8rem;

  font-weight: 500;

  transition: border-color 150ms ease, transform 150ms ease, box-shadow 150ms ease;

}

.logo-pill:hover {

  border-color: var(--border-strong);

  transform: translateY(-2px);

  box-shadow: 0 12px 30px rgba(15, 23, 42, 0.08);

}

2.6. Wiring it up in App.tsx// src/App.tsx

import './styles/global.css';

import { Hero } from './components/Hero';

import { LogoStrip } from './components/LogoStrip';

function App() {

  return (

    <div className="page">

      <Hero />

      <LogoStrip />

      {/* next sections: PlatformPillars, Capabilities, etc. */}

    </div>

  );

}

export default App;

Step-by-step reasoning for implementation

1. Set up React + Framer Motion

 ▫ Create a Vite React project.

 ▫ ‎⁠npm install framer-motion⁠.

 ▫ Add ‎⁠global.css⁠ with fonts and color tokens. Import it in ‎⁠main.tsx⁠/‎⁠App.tsx⁠.

2. Define typography + color tokens first

 ▫ This locks in the vibe (like Sarvam) before layout.

 ▫ Use CSS variables and ‎⁠@font-face⁠ for your chosen display/body fonts.

3. Build the hero layout in raw HTML/JSX

 ▫ Grid with two columns: text + side card.

 ▫ Use proper semantic tags (‎⁠section⁠, ‎⁠h1⁠, ‎⁠p⁠, ‎⁠button⁠).

 ▫ Keep copy short and stacked like on sarvam.ai.

4. Style hero with CSS

 ▫ Match spacing: big top padding, constrained text width.

 ▫ Badge style for “India’s Sovereign AI Platform”.

 ▫ High-contrast headline, muted subcopy.

5. Add Framer Motion for entrance animations

 ▫ Create a shared ‎⁠fadeUp⁠ variant and assign ‎⁠custom={index}⁠ to elements.

 ▫ Animate hero side card separately so it feels like it slides in as one block.

6. Implement partner logo strip

 ▫ Create a simple grid with pill-shaped tiles.

 ▫ Use Framer Motion ‎⁠whileInView⁠ for subtle entrance when scrolled into view.

7. Refine micro-interactions

 ▫ Hover on buttons and pillar cards: tiny ‎⁠translateY⁠ changes, box-shadow bump.

 ▫ Keep motion durations between 0.3–0.6s; easing cubic-bezier to feel smooth.

Fonts and color recap

Fonts you may want to download (or equivalents):

- ‎⁠Satoshi⁠ (or ‎⁠General Sans⁠, ‎⁠Metropolis⁠) for headings and body.

- Alternative system-safe stack: ‎⁠system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif⁠.

Key CSS color variables (you can tweak to taste):--bg-body: #f5f5f7;

--bg-surface: #ffffff;

--bg-subtle: #f0f2f5;

--primary: #111827;

--primary-soft: #1f2937;

--primary-contrast: #f97316;

--border-subtle: #e5e7eb;

--border-strong: #d1d5db;

--text-primary: #020617;

--text-muted: #6b7280;

--text-soft: #9ca3af;

--badge-bg: #eef2ff;

--badge-text: #4338ca;

--shadow-soft: 0 18px 45px rgba(15, 23, 42, 0.08);

If you want, we can next drill down into another specific element from the page (e.g., the “See it in action” tabbed section) and build that as a React + Framer Motion component set.