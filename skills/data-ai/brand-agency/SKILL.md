---
name: brand-agency
description: "Expert branding agent that guides users through questionnaire, researches fonts/colors online, and generates comprehensive brand guidelines (MD + TXT files) with visual swatches. Triggers on: brand, branding, brand guidelines, brand identity, create brand."
---

# Brand Agency

Guide users through comprehensive brand identity development, from discovery questionnaire to final brand guidelines with visual assets.

---

## The Job

1. Guide user through branding questionnaire (35 questions across 7 sections)
2. Let user choose conversation flow style (one-by-one, grouped, or custom)
3. Research fonts and color palettes online based on user's brand direction
4. Generate color swatches with rationale for each color
5. Recommend Google Fonts with usage rules
6. Output brand guidelines and executive summary
7. Evaluate output and iterate until requirements are met

---

## Output Files

The skill generates three types of output in the `./branding/` directory:

- **brand-guidelines.md** — Comprehensive brand guidelines with strict rules for logo usage, colors, typography, and visual identity
- **brand-summary.txt** — Executive summary highlighting key brand decisions for stakeholders
- **swatches/** — Visual color swatch images generated using the painter tool

---

## Workflow Overview

### Phase 1: Conversation Flow Selection

**Start every branding session by asking the user to choose their preferred flow:**

```
How would you like to proceed through the branding questionnaire?

1. **One-by-one** — I'll ask each question individually (best for thorough exploration)
2. **Grouped** — I'll present questions section by section (balanced approach)
3. **Custom** — You choose which sections/questions are relevant (fastest)
```

**Flow Logic by Selection:**

#### Option 1: One-by-One Flow
- Ask each of the 35 questions individually
- Wait for user response before proceeding to next question
- Allow user to say "skip" to move past any question
- Provide brief context for why each question matters
- After each response, acknowledge and move to next question

#### Option 2: Grouped Flow
- Present all questions within a section at once
- List the section name and all questions in that section
- User can answer all at once or respond to specific numbers
- After completing a section, summarize responses and move to next section
- Section order: Identity → Audience → Messaging → Visual → Marketing → Operational → Additional

#### Option 3: Custom Flow
- First, show the 7 section titles with question counts:
  ```
  Which sections are relevant to your project?
  
  1. Brand Identity and Story (6 questions)
  2. Target Audience and Market Position (5 questions)
  3. Brand Messaging and Communication (4 questions)
  4. Visual Identity and Design Preferences (5 questions)
  5. Marketing and Promotional Strategy (4 questions)
  6. Operational and Strategic Insights (5 questions)
  7. Additional Insights & Bonus Questions (6 questions)
  
  Enter section numbers (e.g., "1, 2, 4") or "all":
  ```
- After user selects sections, ask: "Within these sections, one-by-one or grouped?"
- Proceed with only the selected sections using chosen sub-flow

**Handle interruptions gracefully:**
- If user says "let's move on" or "skip this section" — proceed to next phase
- If user provides partial answers — work with what you have
- If user wants to revisit — allow jumping back to any section

---

### Phase 2: Brand Discovery Questionnaire

Guide user through 35 branding questions organized into 7 sections. Use the flow style selected in Phase 1.

#### Section 1: Brand Identity and Story (Questions 1-6)

| # | Question |
|---|----------|
| 1 | Tell us briefly about your business. What products or services do you provide? |
| 2 | Why did you start your business, what was the motivator for it? |
| 3 | What mission statement and core values drive your brand (if any)? |
| 4 | In 3 words, how would you characterize your brand's personality? |
| 5 | Besides personality traits, what are 5 words that capture your brand's essence? |
| 6 | What emotional triggers do you want your brand to evoke in customers? |

**Follow-up prompts:** If answers are vague, ask "Can you give me a specific example?" or "What would that look like in practice?"

#### Section 2: Target Audience and Market Position (Questions 7-11)

| # | Question |
|---|----------|
| 7 | Describe your ideal customer in detail, covering personality and behavior. |
| 8 | Where would your brand fit into the customer's life? |
| 9 | What struggles does your ideal customer encounter that you solve? |
| 10 | Can you share 3–4 examples of direct competitor websites? |
| 11 | What's your stance against competitors — is it a competitive field? |

**Follow-up prompts:** If user struggles with Q7, suggest demographics (age, income, location) and psychographics (values, interests, lifestyle).

#### Section 3: Brand Messaging and Communication (Questions 12-15)

| # | Question |
|---|----------|
| 12 | What essential messages must your brand convey to your audience? |
| 13 | How does your brand speak and interact with your audience — what is its tone? |
| 14 | How does your brand motivate customers to make a purchase? |
| 15 | What results should customers expect after buying from you? |

**Follow-up prompts:** For tone (Q13), offer examples: "formal vs casual, playful vs serious, authoritative vs friendly."

#### Section 4: Visual Identity and Design Preferences (Questions 16-20)

| # | Question |
|---|----------|
| 16 | Share 2–3 logo designs that inspire you and explain your preferences. |
| 17 | What color palette do you prefer, and are there any colors to avoid? |
| 18 | Do you have any specific typography or font styles in mind? |
| 19 | How would you describe your brand's visual style? |
| 20 | Are there specific design assets you've already produced or want to use? |

**Follow-up prompts:** For Q19, offer style spectrum: "minimalist vs ornate, modern vs classic, bold vs subtle, geometric vs organic."

#### Section 5: Marketing and Promotional Strategy (Questions 21-24)

| # | Question |
|---|----------|
| 21 | What marketing materials do you anticipate needing (e.g., brochure, website)? |
| 22 | Describe your current or desired social media presence. |
| 23 | How do you intend to reach your ideal customer? |
| 24 | What makes your brand's offering unique vs competitors? |

**Follow-up prompts:** If Q24 is weak, ask "What do customers tell you they love about working with you?"

#### Section 6: Operational and Strategic Insights (Questions 25-29)

| # | Question |
|---|----------|
| 25 | What are your brand's short- and long-term objectives? |
| 26 | Where do you envision your brand in 1, 3, and 5 years? |
| 27 | What are your deadlines and timelines for this branding project? |
| 28 | How do you plan to implement the branding strategy in business operations? |
| 29 | How do you prefer to provide feedback, and what turnaround time do you expect? |

**Follow-up prompts:** For Q25-26, distinguish between revenue goals, market position, and brand perception goals.

#### Section 7: Additional Insights & Bonus Questions (Questions 30-35)

| # | Question |
|---|----------|
| 30 | How have previous solutions not met your ideal customer's needs? |
| 31 | What inspired your current logo, and does it still represent your brand? |
| 32 | How does your brand balance contrasting characteristics (e.g., modern/classic)? |
| 33 | What other brands (competitor or not) do you admire, and why? |
| 34 | Describe the type of imagery you envision for your brand. |
| 35 | How do you want customers to experience your brand overall? |

**Follow-up prompts:** Q33 is valuable for visual research — ask for specific examples and what elements they admire.

---

#### Skipping and Adaptive Selection

**When to skip questions:**
- User says "skip", "not applicable", or "I don't know" — move on without pressure
- For startups: Skip Q31 (current logo) if they don't have one yet
- For rebrand projects: Prioritize Q31, Q30, Q32 to understand what's changing
- For time-constrained users: Sections 1-4 are essential; Sections 5-7 are supplementary

**Minimum viable questionnaire (10 essential questions):**
If user requests the fastest path, use: Q1, Q4, Q6, Q7, Q9, Q13, Q16, Q17, Q19, Q24

**Recording responses:**
- Keep a running summary of responses for each section
- After completing all questions, present a consolidated summary for user confirmation
- Allow user to revise any answer before proceeding to research phase

---

### Phase 3: Research

Based on questionnaire responses, research fonts, colors, and competitors using web tools.

#### 3.1 Google Fonts Research

**When to research:** After capturing brand personality (Q4), visual style (Q19), and typography preferences (Q18).

**Search workflow:**

1. Use `web_search` with brand descriptors:
   ```
   web_search: "Google Fonts [personality trait] [visual style]"
   Example: "Google Fonts modern minimalist sans-serif"
   ```

2. Read Google Fonts directly for pairing suggestions:
   ```
   read_web_page: "https://fonts.google.com" 
   objective: "Find [style] fonts suitable for [brand personality]"
   ```

3. For each font candidate, verify:
   - Available weights (need at least regular, medium, bold)
   - Language support if brand operates internationally
   - Free for commercial use (all Google Fonts are)

**Capture for each font:**
| Field | Example |
|-------|---------|
| Name | Inter |
| Category | Sans-serif |
| Recommended use | Body text |
| Personality match | Clean, modern, highly readable |
| Pairing suggestion | Works well with Playfair Display for headings |

#### 3.2 Color Palette Research

**When to research:** After capturing emotional triggers (Q6), color preferences (Q17), and industry context (Q1).

**Search workflow:**

1. Research industry color conventions:
   ```
   web_search: "[industry] brand color palette examples"
   Example: "fintech brand color palette examples"
   ```

2. Research color psychology for brand emotions:
   ```
   web_search: "color psychology [emotion] branding"
   Example: "color psychology trust professional branding"
   ```

3. Find palette inspiration:
   ```
   read_web_page: "https://coolors.co/palettes/trending"
   objective: "Find [mood] color palettes for [industry]"
   ```

4. Cross-reference with competitor colors (avoid similarity):
   - Note competitor primary colors from Phase 3.3
   - Ensure differentiation while staying industry-appropriate

**Capture for each color:**
| Field | Example |
|-------|---------|
| Hex code | #2563EB |
| Name | Trust Blue |
| Role | Primary brand color |
| Rationale | Conveys reliability; differentiates from competitor's teal |
| Usage context | Headlines, CTAs, primary buttons |

#### 3.3 Competitor Analysis

**When to research:** After user provides competitor examples (Q10) and competitive stance (Q11).

**Search workflow:**

1. For each competitor URL provided:
   ```
   read_web_page: "[competitor URL]"
   objective: "Identify visual identity: primary colors, typography style, logo characteristics, overall aesthetic"
   ```

2. If competitor URLs not provided, search for them:
   ```
   web_search: "[industry] [location] companies similar to [user's business]"
   ```

3. Analyze competitor patterns:
   - Common color choices in the industry
   - Typography trends (serif vs sans-serif dominance)
   - Visual density (minimalist vs information-rich)
   - Imagery style (photography vs illustration)

**Capture competitor summary:**

```
## Competitor: [Name]
- **Primary color:** [hex + description]
- **Typography:** [serif/sans-serif, specific fonts if identifiable]
- **Visual style:** [minimalist/bold/elegant/playful]
- **Differentiation opportunity:** [what user's brand can do differently]
```

#### Research Summary Template

After completing research, present findings to user:

```
## Research Summary

### Font Recommendations
Based on your brand personality ([traits]) and visual style ([style]):
1. **Heading font:** [Name] - [why it fits]
2. **Body font:** [Name] - [why it fits]
3. **Accent font:** [Name] - [why it fits]

### Color Direction
Based on [industry] conventions and your desired emotions ([emotions]):
- Primary: [color] - [rationale]
- Secondary: [color] - [rationale]
- Accent: [color] - [rationale]
- Neutrals: [colors]

### Competitor Landscape
[2-3 sentence summary of what competitors are doing visually]
- Differentiation opportunity: [specific recommendation]

Does this direction align with your vision? Any adjustments before we proceed?
```

**User confirmation required** before proceeding to Phase 4.

---

### Phase 4: Color Palette Generation

Generate a complete color palette based on research findings and user preferences.

#### 4.1 Swatch Count Selection

**Ask user before generating:**

```
How many colors would you like in your brand palette?

Recommended options:
- **5 colors** — Minimal palette (primary, secondary, accent, 2 neutrals)
- **7 colors** — Standard palette (adds light/dark variants)
- **10 colors** — Extended palette (full system with semantic colors)

Enter a number or choose a recommendation:
```

**Palette structure by count:**

| Count | Includes |
|-------|----------|
| 5 | Primary, Secondary, Accent, Light neutral, Dark neutral |
| 7 | Above + Primary light variant, Primary dark variant |
| 10 | Above + Success, Warning, Error (semantic colors) |

#### 4.2 Color Research Workflow

**Build on Phase 3.2 research** — reference the color direction already approved by user.

1. For each color role, search for specific inspiration:
   ```
   web_search: "[color role] color [industry] [brand personality]"
   Example: "primary color fintech trustworthy modern"
   ```

2. Validate color accessibility:
   ```
   web_search: "[hex code] contrast ratio accessibility"
   ```

3. Check color harmony:
   ```
   read_web_page: "https://coolors.co/contrast-checker"
   objective: "Verify [primary] and [secondary] have sufficient contrast"
   ```

4. Research complementary/analogous options:
   ```
   web_search: "colors that pair well with [primary hex]"
   ```

#### 4.3 Color Definition Template

**For each color, capture:**

| Field | Description | Example |
|-------|-------------|---------|
| Hex code | 6-digit hex value | #2563EB |
| Name | Memorable brand-specific name | Trust Blue |
| Role | Function in the palette | Primary |
| Rationale | Why this color fits the brand | Conveys reliability and professionalism; differentiates from competitor's teal |
| Usage contexts | Where to apply this color | Headlines, CTAs, primary buttons, hero backgrounds |
| Accessibility notes | Contrast requirements | Use only on white/light backgrounds; pair with #FFFFFF text |

#### 4.4 Palette Output Format

Present the complete palette to user for approval:

```
## Your Brand Color Palette

### Primary Colors
| Color | Hex | Role | Usage |
|-------|-----|------|-------|
| 🔵 Trust Blue | #2563EB | Primary | Headlines, CTAs, buttons |
| 🟢 Growth Green | #10B981 | Secondary | Success states, highlights |

### Accent Colors
| Color | Hex | Role | Usage |
|-------|-----|------|-------|
| 🟡 Energy Gold | #F59E0B | Accent | Callouts, badges, icons |

### Neutral Colors
| Color | Hex | Role | Usage |
|-------|-----|------|-------|
| ⚪ Cloud White | #F9FAFB | Light neutral | Backgrounds, cards |
| ⚫ Charcoal | #1F2937 | Dark neutral | Body text, footers |

### Color Rationale Summary
[2-3 sentences explaining how the palette supports brand personality and differentiates from competitors]

Does this palette align with your brand vision? Any colors to adjust?
```

#### 4.5 Iteration Handling

**If user requests changes:**
- "Too bold" → Research more muted/desaturated variants
- "Too safe" → Research more vibrant/saturated options
- "Doesn't feel [emotion]" → Re-research color psychology for that emotion
- "Too similar to [competitor]" → Find alternative hues that maintain brand personality

**User approval required** before proceeding to Phase 5.

#### 4.6 Visual Swatch Generation

**After user approves the palette**, generate visual color swatches using the painter tool.

**Swatch generation workflow:**

1. Create the output directory structure:
   ```
   ./branding/swatches/
   ```

2. For each color in the approved palette, use the painter tool:
   ```
   painter: {
     "prompt": "Solid color swatch, single color fill #[HEX], 400x400 pixels, 
               no gradients, no textures, pure flat color, square format"
   }
   ```

3. Save swatches with consistent naming convention.

**Swatch naming convention:**

| Role | Filename Pattern | Example |
|------|------------------|---------|
| Primary | `primary-[name].png` | `primary-trust-blue.png` |
| Secondary | `secondary-[name].png` | `secondary-growth-green.png` |
| Accent | `accent-[name].png` | `accent-energy-gold.png` |
| Light neutral | `neutral-light-[name].png` | `neutral-light-cloud-white.png` |
| Dark neutral | `neutral-dark-[name].png` | `neutral-dark-charcoal.png` |

**Naming rules:**
- Lowercase only
- Hyphens instead of spaces
- Role prefix for easy sorting
- Brand name suffix from Color Definition (4.3)

**Complete swatch set example:**
```
./branding/swatches/
├── primary-trust-blue.png
├── secondary-growth-green.png
├── accent-energy-gold.png
├── neutral-light-cloud-white.png
└── neutral-dark-charcoal.png
```

**Swatch generation prompts by palette size:**

| Palette Size | Files Generated |
|--------------|-----------------|
| 5 colors | 5 swatch files |
| 7 colors | 7 swatch files |
| 10 colors | 10 swatch files |

**Validation checklist:**
- [ ] All approved colors have corresponding swatch files
- [ ] Filenames follow naming convention (lowercase, hyphens, role prefix)
- [ ] All files are in ./branding/swatches/ directory
- [ ] Swatch colors match hex codes exactly
- [ ] No duplicate filenames

**Inform user upon completion:**
```
Visual color swatches generated in ./branding/swatches/

Files created:
- primary-[name].png (#XXXXXX)
- secondary-[name].png (#XXXXXX)
- accent-[name].png (#XXXXXX)
- neutral-light-[name].png (#XXXXXX)
- neutral-dark-[name].png (#XXXXXX)

Proceeding to typography selection...
```

---

### Phase 5: Font Guidelines Generation

Generate a complete typography system with 4 Google Font recommendations based on research findings and brand personality.

#### 5.1 Font Category Selection

**Present the 4 font categories to user:**

```
Your brand needs fonts for 4 purposes. I'll recommend Google Fonts for each:

1. **Heading font** — Used for page titles, section headers, hero text
2. **Body font** — Used for paragraphs, descriptions, general content
3. **Accent font** — Used for callouts, quotes, special emphasis
4. **Display font** — Used for logos, large decorative text, marketing materials

Would you like me to proceed with recommendations based on your brand personality ([traits from Q4])?
```

**Font category requirements:**

| Category | Key Requirements | Typical Styles |
|----------|-----------------|----------------|
| Heading | Bold weights, distinctive character | Sans-serif, Slab serif, Display |
| Body | High readability, multiple weights | Sans-serif, Serif |
| Accent | Personality-forward, limited use | Script, Display, Decorative |
| Display | Maximum impact, brand recognition | Display, Custom, Decorative |

#### 5.2 Google Fonts Research Workflow

**Build on Phase 3.1 research** — reference the font direction already explored.

1. For each font category, search with brand context:
   ```
   web_search: "Google Fonts best [category] font [brand personality]"
   Example: "Google Fonts best heading font modern minimalist"
   ```

2. Browse Google Fonts directly for specific styles:
   ```
   read_web_page: "https://fonts.google.com/?category=Sans+Serif"
   objective: "Find [style] fonts suitable for [category] with [personality traits]"
   ```

3. Research font pairings that work together:
   ```
   web_search: "[heading font name] font pairing suggestions"
   Example: "Playfair Display font pairing suggestions"
   ```

4. Verify font availability and weights:
   ```
   read_web_page: "https://fonts.google.com/specimen/[FontName]"
   objective: "Check available weights, styles, and language support"
   ```

**Validation checklist for each font:**
- [ ] Available on Google Fonts (free commercial use)
- [ ] Has required weights (regular, medium, bold minimum)
- [ ] Supports brand's target languages
- [ ] Pairs well with other selected fonts
- [ ] Matches brand personality traits

#### 5.3 Font Definition Template

**For each font, capture:**

| Field | Description | Example |
|-------|-------------|---------|
| Name | Google Fonts name | Playfair Display |
| Category | Heading/Body/Accent/Display | Heading |
| Style | Serif/Sans-serif/Script/Display | Serif |
| Personality match | How it reflects brand traits | Elegant, sophisticated, timeless |
| Recommended weights | Which weights to use | Regular (400), Bold (700) |
| Pairing rationale | Why it works with other fonts | Contrasts well with Inter's clean geometry |

#### 5.4 Usage Rules Template

**Define usage rules for each font category:**

```markdown
## Heading Font: [Name]

**When to use:**
- Page titles and H1 headings
- Section headers (H2, H3)
- Hero text and taglines
- Navigation labels

**Sizing guidelines:**
- H1: 48-64px (desktop), 32-40px (mobile)
- H2: 32-40px (desktop), 24-28px (mobile)
- H3: 24-28px (desktop), 20-24px (mobile)

**Weight usage:**
- Bold (700): Primary headings
- Medium (500): Subheadings
- Regular (400): Navigation, labels

**Do:**
- Use for short, impactful text
- Maintain consistent hierarchy
- Allow adequate line height (1.2-1.3)

**Don't:**
- Use for body paragraphs
- Mix with display font in same context
- Use light weights for headings
```

**Repeat for each font category with appropriate rules.**

#### 5.5 Typography Output Format

Present the complete typography system to user for approval:

```
## Your Brand Typography System

### Font Stack Overview
| Category | Font | Style | Primary Use |
|----------|------|-------|-------------|
| Heading | Playfair Display | Serif | Titles, headers |
| Body | Inter | Sans-serif | Paragraphs, content |
| Accent | Lora | Serif italic | Quotes, callouts |
| Display | Bebas Neue | Display | Logo, marketing |

### Font Pairing Rationale
[2-3 sentences explaining why these fonts work together and support the brand personality]

### Heading Font: [Name]
- **Style:** [Serif/Sans-serif]
- **Weights:** [List weights to use]
- **Personality:** [How it reflects brand]
- **Usage:** [Primary use cases]

### Body Font: [Name]
- **Style:** [Serif/Sans-serif]
- **Weights:** [List weights to use]
- **Personality:** [How it reflects brand]
- **Usage:** [Primary use cases]

### Accent Font: [Name]
- **Style:** [Serif/Sans-serif/Script]
- **Weights:** [List weights to use]
- **Personality:** [How it reflects brand]
- **Usage:** [Primary use cases]

### Display Font: [Name]
- **Style:** [Display/Decorative]
- **Weights:** [List weights to use]
- **Personality:** [How it reflects brand]
- **Usage:** [Primary use cases]

### Implementation
Google Fonts import link:
\`\`\`html
<link href="https://fonts.googleapis.com/css2?family=[Font1]&family=[Font2]&display=swap" rel="stylesheet">
\`\`\`

CSS variables:
\`\`\`css
:root {
  --font-heading: '[Heading Font]', serif;
  --font-body: '[Body Font]', sans-serif;
  --font-accent: '[Accent Font]', serif;
  --font-display: '[Display Font]', sans-serif;
}
\`\`\`

Does this typography system align with your brand vision? Any fonts to adjust?
```

#### 5.6 Iteration Handling

**If user requests changes:**
- "Too formal" → Research more casual, friendly font alternatives
- "Too playful" → Research more professional, structured fonts
- "Hard to read" → Prioritize readability metrics, increase x-height preference
- "Doesn't match [competitor]" → Research what fonts competitor uses, find differentiated alternatives
- "Need more variety" → Research additional weights or complementary fonts

**Common font swaps by feedback:**

| Feedback | Current Style | Try Instead |
|----------|--------------|-------------|
| "Too corporate" | Geometric sans | Humanist sans, Rounded sans |
| "Too trendy" | Display, Script | Classic serif, Grotesque sans |
| "Too traditional" | Serif | Modern sans, Slab serif |
| "Low contrast" | Similar x-heights | Serif + Sans pairing |

**User approval required** before proceeding to Phase 6.

---

### Phase 6: Brand Guidelines Assembly

Compile all brand elements from Phases 4-5 into comprehensive guidelines with clear usage rules for every context.

#### 6.1 Logo Usage Guidelines

**Build on questionnaire responses** — reference Q16 (logo inspiration), Q20 (existing assets), Q31 (current logo).

**Present logo usage framework to user:**

```
Before I create your brand guidelines, let's define logo usage rules.

Do you have an existing logo, or will one be designed separately?
- **Existing logo** — I'll create placement and usage rules for it
- **Logo pending** — I'll create placeholder guidelines for future logo
```

**Logo usage template:**

```markdown
## Logo Usage Guidelines

### Primary Logo
[Description or placeholder for primary logo]

### Logo Variations
| Variation | Use Case | Background |
|-----------|----------|------------|
| Full color | Primary use, marketing | Light backgrounds |
| Reversed | Dark backgrounds | Dark backgrounds |
| Monochrome | Single-color printing | Any |
| Icon only | Favicons, app icons, small spaces | Any |

### Clear Space Requirements
- Minimum clear space: Equal to the height of the logo mark (or "X" unit)
- No text, graphics, or other elements within the clear space zone
- Clear space applies to all edges of the logo

### Minimum Size Requirements
| Format | Minimum Width |
|--------|---------------|
| Print | 1 inch (25mm) |
| Digital | 120px |
| Favicon | 32px (icon only) |

### Logo Placement

**Preferred positions:**
- Top-left for headers/navigation
- Center for hero sections and marketing
- Bottom-center for footers

**Placement rules by context:**
| Context | Position | Size | Variation |
|---------|----------|------|-----------|
| Website header | Top-left | 40-60px height | Full color |
| Email signature | Bottom-left | 80-120px width | Full color |
| Social media profile | Center | Platform specs | Full color or icon |
| Business cards | Top-left or center | 0.75-1 inch | Full color |
| Presentations | Title slide center, footer | Per template | Full color |

### Logo Don'ts
- ❌ Do not stretch, distort, or rotate the logo
- ❌ Do not change logo colors outside approved palette
- ❌ Do not add effects (shadows, gradients, outlines)
- ❌ Do not place on busy backgrounds without sufficient contrast
- ❌ Do not recreate or redraw the logo
- ❌ Do not use outdated versions of the logo
```

#### 6.2 Color Usage Contexts

**Build on Phase 4 color palette** — reference the colors already approved by user.

**Define color roles and contexts:**

```markdown
## Color Usage Contexts

### Color Hierarchy
| Role | Color | Usage Percentage |
|------|-------|------------------|
| Primary | [Primary color] | 60% of brand visuals |
| Secondary | [Secondary color] | 30% of brand visuals |
| Accent | [Accent color] | 10% for emphasis |

### Context-Specific Usage

#### Digital Applications
| Element | Color | Hex | Notes |
|---------|-------|-----|-------|
| Primary buttons | Primary | #XXXXXX | Call-to-action buttons |
| Secondary buttons | Secondary | #XXXXXX | Less prominent actions |
| Links | Primary or Accent | #XXXXXX | Ensure contrast |
| Backgrounds | Light neutral | #XXXXXX | Main content areas |
| Headers/Footers | Dark neutral | #XXXXXX | Navigation, footer |
| Hover states | Primary (darkened 10%) | #XXXXXX | Interactive feedback |
| Error states | Red variant | #DC2626 | Form validation |
| Success states | Green variant | #16A34A | Confirmations |

#### Print Applications
| Material | Primary Color Use | Secondary Color Use | Notes |
|----------|------------------|---------------------|-------|
| Business cards | Logo, name | Contact details | White background |
| Letterhead | Header, logo | Accent lines | Conservative use |
| Brochures | Headlines, CTAs | Body accents | Balance with white space |
| Packaging | Dominant brand color | Supporting elements | Consider shelf impact |

#### Social Media
| Platform | Profile | Cover/Banner | Posts |
|----------|---------|--------------|-------|
| LinkedIn | Logo on brand color | Brand gradient or primary | Professional, muted |
| Instagram | Logo icon | Brand imagery | Vibrant, on-brand |
| Twitter/X | Logo icon | Brand pattern or color | Consistent with posts |
| Facebook | Logo icon | Brand imagery | Balanced color use |

### Color Pairing Rules

**Allowed combinations:**
- Primary + Light neutral (high contrast content)
- Secondary + Dark neutral (subtle headers)
- Accent + Primary (emphasis within primary content)
- Light neutral + Dark neutral (clean, minimal layouts)

**Avoid combinations:**
- Primary + Secondary directly adjacent (unless tested for harmony)
- Accent + Accent (never use multiple accents together)
- Any combination that fails WCAG 2.1 AA contrast requirements

### Accessibility Requirements
| Combination | Minimum Contrast | Use Case |
|-------------|------------------|----------|
| Body text on background | 4.5:1 | All body copy |
| Large text on background | 3:1 | Headlines 18px+ |
| UI components | 3:1 | Buttons, inputs |
| Non-text elements | 3:1 | Icons, borders |
```

#### 6.3 Font Pairing Rules

**Build on Phase 5 typography system** — reference the 4 fonts already approved.

**Define pairing hierarchies:**

```markdown
## Font Pairing Rules

### The Font Stack
| Role | Font | Fallback Stack |
|------|------|----------------|
| Heading | [Heading font] | Georgia, serif |
| Body | [Body font] | Arial, sans-serif |
| Accent | [Accent font] | Georgia, serif |
| Display | [Display font] | Impact, sans-serif |

### Pairing Combinations

**Approved pairings (use these):**
| Heading | Body | Context |
|---------|------|---------|
| [Heading font] | [Body font] | Standard content pages |
| [Display font] | [Body font] | Marketing materials, landing pages |
| [Heading font] | [Accent font] | Editorial, blog posts |

**Avoid these pairings:**
- Display + Accent (too decorative, reduces readability)
- Two fonts from the same category in adjacent elements
- More than 2 fonts on a single page/screen

### Hierarchy Rules

**Content hierarchy pattern:**

```
H1 (Hero)     → Display font, 48-64px, Bold
H1 (Page)     → Heading font, 36-48px, Bold  
H2 (Section)  → Heading font, 28-32px, Bold
H3 (Subsect)  → Heading font, 22-26px, Semi-bold
Body          → Body font, 16-18px, Regular
Caption       → Body font, 12-14px, Regular
Accent text   → Accent font, 16-20px, Italic
```

**Line height by context:**
| Element | Line Height | Letter Spacing |
|---------|-------------|----------------|
| Headlines | 1.1-1.2 | -0.02em to 0 |
| Body text | 1.5-1.7 | 0 |
| Captions | 1.4-1.5 | 0.02em |
| Buttons | 1.2 | 0.05em |

### Context-Specific Typography

#### Website
| Element | Font | Size | Weight |
|---------|------|------|--------|
| Navigation | Body | 14-16px | Medium (500) |
| Hero headline | Display | 48-72px | Bold (700) |
| Section titles | Heading | 32-40px | Bold (700) |
| Body copy | Body | 16-18px | Regular (400) |
| Buttons | Body | 14-16px | Semi-bold (600) |
| Footer links | Body | 14px | Regular (400) |

#### Print Materials
| Element | Font | Size | Weight |
|---------|------|------|--------|
| Document titles | Heading | 24-32pt | Bold |
| Section heads | Heading | 18-24pt | Bold |
| Body copy | Body | 10-12pt | Regular |
| Captions | Body | 8-10pt | Regular |
| Pull quotes | Accent | 14-18pt | Italic |

#### Presentations
| Element | Font | Size | Weight |
|---------|------|------|--------|
| Slide titles | Heading | 36-44pt | Bold |
| Bullet points | Body | 24-28pt | Regular |
| Annotations | Body | 18-20pt | Regular |
```

#### 6.4 Visual Identity Rules

**Define overall visual language beyond colors and fonts:**

```markdown
## Visual Identity Rules

### Photography Style

**Based on brand personality ([traits from Q4]):**

| Attribute | Guideline |
|-----------|-----------|
| Mood | [Derived from Q6 emotional triggers] |
| Lighting | Natural/studio, bright/moody |
| Color treatment | True color, warm filter, desaturated |
| Subjects | People, products, abstract, lifestyle |
| Composition | Centered, rule-of-thirds, asymmetric |

**Photography dos:**
- ✓ Use images that reflect brand personality
- ✓ Maintain consistent color grading across all imagery
- ✓ Feature diverse, authentic subjects when using people
- ✓ Ensure adequate resolution (min 72dpi web, 300dpi print)

**Photography don'ts:**
- ✗ Generic stock photos with forced smiles
- ✗ Inconsistent filters or color treatments
- ✗ Busy, cluttered compositions
- ✗ Low-quality or pixelated images

### Iconography

| Attribute | Guideline |
|-----------|-----------|
| Style | Line/solid, rounded/sharp, thick/thin |
| Stroke weight | Consistent (e.g., 2px for all icons) |
| Corner radius | Match brand style (e.g., 4px rounded) |
| Color | Primary, secondary, or neutral only |
| Size | Multiples of 8px (16, 24, 32, 48px) |

### Spacing and Layout

**Spacing scale (8px base unit):**
| Token | Value | Use |
|-------|-------|-----|
| xs | 4px | Inline elements, tight groups |
| sm | 8px | Related elements, list items |
| md | 16px | Standard component spacing |
| lg | 24px | Section padding, card margins |
| xl | 32px | Major section breaks |
| 2xl | 48px | Page sections |
| 3xl | 64px | Hero spacing, major divisions |

**Layout principles:**
- Use consistent grid system (12-column recommended)
- Maintain generous white space (don't crowd elements)
- Align elements to grid for visual harmony
- Group related content with consistent spacing

### Graphic Elements

**Patterns and textures:**
- [Define if brand uses patterns — derived from Q19, Q34]
- Usage: backgrounds, section dividers, accent areas
- Never compete with content for attention

**Shapes and containers:**
| Element | Style | Border Radius | Border |
|---------|-------|---------------|--------|
| Cards | Subtle shadow | 8-12px | None or 1px neutral |
| Buttons | Solid fill | 4-8px | None |
| Input fields | Outlined | 4px | 1px neutral |
| Tags/badges | Solid or outlined | 4px or full | Optional 1px |

### Brand Collateral Templates

**Required templates for consistency:**
| Template | Key Elements |
|----------|--------------|
| Email signature | Logo, name, title, contact, social icons |
| Presentation | Title slide, content slide, section divider |
| Social post | Brand colors, logo placement, text overlay |
| Document | Letterhead, header, footer with logo |
```

#### 6.5 Brand Guidelines Confirmation

**Present compiled guidelines summary to user:**

```
## Brand Guidelines Summary

I've compiled your brand guidelines covering:

✓ **Logo Usage** — Variations, clear space, minimum sizes, placement rules
✓ **Color Contexts** — Digital, print, social media, accessibility requirements
✓ **Font Pairing** — Approved combinations, hierarchy, sizing by context
✓ **Visual Identity** — Photography, iconography, spacing, graphic elements

Before I generate the final files, please confirm:
1. Do the logo usage rules match your needs?
2. Are the color usage contexts complete for your applications?
3. Do the font pairing rules align with your vision?
4. Any visual identity elements to add or modify?

Once approved, I'll generate:
- `./branding/brand-guidelines.md` — Complete brand guidelines
- `./branding/brand-summary.txt` — Executive summary
- `./branding/swatches/` — Visual color swatches
```

#### 6.6 Iteration Handling

**If user requests changes:**
- "Need more contexts" → Add specific use cases (signage, merchandise, video)
- "Too restrictive" → Relax rules, provide more flexibility language
- "Not restrictive enough" → Add more specific don'ts and requirements
- "Missing [element]" → Add subsection for requested element

**User approval required** before proceeding to file generation (Tasks 8-10).

#### 6.7 Markdown File Generation

**After user approval in 6.5**, generate `./branding/brand-guidelines.md` using this template:

```markdown
# [Brand Name] Brand Guidelines

*Version 1.0 | Generated [Date]*

---

## Table of Contents

1. [Brand Overview](#brand-overview)
2. [Logo Usage](#logo-usage)
3. [Color Palette](#color-palette)
4. [Typography](#typography)
5. [Visual Identity](#visual-identity)
6. [Application Guidelines](#application-guidelines)
7. [Quick Reference](#quick-reference)

---

## Brand Overview

### Brand Essence
[3-5 sentences summarizing brand personality, derived from Q4, Q5, Q6]

### Mission & Values
[From Q3 mission statement and core values]

### Brand Personality
| Trait | Description |
|-------|-------------|
| [Trait 1 from Q4] | How this manifests in brand expression |
| [Trait 2 from Q4] | How this manifests in brand expression |
| [Trait 3 from Q4] | How this manifests in brand expression |

### Target Audience
[Summary from Q7-Q9: ideal customer, where brand fits, problems solved]

---

## Logo Usage

### Primary Logo
[Insert logo or placeholder description]

### Logo Variations
| Variation | Use Case | File Name |
|-----------|----------|-----------|
| Full color | Primary use, light backgrounds | logo-full-color.svg |
| Reversed | Dark backgrounds | logo-reversed.svg |
| Monochrome | Single-color printing | logo-mono.svg |
| Icon only | Favicons, app icons, small spaces | logo-icon.svg |

### Clear Space
Minimum clear space equals the height of the logo mark on all sides.

![Clear space diagram placeholder]

### Minimum Sizes
| Format | Minimum Width |
|--------|---------------|
| Print | 1 inch (25mm) |
| Digital | 120px |
| Favicon | 32px (icon only) |

### Logo Placement by Context
| Context | Position | Size | Variation |
|---------|----------|------|-----------|
| Website header | Top-left | 40-60px height | Full color |
| Email signature | Bottom-left | 80-120px width | Full color |
| Social media profile | Center | Platform specs | Full color or icon |
| Business cards | Top-left or center | 0.75-1 inch | Full color |
| Presentations | Title slide center, footer | Per template | Full color |

### Logo Don'ts
- ❌ Do not stretch, distort, or rotate the logo
- ❌ Do not change logo colors outside approved palette
- ❌ Do not add effects (shadows, gradients, outlines)
- ❌ Do not place on busy backgrounds without sufficient contrast
- ❌ Do not recreate or redraw the logo
- ❌ Do not use outdated versions of the logo

---

## Color Palette

### Primary Colors
| Color | Hex | RGB | Role |
|-------|-----|-----|------|
| [Name] | #XXXXXX | rgb(X, X, X) | Primary brand color |
| [Name] | #XXXXXX | rgb(X, X, X) | Secondary brand color |

### Accent Colors
| Color | Hex | RGB | Role |
|-------|-----|-----|------|
| [Name] | #XXXXXX | rgb(X, X, X) | Call-to-action, emphasis |

### Neutral Colors
| Color | Hex | RGB | Role |
|-------|-----|-----|------|
| [Name] | #XXXXXX | rgb(X, X, X) | Backgrounds |
| [Name] | #XXXXXX | rgb(X, X, X) | Text |
| [Name] | #XXXXXX | rgb(X, X, X) | Borders |

### Color Hierarchy
| Role | Color | Usage Percentage |
|------|-------|------------------|
| Primary | [Primary color] | 60% of brand visuals |
| Secondary | [Secondary color] | 30% of brand visuals |
| Accent | [Accent color] | 10% for emphasis |

### Digital Color Usage
| Element | Color | Hex | Notes |
|---------|-------|-----|-------|
| Primary buttons | Primary | #XXXXXX | Call-to-action |
| Secondary buttons | Secondary | #XXXXXX | Less prominent actions |
| Links | Primary or Accent | #XXXXXX | Ensure contrast |
| Backgrounds | Light neutral | #XXXXXX | Main content areas |
| Headers/Footers | Dark neutral | #XXXXXX | Navigation, footer |
| Hover states | Primary (darkened 10%) | #XXXXXX | Interactive feedback |
| Error states | Red | #DC2626 | Form validation |
| Success states | Green | #16A34A | Confirmations |

### Print Color Usage
| Material | Primary | Secondary | Notes |
|----------|---------|-----------|-------|
| Business cards | Logo, name | Contact details | White background |
| Letterhead | Header, logo | Accent lines | Conservative use |
| Brochures | Headlines, CTAs | Body accents | Balance with white space |
| Packaging | Dominant brand | Supporting elements | Consider shelf impact |

### Accessibility Requirements
| Combination | Minimum Contrast | Use Case |
|-------------|------------------|----------|
| Body text on background | 4.5:1 | All body copy |
| Large text on background | 3:1 | Headlines 18px+ |
| UI components | 3:1 | Buttons, inputs |
| Non-text elements | 3:1 | Icons, borders |

---

## Typography

### Font Stack
| Category | Font | Fallback | Primary Use |
|----------|------|----------|-------------|
| Heading | [Font name] | Georgia, serif | Titles, headers |
| Body | [Font name] | Arial, sans-serif | Paragraphs, content |
| Accent | [Font name] | Georgia, serif | Quotes, callouts |
| Display | [Font name] | Impact, sans-serif | Logo, marketing |

### Google Fonts Implementation

```html
<link href="https://fonts.googleapis.com/css2?family=[Font1]:wght@400;500;700&family=[Font2]:wght@400;600&display=swap" rel="stylesheet">
```

```css
:root {
  --font-heading: '[Heading Font]', Georgia, serif;
  --font-body: '[Body Font]', Arial, sans-serif;
  --font-accent: '[Accent Font]', Georgia, serif;
  --font-display: '[Display Font]', Impact, sans-serif;
}
```

### Type Scale — Web
| Element | Font | Size | Weight | Line Height |
|---------|------|------|--------|-------------|
| H1 (Hero) | Display | 48-64px | Bold (700) | 1.1 |
| H1 (Page) | Heading | 36-48px | Bold (700) | 1.2 |
| H2 | Heading | 28-32px | Bold (700) | 1.2 |
| H3 | Heading | 22-26px | Semi-bold (600) | 1.2 |
| Body | Body | 16-18px | Regular (400) | 1.6 |
| Caption | Body | 12-14px | Regular (400) | 1.5 |
| Button | Body | 14-16px | Semi-bold (600) | 1.2 |

### Type Scale — Print
| Element | Font | Size | Weight |
|---------|------|------|--------|
| Document titles | Heading | 24-32pt | Bold |
| Section heads | Heading | 18-24pt | Bold |
| Body copy | Body | 10-12pt | Regular |
| Captions | Body | 8-10pt | Regular |
| Pull quotes | Accent | 14-18pt | Italic |

### Approved Font Pairings
| Heading | Body | Context |
|---------|------|---------|
| [Heading font] | [Body font] | Standard content pages |
| [Display font] | [Body font] | Marketing materials, landing pages |
| [Heading font] | [Accent font] | Editorial, blog posts |

### Typography Don'ts
- ❌ Do not use more than 2 fonts on a single page
- ❌ Do not pair Display + Accent fonts together
- ❌ Do not use heading fonts for body paragraphs
- ❌ Do not use light weights for headlines

---

## Visual Identity

### Photography Style
| Attribute | Guideline |
|-----------|-----------|
| Mood | [From Q6 emotional triggers] |
| Lighting | [Natural/studio, bright/moody] |
| Color treatment | [True color, warm filter, desaturated] |
| Subjects | [People, products, abstract, lifestyle] |
| Composition | [Centered, rule-of-thirds, asymmetric] |

**Photography Dos:**
- ✓ Use images that reflect brand personality
- ✓ Maintain consistent color grading
- ✓ Feature diverse, authentic subjects
- ✓ Ensure adequate resolution (72dpi web, 300dpi print)

**Photography Don'ts:**
- ✗ Generic stock photos with forced expressions
- ✗ Inconsistent filters or treatments
- ✗ Busy, cluttered compositions
- ✗ Low-quality or pixelated images

### Iconography
| Attribute | Guideline |
|-----------|-----------|
| Style | [Line/solid, rounded/sharp] |
| Stroke weight | [Consistent, e.g., 2px] |
| Corner radius | [Match brand style, e.g., 4px] |
| Color | Primary, secondary, or neutral only |
| Size | Multiples of 8px (16, 24, 32, 48px) |

### Spacing Scale
| Token | Value | Use |
|-------|-------|-----|
| xs | 4px | Inline elements, tight groups |
| sm | 8px | Related elements, list items |
| md | 16px | Standard component spacing |
| lg | 24px | Section padding, card margins |
| xl | 32px | Major section breaks |
| 2xl | 48px | Page sections |
| 3xl | 64px | Hero spacing, major divisions |

### Graphic Elements
| Element | Style | Border Radius | Notes |
|---------|-------|---------------|-------|
| Cards | Subtle shadow | 8-12px | None or 1px neutral border |
| Buttons | Solid fill | 4-8px | No border |
| Input fields | Outlined | 4px | 1px neutral border |
| Tags/badges | Solid or outlined | 4px or full | Optional 1px border |

---

## Application Guidelines

### Social Media
| Platform | Profile | Cover/Banner | Posts |
|----------|---------|--------------|-------|
| LinkedIn | Logo on brand color | Brand gradient or primary | Professional, muted |
| Instagram | Logo icon | Brand imagery | Vibrant, on-brand |
| Twitter/X | Logo icon | Brand pattern or color | Consistent with posts |
| Facebook | Logo icon | Brand imagery | Balanced color use |

### Email Signature
```
[Employee Name]
[Title]

[Company Logo - 80-120px width]

[Phone] | [Email]
[Website]
```

### Presentation Templates
| Slide Type | Key Elements |
|------------|--------------|
| Title slide | Centered display font, logo, brand colors |
| Content slide | Heading font for titles, body font for content |
| Section divider | Brand color background, display font |

---

## Quick Reference

### Brand at a Glance
| Element | Value |
|---------|-------|
| Primary Color | #XXXXXX |
| Secondary Color | #XXXXXX |
| Accent Color | #XXXXXX |
| Heading Font | [Font name] |
| Body Font | [Font name] |
| Logo Min Size (Digital) | 120px |
| Logo Min Size (Print) | 1 inch |

### Do's and Don'ts Summary

**Always:**
- ✓ Use approved color palette
- ✓ Maintain logo clear space
- ✓ Follow typography hierarchy
- ✓ Ensure WCAG accessibility

**Never:**
- ✗ Distort or rotate logo
- ✗ Use unapproved colors
- ✗ Mix more than 2 fonts per page
- ✗ Ignore minimum size requirements

---

*These guidelines ensure consistent brand expression across all touchpoints. For questions or exceptions, contact [brand owner/team].*
```

**File generation workflow:**

1. Copy template above
2. Replace all placeholders `[...]` with actual values from questionnaire and Phase 4-6 outputs
3. Replace `#XXXXXX` hex codes with actual palette colors
4. Update font names with Phase 5 selections
5. Remove placeholder image references if no assets exist
6. Save to `./branding/brand-guidelines.md`

**Validation checklist before saving:**
- [ ] All placeholders replaced with actual values
- [ ] Brand name appears in title and overview
- [ ] All colors from Phase 4 palette included
- [ ] All fonts from Phase 5 included with correct names
- [ ] Logo usage rules match Phase 6.1
- [ ] Color contexts match Phase 6.2
- [ ] Font pairing rules match Phase 6.3
- [ ] Visual identity rules match Phase 6.4
- [ ] No orphaned placeholder text (`[...]`, `#XXXXXX`, `[Font name]`)

#### 6.8 Text File Generation (Executive Summary)

Generate a concise executive summary for stakeholders who need key brand decisions at a glance.

**Purpose:** Unlike the comprehensive brand-guidelines.md, this summary highlights only the most critical decisions for quick reference by executives, partners, and vendors.

**File location:** `./branding/brand-summary.txt`

**Template:**

```
================================================================================
                         [BRAND NAME] — BRAND SUMMARY
================================================================================

Generated: [Date]
Version: 1.0

--------------------------------------------------------------------------------
BRAND IDENTITY
--------------------------------------------------------------------------------

Mission:
[One sentence mission statement from Q3]

Brand Personality (3 words):
[From Q4]

Emotional Triggers:
[From Q6 — what customers should feel]

Target Customer:
[One sentence summary from Q7]

Unique Value Proposition:
[From Q24 — what makes the brand different]

--------------------------------------------------------------------------------
VISUAL IDENTITY
--------------------------------------------------------------------------------

PRIMARY COLORS
  Primary:    [Color name] — #XXXXXX
  Secondary:  [Color name] — #XXXXXX
  Accent:     [Color name] — #XXXXXX

NEUTRAL COLORS
  Light:      [Color name] — #XXXXXX
  Dark:       [Color name] — #XXXXXX

TYPOGRAPHY
  Headings:   [Font name] (Google Fonts)
  Body:       [Font name] (Google Fonts)
  Accent:     [Font name] (Google Fonts)
  Display:    [Font name] (Google Fonts)

VISUAL STYLE
  [2-3 word description from Q19, e.g., "Modern, minimal, bold"]

--------------------------------------------------------------------------------
KEY DECISIONS
--------------------------------------------------------------------------------

1. [Critical decision #1 — e.g., "Blue primary color chosen for trust and professionalism"]
2. [Critical decision #2 — e.g., "Sans-serif typography for modern, accessible feel"]
3. [Critical decision #3 — e.g., "Warm accent color to balance corporate palette"]
4. [Critical decision #4 — e.g., "Photography style: authentic, natural lighting"]
5. [Critical decision #5 — e.g., "Tone of voice: friendly but professional"]

--------------------------------------------------------------------------------
QUICK RULES
--------------------------------------------------------------------------------

✓ DO:
  • Use primary color for CTAs and key actions
  • Maintain logo clear space (equal to logo height)
  • Follow heading/body font pairing for all content
  • Ensure 4.5:1 contrast ratio for body text

✗ DON'T:
  • Distort, rotate, or recolor the logo
  • Use more than 2 fonts on a single page
  • Place logo on busy backgrounds
  • Use unapproved color combinations

--------------------------------------------------------------------------------
FILES & RESOURCES
--------------------------------------------------------------------------------

Full Brand Guidelines:  ./branding/brand-guidelines.md
Color Swatches:         ./branding/swatches/
Google Fonts Link:      https://fonts.google.com/share?selection.family=[fonts]

--------------------------------------------------------------------------------
NEXT STEPS
--------------------------------------------------------------------------------

1. [ ] Review and approve brand guidelines
2. [ ] Create/finalize logo based on guidelines
3. [ ] Set up brand assets in design tools
4. [ ] Brief team and external partners
5. [ ] Apply brand to priority touchpoints

================================================================================
                    For full guidelines, see brand-guidelines.md
================================================================================
```

**Generating the summary:**

1. Extract key values from questionnaire responses (Q3, Q4, Q6, Q7, Q24)
2. Pull color palette from Phase 4 output
3. Pull font selections from Phase 5 output
4. Identify 5 most critical decisions from the branding process
5. Save to `./branding/brand-summary.txt`

**Key decisions selection criteria:**

Choose decisions that:
- Differentiate this brand from competitors
- Required significant discussion or iteration
- Impact multiple touchpoints
- Represent strategic trade-offs

**Content guidelines:**
- Keep each line under 80 characters for readability
- Use plain text formatting (no markdown)
- Include hex codes for easy copy/paste
- Avoid jargon — executives may not know design terms

**Validation checklist:**
- [ ] Brand name in header
- [ ] All placeholders replaced
- [ ] All hex codes are valid 6-character codes
- [ ] Font names match Google Fonts exactly
- [ ] Key decisions are specific, not generic
- [ ] Next steps are actionable
- [ ] File paths are correct relative paths

---

### Phase 7: Evaluation and Approval

After generating all brand assets, systematically evaluate output quality and alignment with user requirements before final delivery.

#### 7.1 Output Completeness Checklist

**Verify all required files exist:**

| File | Location | Status |
|------|----------|--------|
| Brand Guidelines | `./branding/brand-guidelines.md` | [ ] Exists |
| Executive Summary | `./branding/brand-summary.txt` | [ ] Exists |
| Primary Color Swatch | `./branding/swatches/primary-*.png` | [ ] Exists |
| Secondary Color Swatch | `./branding/swatches/secondary-*.png` | [ ] Exists |
| Accent Color Swatch | `./branding/swatches/accent-*.png` | [ ] Exists |
| Neutral Swatches | `./branding/swatches/neutral-*.png` | [ ] Exist |

**Verify content completeness:**

| Section | Requirement | Check |
|---------|-------------|-------|
| Brand Overview | Mission, values, personality defined | [ ] |
| Color Palette | All colors have hex, RGB, rationale | [ ] |
| Typography | 4 fonts with usage rules | [ ] |
| Logo Guidelines | Placement, sizing, don'ts documented | [ ] |
| Visual Identity | Photography, icons, spacing defined | [ ] |
| Quick Reference | Summary table present | [ ] |

#### 7.2 Questionnaire Alignment Verification

**Compare outputs against user's original answers:**

| Question | User Answer | Output Alignment | Match? |
|----------|-------------|------------------|--------|
| Q4: Brand personality (3 words) | [recorded answer] | Reflected in color rationale, font choices | [ ] |
| Q6: Emotional triggers | [recorded answer] | Color psychology aligns | [ ] |
| Q13: Brand tone | [recorded answer] | Typography personality matches | [ ] |
| Q17: Color preferences | [recorded answer] | Palette includes preferences, avoids excluded colors | [ ] |
| Q18: Typography preferences | [recorded answer] | Font selections align | [ ] |
| Q19: Visual style | [recorded answer] | Overall visual identity matches | [ ] |
| Q24: Unique value proposition | [recorded answer] | Differentiation visible in guidelines | [ ] |

**Alignment scoring:**
- **Strong match (✓):** Output directly reflects user input
- **Partial match (~):** Output somewhat reflects input, may need refinement
- **Mismatch (✗):** Output contradicts or ignores user input

**Minimum threshold:** All critical questions (Q4, Q6, Q13, Q17, Q19) must be Strong or Partial match.

#### 7.3 Quality Standards Verification

**Color Palette Quality:**
- [ ] Primary color is distinctive and memorable
- [ ] Color roles are clear (primary, secondary, accent, neutrals)
- [ ] Accessibility: Text colors meet WCAG 2.1 AA contrast (4.5:1)
- [ ] Colors work together harmoniously
- [ ] Rationale explains why each color was chosen

**Typography Quality:**
- [ ] Heading and body fonts pair well together
- [ ] All fonts available on Google Fonts
- [ ] Font weights specified (regular, medium, bold minimum)
- [ ] Sizing guidelines are practical (not too small, not too large)
- [ ] Fallback fonts defined

**Guidelines Quality:**
- [ ] No orphaned placeholders (`[...]` or `#XXXXXX`)
- [ ] All hex codes are valid 6-character format
- [ ] Rules are specific and actionable, not generic
- [ ] Context-specific guidance provided (web, print, social)
- [ ] Do's and Don'ts are clear

#### 7.4 User Approval Checkpoint

**Present evaluation summary to user:**

```
## Brand Guidelines Evaluation

I've completed your brand guidelines. Here's my evaluation:

### Completeness
✓ All 3 output files generated
✓ [X] color swatches created
✓ All required sections present

### Alignment with Your Vision
| Your Input | How It's Reflected |
|------------|-------------------|
| Personality: [3 words from Q4] | [How colors/fonts reflect this] |
| Emotional triggers: [from Q6] | [How palette evokes this] |
| Visual style: [from Q19] | [How overall identity matches] |

### Quality Check
✓ Accessibility requirements met
✓ All placeholders replaced
✓ Actionable guidelines provided

---

**Please review the generated files:**
- `./branding/brand-guidelines.md` — Full brand guidelines
- `./branding/brand-summary.txt` — Executive summary
- `./branding/swatches/` — Visual color swatches

**Do you approve these brand guidelines?**
1. **Approve** — Guidelines are complete and accurate
2. **Request changes** — Specify what needs adjustment
3. **Restart section** — Redo a specific phase (colors, fonts, guidelines)
4. **Full restart** — Begin questionnaire again with different direction
```

#### 7.5 Rejection Handling

**If user selects "Request changes":**

1. Ask user to specify which elements need adjustment:
   - Colors (which ones, what's wrong)
   - Fonts (which category, what's the issue)
   - Guidelines (which section, what's missing)

2. Return to the relevant phase:
   | Issue Area | Return To |
   |------------|-----------|
   | Color changes | Phase 4.5 (Iteration Handling) |
   | Font changes | Phase 5.6 (Iteration Handling) |
   | Guideline structure | Phase 6.6 (Iteration Handling) |
   | Missing content | Relevant section in Phase 6 |

3. After changes, regenerate affected output files
4. Return to Phase 7.4 for re-evaluation

**If user selects "Restart section":**

1. Ask which phase to restart:
   ```
   Which section would you like to redo?
   1. Color Palette (Phase 4)
   2. Typography (Phase 5)
   3. Brand Guidelines (Phase 6)
   ```

2. Clear previous outputs for that section
3. Resume from beginning of selected phase
4. Carry forward unchanged sections

**If user selects "Full restart":**

1. Confirm with user:
   ```
   Starting over will discard all current work. Are you sure?
   - Yes, start fresh
   - No, let me request specific changes instead
   ```

2. If confirmed:
   - Archive current `./branding/` to `./branding-v1/`
   - Return to Phase 1 (Conversation Flow Selection)
   - Begin questionnaire anew

#### 7.6 Iteration Limits

**Maximum iterations per phase:**
- Color palette: 3 major revisions
- Typography: 3 major revisions
- Guidelines structure: 2 major revisions

**After reaching limit:**
```
We've made [X] revisions to [section]. To ensure we're aligned:

1. Let's review your original answers to questions [relevant Q#s]
2. Can you share a specific example of a brand that has what you're looking for?
3. What single word best describes what's missing?

This will help me understand the gap and propose a better solution.
```

**Full restart limit:** 2 complete restarts

After 2 full restarts, recommend:
```
We've tried two different directions. Before starting again, I recommend:

1. Collect 3-5 brand examples you love and note specifically what you like
2. Write a short paragraph describing your ideal customer's first impression
3. List any absolute requirements (colors to use/avoid, style constraints)

This preparation will help us nail it on the next attempt.
```

#### 7.7 Approval and Delivery

**When user approves:**

1. Confirm delivery:
   ```
   ✓ Brand guidelines approved!
   
   Your brand assets are ready in ./branding/:
   - brand-guidelines.md — Full brand documentation
   - brand-summary.txt — Executive summary for stakeholders
   - swatches/ — Visual color swatches
   
   **Recommended next steps:**
   1. Share brand-summary.txt with stakeholders for alignment
   2. Use brand-guidelines.md as the source of truth for all design work
   3. Create or finalize your logo based on the visual identity guidelines
   4. Set up brand colors and fonts in your design tools
   ```

2. Offer follow-up options:
   ```
   Would you like me to help with anything else?
   - Generate additional assets (social media templates, email signatures)
   - Create a brand presentation deck
   - Document brand voice and messaging guidelines in more detail
   ```

3. Archive the session:
   - Questionnaire responses are captured in the guidelines
   - No separate archive needed unless user requests

---

## Iteration Loop Workflow

The branding process uses a **generate → evaluate → iterate** loop to ensure output quality. This section documents the explicit loop structure.

### Loop Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    BRANDING WORKFLOW LOOP                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 1-3 ──► Phase 4 ──► Phase 5 ──► Phase 6 ──► Phase 7      │
│  (Discovery)   (Colors)    (Fonts)    (Guidelines) (Evaluation)  │
│                   │           │            │            │        │
│                   │           │            │            ▼        │
│                   │           │            │      ┌──────────┐   │
│                   │           │            │      │ APPROVED │───┼──► Exit
│                   │           │            │      └──────────┘   │
│                   │           │            │            │        │
│                   │           │            │      ┌──────────┐   │
│                   │           │            │      │ REJECTED │   │
│                   │           │            │      └────┬─────┘   │
│                   │           │            │           │         │
│                   ◄───────────┼────────────┼───────────┘         │
│                   │           │            │    (Return to       │
│                   │           │            │     relevant        │
│                   │           │            │     phase)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Loop Entry Points

| User Feedback | Return To | What Happens |
|---------------|-----------|--------------|
| "Change colors" | Phase 4.5 | Revise palette, regenerate swatches, update guidelines |
| "Change fonts" | Phase 5.6 | Select new fonts, update typography rules |
| "Change guidelines" | Phase 6.6 | Adjust structure/content, regenerate files |
| "Restart colors" | Phase 4.1 | Full color phase redo |
| "Restart fonts" | Phase 5.1 | Full typography phase redo |
| "Restart guidelines" | Phase 6.1 | Full guidelines phase redo |
| "Full restart" | Phase 1 | Archive current work, begin anew |

### Loop Execution Rules

**On each iteration:**
1. Make requested changes in the target phase
2. Regenerate all downstream outputs affected by the change
3. Return to Phase 7.1 for completeness check
4. Present updated evaluation in Phase 7.4
5. Wait for user approval or next iteration request

**Downstream dependencies:**

| Change In | Must Regenerate |
|-----------|-----------------|
| Phase 4 (Colors) | Swatches (4.6), Color sections in brand-guidelines.md (6.7), brand-summary.txt (6.8) |
| Phase 5 (Fonts) | Typography sections in brand-guidelines.md (6.7), brand-summary.txt (6.8) |
| Phase 6 (Guidelines) | brand-guidelines.md (6.7), brand-summary.txt (6.8) |

### Exit Conditions

**Successful exit (Phase 7.7):**
- User explicitly approves output
- All completeness checks pass (7.1)
- All alignment verifications pass (7.2)
- All quality standards met (7.3)

**Escalation exit (after iteration limits):**
- 3+ revisions per phase without approval → Pause for user clarification (7.6)
- 2+ full restarts → Recommend preparation steps before continuing (7.6)

### Iteration Limits (Reference)

| Scope | Maximum Iterations | After Limit |
|-------|-------------------|-------------|
| Color palette | 3 revisions | Clarification dialog |
| Typography | 3 revisions | Clarification dialog |
| Guidelines | 2 revisions | Clarification dialog |
| Full restart | 2 complete restarts | Preparation recommendations |

### Loop State Tracking

**Track during session:**
- Current iteration count per phase
- Full restart count
- User feedback history (what was changed and why)

**Carry forward on section restart:**
- Approved outputs from other phases
- User preferences and constraints captured in questionnaire

**Archive on full restart:**
- Move `./branding/` to `./branding-v[N]/` (where N = restart count)
- Preserve for reference but start fresh

---

## Checklist Before Output

- [ ] User selected conversation flow style
- [ ] All relevant questionnaire sections completed
- [ ] Color palette researched and generated with rationale
- [ ] Font recommendations include usage rules
- [ ] brand-guidelines.md created in ./branding/
- [ ] brand-summary.txt created in ./branding/
- [ ] Visual swatches generated in ./branding/swatches/
- [ ] Output verified by evaluator
- [ ] User approved final output (Phase 7.7)
