---
name: faf-docs
description: Access FAF documentation, guides, and resources. Answers questions about The Reading Order, IANA registration, Podium scoring, format specification, and best practices. Use when user asks "how does FAF work", "show me docs", "explain The Reading Order", or needs reference information.
allowed-tools: WebFetch, Read
---

# FAF Docs - Documentation & Resources

## Purpose

Provide quick access to FAF documentation, guides, specifications, and learning resources. Answers common questions and explains FAF concepts without requiring users to leave their workflow.

**The Goal:** Instant answers. Zero context switching. Championship-grade documentation.

## When to Use

This skill activates automatically when the user:
- Asks "How does FAF work?"
- Says "Show me the docs"
- Says "Explain The Reading Order"
- Asks "What is IANA registration?"
- Says "How do I use Podium scoring?"
- Asks "Where are the FAF docs?"
- Needs format specification reference
- Wants to learn best practices
- Asks "What's the difference between project.faf and CLAUDE.md?"

**Trigger Words:** docs, documentation, guide, explain, how does, what is, show me, reference, spec, help

## How It Works

### Step 1: Identify Question Type

Determine what the user needs:

**Conceptual:**
- "What is FAF?" → Use **faf-teacher** skill for comprehensive overview
- "What is The Reading Order?" → Explain reading sequence
- "What is Podium scoring?" → Explain 0-100% AI-readiness system

**Reference:**
- "Show format spec" → Provide YAML structure reference
- "List all commands" → Show faf-cli command reference
- "Show IANA details" → Explain registration

**Practical:**
- "How do I improve my score?" → Use **faf-enhance** skill
- "How do I sync files?" → Use **faf-sync** skill
- "How do I validate?" → Use **faf-validate** skill

### Step 2: Provide Answer

**For conceptual questions:**
- Explain concept clearly
- Provide examples
- Link to related skills
- Suggest next steps

**For reference questions:**
- Show relevant specification
- Provide code examples
- Link to full documentation

**For practical questions:**
- Activate appropriate skill
- Guide through process
- Show command examples

### Step 3: Offer Additional Resources

After answering, provide:
- Related documentation links
- Relevant skills to use
- Next learning steps
- Official resources

## Key Concepts

### The Reading Order

**What it is:**
Optimal sequence for AI to read project files for complete context understanding.

**The Order:**
```
1. project.faf     → Project DNA (architecture, purpose, stack)
2. CLAUDE.md       → Workflow instructions (how to work here)
3. README.md       → User documentation (how humans use it)
4. package.json    → Dependencies (what it needs)
5. Config files    → Build settings (how to compile)
6. Code            → Implementation (what it does)
```

**Why this order:**
- Architecture BEFORE implementation
- Context BEFORE code
- Complete picture BEFORE details
- Persistent BEFORE temporary

**Analogy:**
Like reading blueprints before examining bricks. You understand WHAT you're building before diving into HOW it's built.

### IANA Registration

**What it is:**
FAF is officially registered with the Internet Assigned Numbers Authority as a recognized media type.

**Details:**
- **Media Type:** `application/vnd.faf+yaml`
- **Registration Date:** October 31, 2025
- **Authority:** IANA (same authority that registered PDF, JSON, XML)
- **Status:** Official Internet standard

**Why it matters:**
- Universal recognition (browsers, email clients, APIs)
- Format authority (not a tool, foundational infrastructure)
- Same level as PDF, JSON, XML
- Professional legitimacy

**How to verify:**
```yaml
# In your project.faf
metadata:
  iana_media_type: application/vnd.faf+yaml
```

### Podium Scoring

**What it is:**
AI-readiness measurement system (0-100%) showing how well AI can understand your project.

**Tiers:**
- 🏆 **Trophy (85-100%)** - Elite AI-readiness
- 🥇 **Gold (70-84%)** - Excellent AI-readiness
- 🥈 **Silver (55-69%)** - Good AI-readiness
- 🥉 **Bronze (40-54%)** - Basic AI-readiness
- 🟢 **Green (35-39%)** - Minimal AI-readiness
- 🟡 **Yellow (20-34%)** - Very limited AI-readiness
- 🔴 **Red (10-19%)** - Critical gaps
- 🤍 **White (<10%)** - No effective AI-readiness

**What gets scored:**
- Project purpose clarity
- Architecture documentation
- Stack information completeness
- Testing approach detail
- Dependencies specification
- Format compliance

**How to check:**
```bash
faf score
```

**How to improve:**
```bash
faf enhance
```

### Context-Mirroring

**What it is:**
Bidirectional synchronization between project.faf and CLAUDE.md in <10ms (achieved: 8ms).

**How it works:**
- Changes in project.faf → auto-sync to CLAUDE.md
- Changes in CLAUDE.md → auto-sync to project.faf
- Architecture stays consistent
- Workflow stays separate
- Zero manual work

**When to use:**
```bash
# After editing either file
faf bi-sync
```

**Performance:**
- Target: <10ms
- Achieved: 8ms average
- F1-grade engineering
- Championship performance

## Common Questions

### "What's the difference between project.faf and CLAUDE.md?"

**project.faf (Project DNA):**
- **Purpose:** Architecture and project structure
- **Audience:** AI understanding "WHAT it is"
- **Changes:** Rarely (only when architecture changes)
- **Format:** YAML (machine-readable, human-friendly)
- **Contains:** Stack, framework, dependencies, testing, architecture

**CLAUDE.md (Workflow Instructions):**
- **Purpose:** How to work in this codebase
- **Audience:** AI understanding "HOW to work here"
- **Changes:** Frequently (as workflow evolves)
- **Format:** Markdown (human-readable)
- **Contains:** Git protocol, code style, development commands, team conventions

**Together:** Complete AI context (architecture + workflow)

### "Do I need both files?"

**Minimum:**
- project.faf alone = Good (architecture context)

**Recommended:**
- project.faf + CLAUDE.md = Excellent (architecture + workflow)

**Best:**
- project.faf + CLAUDE.md + bi-sync = Elite (perfect sync)

### "How often do I update project.faf?"

**Update when:**
- ✅ Major framework upgrade (Next.js 13 → 14)
- ✅ Runtime version change (Node 18 → 20)
- ✅ Architecture shift (REST → GraphQL)
- ✅ Testing framework change (Jest → Vitest)
- ✅ New major dependency added

**Don't update for:**
- ❌ New features (code shows that)
- ❌ Bug fixes (code shows that)
- ❌ Dependency version bumps (package.json shows that)

**Think:** "Would a new developer need to know this?" If yes, update project.faf.

### "Can I use FAF with [other AI tool]?"

**Yes. FAF works with ALL AI tools:**
- ✅ Claude Code (native support)
- ✅ Claude Desktop
- ✅ Cursor
- ✅ Gemini CLI
- ✅ OpenAI Codex CLI
- ✅ Windsurf
- ✅ Warp
- ✅ ANY AI tool (it's just text)

**Why:** FAF is a text file in YAML format. Any AI can read YAML. No special integration required. Universal by design.

### "What if my team doesn't use FAF?"

**You can still benefit personally:**

1. Create project.faf for YOUR understanding
2. Drop it into YOUR AI conversations
3. Save 10-30 min per session
4. Share with team when they see the value

**Team adoption is automatic:**
- Commit project.faf to repository
- Team members' AI tools read it automatically
- Everyone saves time
- No coordination required

## Format Specification Reference

### Minimal Valid Format

```yaml
name: project-name
purpose: What this project does
version: 1.0.0

metadata:
  format_version: 3.0.0
  iana_media_type: application/vnd.faf+yaml

architecture:
  type: web-app | library | api | cli
  language: TypeScript | Python | etc.
```

### Comprehensive Format

```yaml
name: project-name
purpose: Detailed description of what this project does and why it exists
version: 1.0.0

metadata:
  format_version: 3.0.0
  iana_media_type: application/vnd.faf+yaml
  created: 2025-11-02
  last_updated: 2025-11-02

architecture:
  type: web-app
  language: TypeScript
  framework: Next.js 14
  pattern: Server-side rendering with App Router
  description: |
    Comprehensive architecture description explaining
    the structure, patterns, and design decisions.

stack:
  runtime: Node.js 18+
  framework: Next.js 14 (App Router)
  dependencies:
    - react: "^18.2.0"
    - typescript: "^5.3.3"
  styling: Tailwind CSS
  database: PostgreSQL 15 (Prisma ORM)

testing:
  framework: Jest + React Testing Library
  approach: |
    - Unit tests for utilities
    - Component tests for UI
    - Integration tests for API routes
  coverage: 78% overall, 90%+ critical paths
  commands:
    test: npm test
    coverage: npm run test:coverage

build:
  tool: Next.js (Turbopack)
  commands:
    dev: npm run dev
    build: npm run build
    start: npm start

deployment:
  platform: Vercel
  environment: Node.js 18
  env_variables:
    - DATABASE_URL
    - JWT_SECRET

architecture_decisions:
  - decision: "Chose App Router over Pages Router"
    rationale: "Better performance, simpler data fetching"
  - decision: "TypeScript strict mode"
    rationale: "Catch errors at compile time"

performance:
  - React Server Components for data fetching
  - Image optimization with next/image
  - Code splitting by route (automatic)
```

## Command Reference

### faf-cli Commands

**Essential:**
```bash
faf init        # Initialize project.faf
faf score       # Check AI-readiness (0-100%)
faf enhance     # Improve project.faf
faf validate    # Check format compliance
faf bi-sync     # Sync project.faf ↔ CLAUDE.md
faf migrate     # Upgrade to latest format
```

**Discovery:**
```bash
faf auto        # Auto-detect and generate everything
faf doctor      # Health check for project
faf verify      # Verify installation
```

**Analysis:**
```bash
faf formats     # Show detected formats
faf stacks      # Show detected tech stacks
faf vibe        # AI-powered project analysis
```

**Utility:**
```bash
faf show        # Display current project.faf
faf version     # Show faf-cli version
faf help        # Show help
```

## Official Resources

**Website:**
- https://faf.one - Main website
- https://faf.one/docs - Full documentation

**GitHub:**
- faf-cli: https://github.com/Wolfe-Jam/faf-cli
- claude-faf-mcp: https://github.com/Wolfe-Jam/claude-faf-mcp

**npm:**
- faf-cli: https://npmjs.com/package/faf-cli
- claude-faf-mcp: https://npmjs.com/package/claude-faf-mcp

**Homebrew:**
```bash
brew install wolfe-jam/faf/faf-cli
```

## Related Skills

For specific tasks, use:
- **faf-teacher** - Comprehensive FAF overview
- **faf-init** - Create project.faf
- **faf-score** - Measure AI-readiness
- **faf-enhance** - Improve quality
- **faf-sync** - Bidirectional sync
- **faf-validate** - Format compliance
- **faf-migrate** - Version upgrade

## Examples

### Example 1: "What is The Reading Order?"

**Answer:**

"The Reading Order is the optimal sequence for AI to read your project files:

```
1. project.faf   → Project DNA (WHAT it is)
2. CLAUDE.md     → Workflow (HOW to work)
3. README.md     → Docs (HOW humans use it)
4. package.json  → Dependencies
5. Config files  → Build settings
6. Code          → Implementation
```

**Why this order matters:**
- Architecture BEFORE code = Better understanding
- Context BEFORE details = Fewer questions
- Complete picture FIRST = Correct suggestions

**Analogy:** Reading blueprints before examining bricks. You understand the building plan before diving into construction details.

**Result:** AI has complete context in <1 second vs 10-30 minutes of questions."

### Example 2: "Show me the format spec"

**Answer:**

"Here's the IANA-registered format specification:

**Minimal valid project.faf:**
```yaml
name: project-name
purpose: What it does
version: 1.0.0

metadata:
  format_version: 3.0.0
  iana_media_type: application/vnd.faf+yaml

architecture:
  type: web-app
  language: TypeScript
```

**Create yours:**
```bash
faf init  # Auto-generates valid format
```

**Full spec:** https://faf.one/docs/format"

### Example 3: "How do I improve my score?"

**Answer:**

"Use the **faf-enhance** skill for guided improvement:

```bash
faf enhance
```

This will:
1. Analyze your current project.faf
2. Identify missing or weak sections
3. Ask questions to gather information
4. Add improvements incrementally
5. Re-score after each change
6. Guide you to your target tier

**Typical progression:**
- Start: 58% (Silver)
- +6% add testing docs → 64%
- +4% add API docs → 68%
- +2% add architecture → 70% (Gold) ✨

Want me to activate the faf-enhance skill now?"

## Key Principles

**Documentation as Code:**
- Docs live with code
- Examples are runnable
- Specifications are tested
- Truth in one place

**NO BS ZONE:**
- Only verified claims
- Real examples
- Honest limitations
- Professional tone

**Format-Driven:**
- IANA-registered standard
- Not a tool, infrastructure
- Universal compatibility
- Long-term stability

## Success Metrics

When this skill succeeds, users should:
1. Understand FAF concepts clearly
2. Know where to find detailed docs
3. Have answers to their specific questions
4. Know which skill to use for their task
5. Feel confident using FAF
6. Trust the documentation

---

**Generated by FAF Skill: faf-docs v1.0.0**
**Documentation Edition: Instant Answers**
**"The Reading Order. IANA registration. Championship docs."**
