---
name: create-complementary
description: Creates a new complementary component in Strapi CMS and Next.js frontend. Use when user says "create complementary", "add complementary", "new complementary", "vytvoř complementary", or "přidej complementary". Guides through component name, icon, fields, usage context, app-context needs, then creates CMS schema, type interface, transformer, and relay fragment.
compatibility: Strapi boilerplate monorepo (frontend/ Next.js + cms/ Strapi)
---


# Create New Complementary Component

Creates a complete complementary component - from CMS schema to TypeScript types and GraphQL fragments.

**Complementary components** are reusable sub-components that are nested within blocks or other content types (e.g., button, video, send-email).

**Reference files in this folder:**
- [references/file-templates.md](references/file-templates.md) - TypeScript templates for generated files
- [references/graphql-reference.md](references/graphql-reference.md) - GraphQL fragment examples for common field types
- [references/cms-reference.md](references/cms-reference.md) - CMS field type definitions

---

## Step 1: Guide User Through Questions (ONE AT A TIME)

**Before starting:** Silently scan existing complementary components to check for duplicates.

### 1.1 Component Name
```
What will the complementary component be called?
(in English, singular, e.g., 'image-gallery', 'testimonial', 'author')
```

**Validation:**
| Input | Action |
|-------|--------|
| Czech word (`autor`) | ⚠️ Ask for English name |
| Plural (`testimonials`) | ⚠️ Suggest singular, wait for confirmation |
| PascalCase/camelCase (`ImageGallery`) | ✅ Auto-fix to kebab-case |
| Underscores (`image_gallery`) | ✅ Auto-fix to hyphens |
| Spaces (`image gallery`) | ✅ Auto-fix to hyphens |
| Uppercase (`IMAGE`) | ✅ Auto-fix to lowercase |
| Special chars/diacritics | ✅ Auto-remove |
| Starts with number | ⚠️ Ask for different name |
| Already exists | ❌ Ask for different name |

**⚠️ CRITICAL: Duplicate check has HIGHEST PRIORITY!**
- After auto-fix, ALWAYS check if the resulting name already exists
- Example: Input `VIDEO` → auto-fix to `video` → but `video` exists → ❌ Ask for different name
- The duplicate check runs AFTER all auto-fix transformations

**⚠️ IMPORTANT: When auto-fix is applied, ALWAYS inform the user what was changed:**

```
✓ Component name: `image-gallery`
  (auto-fixed: spaces → hyphens)
```

Examples:
| Input | Output | Message |
|-------|--------|---------|
| `author` | `author` | `✓ Component name: author` |
| `ImageGallery` | `image-gallery` | `✓ Component name: image-gallery` (auto-fixed: PascalCase → kebab-case) |
| `image gallery` | `image-gallery` | `✓ Component name: image-gallery` (auto-fixed: spaces → hyphens) |
| `image_gallery` | `image-gallery` | `✓ Component name: image-gallery` (auto-fixed: underscores → hyphens) |
| `IMAGE` | `image` | `✓ Component name: image` (auto-fixed: uppercase → lowercase) |
| `image!` | `image` | `✓ Component name: image` (auto-fixed: special characters removed) |

### 1.2 Icon (Optional)
```
Which icon to use? (Optional - press Enter to skip)
```
Options: crown, picture, book, envelope, star, heart, quote, layout, bulletList, phone, user, file, play, question, calendar, clock, gift, globe, grid, home, link, video, bold, italic

**Show:** `✓ Icon: picture` or `✓ Icon: (none)`

**Auto-generate displayName:**
- Automatically convert kebab-case to PascalCase (no spaces)
- Examples:
  - `donate-amount` → `DonateAmount`
  - `image-gallery` → `ImageGallery`
  - `author` → `Author`
  - `send-email` → `SendEmail`

**Show:** `✓ displayName: DonateAmount` (auto-generated)

### 1.3 Define Fields

**Before starting, silently scan existing complementary components** in `cms/src/components/complementary/` to offer them when field type is "component".

#### 1.3.1 Ask for Field List
```
What fields should the component contain?

You can:
- List field names: "title, description, amount, button"
- With types: "title: string, description: text, amount: number"
- With notes: "title (required), description (optional)"
- Natural language: "title, short description, donation amount, submit button"

Examples:
  - name, email, phone
  - title: string, description: text, image: media
  - label (required), link (optional), icon
  - title, description, button (for nested component)
```

**Parse the user's input** and extract field names. Be flexible with formats:
- Comma-separated: `name, bio, avatar`
- With types: `name: string, bio: text`
- With notes: `label (required), link (optional)`
- Natural language: "author name, short biography, profile image"

**Show extracted fields:**
```
✓ Fields detected: title, description, amount, button

Now let's configure each field (4 total)
```

#### 1.3.2 Iteratively Ask for Field Details

For each detected field, ask for missing details **one field at a time**.

**Show clear progress with field number:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Field 1/4: title
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

What type should this field be?
1. string - Short text (e.g., names, titles)
2. text - Long text / paragraphs
3. richtext - Rich text editor with formatting
4. media - Images or files
5. boolean - True/False checkbox
6. number - Integer or decimal
7. date - Date picker
8. datetime - Date and time picker
9. relation - Relation to another content type
10. email - Email address
11. enumeration - Dropdown with predefined options
12. component - Nested component (repeatable or single)

Type:
```

After type is selected:
```
Is this field required? (Yes/No)
```

If type is `enumeration`, also ask:
```
What are the enum values? (comma-separated, e.g., "draft, published, archived")
```

If type is `media`, also ask:
```
What media types? (images, videos, files, or "all")
```

If type is `relation`, also ask:
```
Which content type to relate to? (e.g., "page", "article", "category")
```

If type is `component`, also ask:
```
Which component would you like to use?

Available complementary components:
1. complementary.button (label, page, linkExternal, openInNewTab, anchor)
2. complementary.video (uploadedVideo, externalVideo, optionalImage)
3. complementary.ecomail (apiKey, listId, ...)
4. complementary.send-email (recipient, subject, ...)
5. Other (specify manually, e.g., "block.hero-block")

Choose: 1

Is it repeatable? (Yes/No)
```

**Show progress after each field:**
```
✓ Field 1/4: title (string, required)
✓ Field 2/4: description (text, optional)
✓ Field 3/4: amount (integer, required)
✓ Field 4/4: button (component: complementary.button, single)
```

**After all fields are defined, show summary:**
```
✓ All fields configured (4/4):
  - title: string (required)
  - description: text (optional)
  - amount: integer (required)
  - button: component (complementary.button, single)
```

### 1.4 Where Will It Be Used?
```
Where will this component be used?
(e.g., 'in VideoBlock', 'in multiple blocks', 'in Article content type')
```
**This is informational only** - helps with documentation and context.

**Show:** `✓ Usage: in VideoBlock and ArticleBlock`

### 1.5 Needs App Context?
```
Will this component need access to app context (IApp)?
(e.g., for page relations, locale, or global settings)
```
Options: Yes / No

**Examples needing app:**
- Button (has page relation)
- Menu items (has page relations)

**Examples NOT needing app:**
- Video (standalone media)
- Send-email (standalone config)

**Show:** `✓ Needs app context: Yes`

### 1.6 Summary & Confirmation
```
📋 New Complementary Component Summary:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Name:          donation-form
displayName:   DonationForm (auto-generated)
Icon:          gift
Fields:        4 fields defined
  - title: string (required)
  - description: text (optional)
  - amount: integer (required)
  - button: component (complementary.button, single)
Usage:         in fundraising pages
Needs app:     No

Files to create/update:
- cms/src/components/complementary/donation-form.json (new, with 4 fields)
- frontend/src/types/donationForm.ts (new)
- frontend/src/utils/strapi/getDonationFormType.ts (new)
- frontend/src/relay/app.ts (update - add fragment)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Create complementary component? (Yes/No)
```

**Notes:**
- If fields defined, show: `Fields: 4 fields defined` with full list
- If no fields defined, show: `Fields: Empty (add fields in Strapi later)`
- Only show field list if fields were defined

**Only create files after user confirms!**

---

## Step 2: Calculate Names

```javascript
// Input: "image-gallery"
componentNameKebab = "image-gallery"          // CMS schema name
componentNamePascal = "ImageGallery"          // Type interface name + displayName
componentNameCamel = "imageGallery"           // Type file name
fragmentName = "appImageGalleryFragment"      // GraphQL fragment name
typeName = "ComponentComplementaryImageGallery" // GraphQL typename
transformerName = "getImageGalleryType"       // Transformer function name
displayName = "ImageGallery"                  // Auto-generated from kebab-case (same as Pascal)
```

---

## Step 3: Create Files

### 3.1 CMS Schema
**File:** `cms/src/components/complementary/{componentNameKebab}.json`

#### If fields were defined:
```json
{
  "collectionName": "components_complementary_{componentNameKebab}s",
  "info": {
    "displayName": "{displayName}"
    // Add "icon": "{icon}" only if icon was provided
  },
  "options": {},
  "attributes": {
    "fieldName": {
      "type": "string",
      "required": true
    },
    "anotherField": {
      "type": "text"
    }
    // ... add all defined fields here
  }
}
```

#### If no fields defined (empty):
```json
{
  "collectionName": "components_complementary_{componentNameKebab}s",
  "info": {
    "displayName": "{displayName}"
    // Add "icon": "{icon}" only if icon was provided
  },
  "options": {},
  "attributes": {}
}
```

**Field type mapping:**
| User Input | Strapi Type | Additional Properties |
|------------|-------------|----------------------|
| string | `"type": "string"` | - |
| text | `"type": "text"` | - |
| richtext | `"type": "richtext"` | - |
| media | `"type": "media", "multiple": false, "allowedTypes": ["images"]` | Set allowedTypes based on user choice |
| boolean | `"type": "boolean", "default": false` | - |
| number | `"type": "integer"` or `"type": "decimal"` | Ask user which one |
| date | `"type": "date"` | - |
| datetime | `"type": "datetime"` | - |
| email | `"type": "email"` | - |
| enumeration | `"type": "enumeration", "enum": ["value1", "value2"]` | Use user-provided values |
| relation | `"type": "relation", "relation": "oneToOne", "target": "api::page.page"` | Use user-specified target |
| component | `"type": "component", "repeatable": true/false, "component": "complementary.button"` | Use user choice |

**Note:**
- Only include `"icon"` field if user provided an icon. If they skipped it, omit the field entirely.
- For required fields, add `"required": true`
- See [references/cms-reference.md](references/cms-reference.md) for complete field type examples

### 3.2 TypeScript Type Interface
**File:** `frontend/src/types/{componentNameCamel}.ts`

See [references/file-templates.md](references/file-templates.md) for template.

### 3.3 Transformer Utility
**File:** `frontend/src/utils/strapi/{transformerName}.ts`

See [references/file-templates.md](references/file-templates.md) for templates (with and without app context).

**Important:** Always include both functions:
- `{transformerName}()` - Transform single item
- `{transformerName}List()` - Transform array of items (for repeatable components)

### 3.4 Update relay/app.ts
**File:** `frontend/src/relay/app.ts`

Add GraphQL fragment (alphabetically in the complementary fragments section):

```typescript
graphql`
    fragment {fragmentName} on {typeName} {
        id
        # TODO: Add fields after adding them in CMS
    }
`;
```

**⚠️ CRITICAL:** Add fragment in the correct section (after other complementary fragments like `appVideoFragment`, `appButtonFragment`).

---

## Step 4: Show TODO List

### If fields were defined:
```
✅ Complementary component "{componentNameKebab}" created successfully with {X} fields!

📝 TODO (in this order):

1. [ ] Restart Strapi CMS
   cd cms && npm run develop

2. [ ] Verify fields in Strapi Content-Type Builder
   Components → complementary → "{displayName}"
   (Fields are already created, you may adjust if needed)

3. [ ] Refresh GraphQL schema
   cd frontend && npm run graphql-codegen

4. [ ] Update GraphQL fragment in relay/app.ts
   fragment {fragmentName} on {typeName} {
       id
       fieldName1
       fieldName2
       ...
   }
   (see references/graphql-reference.md for field type examples)

5. [ ] Run Relay compiler
   cd frontend && npm run relay

6. [ ] Use in blocks or content types by adding component relation:
   - In CMS: Add component field → Select "complementary.{componentNameKebab}"
   - In block fragment: Add field with fragment spread

7. [ ] Test in CMS and frontend
```

### If no fields defined (empty):
```
✅ Complementary component "{componentNameKebab}" created successfully!

📝 TODO (in this order):

1. [ ] Restart Strapi CMS
   cd cms && npm run develop

2. [ ] Add fields in Strapi Content-Type Builder
   Components → complementary → "{displayName}"

3. [ ] Refresh GraphQL schema
   cd frontend && npm run graphql-codegen

4. [ ] Update GraphQL fragment in relay/app.ts
   fragment {fragmentName} on {typeName} { ... }
   (see references/graphql-reference.md for examples)

5. [ ] Run Relay compiler
   cd frontend && npm run relay

6. [ ] Use in blocks or content types by adding component relation:
   - In CMS: Add component field → Select "complementary.{componentNameKebab}"
   - In block fragment: Add field with fragment spread

7. [ ] Test in CMS and frontend
```

---

## Checklist

### Questions (one at a time):
- [ ] Component name (validate: English, singular, no duplicates)
- [ ] Auto-generate displayName (kebab-case → PascalCase)
- [ ] Icon (optional)
- [ ] Define fields:
  - [ ] Silently scan existing complementary components for component type suggestions
  - [ ] Ask for field list (free-form text)
  - [ ] Parse and extract field names
  - [ ] Show "Now let's configure each field (X total)"
  - [ ] For each field, ask **with clear numbering** "Field X/Y: fieldName":
    - [ ] Field type
    - [ ] Required? (Yes/No)
    - [ ] Additional properties (enum values, media types, relation target, etc.)
    - [ ] If type is "component": Offer existing complementary components (button, video, etc.)
  - [ ] Show progress after each: "✓ Field X/Y: name (type, required)"
  - [ ] After all fields: Show summary "✓ All fields configured (X/X)"
- [ ] Usage description
- [ ] Needs app context (Yes/No)
- [ ] Summary & confirmation

### File creation (after confirmation):
- [ ] CMS schema (`cms/src/components/complementary/{componentNameKebab}.json`)
  - Empty attributes if no fields defined
  - Full field definitions if fields were defined
- [ ] Type interface (`frontend/src/types/{componentNameCamel}.ts`)
- [ ] Transformer utility with list helper (`frontend/src/utils/strapi/{transformerName}.ts`)
  - Single item transformer: `{transformerName}()`
  - Array transformer: `{transformerName}List()`
- [ ] `relay/app.ts` updated (add fragment)
- [ ] TODO list shown to user (varies based on whether fields were defined)

---

## Examples of Existing Complementary Components

### Simple (no app context):
- **video** - Video with uploaded/external source and optional image
- **send-email** - Email configuration with recipient and subject

### With app context:
- **button** - Button/link with page relation, external link, and label
- **menu-item** - Navigation item with page relation

### Usage patterns:
```typescript
// In a block that uses video:
import { getVideoType } from '../../../utils/strapi/getVideoType';

const video = getVideoType(blocksData.video);
// video is IVideo | null

// In a block that uses button (needs app):
import { getButtonType } from '../../../utils/strapi/getButtonType';

const button = getButtonType(blocksData.button, app);
// button is IButton | null
```
