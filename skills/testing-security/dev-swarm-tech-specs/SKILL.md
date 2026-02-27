---
name: dev-swarm-tech-specs
description: Define technical specifications including tech stack, security, theme standards (from UX mockup), coding standards, and testing standards. Use when user asks to define tech specs, choose tech stack, or start Stage 7 after architecture.
---

# AI Builder - Technical Specifications

This skill creates/updates the technical specifications documentation defining the technology stack, security posture, theme standards (extracted from UX mockup), coding standards, testing standards, and security standards.

## When to Use This Skill

- User asks to "define tech specs" or "choose tech stack"
- User requests to start Stage 7 or the next stage after architecture
- User wants to select technologies and frameworks
- User wants to establish coding and testing standards
- User needs to define security standards

## Prerequisites

This skill requires **06-architecture** to be completed for L3+ projects. For L2 projects, this stage can proceed without architecture documentation.

## Your Roles in This Skill

- **Tech Manager (Architect)**: Lead tech stack selection and standards definition. Review architecture to choose appropriate technologies. Define coding and testing standards. Ensure technical choices align with requirements and constraints.
- **Security Engineer**: Define security posture, authentication approach, and secure coding standards. Identify security threats and mitigation strategies. Establish security testing and compliance requirements.
- **UI Designer**: Extract theme standards from approved UX mockup. Document colors, fonts, spacing, and design tokens. Ensure design consistency rules are clear for implementation.
- **DevOps Engineer**: Review tech stack for deployment and operational feasibility. Provide input on infrastructure compatibility. Consider monitoring and logging requirements.

## Role Communication

As an expert in your assigned roles, you must announce your actions before performing them using the following format:

As a {Role} [and {Role}, ...], I will {action description}

This communication pattern ensures transparency and allows for human-in-the-loop oversight at key decision points.
## Instructions

Follow these steps in order:

### Step 0: Verify Prerequisites and Gather Context

First read and understand rules: `dev-swarm/docs/research-specs-rules.md` then:

1. **Check for Project Scale (L2 vs L3+):**
   - Check `00-init-ideas/README.md` or classification to determine project scale.

2. **Check if `06-architecture/` folder exists:**
   - **For L3+ projects (Mandatory):**
     - If NOT found: Inform user they need to create architecture first, then STOP.
     - If found: Read all files.
   - **For L2 projects (Optional):**
     - If found: Read files.
     - If NOT found: Proceed without it.

3. **Check if `05-ux/` folder exists (Mandatory for L3+):**
   - If NOT found and project is L3+: Warn user.
   - For L2: Skip if not relevant.
     - **CSS variables and design tokens**
     - **Color palette**
     - **Typography (fonts, sizes)**
     - **Spacing system**
     - **Border radius and shadows**
     - **Component styles**

3. **Check if `00-init-ideas/` folder exists (recommended):**
   - If found: Read all files to understand it

4. **Check if `04-prd/` folder exists (recommended):**
   - If found: Read to understand:
     - Non-functional requirements (performance, security, compliance)
     - Technical constraints

5. **Check if `03-mvp/` folder exists (recommended):**
   - If found: Read to understand:
     - MVP scope (prioritize tech choices for MVP)
     - Timeline constraints

6. **Check if this stage should be skipped:**
   - Check if `07-tech-specs/SKIP.md` exists
   - **If SKIP.md exists:**
     - Read SKIP.md to understand why this stage was skipped
     - Inform the user: "Stage 7 (tech-specs) is marked as SKIP because [reason from SKIP.md]"
     - Ask the user: "Would you like to proceed to the next stage (devops)?"
     - **If user says yes:**
       - Exit this skill and inform them to run the next stage skill
     - **If user says no:**
       - Ask if they want to proceed with tech specs anyway
       - If yes, delete SKIP.md and continue with this skill
       - If no, exit the skill

7. **Check if `07-tech-specs/` folder exists:**
   - If exists: Read all existing files to understand current tech specs state
   - If NOT exists: Will create new structure

8. **If README.md exists:** Check whether it requires diagrams. If it does,
   follow `dev-swarm/docs/mermaid-diagram-guide.md` and use the
   `dev-swarm-mermaid` skill to render outputs.

9. **Read source code structure guidance (mandatory):**
   - Read `dev-swarm/docs/source-code-structure.md`
   - Use it as the baseline when creating `07-tech-specs/source-code-structure.md`

10. Proceed to Step 1 with gathered context

### Step 1: Refine Design Requirements in README and Get Approval

**CRITICAL: Create/update README.md first based on previous stage results, get user approval, then create other docs.**

1. **Analyze information from previous stages:**
   - Read `06-architecture/` to understand system components and deployment
   - Read `05-ux/mockups/styles.css` to extract theme (CRITICAL for theme-standards.md)
   - Read `04-prd/` to understand non-functional requirements
   - Read `03-mvp/` (if exists) to understand what to prioritize
   - Consider cost-budget constraints for this stage

2. **Create or update 07-tech-specs/README.md with refined requirements:**
   - **For L2 projects:** Create a simple README (just several lines) indicating the project level and that only `tech-stack.md` is required.
   - **For L3+ projects:** List deliverables explicitly in README (typical: tech-stack.md, security.md, theme-standards.md, coding-standards.md, source-code-structure.md, testing-standards.md, security-standards.md)
   - **Stage overview and objectives** (based on previous stage context)
   - **Owners:** Tech Manager (lead), Security Engineer, UI Designer, DevOps Engineer
   - **Diagrams (if required by project init):**
     - Reference `dev-swarm/docs/mermaid-diagram-guide.md`
     - Include `diagram/` deliverables when needed
   - **What tech specs will include:**
     - Technology stack selection with rationale
     - Security posture and authentication approach
     - Theme standards extracted from UX mockup (CRITICAL)
     - Coding standards and best practices
     - Testing standards and coverage requirements
     - Security standards for secure coding
   - **Methodology:**
     - How tech stack will be selected (based on architecture + requirements)
     - How theme will be extracted from mockup CSS (DO NOT invent values)
   - **Deliverables planned:**
     - List of files that will be created (tech-stack.md, theme-standards.md, etc.)
   - **Budget allocation for this stage** (from cost-budget.md)
   - **Status:** In Progress (update to "Completed" after implementation)

3. **Present README to user:**
   - Show the tech specs approach and what will be defined
   - Show what documentation files will be created
   - Explain how it aligns with previous stages
   - Ask: "Does this tech specs plan look good? Should I proceed with defining technology stack and standards?"

4. **Wait for user approval:**
   - **If user says yes:** Proceed to Step 2
   - **If user says no:**
     - Ask what needs to be changed
     - Update README based on feedback
     - Ask for approval again

### Step 2: Create/Update Tech Specs Structure

**Only after user approves the README:**

1. **Create files as specified in the approved README.md:**

   **IMPORTANT:** The file structure below is a SAMPLE only. The actual files you create must follow what was approved in the README.md in Step 1.

   **Typical structure (example):**
   ```
   07-tech-specs/
   ├── README.md (already created and approved in Step 1)
   ├── tech-stack.md (if specified in README)
   ├── security.md (if specified in README)
   ├── theme-standards.md (if specified in README - MUST extract from UX mockup)
   ├── coding-standards.md (if specified in README)
   ├── source-code-structure.md (if specified in README)
   ├── testing-standards.md (if specified in README)
   └── security-standards.md (if specified in README)
   ```

   **Create only the files listed in the README's "Deliverables planned" section.**

### Step 3: Create/Update Technical Specifications Documentation

**IMPORTANT: Only create tech specs documentation after README is approved in Step 1.**

**NOTE:** The content structure below provides GUIDELINES for typical tech specs documentation. Adapt based on the approved README and project needs.

**07-tech-specs/README.md:**
- Stage overview and objectives
- Specify the owners: Tech Manager (lead), Security Engineer, UI Designer, DevOps Engineer
- Summary of technical approach and key decisions
- Links to all tech specs documentation files
- Rationale for major technical decisions

**tech-stack.md:**

Define the complete technology stack:

1. **Technology Selection Criteria:**
   - Must support requirements from PRD
   - Must work with chosen architecture
   - Team familiarity and expertise
   - Community support and ecosystem
   - Performance and scalability
   - Cost considerations
   - Long-term maintainability

2. **Frontend Stack:**
   - Define framework/library choice and rationale
   - Specify language (JavaScript/TypeScript) and why
   - Select UI component library approach
   - Choose state management solution
   - Define styling approach
   - Specify build tool and package manager
   - For mobile: Define framework and navigation approach
   - For desktop: Define framework if applicable

3. **Backend Stack:**
   - Select language and version with rationale
   - Choose framework and explain why
   - Define API style (REST/GraphQL/gRPC/etc.)
   - Specify background job processing approach
   - Define scheduler if needed

4. **Database & Data Storage:**
   - Select primary database type and version with rationale
   - Choose caching solution
   - Define object storage approach
   - Specify search solution if applicable

5. **Infrastructure & Deployment:**
   - Select cloud provider with rationale
   - Choose compute service approach
   - Define container orchestration if applicable
   - Specify CDN service

6. **DevOps & Tools:**
   - Define version control platform and branching strategy
   - Choose CI/CD platform with rationale
   - Select application monitoring solution
   - Choose error tracking service
   - Define logging approach
   - Select infrastructure as code tool

7. **External Services & APIs:**
   - Choose authentication service approach
   - Select email delivery service
   - Define payment processing if applicable
   - Choose analytics service
   - Select SMS service if applicable

**security.md:**

Define the security posture and approach:

1. **Security Principles:**
   - Security by design
   - Defense in depth
   - Principle of least privilege
   - Zero trust architecture
   - Fail securely

2. **Authentication Approach:**
   - **Method**: [JWT / Session-based / OAuth 2.0 / SAML / etc.]
   - **Token Storage**: [httpOnly cookies / localStorage / sessionStorage]
   - **Token Expiration**: [Access token: 15 minutes, Refresh token: 7 days, etc.]
   - **Multi-Factor Authentication (MFA)**: [Required / Optional / Not implemented]
   - **Password Policy**:
     - Minimum length: [8-12 characters]
     - Complexity requirements
     - Password hashing: [bcrypt / Argon2 / PBKDF2]
     - Rounds/iterations: [Specify]

3. **Authorization Model:**
   - **Approach**: [RBAC (Role-Based) / ABAC (Attribute-Based) / ACL / etc.]
   - **Roles**: [List of roles: Admin, User, Moderator, etc.]
   - **Permissions**: [How permissions are defined and checked]

4. **Secrets Management:**
   - **Approach**: [AWS Secrets Manager / HashiCorp Vault / Environment Variables / etc.]
   - **API Keys**: [How stored and rotated]
   - **Database Credentials**: [How stored and accessed]
   - **Encryption Keys**: [How managed]

5. **Data Security:**
   - **Encryption at Rest**: [AES-256 / etc.]
   - **Encryption in Transit**: [TLS 1.3 / etc.]
   - **PII Handling**: [How personally identifiable information is protected]
   - **Data Retention**: [How long data is kept]
   - **Data Deletion**: [Hard delete vs soft delete approach]

6. **Threat Mitigation:**

   **OWASP Top 10 Protections:**
   - [ ] **Injection**: Parameterized queries, input validation
   - [ ] **Broken Authentication**: Secure session management, MFA
   - [ ] **Sensitive Data Exposure**: Encryption, secure storage
   - [ ] **XML External Entities (XXE)**: Disable XML external entities
   - [ ] **Broken Access Control**: Authorization checks on every request
   - [ ] **Security Misconfiguration**: Secure defaults, minimal attack surface
   - [ ] **Cross-Site Scripting (XSS)**: Input sanitization, Content Security Policy
   - [ ] **Insecure Deserialization**: Validate serialized data
   - [ ] **Using Components with Known Vulnerabilities**: Dependency scanning
   - [ ] **Insufficient Logging & Monitoring**: Comprehensive logging, alerts

   **Additional Protections:**
   - **CSRF Protection**: CSRF tokens on state-changing requests
   - **Rate Limiting**: Prevent brute force and DDoS
   - **SQL Injection**: ORM usage, parameterized queries
   - **Clickjacking**: X-Frame-Options header
   - **CORS**: Proper CORS configuration

7. **Compliance Requirements:**
   - **GDPR** (if applicable): Data privacy, right to deletion, consent
   - **CCPA** (if applicable): California privacy rights
   - **HIPAA** (if healthcare): Healthcare data protection
   - **PCI-DSS** (if payment): Payment card data security
   - **SOC 2** (if enterprise): Security, availability, confidentiality

**theme-standards.md (CRITICAL - Based on Approved UX Mockup):**

**IMPORTANT**: This file MUST be extracted from the approved UX mockup in `05-ux/mockups/styles.css`. Do NOT invent theme values - extract them from the actual mockup.

Extract and document the UI theme from the UX mockup:

1. **Theme Extraction from Mockup:**

   **Step 1**: Read `05-ux/mockups/styles.css` file
   **Step 2**: Extract all CSS variables defined in `:root`
   **Step 3**: Document them here with exact values from mockup

2. **Color Palette:**
   - Extract all color variables from mockup CSS (primary, secondary, accent, neutral, semantic colors)
   - Document use cases for each color
   - Ensure color contrast requirements (WCAG AA minimum)
   - Define dark mode variations if applicable

3. **Typography:**
   - Extract font families and their use cases
   - Document font sizes scale (xs through 3xl)
   - Define font weights (regular, medium, semibold, bold)
   - Specify line heights for different content types
   - Define typography hierarchy and usage rules

4. **Spacing System:**
   - Extract spacing scale variables from mockup
   - Define spacing usage rules for components, margins, and gaps
   - Specify default spacing for common UI patterns

5. **Border Radius:**
   - Extract border radius values from mockup
   - Define usage rules for different component types

6. **Shadows:**
   - Extract shadow values from mockup
   - Define shadow usage for elevation and states

7. **Transitions:**
   - Extract transition timing values from mockup
   - Define transition usage for interactions and animations

8. **Component-Specific Styles:**
   - Extract button styles (variants, sizes, states)
   - Extract form styles (inputs, focus, error, success states)
   - Extract card styles (background, padding, shadow, border)
   - Document other component-specific styles as needed

9. **Responsive Breakpoints:**
   - Extract breakpoints from mockup media queries
   - Define responsive behavior rules for fonts, spacing, and layouts

10. **Design Tokens for Implementation:**
    - Provide design tokens in format suitable for the chosen tech stack
    - Ensure tokens match extracted CSS variable values
    - Include all theme values (colors, fonts, spacing, etc.)

**CRITICAL RULES for theme-standards.md:**
- ❌ **DO NOT** invent or guess theme values
- ✅ **DO** extract exact values from `05-ux/mockups/styles.css`
- ✅ **DO** preserve the exact CSS variable names
- ✅ **DO** include usage rules and accessibility notes
- ✅ **DO** provide code examples for implementation

**coding-standards.md:**

Define code style rules and conventions:

1. **General Principles:**
   - Write clean, readable code
   - Follow DRY (Don't Repeat Yourself)
   - KISS (Keep It Simple, Stupid)
   - SOLID principles (for OOP languages)
   - Functional programming principles (where applicable)

2. **Naming Conventions:**

   **Variables:**
   - camelCase for JavaScript/TypeScript: `userName`, `isActive`
   - snake_case for Python: `user_name`, `is_active`
   - PascalCase for classes/components: `UserProfile`, `LoginForm`

   **Functions/Methods:**
   - Verb-first naming: `getUserData()`, `validateEmail()`, `handleSubmit()`
   - Boolean functions: `isValid()`, `hasPermission()`, `canEdit()`

   **Constants:**
   - UPPER_SNAKE_CASE: `MAX_RETRIES`, `API_BASE_URL`

   **Files:**
   - Component files: PascalCase (`LoginForm.tsx`, `UserProfile.jsx`)
   - Utility files: camelCase (`formatDate.js`, `validateInput.ts`)
   - Keep filenames descriptive and consistent

3. **Code Organization:**
   - Define frontend folder structure (components, pages, hooks, utils, services, store, styles, types, constants)
   - Define backend folder structure (controllers, models, services, routes, middleware, utils, config, validators)
   - Ensure clear separation of concerns

4. **Code Style:**

   **Indentation:**
   - Use 2 spaces for JavaScript/TypeScript/HTML/CSS
   - Use 4 spaces for Python
   - No tabs

   **Line Length:**
   - Maximum 80-100 characters per line
   - Break long lines at logical points

   **Comments:**
   - Use comments to explain WHY, not WHAT
   - Write JSDoc/TSDoc for functions
   - Keep comments up-to-date

   **Formatting:**
   - Use Prettier for auto-formatting (JavaScript/TypeScript)
   - Use Black for Python formatting
   - Configure IDE to format on save

5. **Best Practices:**

   **Error Handling:**
   - Always handle errors, never ignore them
   - Use try-catch for async operations
   - Provide meaningful error messages
   - Log errors with context

   **Async Code:**
   - Use async/await over promises.then()
   - Handle promise rejections
   - Avoid callback hell

   **Security:**
   - Never commit secrets or credentials
   - Validate all user input
   - Sanitize data before rendering
   - Use prepared statements for SQL

   **Performance:**
   - Avoid unnecessary re-renders (React)
   - Use proper indexing (Database)
   - Cache when appropriate
   - Lazy load when possible

6. **Version Control:**

   **Commit Messages:**
   - Format: `<type>: <description>`
   - Types: feat, fix, docs, style, refactor, test, chore
   - Examples:
     - `feat: add user authentication`
     - `fix: resolve login redirect bug`
     - `docs: update API documentation`

   **Branch Naming:**
   - feature/[feature-name]
   - bugfix/[bug-description]
   - hotfix/[issue-number]

**source-code-structure.md**

Define the organization and structure of source code under the `src/` folder based on the project's needs. Follow `dev-swarm/docs/source-code-structure.md` as the baseline:

1. **Overview:**
   - Explain the chosen code organization approach
   - Purpose: Enable AI developers to navigate and locate code efficiently
   - Maintain clear separation of concerns
   - Support scalability and maintainability

2. **Choose Organization Strategy:**

   Select and document the code organization strategy that best fits the project. Use the options listed in `dev-swarm/docs/source-code-structure.md` for the sample structures.

3. **File Naming Conventions:**
   - Define file naming conventions (e.g., kebab-case, snake_case, PascalCase)
   - Test file conventions (e.g., `.test.*`, `.spec.*`, `test_*`)
   - Type definition conventions (e.g., `.types.*`, `types.*`)

4. **Code Organization Principles:**
   - **Single Responsibility**: One file, one primary purpose
   - **Clear Entry Points**: Well-defined public APIs
   - **Co-located Tests**: Tests live near the code they test
   - **Shared Code**: Define when code should be shared vs duplicated

5. **Documentation Requirements:**
   - Document where each type of code belongs
   - Provide examples for common scenarios
   - Explain the rationale for the chosen structure
   - Define guidelines for adding new code

**testing-standards.md:**

Define testing requirements and standards:

1. **Testing Principles:**
   - Write tests before or alongside code (TDD encouraged)
   - Test behavior, not implementation
   - Keep tests simple and readable
   - One assertion per test (when possible)
   - Tests should be independent and isolated

2. **Testing Pyramid:**
   ```
        /\
       /E2E\     <- Few (10%)
      /------\
     /Integration\ <- Some (30%)
    /------------\
   /  Unit Tests  \  <- Many (60%)
   ```

3. **Required Test Coverage:**
   - **Minimum Coverage**: 80% overall
   - **Critical Code**: 100% coverage (authentication, payment, data validation)
   - **Nice-to-have Code**: 60-70% coverage (UI components, utilities)

4. **Unit Testing:**

   **What to Test:**
   - All business logic functions
   - Data transformations and calculations
   - Validation functions
   - Utility functions
   - Component logic (React hooks, etc.)

   **Testing Framework:**
   - JavaScript/TypeScript: Jest, Vitest
   - Python: pytest, unittest
   - Go: testing package

   **Naming Convention:**
   - Test file: `fileName.test.js` or `fileName.spec.js`
   - Test description: `describe('functionName', () => { it('should do something', () => {}) })`
   - Follow testing framework conventions for structure

5. **Integration Testing:**

   **What to Test:**
   - API endpoints (request/response)
   - Database operations
   - External service integrations
   - Authentication flows

   **Testing Tools:**
   - API Testing: Supertest, Postman/Newman
   - Database: In-memory DB or test DB
   - Mocking: Mock external services

6. **End-to-End (E2E) Testing:**

   **What to Test:**
   - Critical user flows (from UX design)
   - User registration and login
   - Core feature workflows
   - Payment flows (if applicable)

   **Testing Framework:**
   - Playwright (recommended)
   - Cypress
   - Selenium

   **E2E Test Approach:**
   - Write tests that simulate real user interactions
   - Test complete workflows from start to finish
   - Verify expected outcomes and navigation

7. **Test Execution:**

   **Local Development:**
   - Run tests before committing: `npm test`
   - Run tests in watch mode: `npm test --watch`

   **CI/CD Pipeline:**
   - Run all tests on pull requests
   - Block merge if tests fail
   - Run tests before deployment

   **Test Commands:**
   - `npm test` - Run all tests
   - `npm test:unit` - Run unit tests only
   - `npm test:integration` - Run integration tests
   - `npm test:e2e` - Run E2E tests
   - `npm test:coverage` - Generate coverage report

8. **Minimum Test Gates:**

   **Before Merging to Main:**
   - [ ] All tests pass
   - [ ] Code coverage >= 80%
   - [ ] No critical bugs
   - [ ] E2E tests pass for critical flows

   **Before Deployment:**
   - [ ] All tests pass in production-like environment
   - [ ] E2E tests pass on staging
   - [ ] Performance tests pass (if applicable)
   - [ ] Security tests pass

**security-standards.md:**

Define secure coding rules and practices:

1. **Secure Coding Principles:**
   - Validate all input, trust no one
   - Fail securely (default deny)
   - Defense in depth
   - Principle of least privilege
   - Keep security simple
   - Don't rely on security through obscurity

2. **Input Validation:**
   - Validate on server-side (never trust client)
   - Use allowlists, not denylists
   - Sanitize all user input
   - Validate data types, lengths, formats
   - Reject invalid input, don't try to fix it

3. **Output Encoding:**
   - Encode output based on context (HTML, URL, JavaScript)
   - Use framework-provided encoding functions
   - Prevent XSS by encoding user data

4. **Authentication & Authorization:**
   - Never store passwords in plain text
   - Use bcrypt/Argon2 for password hashing
   - Implement rate limiting on auth endpoints
   - Require re-authentication for sensitive operations
   - Check authorization on EVERY request

5. **Sensitive Data Handling:**
   - Never log sensitive data (passwords, tokens, SSNs, credit cards)
   - Encrypt sensitive data at rest
   - Use HTTPS for all data in transit
   - Redact sensitive data in logs and errors

6. **Secret Management:**
   - Never commit secrets to version control
   - Use environment variables or secret management services
   - Rotate secrets regularly
   - Use different secrets for dev/staging/production

7. **Dependency Security:**
   - Regularly scan dependencies for vulnerabilities
   - Keep dependencies up-to-date
   - Use `npm audit` or `pip-audit` in CI/CD
   - Remove unused dependencies

8. **SQL Injection Prevention:**
   - Use parameterized queries (prepared statements)
   - Use ORM frameworks properly
   - Never concatenate SQL strings with user input
   - Validate input before queries

9. **Logging & Monitoring:**
   - Log all authentication events (success and failure)
   - Log all authorization failures
   - Log all data access to sensitive resources
   - Set up alerts for suspicious activity
   - **Redaction Rules**:
     - Redact passwords, tokens, API keys
     - Redact credit card numbers, SSNs
     - Redact PII in production logs

10. **Security Checklist for Code Review:**
    - [ ] All user input is validated
    - [ ] SQL uses parameterized queries
    - [ ] No secrets in code
    - [ ] Sensitive data is not logged
    - [ ] Authorization is checked
    - [ ] HTTPS is enforced
    - [ ] CSRF protection is enabled
    - [ ] XSS prevention is in place
    - [ ] Error messages don't leak information

### Step 4: Ensure Alignment

Make sure tech specs align with:
- Architecture from 06-architecture/
- Non-functional requirements from 04-prd/non-functional-requirements.md
- **UX mockup theme** from 05-ux/mockups/styles.css (CRITICAL for theme-standards.md)
- MVP scope from 03-mvp/ (prioritize tech choices for MVP)

Verify that:
- Tech stack can implement the architecture
- Theme standards match the UX mockup exactly
- Security standards address requirements
- Testing standards ensure quality
- Coding standards are clear and enforceable

### Step 5: Final User Review

1. **Inform user that tech specs are complete**
2. **Update README.md:**
   - Change **Status** from "In Progress" to "Completed"
   - Add a **Summary** section with key insights (2-3 paragraphs)
   - Add a **Created Files** section listing all created files

3. **Present completed work to user:**
   - Review chosen tech stack and rationale
   - Show theme standards extracted from UX mockup
   - Explain security approach
   - Walk through coding and testing standards

4. **Highlight key insights:**
   - Frontend framework choice and why
   - Backend framework choice and why
   - Database choice and why
   - **Theme values extracted from mockup** (show side-by-side)
   - Security compliance level
   - Test coverage requirements

5. **Ask questions:**
   - Comfortable with tech stack choices?
   - Theme standards match their vision?
   - Any security concerns?
   - Testing requirements achievable?
   - Ready to proceed to next stage (DevOps)?

6. Make adjustments based on user feedback if needed

### Step 6: Commit to Git (if user confirms)

1. **If user confirms tech specs are complete:**
   - Ask if they want to commit to git
2. **If user wants to commit:**
   - Stage all changes in `07-tech-specs/`
   - Commit with message: "Define tech stack and engineering standards (Stage 7)"

## Expected Project Structure

### For L3+ Projects (Full Scale)
```
project-root/
├── 00-init-ideas/
│   └── [existing files]
├── 01-market-research/ (optional)
│   └── [existing files if present]
├── 02-personas/
│   └── [existing files]
├── 03-mvp/
│   └── [existing files]
├── 04-prd/
│   └── [existing files]
├── 05-ux/
│   └── [existing files including mockups/]
├── 06-architecture/
│   └── [existing files]
└── 07-tech-specs/
    ├── README.md (with owners and summary)
    ├── tech-stack.md (languages, frameworks, tools)
    ├── security.md (auth, secrets, threats)
    ├── theme-standards.md (EXTRACTED FROM UX MOCKUP)
    ├── coding-standards.md (style, naming, organization)
    ├── source-code-structure.md (src/ organization)
    ├── testing-standards.md (coverage, frameworks, gates)
    └── security-standards.md (secure coding, logging redaction)
```

### For L2 Projects (Tools)
```
project-root/
├── 00-init-ideas/
│   └── [existing files]
└── 07-tech-specs/
    ├── README.md (minimal requirements)
    └── tech-stack.md (language and package selection)
```

## Key Tech Specs Principles

1. **Justify Choices**: Explain WHY each technology was chosen
2. **Extract Theme from Mockup**: Never invent theme values, extract from approved UX
3. **Standards Over Preferences**: Define standards that everyone follows
4. **Security First**: Build security into standards, not bolt on later
5. **Testable**: Make standards measurable and enforceable
6. **Practical**: Standards should be realistic and achievable
7. **Documented**: Clear documentation prevents confusion

## Theme Standards Best Practices (CRITICAL)

1. **Always Extract from Mockup**: Read `05-ux/mockups/` first
2. **Preserve Variable Names**: Keep CSS variable names from mockup
3. **Document Usage**: Explain when and how to use each value
4. **Accessibility Check**: Ensure color contrast meets WCAG standards
5. **Provide Examples**: Show code examples for implementation
6. **Design Tokens**: Convert CSS variables to JS/TS tokens for frameworks
7. **Keep In Sync**: If mockup changes, update theme standards immediately

## Deliverables

By the end of this stage, you should have:
- Complete tech stack definition with rationale for each choice
- Security posture and authentication approach defined
- **Theme standards extracted from approved UX mockup** (colors, fonts, spacing)
- Coding standards for style, naming, and organization
- **Source code structure for feature-driven development** (src/ organization)
- Testing standards with coverage requirements and frameworks
- Security standards for secure coding and logging
- Foundation for DevOps setup (next stage)
- Clear implementation guidelines for developers
