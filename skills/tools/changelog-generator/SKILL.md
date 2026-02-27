---
name: "Changelog Generator"
description: "Generate professional changelogs from git commits with industry-standard categories. Automatically activates when users request changelog generation with time periods."
version: "1.0.0"
dependencies:
  - "git"
  - "bash"
---

# Changelog Generator

Automatically generates professional changelogs from git commits with intelligent filtering, categorization, and multiple output formats for the Quorum project.

## Description

This skill provides comprehensive changelog generation for Quorum project releases, creating both technical documentation and user-friendly communications. It analyzes git commits, filters for relevant changes, categorizes improvements, and generates polished output in multiple formats.

**Use this skill when the user mentions:**
- Generating changelogs with any time period specification
- Creating release notes or development summaries
- Preparing user communications about new features
- Summarizing changes for stakeholders or beta testers
- Analyzing development progress over specific timeframes

**Automatically triggers on phrases like:**
- "generate changelog" / "generate a changelog"
- "create changelog" / "create a changelog"
- "changelog for the past [X] days"
- "changelog for the last [X] weeks"
- "generate changelog for [time period]"
- "create release notes"
- "summarize recent changes"
- "changelog for [X] days"

**Time period examples that trigger the skill:**
- "generate changelog for the past 30 days"
- "create changelog for last 3 days"
- "changelog for 2 weeks"
- "generate changelog for yesterday"
- "changelog since last month"

## Core Capabilities

### 1. Technical Changelog Generation
- **Script**: `changelog-generator.sh`
- **Audience**: Developers, technical stakeholders
- **Content**: Detailed technical changes with GitHub links
- **Categories**: New Features, Enhancements, Bug Fixes, Localization, Compatibility, Maintenance
- **Format**: Markdown + Text versions with commit references

### 2. User-Friendly Changelog Generation
- **Script**: `changelog-user-friendly.sh`
- **Audience**: End users, beta testers, non-technical stakeholders
- **Content**: Benefit-focused descriptions without technical jargon
- **Categories**: New Features, Enhancements, Bug Fixes, Localization, Compatibility, Maintenance
- **Format**: Engaging copy optimized for user excitement

### 3. Intelligent Commit Analysis
- **Conventional Commit Support**: Leverages consistent emoji + conventional commit format
- **Precise Categorization**: Maps commit types directly to changelog categories
- **Smart Filtering**: Uses configurable patterns to identify user-facing changes
- **Quality Enhancement**: Post-processes output for clarity and engagement

#### Commit Type → Category Mapping
- `✨ feat:` / `feat:` → 🎉 **New Features**
- `🎨 style:` / `🚀 perf:` / `perf:` → ✨ **Enhancements**
- `🐛 fix:` / `fix:` → 🐛 **Bug Fixes**
- `🌐 i18n:` / `i18n:` → 🌐 **Localization**
- `🧹 chore:` / `⚙️ refactor:` → 🔧 **Maintenance** (technical only)
- Platform-specific commits → 📱 **Compatibility**

### 4. Multiple Output Formats
- **Markdown**: For documentation, GitHub releases, and web content
- **Plain Text**: For social media, emails, and messaging platforms
- **Analysis Files**: For review and further processing

## Available Scripts

### Technical Changelog (`changelog-generator.sh`)
```bash
./changelog-generator.sh [DAYS] [BRANCH] [CLAUDE_ENHANCE] [OUTPUT_DIR] [--show]
```

**Parameters:**
- `DAYS`: Number of days to analyze (default: 7)
- `BRANCH`: Git branch to analyze (default: cross-platform)
- `CLAUDE_ENHANCE`: Enable Claude post-processing (default: false)
- `OUTPUT_DIR`: Output directory (default: skill/changelogs/)
- `--show`: Display results immediately

**Example:** `./changelog-generator.sh 14 develop true`

### User-Friendly Changelog (`changelog-user-friendly.sh`)
```bash
./changelog-user-friendly.sh [DAYS] [BRANCH] [CLAUDE_ENHANCE] [OUTPUT_DIR] [--show]
```

**Parameters:**
- `DAYS`: Number of days to analyze (default: 14)
- `BRANCH`: Git branch to analyze (default: cross-platform)
- `CLAUDE_ENHANCE`: Enable Claude enhancement (default: true)
- `OUTPUT_DIR`: Output directory (default: skill/changelogs/)
- `--show`: Display preview

**Example:** `./changelog-user-friendly.sh 30 main true`

### Output Structure

All changelogs are stored within the skill directory:
```
.claude/skills/changelog-generator/
├── changelogs/
│   ├── quorum-changelog_2025-11-16.md
│   ├── quorum-changelog_2025-11-16.txt
│   ├── quorum-changelog-user-friendly_2025-11-16_14-30.md
│   ├── quorum-changelog-user-friendly_2025-11-16_14-30.txt
│   └── commits-analysis_2025-11-16_14-30.txt
└── ...
```

## Configuration Files

### Commit Filters (`filters/`)

**User-Facing Commits** (`user-facing-commits.txt`):
```
✨ feat:
feat:
🐛 fix:
fix:
🎨 style:
style:
🚀 perf:
perf:
Add
Implement
Fix
Improve
Enhance
Update
```

**Exclude Patterns** (`exclude-patterns.txt`):
```
playground
audit
component complexity
primitive
typescript
build
lint
task:
doc:
chore:
refactor:
test:
build:
```

### Templates (`templates/`) - MANDATORY

**IMPORTANT: You MUST read and follow these templates exactly for consistent output.**

Templates define the exact structure, formatting, and layout for each output format. Before generating any changelog:

1. **Read the appropriate template file(s)** for the requested output format
2. **Follow the template structure exactly** - same headers, separators, sections
3. **Use the same formatting conventions** - bullet styles, emoji placement, spacing

| Output Type | Template File | Purpose |
|-------------|---------------|---------|
| User-friendly (Markdown) | `templates/user-friendly-format.md` | Discord, documentation, web |
| User-friendly (Plain text) | `templates/user-friendly-format.txt` | Telegram, email, messaging |
| Technical (Markdown) | `templates/technical-format.md` | GitHub releases, dev docs |
| Social media | `templates/social-media-format.txt` | Twitter, short announcements |

**Template Variables:**
- `{{DATE_RANGE}}` - The date range covered (e.g., "December 11-13, 2025")
- `{{PROJECT_NAME}}` - Project name (Quorum)
- `{{CONTENT}}` - Main changelog content

**Key Formatting Rules from Templates:**
- Use `---` as section separators in Markdown (NOT `━━━` unicode lines)
- Use `---` as section separators in Plain text (NOT `━━━` unicode lines)
- Use `•` for bullet points
- Use subcategory emojis from the reference table in templates
- Include the footer with improvement count and date range
- Include the Quorum links at the bottom

## Instructions

When the user requests changelog generation, follow this workflow:

### Step 1: Determine Changelog Type
Analyze the request to identify:
- **Technical Changelog**: For developers, detailed commit tracking, technical stakeholders
- **User-Friendly Changelog**: For end users, marketing, beta announcements, feature highlights
- **Custom Analysis**: Specific date ranges, branches, or focus areas

### Step 2: Read Templates (MANDATORY)

**Before writing any output, you MUST read the template files:**

```
# For user-friendly changelogs, read BOTH:
templates/user-friendly-format.md   # For markdown output
templates/user-friendly-format.txt  # For plain text output

# For technical changelogs:
templates/technical-format.md
```

This ensures consistent formatting across all changelogs.

### Step 3: Execute Appropriate Script & Generate Output

**For Technical Changelogs:**
1. Run `changelog-generator.sh` with appropriate parameters
2. Review generated output for accuracy
3. Enhance with additional context if needed
4. Save in both markdown and text formats **following template structure**

**For User-Friendly Changelogs:**
1. Run `changelog-user-friendly.sh` to generate commit analysis (or use git log directly)
2. Use the analysis to create engaging, benefit-focused descriptions
3. Apply user-friendly writing guidelines:
   - Focus on user capabilities, not technical details
   - Use active language and exciting tone
   - Keep bullet points concise (under 15 words)
   - Group related changes into single descriptions
4. Generate both markdown and text versions **using the exact template structure**
5. Include project-specific formatting (test links, access instructions)

### Step 4: Apply Enhancement Guidelines

**User-Friendly Enhancement Rules:**

✅ **DO:**
- Focus on what users can accomplish with new features
- Use exciting, active language that motivates testing
- Group multiple technical commits into single user-facing features
- Keep descriptions scannable and concise
- Include practical benefits and improvements

❌ **DON'T:**
- Use technical jargon (refactor, implement, optimize, etc.)
- Mention file paths, code references, or technical debt
- Write obvious statements ("works better now")
- Create long, detailed explanations
- Include internal development processes

**Example Transformations:**

*Multiple commits:*
- "Add pin message functionality to MessageComponent"
- "Implement pin UI component with animations"
- "Add unpin capability and state management"

*Becomes:*
📌 **Pinned Messages & Content Management**
- Pin important messages in channels for easy member access

### Step 5: Quality Assurance

**Technical Changelog Check:**
- All commit links work and point to correct repository
- Categories are properly populated with relevant changes
- Statistics accurately reflect included vs. filtered commits
- Markdown formatting is valid and renders correctly

**User-Friendly Changelog Check:**
- Non-technical users can understand all descriptions
- Each bullet point focuses on user benefits
- Tone is exciting and encourages feature testing
- No technical jargon or implementation details remain
- Categories only appear if they contain relevant content

### Step 6: Output and Delivery

1. **Save files** in the specified output directory
2. **Provide file paths** to the user for easy access
3. **Show preview** if requested with --show flag
4. **Offer additional formats** if needed (social media posts, email content, etc.)

## Example Usage Scenarios

### Scenario 1: Weekly Technical Update
**User Request:** "Generate a technical changelog for the last week"

**Skill Response:**
1. Runs `changelog-generator.sh 7 cross-platform false`
2. Generates technical changelog with categorized commits
3. Provides markdown and text versions
4. Shows summary statistics

### Scenario 2: User-Friendly Release Announcement
**User Request:** "Create a user-friendly changelog for the beta users"

**Skill Response:**
1. Runs `changelog-user-friendly.sh 14 cross-platform true`
2. Analyzes commits and categorizes by user impact
3. Generates engaging descriptions focusing on user benefits
4. Creates both markdown (for documentation) and text (for messaging)
5. Includes project-specific formatting and test links

### Scenario 3: Custom Date Range Analysis
**User Request:** "What changed between September 1st and October 15th?"

**Skill Response:**
1. Calculates appropriate parameters for date range
2. Runs appropriate script with custom parameters
3. Generates comprehensive changelog covering the period
4. Highlights major features and improvements

### Scenario 4: Multi-Format Release Package
**User Request:** "Prepare release materials for version 2.1"

**Skill Response:**
1. Generates technical changelog for developer documentation
2. Creates user-friendly version for announcement
3. Provides text versions for social media and email
4. Suggests additional formats (GitHub release notes, newsletter content)

## Integration with Development Workflow

### Pre-Release Process
1. **Generate technical changelog** for internal review
2. **Create user-friendly version** for public announcement
3. **Review with stakeholders** using generated previews
4. **Finalize content** with any manual adjustments
5. **Deploy across channels** using appropriate formats

### Continuous Documentation
- **Weekly technical summaries** for development team
- **Bi-weekly user updates** for beta testing community
- **Monthly feature highlights** for broader audience
- **Ad-hoc analysis** for specific feature releases

### Quality Standards
- **Accuracy**: All information verified against actual commits
- **Clarity**: Technical content accessible to intended audience
- **Engagement**: User-friendly content motivates feature adoption
- **Consistency**: Formatting and tone align with project standards

## Advanced Features

### Custom Filtering
- Modify `filters/` configuration files for project-specific needs
- Add new commit patterns or exclude additional noise
- Create branch-specific or feature-specific filters

### Template Customization
- Edit `templates/` files to match brand guidelines
- Add project-specific headers, footers, or formatting
- Include custom variables and substitutions

### Multi-Branch Analysis
- Compare changes across multiple branches
- Generate merge-specific changelogs
- Track feature branch progress

### Integration Hooks
- Connect with CI/CD pipelines for automated changelog generation
- Integrate with release management tools
- Export to external documentation systems

This skill transforms the manual, error-prone process of changelog creation into an automated, intelligent system that produces professional documentation while saving significant time and ensuring consistency.