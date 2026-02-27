---
name: codebase-analyzer
description: Understand existing codebase patterns and context to inform feature judgement.
---

# Codebase Analyzer

Understand existing codebase patterns and context to inform feature judgement.

## Purpose

Before judges evaluate a feature proposal, they need to understand:
- What similar features already exist?
- What's the tech stack and architecture?
- What patterns and conventions are used?
- What files/code would be affected?

This skill provides that context through hybrid search strategy (fast → deep as needed).

## Input

```typescript
interface AnalyzerInput {
  parsedProposal: ParsedProposal;  // From feature-parser
  workingDirectory: string;  // Current project root
}
```

Key data used from ParsedProposal:
- `keywords`: For searching similar features
- `constraints`: Tech stack hints (e.g., "Laravel 12.x")
- `requirements.mustHave`: Integration points to search for

## Strategy: Hybrid Search

### Phase 1: Targeted Fast Search (Always runs)

Quick reconnaissance using keywords:

**1. Identify Tech Stack:**
```markdown
Search for config/manifest files:
- package.json → Node.js, npm/yarn, frameworks (React, Vue, Express)
- composer.json → PHP, Composer, Laravel/Symfony
- requirements.txt / Pipfile → Python, Django/Flask
- go.mod → Go
- Cargo.toml → Rust
- pom.xml / build.gradle → Java
- Gemfile → Ruby, Rails

Use Glob tool:
- Glob pattern: "**/package.json"
- Glob pattern: "**/composer.json"
- Glob pattern: "**/requirements.txt"
- Glob pattern: "**/go.mod"

Read first match to extract:
- Language & version
- Framework & version
- Key dependencies
```

**2. Find Similar Features:**
```markdown
Use keywords from proposal to search:

Grep for keywords in code:
- Pattern: keywords joined with OR regex
- Example: "notification|notify|alert" if proposal is about notifications
- Output mode: "files_with_matches"
- File types: source code only (js, ts, py, php, go, rs, java, rb)

Example searches based on feature type:
- Notifications: grep "notification|notify|alert|email.*send"
- Authentication: grep "auth|login|session|token|jwt"
- API: grep "api|endpoint|route|controller"
- Database: grep "model|schema|migration|query"

Glob for related file patterns:
- Notifications: "**/*{notif,alert,email}*.{ts,js,py,php}"
- Auth: "**/*{auth,login,session,token}*.{ts,js,py,php}"
- Admin: "**/*{admin,dashboard}*.{ts,js,py,php}"
```

**3. Detect Architecture Patterns:**
```markdown
Common patterns to look for:

**MVC Pattern:**
- Glob: "**/models/**", "**/views/**", "**/controllers/**"
- If found → Architecture: "MVC"

**Service Layer:**
- Glob: "**/services/**", "**/lib/services/**"
- If found → Pattern: "Service Layer"

**Repository Pattern:**
- Glob: "**/repositories/**", "**/repo/**"
- If found → Pattern: "Repository Pattern"

**Clean Architecture:**
- Glob: "**/domain/**", "**/application/**", "**/infrastructure/**"
- If found → Architecture: "Clean Architecture" or "Layered"

**Monorepo:**
- Glob: "**/packages/**" OR "**/apps/**"
- If found → Structure: "Monorepo"

**Framework-specific:**
- Laravel: app/Models, app/Http/Controllers, routes/
- Django: models.py, views.py, urls.py
- Express: routes/, controllers/, middleware/
- Next.js: pages/, app/, components/
```

**4. Identify Testing Strategy:**
```markdown
Glob for test files:
- "**/*.test.{js,ts,jsx,tsx}"
- "**/*.spec.{js,ts,jsx,tsx}"
- "**/test/**"
- "**/tests/**"
- "**/__tests__/**"

Read package.json or composer.json to find test frameworks:
- Jest, Mocha, Vitest, Jasmine → JavaScript
- PHPUnit, Pest → PHP
- pytest, unittest → Python
- Go test → Go
```

**5. Find Integration Points:**
```markdown
Based on requirements, search for existing integrations:

If proposal mentions "User model" or "auth":
- Grep: "User.*model|user.*entity|authentication"
- Glob: "**/User.{ts,js,py,php,go}"
- Glob: "**/models/User.*"

If proposal mentions "database":
- Glob: "**/migrations/**"
- Glob: "**/schema/**"
- Read database config files

If proposal mentions specific services (email, payment, etc):
- Grep: "email.*service|mail.*send"
- Grep: "payment|stripe|paypal"
```

**Phase 1 Duration:** 30-60 seconds

### Phase 2: Deep Analysis (Conditional)

**Trigger Deep Analysis if:**
- Phase 1 found < 3 relevant files
- Proposal completeness score < 40 (vague proposal)
- High complexity detected (mentions "migration", "breaking change", "refactor")

**Deep Analysis using Task Agent:**
```markdown
Spawn Task agent with subagent_type=Explore

Prompt for agent:
"Analyze codebase architecture and patterns for implementing [feature title].

Context:
- Proposal: [brief summary of goals and requirements]
- Keywords: [keywords list]
- Initial findings: [Phase 1 results summary]

Focus on:
1. Overall architecture style (MVC, Clean, Microservices, etc)
2. Existing similar features or modules
3. Code organization conventions
4. Testing patterns
5. Files/modules that would be affected by [feature]

Provide comprehensive understanding of how [feature] would fit into current codebase."

Thoroughness level: "medium"
```

**Phase 2 Duration:** 1-2 minutes (if needed)

## Output Format

```typescript
interface CodebaseContext {
  // Similar existing features
  similarFeatures: Array<{
    path: string;
    purpose: string;  // What this file/feature does
    relevance: "high" | "medium" | "low";
  }>;

  // Technology stack
  techStack: {
    languages: string[];  // ["TypeScript", "JavaScript"]
    frameworks: string[];  // ["Laravel 12.2", "Vue 3.4"]
    libraries: string[];  // ["axios", "lodash", "moment"]
    database?: string;  // "PostgreSQL", "MySQL", "MongoDB"
    testing?: string;  // "Jest", "PHPUnit", "pytest"
  };

  // Architecture & patterns
  patterns: {
    architecture: string;  // "MVC", "Clean Architecture", "Layered"
    structure: string;  // "Monorepo", "Monolith", "Modular"
    serviceLayer: boolean;  // Uses service layer pattern?
    repository: boolean;  // Uses repository pattern?
    testingStrategy: string;  // "Unit + Integration", "E2E", "TDD"
    conventions: string[];  // ["TypeScript strict mode", "ESLint", "PSR-12"]
  };

  // Files that might need changes
  potentialConflicts: Array<{
    path: string;
    reason: string;  // Why this file might conflict/need changes
  }>;

  // Integration points
  integrationPoints: Array<{
    type: string;  // "auth", "database", "queue", "email"
    location: string;  // File path or description
    details: string;  // How to integrate
  }>;

  // Analysis metadata
  confidence: "High" | "Medium" | "Low";
  searchPhase: "Targeted" | "Deep";  // Which phase provided this data
  analysisTime: number;  // Seconds taken
}
```

## Example Output

**Scenario:** Laravel app, notification feature proposal

**Input:**
```typescript
{
  parsedProposal: {
    title: "Real-time User Notifications",
    keywords: ["notification", "user", "email", "real-time", "laravel", "database", "queue"]
  },
  workingDirectory: "/home/user/my-laravel-app"
}
```

**Phase 1 Search Results:**
```
Tech Stack Detection:
- Found: composer.json → Laravel 12.2, PHP 8.3
- Found: package.json → Vue 3.4, Vite

Similar Features:
- app/Notifications/WelcomeEmail.php (high relevance)
- app/Services/EmailService.php (medium relevance)
- app/Models/User.php (high relevance)
- config/mail.php (medium relevance)
- config/queue.php (high relevance)

Architecture:
- Found: app/Models/ → MVC pattern
- Found: app/Services/ → Service layer
- Found: tests/Feature/ → Feature tests with Pest

Integration Points:
- User model: app/Models/User.php
- Mail config: config/mail.php (SMTP via Mailgun)
- Queue: config/queue.php (Redis driver)
```

**Output:**
```json
{
  "similarFeatures": [
    {
      "path": "app/Notifications/WelcomeEmail.php",
      "purpose": "Existing notification example - sends welcome email to new users",
      "relevance": "high"
    },
    {
      "path": "app/Services/EmailService.php",
      "purpose": "Email sending service - handles SMTP delivery",
      "relevance": "medium"
    },
    {
      "path": "app/Models/User.php",
      "purpose": "User model - target for notifications",
      "relevance": "high"
    }
  ],

  "techStack": {
    "languages": ["PHP 8.3", "JavaScript"],
    "frameworks": ["Laravel 12.2", "Vue 3.4"],
    "libraries": ["Mailgun", "Redis", "Vite"],
    "database": "MySQL",
    "testing": "Pest (PHPUnit)"
  },

  "patterns": {
    "architecture": "MVC with Service Layer",
    "structure": "Monolith",
    "serviceLayer": true,
    "repository": false,
    "testingStrategy": "Feature tests + Unit tests",
    "conventions": [
      "Laravel coding standards",
      "Service layer for business logic",
      "Pest for testing",
      "Vue 3 Composition API for frontend"
    ]
  },

  "potentialConflicts": [
    {
      "path": "app/Models/User.php",
      "reason": "May need Notifiable trait (likely already has it)"
    },
    {
      "path": "config/queue.php",
      "reason": "Queue configuration for async notification delivery"
    }
  ],

  "integrationPoints": [
    {
      "type": "auth",
      "location": "app/Models/User.php",
      "details": "User model should use Notifiable trait for Laravel notifications"
    },
    {
      "type": "queue",
      "location": "config/queue.php",
      "details": "Redis queue configured and ready for async jobs"
    },
    {
      "type": "email",
      "location": "config/mail.php",
      "details": "Mailgun SMTP configured for email delivery"
    }
  ],

  "confidence": "High",
  "searchPhase": "Targeted",
  "analysisTime": 45
}
```

## Usage in Judgement Flow

Judges receive this context to make informed decisions:

**Pragmatist uses:**
- `similarFeatures` → Can we reuse existing code?
- `integrationPoints` → Integration effort estimate
- `techStack.testing` → Is testing setup ready?

**Purist uses:**
- `patterns.architecture` → Does proposal fit current architecture?
- `patterns.conventions` → Will this follow existing conventions?
- `serviceLayer/repository` → Proper separation of concerns?

**Innovator uses:**
- `techStack.libraries` → Are we using latest/best libraries?
- `patterns` → Opportunity to improve architecture?

**Skeptic uses:**
- `potentialConflicts` → What could break?
- `integrationPoints` → Security implications of integrations?

**Optimizer uses:**
- `techStack.database` → Database optimization opportunities?
- `similarFeatures` → Performance patterns to follow/avoid?

## Edge Cases

**Empty/New Project:**
```json
{
  "similarFeatures": [],
  "techStack": {
    "languages": [],
    "frameworks": [],
    "libraries": []
  },
  "patterns": {
    "architecture": "Unknown",
    "structure": "Unknown",
    "serviceLayer": false,
    "repository": false,
    "testingStrategy": "None detected",
    "conventions": []
  },
  "potentialConflicts": [],
  "integrationPoints": [],
  "confidence": "Low",
  "searchPhase": "Targeted",
  "analysisTime": 15
}
```

Judges should note: "New/empty codebase - no existing patterns to follow"

**Large Monorepo (1000+ files):**
- Limit search depth to relevant packages/apps
- Use glob patterns to narrow scope
- May need Phase 2 deep analysis
- Increase timeout for searches

**Multiple Frameworks Detected:**
```json
{
  "techStack": {
    "frameworks": ["Laravel 12.2 (backend)", "Next.js 14 (frontend)", "React Native (mobile)"]
  }
}
```

Judges should consider: Which part of the stack does feature target?

## Performance Optimization

**Caching:**
- Cache tech stack detection (rarely changes)
- Cache file structure for 5 minutes
- Don't re-scan if same proposal keywords

**Timeouts:**
- Phase 1: 60 second timeout
- Phase 2: 120 second timeout
- If timeout: return partial results with confidence: "Low"

**Parallel Searches:**
- Run Glob and Grep in parallel when possible
- Tech stack detection parallel to feature search

## Error Handling

**No package/config files found:**
- Continue with generic analysis
- Set confidence: "Low"
- Recommend manual tech stack specification

**Grep/Glob errors:**
- Log error but continue
- Try alternative search patterns
- Fallback to Phase 2 if available

**Permission denied errors:**
- Skip inaccessible directories
- Note in output: "Some directories inaccessible"
- Reduce confidence accordingly

**Phase 2 agent fails:**
- Fall back to Phase 1 results
- Set confidence: "Medium" (downgrade from High)
- Note: "Deep analysis unavailable"

## Success Criteria

**Excellent Analysis (confidence: High):**
- Found 5+ similar features
- Tech stack fully identified
- Architecture pattern clear
- Integration points identified

**Good Analysis (confidence: Medium):**
- Found 2-4 similar features
- Tech stack partially identified
- Some architecture clues
- Basic integration points

**Poor Analysis (confidence: Low):**
- Found 0-1 similar features
- Tech stack unknown
- Architecture unclear
- No integration points

Judges will weight their feedback based on confidence level.
