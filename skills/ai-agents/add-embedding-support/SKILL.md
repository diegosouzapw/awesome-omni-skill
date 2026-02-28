---
name: add-embedding-support
description: Add Qdrant embedding support to v3 WordPress components for RAG chatbot. Implements component-level content chunking for searchable, structured embeddings. Use when adding embedding to new or existing v3 components.
---

# Add Embedding Support Skill

You are helping add Qdrant embedding support to WordPress v3 components. This enables component content to be indexed
and searched via a RAG-based chatbot powered by Claude's API.

## System Overview

The embedding system:

- **Chunks at component level**: Each component becomes one or more embedding chunks
- **Avoids sub-component loading**: Write extraction code directly in the component class
- **Supports sections**: Complex components add multiple sections (sub-chunks) per instance
- **Tracks metadata**: Links, dates, and custom metadata stored separately
- **Respects skip markers**: Components can opt-out via `ComponentEmbeddingSkipAwareInterface`

### How It Works

1. CLI command `wp vendi embedding:generate` runs
2. Sets global constant `VENDI_RENDER_CONTEXT` to `RenderingContextEnum::EMBEDDING`
3. Loads each component via `vendi_load_component_v3()`
4. Template detects context and returns component instance (no HTML rendering)
5. Component's `getEmbedding()` method extracts structured data
6. `ComponentEmbedding` DTO formats data into JSON chunks for Qdrant

### Output Format

Each component produces a JSON object like this:

```json
{
  "content": "Heading: Ask a Researcher\nBody: Are you a CRNA with research questions?\nLinks: Contact us",
  "metadata": {
    "type": "page",
    "url": "https://example.com/page/",
    "created": "2022-11-29T21:01:08+00:00",
    "updated": "2024-03-07T09:07:06+00:00",
    "links": [
      {
        "text": "Contact us",
        "url": "https://example.com/contact/"
      }
    ],
    "component_type": "content_callout_full_width"
  },
  "id": "660-3"
}
```

## Component Type Classification

### Embeddable Components

- Implements `ComponentEmbeddingAwareInterface`
- Provides `getEmbedding()` method
- Content is indexed for chatbot

### Skippable Components

- Implements `ComponentEmbeddingSkipAwareInterface` (marker interface)
- No `getEmbedding()` method needed
- Ignored during embedding generation
- Use for: ads, navigation, forms, decorative elements

### Simple Components

- Single chunk with heading and/or body
- No repeater fields
- Auto-extraction via interfaces

### Complex Components

- Multiple sections from repeater/flexible content
- Each item becomes a separate section (sub-chunk)
- May include links/CTAs tracked in metadata

---

# Implementation Patterns

## Pattern 1: Simple Component (Single Chunk)

**When to use**: Component has just heading and/or body copy, no repeater fields

### Choosing the Right Interfaces

**IMPORTANT**: Inspect the actual template file to determine which interfaces to implement:

- **`PrimaryHeadingInterface`** - Use when template displays a component-level heading (outside loops)
  - Example: `<h2><?php esc_html_e(get_sub_field('headline')); ?></h2>` at the top level
  - NOT for headings inside repeater loops

- **`PrimaryCopyInterface`** - Use when template displays component-level body/intro copy (outside loops)
  - Example: `<?php echo wp_kses_post(get_sub_field('intro_copy')); ?>` before any repeaters
  - NOT for copy inside repeater loops

The interfaces should map to what actually exists in the template structure.

### Required Interfaces

```php
use Vendi\Theme\ComponentInterfaces\ComponentEmbeddingAwareInterface;
use Vendi\Theme\ComponentInterfaces\PrimaryHeadingInterface; // If template has top-level heading
use Vendi\Theme\ComponentInterfaces\PrimaryCopyInterface;    // If template has top-level copy
use Vendi\Theme\DTO\Embedding\ComponentEmbedding;
use Vendi\Theme\DTO\Embedding\ComponentEmbeddingInterface;
```

### Class Implementation

```php
class simple_component extends BaseComponent implements
    ComponentEmbeddingAwareInterface,
    PrimaryHeadingInterface,      // Only if template has top-level heading
    PrimaryCopyInterface          // Only if template has top-level copy
{
    public function getEmbedding(): ?ComponentEmbeddingInterface
    {
        return ComponentEmbedding::fromComponent($this);
    }

    public function getPrimaryHeadingText(): ?string
    {
        // Return the field that corresponds to the top-level heading in template
        return get_sub_field('headline');
    }

    public function getPrimaryCopy(): ?string
    {
        // Return the field that corresponds to the top-level copy in template
        return get_sub_field('copy');
    }
}
```

### Output

```
Heading: [from getPrimaryHeadingText() if interface implemented]
Body: [from getPrimaryCopy() if interface implemented]
```

**Key Points:**

- Inspect template first to determine which interfaces are needed
- `fromComponent()` auto-extracts heading and body via interfaces
- Single chunk per component instance
- No manual section creation needed
- Don't guess at structure - base decision on actual template code

---

## Pattern 2: Skippable Component (No Embedding)

**When to use**: Ads, navigation, forms, decorative/visual-only elements

### Required Interface

```php
use Vendi\Theme\ComponentInterfaces\ComponentEmbeddingSkipAwareInterface;
```

### Class Implementation

```php
class ad_component extends VendiComponent implements ComponentEmbeddingSkipAwareInterface
{
    // No getEmbedding() method needed
    // Component completely ignored during embedding generation
}
```

**Key Points:**

- Empty marker interface
- No embedding logic required
- Still add template boilerplate (see Template Requirements)

---

## Pattern 3: Complex Component with Sections

**When to use**: Component has repeater or flexible content fields where each item should be a separate section

### Required Interfaces

```php
use Vendi\Theme\ComponentInterfaces\ComponentEmbeddingAwareInterface;
use Vendi\Theme\ComponentInterfaces\PrimaryHeadingInterface;
use Vendi\Theme\ComponentInterfaces\PrimaryCopyInterface;
use Vendi\Theme\DTO\Embedding\ComponentEmbedding;
use Vendi\Theme\DTO\Embedding\ComponentEmbeddingInterface;
```

### Class Implementation

```php
public function getEmbedding(): ?ComponentEmbeddingInterface
{
    // Start with base embedding (auto-extracts heading/body from interfaces)
    $ret = ComponentEmbedding::fromComponent($this);

    // Loop through repeater field
    while (have_rows('items')) {
        the_row();
        $layout = get_row_layout();

        // CRITICAL: Filter to relevant layouts only
        if (!in_array($layout, ['content_item', 'text_block'], true)) {
            continue;
        }

        $heading = get_sub_field('heading');
        $copy = get_sub_field('copy');

        // CRITICAL: Always clean HTML from user content
        $cleanCopy = ComponentEmbedding::stripAllHtmlFromText($copy);

        // Add section with optional custom label
        $ret->addSection(
            $heading . PHP_EOL . $cleanCopy,
            'Section' // Optional: 'FAQ Item', 'Testimonial', etc.
        );
    }

    return $ret;
}
```

### Output

```
Heading: [component main heading]
Body: [component intro copy]
Section 1: Item 1 Heading
[item 1 copy]
Section 2: Item 2 Heading
[item 2 copy]
```

**Key Points:**

- Filter layouts to process only relevant types
- Use `stripAllHtmlFromText()` for all HTML content
- Each `addSection()` creates a separate sub-chunk
- Sections are auto-numbered (Section 1, Section 2, etc.)

---

## Pattern 4: Component with Links/CTAs

**When to use**: Component has call-to-action buttons or links that should be tracked in metadata

### Class Implementation

```php
public function getEmbedding(): ?ComponentEmbeddingInterface
{
    $ret = ComponentEmbedding::fromComponent($this);

    while (have_rows('cards')) {
        the_row();

        $heading = get_sub_field('heading');
        $copy = get_sub_field('copy');
        $link = get_sub_field('cta');

        // Build structured content with labels
        $contentParts = [];
        if ($heading) {
            $contentParts[] = 'Heading: ' . $heading;
        }
        if ($copy) {
            $contentParts[] = 'Body: ' . $copy;
        }
        if ($link && is_array($link)) {
            $contentParts[] = 'Link: ' . $link['title'];
        }

        // Only add section if there's content
        if ($content = implode(PHP_EOL, array_filter($contentParts))) {
            $ret->addSection($content);
        }

        // CRITICAL: Track link separately in metadata
        if ($link && is_array($link)) {
            $ret->addLink(
                linkText: $link['title'] ?? '',
                linkUrl: $link['url'] ?? ''
            );
        }
    }

    return $ret;
}
```

### Output

```json
{
  "content": "Heading: Component Title\nBody: Intro text\nLinks: Card 1 CTA, Card 2 CTA\nSection 1:\nHeading: Card 1\nBody: Card 1 copy\nLink: Card 1 CTA",
  "metadata": {
    "links": [
      {
        "text": "Card 1 CTA",
        "url": "/page1/"
      },
      {
        "text": "Card 2 CTA",
        "url": "/page2/"
      }
    ],
    "component_type": "card_navigation"
  }
}
```

**Key Points:**

- Links appear in both content text and metadata
- Metadata links enable advanced RAG features
- Use structured content with labels (Heading:, Body:, Link:)
- Filter empty content before adding sections

---

## Pattern 5: Component with HTML Content Containing Links

**When to use**: Component has HTML content (bios, articles, descriptions) with embedded `<a>` tags that should be tracked

### Class Implementation

```php
public function getEmbedding(): ?ComponentEmbeddingInterface
{
    $ret = ComponentEmbedding::fromComponent($this);

    while (have_rows('items')) {
        the_row();

        $name = get_sub_field('name');
        $bio = get_sub_field('bio'); // Contains HTML with links

        // CRITICAL: Extract links BEFORE stripping HTML
        // Use name as prefix for context
        ComponentEmbedding::extractAndAddLinksFromHtml($ret, $bio, $name);

        // Now strip HTML for text content
        $cleanBio = ComponentEmbedding::stripAllHtmlFromText($bio);

        $ret->addSection(
            'Name: ' . $name . PHP_EOL . 'Bio: ' . $cleanBio,
            'Person'
        );
    }

    return $ret;
}
```

### Output

If bio contains: `<p>Follow me on <a href="https://twitter.com/jdoe">Twitter</a></p>`

```json
{
  "content": "Person 1: Name: John Doe\nBio: Follow me on Twitter",
  "metadata": {
    "links": [
      {"text": "John Doe Twitter", "url": "https://twitter.com/jdoe"}
    ]
  }
}
```

**Key Points:**

- Call `extractAndAddLinksFromHtml()` BEFORE `stripAllHtmlFromText()`
- Use contextual prefix (name, title, etc.) to avoid duplicate generic link text
- Links preserved in metadata even after HTML is stripped from content

---

## Pattern 6: Component with Related Posts

**When to use**: Component displays content from related WP_Post objects (testimonials, people, etc.)

### Class Implementation

```php
public function getEmbedding(): ?ComponentEmbeddingInterface
{
    $ret = ComponentEmbedding::fromComponent($this);

    foreach ($this->getRelatedPosts() as $post) {
        // CRITICAL: Validate post object before accessing fields
        if (!$post instanceof WP_Post) {
            continue;
        }

        $name = get_field('name', $post->ID);
        $bio = get_field('bio', $post->ID);

        // Clean HTML and add with custom section label
        $ret->addSection(
            $name . PHP_EOL . ComponentEmbedding::stripAllHtmlFromText($bio),
            'Person' // Custom label: 'Testimonial', 'Team Member', etc.
        );
    }

    return $ret;
}
```

**Key Points:**

- Always check `instanceof WP_Post` before accessing post fields
- Access fields with post ID: `get_field('field_name', $post->ID)`
- Use descriptive section labels

---

# Template File Requirements

**CRITICAL**: Every embeddable component template must include this boilerplate at the top.

## Required Boilerplate

```php
<?php

use Vendi\Theme\Component\{component_name};
use Vendi\Theme\ComponentUtility;
use Vendi\Theme\Enums\RenderingContextEnum;

/** @var {component_name} $component */
$component = ComponentUtility::get_new_component_instance({component_name}::class);

// CRITICAL: Early return for embedding context
if (defined('VENDI_RENDER_CONTEXT') && VENDI_RENDER_CONTEXT === RenderingContextEnum::EMBEDDING->value) {
    return $component;
}

if (!$component->renderComponentWrapperStart()) {
    return;
}

?>
    <!-- HTML template here -->
<?php

$component->renderComponentWrapperEnd();
```

## Why This Matters

Without the embedding context check:

- Template will render HTML instead of returning component instance
- `getEmbedding()` method will never be called
- Component will be skipped in embedding output

**This boilerplate is required even for skippable components** (for consistency).

---

# Key Methods & Utilities

## ComponentEmbedding Static Factory

### `fromComponent($this)`

**Purpose**: Create base embedding with auto-extraction

**Auto-extracts:**

- Component type (class short name)
- Post ID and URL
- Creation and modification dates
- Primary heading (if `PrimaryHeadingInterface` implemented - based on template inspection)
- Primary body copy (if `PrimaryCopyInterface` implemented - based on template inspection)

**Usage**: Always first line of `getEmbedding()`

```php
public function getEmbedding(): ?ComponentEmbeddingInterface
{
    $ret = ComponentEmbedding::fromComponent($this);
    // ... add sections, links, etc.
    return $ret;
}
```

**Note**: The heading and body auto-extraction only works if you've implemented the corresponding interfaces based on what actually exists in the template (see Pattern 1 for details).

## Content Building Methods

### `addSection(string $text, string $sectionLabel = 'Section')`

Adds a labeled section to the embedding. Sections are auto-numbered (Section 1, Section 2, etc.).

**Best Practice**: Use descriptive labels

```php
// Good: Descriptive
$ret->addSection($content, 'Testimonial');
$ret->addSection($content, 'FAQ Item');
$ret->addSection($content, 'Team Member');

// Acceptable: Default auto-numbering
$ret->addSection($content); // "Section 1", "Section 2", etc.
```

### `addLink(string $linkText, string $linkUrl)`

Adds a link to metadata. Links stored separately from content text for advanced RAG features.

```php
if ($link && is_array($link)) {
    $ret->addLink(
        linkText: $link['title'] ?? '',
        linkUrl: $link['url'] ?? ''
    );
}
```

### `extractAndAddLinksFromHtml(ComponentEmbedding $embedding, ?string $html, string $linkPrefix = '')`

**Purpose**: Extracts all `<a>` tags from HTML content and adds them to the embedding's link metadata.

**When to use**: When content contains HTML with embedded links that should be tracked separately (e.g., biographical text with social media links, articles with reference links).

**Parameters:**
- `$embedding` - The ComponentEmbedding instance to add links to
- `$html` - HTML content to parse for links
- `$linkPrefix` - Optional prefix to add context to link text (e.g., person name)

**Features:**
- Uses DOMDocument for reliable HTML parsing
- Extracts both href and link text
- Filters out links missing href or text
- Adds contextual prefix when provided (useful for avoiding duplicate generic link text)

**Usage:**

```php
// Basic usage - extract links from HTML
ComponentEmbedding::extractAndAddLinksFromHtml($ret, $htmlContent);

// With prefix for context (recommended when looping through items)
foreach ($persons as $person) {
    $name = $person->name;
    $bio = $person->bio; // Contains <a href="...">Twitter</a>, <a href="...">LinkedIn</a>

    // Prefix links with person name: "John Doe Twitter", "John Doe LinkedIn"
    ComponentEmbedding::extractAndAddLinksFromHtml($ret, $bio, $name);

    // Clean HTML after extracting links
    $cleanBio = ComponentEmbedding::stripAllHtmlFromText($bio);
    $ret->addSection("Name: $name\nBio: $cleanBio", 'Person');
}
```

**Why use linkPrefix:**
Without prefix, 20 people with Twitter links produces 20 identical "Twitter" entries. With prefix, you get "Chris Haas Twitter", "Jane Smith Twitter", etc., providing essential context.

**Important:** Call `extractAndAddLinksFromHtml()` BEFORE `stripAllHtmlFromText()` to preserve the links before HTML is removed.

## HTML Cleaning Utility

### `stripAllHtmlFromText(?string $text, bool $preserveLists = false)`

**CRITICAL**: Always use this for user-entered HTML content

**Features:**

- Removes `<script>`, `<style>`, `<form>` tags and HTML comments
- Strips all remaining HTML tags
- Decodes HTML entities (`&amp;` → `&`)
- Collapses whitespace
- Optional: Preserves list structure with proper formatting

**Usage:**

```php
// DO THIS:
$cleanCopy = ComponentEmbedding::stripAllHtmlFromText($copy);
$ret->addSection($cleanCopy);

// NOT THIS:
$ret->addSection($copy); // May contain <div>, <p>, <br> tags
```

---

# Best Practices

## 1. Avoid Loading Sub-Components

**VERY IMPORTANT**: Write extraction code directly in `getEmbedding()`. Do NOT load sub-components.

**Strongly Preferred:**

```php
public function getEmbedding(): ?ComponentEmbeddingInterface
{
    $ret = ComponentEmbedding::fromComponent($this);

    // Write code directly - NO sub-component loading
    while (have_rows('items')) {
        the_row();
        $ret->addSection(get_sub_field('copy'));
    }

    return $ret;
}
```

**Avoid:**

```php
// DON'T load sub-components during embedding
vendi_load_component_v3(['parent', 'child']);
```

**Why**: The system hasn't found a good pattern for sub-component loading in embeddings yet. Keep it simple and direct.

## 2. Always Clean HTML from User Content

```php
// CORRECT:
$cleanCopy = ComponentEmbedding::stripAllHtmlFromText($copy);
$ret->addSection($cleanCopy);

// WRONG:
$ret->addSection($copy); // HTML tags leak into embedding
````

## 5. Use Structured Content with Labels

Makes content more parseable by the RAG system:

```php
$contentParts = [];

if ($heading) {
    $contentParts[] = 'Heading: ' . $heading;
}

if ($subheading) {
    $contentParts[] = 'Subheading: ' . $subheading;
}

if ($copy) {
    $contentParts[] = 'Body: ' . ComponentEmbedding::stripAllHtmlFromText($copy);
}

if ($link) {
    $contentParts[] = 'Link: ' . $link['title'];
}

$ret->addSection(implode(PHP_EOL, $contentParts));
```

---

# Implementation Checklist

## Step 1: Inspect Template File

- [ ] **Read the component's template file** (`.php`) to understand its structure
- [ ] Identify if there's a top-level heading (outside any loops) → Consider `PrimaryHeadingInterface`
- [ ] Identify if there's top-level body/intro copy (outside any loops) → Consider `PrimaryCopyInterface`
- [ ] Note any repeater fields that should become sections
- [ ] Note any links/CTAs that should be tracked in metadata

## Step 2: Class File Changes

- [ ] Add use statements at top of file:
  ```php
  use Vendi\Theme\ComponentInterfaces\ComponentEmbeddingAwareInterface;
  use Vendi\Theme\DTO\Embedding\ComponentEmbedding;
  use Vendi\Theme\DTO\Embedding\ComponentEmbeddingInterface;
  ```
- [ ] **Only if template has top-level heading**: Add interface use statement:
  ```php
  use Vendi\Theme\ComponentInterfaces\PrimaryHeadingInterface;
  ```
- [ ] **Only if template has top-level copy**: Add interface use statement:
  ```php
  use Vendi\Theme\ComponentInterfaces\PrimaryCopyInterface;
  ```
- [ ] Implement `ComponentEmbeddingAwareInterface` in class declaration
- [ ] **Only if template has top-level heading**: Implement `PrimaryHeadingInterface`
- [ ] **Only if template has top-level copy**: Implement `PrimaryCopyInterface`
- [ ] Add `getEmbedding(): ?ComponentEmbeddingInterface` method
- [ ] **If using `PrimaryHeadingInterface`**: Add `getPrimaryHeadingText(): ?string` returning the appropriate field
- [ ] **If using `PrimaryCopyInterface`**: Add `getPrimaryCopy(): ?string` returning the appropriate field

## Step 3: Template File Changes

- [ ] Add use statement at top:
  ```php
  use Vendi\Theme\Enums\RenderingContextEnum;
  ```
- [ ] Add embedding context check after component instantiation:
  ```php
  if (defined('VENDI_RENDER_CONTEXT') && VENDI_RENDER_CONTEXT === RenderingContextEnum::EMBEDDING->value) {
      return $component;
  }
  ```

## Step 4: getEmbedding() Implementation

- [ ] Start with `$ret = ComponentEmbedding::fromComponent($this);`
- [ ] Loop through any repeater/flexible content fields
- [ ] Filter layouts to relevant types only (`in_array()` check)
- [ ] Use `stripAllHtmlFromText()` for all HTML content
- [ ] Add sections with `addSection()` for each logical chunk
- [ ] Add links with `addLink()` if component has CTAs
- [ ] Validate WP_Post objects with `instanceof` before accessing fields
- [ ] Filter empty content before adding sections
- [ ] Write code directly (do NOT load sub-components)
- [ ] Return `$ret`

## For Skippable Components Only

- [ ] Add use statement:
  ```php
  use Vendi\Theme\ComponentInterfaces\ComponentEmbeddingSkipAwareInterface;
  ```
- [ ] Implement `ComponentEmbeddingSkipAwareInterface` in class declaration
- [ ] Do NOT implement `ComponentEmbeddingAwareInterface`
- [ ] Still add template boilerplate (for consistency)
- [ ] No `getEmbedding()` method needed

---

# Testing

After implementation, test with the CLI command:

```bash
wp vendi embedding:generate
```

This command:

1. Iterates through all published posts/pages
2. Sets `VENDI_RENDER_CONTEXT` to `EMBEDDING`
3. Loads each component
4. Calls `getEmbedding()` on embeddable components
5. Outputs structured JSON for Qdrant

## Verify Output

Check the JSON output for:

- ✅ Component appears in embedding data
- ✅ Heading and body extracted correctly
- ✅ Sections appear as separate chunks (Section 1, Section 2, etc.)
- ✅ Links tracked in metadata
- ✅ HTML stripped from content (no `<div>`, `<p>`, `<br>` tags)
- ✅ Content is readable and well-structured

## Sample Output Format

```json
{
  "content": "Heading: Research Topics\nSection 1: AANA's Current Priorities\nWhat are healthcare executives' perceptions...",
  "metadata": {
    "type": "page",
    "url": "https://example.com/page/",
    "created": "2022-11-29T21:01:08+00:00",
    "updated": "2024-03-07T09:07:06+00:00",
    "component_type": "accordion"
  },
  "id": "660-2"
}
```

---

# Common Pitfalls

1. **Forgetting to clean HTML**: Always use `stripAllHtmlFromText()` on user content
2. **Loading sub-components**: Write extraction code directly in `getEmbedding()`
3. **Missing template boilerplate**: Component will render HTML instead of being embedded
4. **Not filtering layouts**: Process only relevant flexible content layouts
5. **Not validating WP_Post**: Check `instanceof WP_Post` before accessing post fields
6. **Adding empty sections**: Filter content before calling `addSection()`
7. **Forgetting to return component**: Template must `return $component;` in embedding context
8. **Extracting links after stripping HTML**: Call `extractAndAddLinksFromHtml()` BEFORE `stripAllHtmlFromText()`
9. **Missing link context**: Use linkPrefix parameter when looping through items to avoid duplicate generic link text

---

# Reference Examples

Examine these components for real-world patterns:

- **basic_copy_block** - Simple: Single chunk with heading/body
- **ad_row** - Skippable: Marked with skip interface
- **accordion** - Complex: Multiple accordion items as sections
- **card_navigation** - Complex: Cards with CTAs tracked as links
- **testimonial** - Related Posts: WP_Post objects as sections with custom label
- **people_image_grid** - Complex: Loops through people, extracts links from bio HTML with name prefix, creates person sections

All located in: `vendi-theme-parts/components/[component_name]/[component_name].class.php`

---

# Your Role

Guide the user through implementing embedding support for a v3 component:

1. **Read the template file**: Inspect the actual `.php` template to understand structure
2. **Identify top-level content**: Determine if component has top-level heading and/or copy (outside loops)
3. **Determine pattern**: Is it simple, complex, skippable? Does it have repeaters? Links?
4. **Choose interfaces**: Based on template inspection, decide which interfaces to implement
5. **Present implementation plan**: Describe changes needed with specific field names from template
6. **Implement changes**: Update class and template files
7. **Test**: Run `wp vendi embedding:generate` and verify output

Remember:

- **Always start by reading the template file** - don't guess at structure
- Implement `PrimaryHeadingInterface` only if template has top-level heading (outside loops)
- Implement `PrimaryCopyInterface` only if template has top-level copy (outside loops)
- User handles `git add` and `git commit` - you should NOT run these
- Write embedding extraction code directly (avoid sub-component loading)
- Always clean HTML from user content
- Use structured content with labels for better RAG performance
