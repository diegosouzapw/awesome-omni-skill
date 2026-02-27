---
name: brand-guidelines
description: Apply comprehensive brand style guidelines to all project work including UI components, code, documentation, and user-facing content. Use when creating or editing Angular components, HTML/CSS, TypeScript code, writing copy, documenting features, or any customer-facing materials to ensure brand consistency.
allowed-tools: Read, Edit, Write, Grep, Glob
---

# Brand Style Guidelines

Apply consistent brand standards across all aspects of the Angular project.

## When to Use

Automatically invoke this skill when:
- Creating new Angular components, modules, or services
- Writing user-facing text (labels, errors, help text, notifications)
- Editing existing components for consistency
- Creating or updating documentation
- Writing TypeScript code (to follow naming and structure conventions)
- User explicitly requests brand compliance check

## Instructions

### Step 1: Identify Work Type

Determine what needs brand guidelines:
- **UI/Visual**: Angular components, templates, CSS/SCSS, styling
- **Copywriting**: Headings, body text, CTAs, messages
- **Code**: Components, services, modules, interfaces, file structure
- **Documentation**: README, API docs, guides, comments

### Step 2: Load Relevant Guidelines

Read the appropriate reference files from `reference/`:
- `visual-ui.md` - Colors, typography, spacing, components
- `writing-tone.md` - Voice, terminology, messaging
- `code-conventions.md` - Naming, structure, Angular patterns
- `documentation.md` - Format, style, organization

### Step 3: Apply Guidelines

#### For UI/Visual Work
- Use only approved brand colors from palette
- Apply correct typography hierarchy and font families
- Follow spacing system (8px grid)
- Use established component patterns
- Ensure WCAG AA accessibility compliance

#### For Copywriting
- Match brand tone of voice (professional yet approachable)
- Use approved terminology consistently
- Write clear, action-oriented copy
- Address users directly ("you")
- Keep language concise and specific

#### For Code
- Follow Angular style guide and naming conventions
- Use consistent file and folder structure
- Apply established architectural patterns (services, guards, interceptors)
- Write self-documenting code with clear intent
- Add JSDoc comments for public APIs

#### For Documentation
- Follow standard document structure
- Use consistent formatting and headers
- Include required sections (Purpose, Usage, Examples)
- Write for the intended audience
- Keep examples current and tested

### Step 4: Use Templates

Start from templates when creating new artifacts:
- `templates/component-template.component.ts` for Angular components
- `templates/service-template.service.ts` for services
- `templates/documentation-template.md` for docs

### Step 5: Validate Compliance

Run through the checklist:
- ✓ Visual elements match brand system
- ✓ Copy matches tone and terminology
- ✓ Code follows Angular conventions
- ✓ Documentation is complete and formatted correctly
- ✓ Accessibility standards met
- ✓ Consistent with existing project patterns

## Quick Reference Rules

### Visual
- **Colors**: Use CSS custom properties (`var(--primary-color)`, not hex codes directly)
- **Typography**: Follow hierarchy (H1: 32px, H2: 24px, Body: 16px)
- **Spacing**: Multiples of 8px only
- **Components**: Reuse existing, don't recreate

### Writing
- **Voice**: Clear, helpful, professional yet friendly
- **Structure**: Active voice, verb-first CTAs
- **Terminology**: Consistent (see writing-tone.md)
- **Length**: Concise and scannable

### Code
- **Naming**:
  - Components: PascalCase with suffix (e.g., `UserProfileComponent`)
  - Services: PascalCase with suffix (e.g., `AuthService`)
  - Files: kebab-case (e.g., `user-profile.component.ts`)
- **Organization**: Feature-based module structure
- **Comments**: JSDoc for public APIs, explain "why" not "what"
- **Patterns**: Follow Angular best practices

### Documentation
- **Structure**: Purpose → Usage → Examples → API
- **Code blocks**: Syntax-highlighted with language tags
- **Links**: Relative paths for internal docs
- **Updates**: Keep in sync with code changes

## Examples

See `examples/before-after.md` for concrete transformations across all guideline types.

## Troubleshooting

**Issue**: Existing code doesn't match guidelines
- **Solution**: Refactor incrementally, prioritize user-facing areas

**Issue**: Guidelines conflict with technical requirements
- **Solution**: Document exceptions with reasoning

**Issue**: Unclear which guideline applies
- **Solution**: Check reference files or ask for clarification

## Validation Checklist

Before completing any work:
- [ ] Follows visual/UI guidelines
- [ ] Copy matches brand voice
- [ ] Code adheres to Angular conventions
- [ ] Documentation is complete
- [ ] Consistent with existing patterns
- [ ] No accessibility issues
- [ ] User-facing content reviewed
