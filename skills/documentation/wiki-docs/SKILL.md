---
name: wiki-docs
description: Document custom Magento 2 site functionality in the project wiki. Systematically captures custom features, modules, configurations, and business logic for client handover and developer onboarding.
---

# Wiki Documentation Skill

## Overview
This skill provides a systematic approach to documenting custom Magento 2 site functionality in the project's GitHub wiki. It ensures custom features, modules, configurations, and business logic are properly documented for client handover, developer onboarding, and long-term maintenance.

## When to Use This Skill
- After implementing a new custom feature or module
- After completing a significant refactoring or migration
- When discovering undocumented functionality
- During client handover or project transitions
- When onboarding new developers to the project
- After resolving complex issues that reveal important system behavior
- When documenting third-party module customizations

## Wiki Location and Access

### Wiki Directory Structure
The wiki is located at the project root in a directory with the pattern:
```
<project_name>.wiki/
```

**For this project:**
- **Wiki Directory:** `m2_ntotank.wiki/`
- **Git Remote:** `git@github.com:uptactics/m2_ntotank.wiki.git`

**For other Magento 2 projects, determine wiki location:**
```bash
# Find wiki directory
find . -maxdepth 1 -type d -name "*.wiki"

# Or check CLAUDE.md for wiki configuration
grep -i "wiki" CLAUDE.md
```

### Wiki Git Workflow
The wiki is a separate git repository (GitHub Wiki standard):
```bash
# Navigate to wiki directory
cd <project_name>.wiki/

# Check status
git status

# Pull latest changes before editing
git pull origin master

# After making changes
git add .
git commit -m "docs: <description of changes>"
git push origin master

# Return to main project
cd ..
```

## Documentation Workflow

### Step 1: Identify What Needs Documentation
**Ask yourself:**
- Is this custom functionality?
- Will clients or future developers need to understand this?
- Does this involve custom business logic?
- Are there non-obvious configuration requirements?
- Is this different from standard Magento behavior?

**Trigger events for documentation:**
- ✅ Created a new custom module
- ✅ Modified or extended third-party module behavior
- ✅ Implemented custom business logic
- ✅ Added custom product attributes with specific purposes
- ✅ Created custom admin configurations
- ✅ Implemented custom APIs or integrations
- ✅ Added custom frontend features (calculators, widgets, etc.)
- ✅ Completed data migration with custom scripts
- ✅ Applied non-standard Magento configurations

**Skip documentation for:**
- ❌ Standard Magento functionality (already documented in Magento docs)
- ❌ Third-party modules without customization (vendor should document)
- ❌ Trivial bug fixes with no behavioral changes
- ❌ Temporary development utilities

### Step 2: Determine Documentation Type

#### A. Feature/Module Documentation
**For custom modules and features:**
- What it does (purpose)
- How it works (high-level architecture)
- How to use it (admin/frontend)
- Configuration options
- Dependencies
- Known limitations

**Example topics:**
- "Grouping Related Products"
- "Custom Shipping Calculator"
- "Product Data ViewModels"

#### B. Configuration Documentation
**For custom configurations:**
- What the configuration controls
- Where to find it (admin path)
- Valid values and their effects
- Dependencies on other settings
- Default values

**Example topics:**
- "Custom Product Attributes"
- "Third-Party Service Integrations"
- "Custom Admin Grids"

#### C. Process Documentation
**For workflows and procedures:**
- When to perform the process
- Step-by-step instructions
- Prerequisites
- Expected outcomes
- Troubleshooting common issues

**Example topics:**
- "Data Migration Process"
- "Deployment Workflow"
- "Cache Management Strategy"

#### D. Troubleshooting Documentation
**For known issues and solutions:**
- Problem description
- Root cause
- Solution or workaround
- Prevention strategies

**Example topics:**
- "Missing Products in Categories"
- "ElasticSuite Reindex Issues"

### Step 3: Choose or Create Wiki Page

#### Existing Pages
**Check for existing pages:**
```bash
cd <project_name>.wiki/
ls -la *.md
```

**Common wiki pages:**
- `Home.md` - Wiki home page with index
- `Magento-2-Site-Docs.md` - General site documentation
- `Data-Migration.md` - Migration-specific docs
- Feature-specific pages (e.g., `Grouping-Related-Products.md`)

**Update existing page if:**
- The topic fits naturally into an existing page
- The page is a general reference (like `Magento-2-Site-Docs.md`)
- You're adding to or correcting existing documentation

#### New Pages
**Create new page if:**
- The topic is substantial enough to warrant its own page
- The feature/module is complex or has multiple aspects
- The documentation will be frequently referenced independently

**Naming convention for new pages:**
- Use PascalCase with hyphens (GitHub Wiki standard)
- Be descriptive and specific
- Examples:
  - `Custom-Shipping-Calculator.md`
  - `Hyva-Theme-Customizations.md`
  - `Product-ViewModels.md`

### Step 4: Write the Documentation

#### Documentation Template for Features/Modules

**IMPORTANT: Structure Documentation for End Users First**

The wiki is primarily used by clients, site administrators, and non-technical users. Always structure documentation with user-facing content first, followed by technical details at the end.

**Recommended Structure:**
1. Overview (brief, what it does)
2. User-facing content (how to use it)
3. Troubleshooting (common issues end users might encounter)
4. External documentation links
5. Horizontal rule separator (`---`)
6. Technical details (for developers)

```markdown
# [Feature/Module Name]

## Overview
Brief 2-3 sentence description of what this feature does and why it exists. Focus on business value and what users can accomplish with it.

## Managing [Feature Name]

### Admin Access
**Path:** Section > Subsection > Page

From here you can:
- Action 1
- Action 2
- Action 3

### Creating/Configuring [Feature]
Step-by-step guide for admin users:
1. Navigate to [path]
2. Configure [settings]
   - **Setting 1:** Description of what it does
   - **Setting 2:** Description of what it does
3. Save and flush cache

### Available Options
**Option 1** - What this option provides
**Option 2** - What this option provides
**Option 3** - What this option provides

## Advanced Features
(If applicable - optional features or advanced usage)

### Import/Export
(If applicable)

### Bulk Operations
(If applicable)

## Troubleshooting

### Issue: [Common Problem]
**Symptoms:** What the user sees

**Possible Causes:**
1. Cause 1
2. Cause 2

**Solution:**
```bash
# Commands or admin steps to fix
```

### Issue: [Another Problem]
**Symptoms:** What the user sees

**Solution:**
1. Clear browser cache
2. Contact developer if issue persists

## Documentation

**Official Documentation:** [External Docs](https://example.com)

**Key Pages:**
- [Page 1](https://example.com/page1)
- [Page 2](https://example.com/page2)

---

## Technical Details

### Module Location
**Module:** `Vendor_ModuleName`
**Version:** X.Y.Z
**Path:** `app/code/Vendor/ModuleName/`
**Dependencies:**
- Magento_Backend
- Magento_Catalog

**Custom Templates:**
- Template 1: `path/to/template.phtml`
- Template 2: `path/to/template2.phtml`

### Features
Technical features list:
- Feature 1
- Feature 2

### Architecture
The system consists of:
1. **Backend Component:** Purpose
2. **Frontend Component:** Purpose
3. **Data Model:** Purpose

### Database Schema
**Tables:**
- `table_name` - Purpose

**Key Fields:**
- `field_1` - Description
- `field_2` - Description

### Key Components

#### [Component 1 - e.g., ViewModel]
**File:** `path/to/file.php:line`
**Purpose:** What this component does

```php
// Key code snippet if helpful
```

### Configuration
**Admin Path:** Stores > Configuration > Section > Subsection
**Settings:**
- `config/path/setting1` - Description
- `config/path/setting2` - Description

### Customization

#### Template Customization
How developers can customize templates

#### Styling
How developers can customize styling

### CLI Commands
```bash
bin/magento command:name
```

### API Access
REST or GraphQL API examples (if applicable)

### Related Features
- [Related Page](Related-Page) - Description
- [Another Feature](Another-Feature) - Description

### Technical Documentation
**Developer Documentation:** [Dev Docs](https://example.com/dev)

**Developer Pages:**
- [Setup](https://example.com/setup)
- [API](https://example.com/api)

## Migration from Magento 1
(If applicable - include migration details at the very end)

### Overview
Migration strategy and approach

### Migration Process
Step-by-step migration details

### Migration Challenges & Solutions
Technical challenges encountered

---

## Maintenance
**Last Updated:** YYYY-MM-DD
**Last Reviewed By:** Developer Name
**Magento Version:** 2.4.x / Mage-OS 1.x.x
```

**Key Principles:**
1. **Users First**: Admin guides, usage instructions, and troubleshooting come before technical details
2. **Clear Separation**: Use `---` horizontal rule to separate user content from technical content
3. **Progressive Disclosure**: Start with simple concepts, progress to complex
4. **Accessibility**: Write troubleshooting steps that non-technical users can follow
5. **Technical Section**: Place all developer-specific content (architecture, database schema, code examples) after the separator

#### Documentation Template for Configuration

```markdown
# [Configuration Topic]

## Overview
What this configuration controls and why it's important.

## Admin Path
**Location:** Stores > Configuration > [Section] > [Subsection]
**Or:** [Alternative location if not in standard config]

## Configuration Options

### [Setting 1 Name]
**Config Path:** `section/group/field`
**Type:** [Text/Select/Yes-No/etc.]
**Default Value:** `value`
**Purpose:** What this setting controls

**Valid Values:**
- `value1` - Description of effect
- `value2` - Description of effect

**Dependencies:**
- Requires [module] to be enabled
- Affects [other setting]

**Example:**
```bash
bin/magento config:set section/group/field value1
```

### [Setting 2 Name]
...

## Use Cases

### Use Case 1: [Scenario]
**Configuration:**
```
setting1 = value
setting2 = value
```
**Result:** What happens with this configuration

## Troubleshooting
Common issues related to these configurations

## Related Documentation
Links to related wiki pages or external docs
```

#### Documentation Template for Processes

```markdown
# [Process Name]

## Overview
What this process does and when to use it.

## Prerequisites
- Requirement 1
- Requirement 2
- Access needed

## Process Steps

### Step 1: [Action]
**Command/Action:**
```bash
command to run
```

**Expected Output:**
```
what you should see
```

**Troubleshooting:**
- If X happens, do Y

### Step 2: [Action]
...

## Verification
How to confirm the process completed successfully:
```bash
verification command
```

## Rollback
If something goes wrong, how to undo:
```bash
rollback command
```

## Frequency
How often this process should be performed.

## Automation
Whether this process is automated and how:
- Cron job configuration
- CI/CD integration

## Related Processes
Links to related procedures
```

### Step 5: Add Visual Assets (When Helpful)

**Include screenshots for:**
- Admin configuration screens
- Custom UI elements
- Complex workflows
- Before/after comparisons

**Include diagrams for:**
- Architecture overviews
- Data flow
- Integration points
- System interactions

**How to add images to GitHub Wiki:**
1. Create issue in GitHub (or use existing)
2. Drag and drop image to issue comment
3. Copy the generated URL: `https://github.com/user/repo/assets/...`
4. Use in markdown: `![description](url)`

**Or use images directory:**
```bash
cd <project_name>.wiki/
mkdir -p images
/bin/cp /path/to/screenshot.png images/
git add images/screenshot.png
```

Reference in markdown:
```markdown
![Description](images/screenshot.png)
```

### Step 6: Update Wiki Home/Index and Sidebar

After creating or updating documentation, ensure it's discoverable in both the Home page and sidebar.

#### A. Update Home.md

**Update `Home.md` with:**
```markdown
## Custom Functionality
- [Feature Name](Feature-Name) - Brief description
- [Another Feature](Another-Feature) - Brief description

## Configuration
- [Config Topic](Config-Topic) - Brief description

## Processes
- [Process Name](Process-Name) - Brief description
```

**CRITICAL: GitHub Wiki Link Format**
- ✅ **CORRECT:** `[Link Text](Page-Name)` (no `.md` extension)
- ✅ **CORRECT with anchor:** `[Link Text](Page-Name#anchor)`
- ❌ **WRONG:** `[Link Text](Page-Name.md)` - Causes raw markdown view
- ❌ **WRONG:** `[Link Text](Page-Name.md#anchor)` - Causes raw markdown view

#### B. Update or Create _Sidebar.md

**IMPORTANT:** Always maintain a custom `_Sidebar.md` file to prevent navigation issues.

**Why `_Sidebar.md` is Required:**
- GitHub auto-generates sidebars from page list if `_Sidebar.md` doesn't exist
- Auto-generated sidebars can cause "raw markdown view" issues when clicking anchor links
- Custom sidebar provides better organization and user experience
- Custom sidebar appears consistently on every wiki page

**Create/Update `_Sidebar.md`:**
```markdown
### Wiki Home
[Home](Home)

---

### Site Management
**[Homepage Management](Homepage-Management)**
- [Configure Category List](Homepage-Management#configure-category-list)

**[General Site Docs](Magento-2-Site-Docs)**
- [Categories](Magento-2-Site-Docs#categories)

---

### Custom Features
[Feature Name](Feature-Name)
[Another Feature](Another-Feature)

---

### Configuration
[Config Topic](Config-Topic)

---

### Troubleshooting
[Issue Name](Issue-Name)
```

**Sidebar Best Practices:**
- Use `**bold**` for main section links
- Use bullets `-` for subsections
- Use `---` horizontal rules for visual separators
- Keep hierarchy shallow (2-3 levels maximum)
- Update sidebar every time you add/remove wiki pages
- Always use proper link format (no `.md` extensions)

### Step 7: Commit and Push Wiki Changes

```bash
cd <project_name>.wiki/

# Check what changed
git status
git diff

# Stage changes (including _Sidebar.md if updated)
git add .

# Commit with descriptive message
git commit -m "docs: Add documentation for [feature/topic]

- Added [new page] documenting [feature]
- Updated [existing page] with [new information]
- Updated _Sidebar.md with new navigation links
- Added screenshots for [feature]

Co-Authored-By: Claude <noreply@anthropic.com>"

# Push to wiki repository
git push origin master

# Return to main project
cd ..
```

**Files to Always Check Before Committing:**
- ✅ New/updated content pages
- ✅ `_Sidebar.md` (update if pages added/removed)
- ✅ `Home.md` (update index if major pages added)
- ✅ Any new images in `images/` directory
- ✅ All links use proper format (no `.md` extensions)

### Step 8: Reference Wiki in Code (Optional but Recommended)

Add wiki references in module README or docblocks:

**Module README.md:**
```markdown
# Vendor_ModuleName

Brief description.

## Documentation
Full documentation available in the [project wiki](https://github.com/user/repo/wiki/Feature-Name).

## Quick Start
...
```

**PHP Docblock:**
```php
/**
 * Custom ViewModel for product data
 *
 * @see https://github.com/uptactics/m2_ntotank/wiki/Product-ViewModels
 */
class ProductData extends AbstractViewModel
{
    // ...
}
```

## Documentation Best Practices

### Writing Style
- **Clear and Concise:** Avoid jargon when possible, explain when necessary
- **Action-Oriented:** Use imperative verbs (Configure, Create, Update)
- **Structured:** Use headings, lists, and code blocks for readability
- **Current:** Date documentation and update when features change
- **Complete:** Include examples, edge cases, and troubleshooting

### Technical Accuracy
- **File References:** Always include file paths and line numbers
  - Format: `app/code/Vendor/Module/Model/Example.php:123`
- **Commands:** Provide exact commands that work in the project environment
- **Configuration Paths:** Use exact config paths from Magento
- **Module Names:** Use exact module names as registered

### Maintenance
- **Review Regularly:** Check documentation during upgrades or major changes
- **Version Tracking:** Note which Magento/Mage-OS version documentation applies to
- **Deprecation Notices:** Mark outdated information clearly
- **Update History:** Keep a changelog section for significant doc updates

### Organization
- **Logical Grouping:** Group related topics together
- **Cross-References:** Link related pages liberally
- **Search-Friendly:** Use clear titles and keywords
- **Navigation:** Maintain a clear hierarchy in Home.md

## Common Documentation Scenarios

### Scenario 1: Just Built a Custom Module

**What to document:**
1. Module purpose and business requirements
2. Key classes and their responsibilities
3. Configuration options (if any)
4. How to use in admin/frontend
5. Extension points for future customization
6. Testing notes

**Wiki page:**
- Create new page: `Custom-Module-Name.md`
- Update `Home.md` with link

### Scenario 2: Customized Third-Party Module

**What to document:**
1. Which module was customized and why
2. What was changed (plugins, overrides, etc.)
3. File locations of customizations
4. Upgrade considerations
5. Alternative approaches considered

**Wiki page:**
- Update existing page or create: `Third-Party-Module-Customizations.md`
- Link from main docs page

### Scenario 3: Completed Data Migration

**What to document:**
1. Migration tool and version used
2. Custom migration scripts created
3. Data transformations applied
4. Post-migration cleanup steps
5. Known data discrepancies
6. Verification queries

**Wiki page:**
- Update `Data-Migration.md`
- Add specific migration date and results

### Scenario 4: Custom Admin Configuration

**What to document:**
1. What the config controls
2. Admin path to settings
3. Each setting and its purpose
4. Valid values and effects
5. Dependencies between settings
6. CLI commands to set values

**Wiki page:**
- Update `Magento-2-Site-Docs.md` or create specific config page

### Scenario 5: Frontend Custom Feature

**What to document:**
1. What the feature does (user perspective)
2. Where it appears on the site
3. How it works technically (templates, ViewModels, Alpine.js)
4. Configuration options
5. Styling customizations
6. JavaScript interactions

**Wiki page:**
- Create: `Custom-Feature-Name.md`
- Link from main docs

### Scenario 6: Complex Bug Fix

**What to document:**
1. Original issue/symptom
2. Root cause discovered
3. Solution implemented
4. Files changed
5. How to prevent recurrence
6. Related issues to watch for

**Wiki page:**
- Add to troubleshooting section of related feature page
- Or update general troubleshooting page

## Wiki Maintenance Schedule

### After Every Feature Implementation
- Document immediately while details are fresh
- Include in same PR/branch or separate docs commit

### Monthly Review
- Check for outdated information
- Update version compatibility notes
- Fix broken links
- Add missing screenshots

### During Magento Upgrades
- Review all custom module documentation
- Update compatibility notes
- Document any required changes
- Archive deprecated feature docs

### During Developer Onboarding
- Use wiki as onboarding material
- Note any confusion or missing information
- Update based on questions asked

## Integration with Main CLAUDE.md

The project's `CLAUDE.md` should reference the wiki for detailed documentation:

```markdown
## Documentation

Custom functionality is documented in the [project wiki](https://github.com/user/repo/wiki):
- [Feature Name](https://github.com/user/repo/wiki/Feature-Name)
- [Another Feature](https://github.com/user/repo/wiki/Another-Feature)

See the wiki for detailed documentation on custom modules, configurations, and processes.
```

## Common Pitfalls to Avoid

### ❌ DON'T: Use .md Extensions in Wiki Links
**Bad:** `[Link](Page-Name.md)` or `[Link](Page-Name.md#anchor)`
**Good:** `[Link](Page-Name)` or `[Link](Page-Name#anchor)`

**Why it's wrong:**
- GitHub wiki links with `.md` extensions can cause "raw markdown view" issues
- Users see raw markdown source instead of rendered wiki pages
- Clicking anchor links from sidebar redirects to `raw.githubusercontent.com`
- This is a GitHub wiki-specific issue that breaks user experience

**Real Example of the Problem:**
```
Bad URL:  raw.githubusercontent.com/wiki/user/repo/Page.md#anchor
Good URL: github.com/user/repo/wiki/Page#anchor
```

**How to Fix:**
- Remove all `.md` extensions from wiki links
- Format: `[Text](Page-Name)` not `[Text](Page-Name.md)`
- Anchors: `[Text](Page#section)` not `[Text](Page.md#section)`

---

### ❌ DON'T: Forget to Create/Update _Sidebar.md
**Bad:** Create new pages without maintaining custom sidebar
**Good:** Always update `_Sidebar.md` when adding/removing pages

**Why it's wrong:**
- GitHub auto-generates sidebar if `_Sidebar.md` doesn't exist
- Auto-generated sidebars cause navigation issues with anchor links
- Users get inconsistent navigation experience
- No organization or categorization of pages

**What Happens Without Custom Sidebar:**
- Sidebar is auto-generated from page list (alphabetical)
- Clicking links with anchors can redirect to raw markdown view
- No ability to organize pages by category
- Sidebar may differ between pages

**How to Fix:**
1. Create `_Sidebar.md` in wiki root directory
2. Add organized navigation structure
3. Update sidebar every time you add/remove wiki pages
4. Use proper link format (no `.md` extensions)
5. Commit `_Sidebar.md` with every wiki update

**Example Custom Sidebar Structure:**
```markdown
### Wiki Home
[Home](Home)

---

### Category Name
**[Main Page](Main-Page)**
- [Subsection 1](Main-Page#subsection-1)
- [Subsection 2](Main-Page#subsection-2)

[Another Page](Another-Page)

---

### Another Category
[Page Name](Page-Name)
```

---

### ❌ DON'T: Skip Sidebar When Adding Pages
**Bad:** Create page → Update Home.md → Commit (forgot sidebar)
**Good:** Create page → Update Home.md → Update _Sidebar.md → Commit all

**Why it's wrong:**
- New pages won't appear in navigation sidebar
- Users can't find the new documentation
- Inconsistent navigation experience
- Have to remember to update sidebar later

**Workflow Checklist:**
- ✅ Create/update wiki page content
- ✅ Update `Home.md` index (if major page)
- ✅ Update `_Sidebar.md` navigation
- ✅ Verify all links use proper format (no `.md`)
- ✅ Commit all files together
- ✅ Test navigation in browser

---

### ❌ DON'T: Mix Link Formats
**Bad:** Some links with `.md`, some without
**Good:** Consistently use proper format (no `.md`) throughout wiki

**Why it's wrong:**
- Inconsistent behavior - some links work, others show raw view
- Confusing for future contributors
- Harder to maintain and debug
- Unprofessional appearance

**How to Prevent:**
- Use search/replace to fix all links: `](.*.md)` → `](Page-Name)`
- Set up editor to highlight `.md` in links
- Review all links before committing
- Add to pull request checklist

---

## Quality Checklist

Before finalizing wiki documentation, verify:

- [ ] Title is clear and descriptive
- [ ] Overview explains purpose in 2-3 sentences
- [ ] File paths include line numbers where relevant
- [ ] Code examples are tested and accurate
- [ ] Screenshots are clear and annotated if needed
- [ ] Configuration paths are exact (not approximate)
- [ ] Commands work in the project environment
- [ ] Links to related pages are included
- [ ] **All wiki links use proper format (NO .md extensions)**
- [ ] **_Sidebar.md is updated with new page links**
- [ ] Troubleshooting section addresses common issues
- [ ] Documentation is dated and attributed
- [ ] Home.md index is updated
- [ ] Wiki changes are committed and pushed
- [ ] No sensitive information (passwords, API keys, etc.)

## Project-Specific Patterns

### For Hyvä Themes Projects
Document:
- Custom Tailwind components and utilities
- Alpine.js reactive components
- ViewModels and their data structures
- Template overrides and customizations
- Tailwind build process customizations

### For Mage-OS Projects
Document:
- Differences from standard Magento 2 behavior
- Mage-OS-specific features used
- Compatibility considerations
- Custom patches applied

### For Migration Projects
Document:
- M1 to M2 migration decisions
- Data transformation logic
- Custom migration scripts
- Post-migration cleanup tasks
- Known data discrepancies

## Examples from This Project

### Good Example: Grouping Related Products
**File:** `m2_ntotank.wiki/Grouping-Related-Products.md`
**Strengths:**
- Clear purpose stated upfront
- Module location specified
- How it works explained
- Usage instructions included

### Good Example: Data Migration
**File:** `m2_ntotank.wiki/Data-Migration.md`
**Strengths:**
- Step-by-step process
- Commands provided
- Troubleshooting included
- Post-migration tasks documented

### Area for Improvement: Magento 2 Site Docs
**File:** `m2_ntotank.wiki/Magento-2-Site-Docs.md`
**Could add:**
- More code examples
- File path references
- Configuration paths
- Developer notes

## When NOT to Use This Skill

**Don't use wiki documentation for:**
- **Code-level documentation** → Use PHPDoc comments in code
- **API documentation** → Use Swagger/OpenAPI specs
- **Inline code explanations** → Use code comments
- **Temporary development notes** → Use `ai/` directory in project
- **Standard Magento features** → Link to official Magento docs
- **Sensitive information** → Store securely, not in public wiki

**Instead:**
- Code comments for implementation details
- `README.md` in module for quick reference
- `ai/` folder for AI conversation summaries
- Private documentation for sensitive configs

## Success Criteria

✅ **Custom functionality is discoverable** - New developers can find it
✅ **Purpose is clear** - Anyone can understand why it exists
✅ **Usage is explained** - Admins and developers know how to use it
✅ **Maintenance is enabled** - Future updates won't break undocumented features
✅ **Troubleshooting is proactive** - Common issues are documented before support tickets
✅ **Client handover is smooth** - Client can maintain site with wiki reference

## Quick Reference

### Common Commands

```bash
# Navigate to wiki
cd <project_name>.wiki/

# Pull latest
git pull origin master

# Create new page
touch New-Feature-Name.md

# Check what changed
git status
git diff

# Commit changes
git add .
git commit -m "docs: description"
git push origin master

# Return to project
cd ..
```

### File Path Format
```
app/code/Vendor/Module/Model/Example.php:123
```

### Config Path Format
```
section/group/field
```

### Admin Path Format
```
Stores > Configuration > Section > Subsection > Field
```

### Wiki Link Format
```markdown
[Link Text](Page-Name.md)
```

### Image Reference Format
```markdown
![Description](images/screenshot.png)
# or
![Description](https://github.com/user/repo/assets/...)
```

## Related Resources

- [GitHub Wiki Documentation](https://docs.github.com/en/communities/documenting-your-project-with-wikis)
- [Markdown Guide](https://www.markdownguide.org/)
- [Magento DevDocs](https://developer.adobe.com/commerce/php/development/)
- [Mage-OS Documentation](https://mage-os.org/)

---

**Remember:** Good documentation is a gift to your future self and your team. Take the time to document well, and you'll save countless hours in the future.
