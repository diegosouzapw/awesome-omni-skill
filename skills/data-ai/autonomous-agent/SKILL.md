---
name: autonomous-agent
description: "Autonomous coding agent that breaks features into small user stories and implements them iteratively with fresh context per iteration. Use when asked to: build a feature autonomously, create a PRD, implement a feature from scratch, run an autonomous coding loop, break down a feature into user stories. Triggers on: autonomous agent, build this autonomously, autonomous mode, implement this feature, create prd, prd to json, user stories, iterative implementation, ralph."
user-invocable: true
context: fork
---

# Autonomous Coding Agent

An autonomous workflow that breaks features into small, testable user stories and implements them one at a time with fresh context per iteration.

## Core Architecture

**Memory persists across iterations via:**

- `prd.json` - Task list with completion status
- `progress.md` - Learnings and implementation notes
- `AGENTS.md` - Long-term patterns for the repository
- Git history - All code changes
- **Memory MCP** - Cross-codebase learnings (patterns, mistakes, preferences)

**Each iteration is stateless** - read these files to understand context.

---

## Memory Integration (Cross-Codebase Learning)

The agent uses the Memory MCP server to learn across different projects. This enables:

- Remembering patterns that work well
- Avoiding mistakes made in other codebases
- Applying user preferences consistently

### Memory Entity Types

| Type                    | Purpose                      | Example                              |
| ----------------------- | ---------------------------- | ------------------------------------ |
| `pattern`               | Reusable solutions           | `pattern:early-returns`              |
| `mistake`               | Things to avoid              | `mistake:env-in-repo`                |
| `preference`            | User's preferred approaches  | `preference:package-manager`         |
| `tech-insight`          | Framework-specific knowledge | `tech-insight:supabase-rls`          |
| `architecture-decision` | High-level design choices    | `architecture-decision:multi-tenant` |

### When to Query Memory

1. **Phase 1 Start** - Query preferences and patterns before asking clarifying questions
2. **Phase 3 Start** - Load relevant tech-insights for the detected stack
3. **Before Implementation** - Check for related mistakes/patterns

### When to Save to Memory

1. **After successful story** - Extract reusable patterns
2. **After fixing a bug** - Save as mistake to avoid
3. **When discovering codebase convention** - Save if broadly applicable

---

## Entry Point Detection

When this skill activates, determine which phase to enter:

| Condition                              | Action                                     |
| -------------------------------------- | ------------------------------------------ |
| No `prd.json` exists                   | Start Phase 1 (PRD Generation)             |
| `prd.json` exists but no markdown PRD  | Start Phase 2 (JSON Conversion)            |
| `prd.json` exists with pending stories | Start Phase 3 (Autonomous Loop)            |
| All stories `passes: true`             | Report completion, ask if more work needed |

**First Action:** Check for existing files:

```bash
ls -la prd.json progress.md tasks/*.md 2>/dev/null
```

---

## Phase 1: PRD Generation

**Goal:** Create a Product Requirements Document from a feature idea.

### Step 1.0: Load User Preferences from Memory

**First action:** Query the Memory MCP for user preferences and patterns:

```
mcp__memory__search_nodes({ query: "preference" })
mcp__memory__search_nodes({ query: "pattern" })
mcp__memory__search_nodes({ query: "architecture-decision" })
```

**Apply learned preferences:**

- Package manager preference (pnpm vs npm vs yarn)
- Deployment targets (Railway, Vercel, Cloudflare)
- Code organization patterns (feature folders, etc.)
- Testing preferences

These preferences inform your clarifying questions and PRD structure.

### Step 1.1: Codebase Discovery

Before asking questions, understand the existing codebase:

```
1. Detect stack: package.json, requirements.txt, go.mod, etc.
2. Find existing patterns: src/ structure, component patterns, API conventions
3. Check for AGENTS.md for documented patterns
4. Identify test patterns and frameworks
5. Cross-reference with memory: tech-insights for detected stack
```

**Query stack-specific learnings:**

```
# If Next.js detected:
mcp__memory__search_nodes({ query: "nextjs" })

# If Supabase detected:
mcp__memory__search_nodes({ query: "supabase" })

# General mistakes to avoid:
mcp__memory__search_nodes({ query: "mistake" })
```

### Step 1.2: Clarifying Questions

Ask 3-5 essential questions with lettered options:

```
I'll help you build [feature]. First, a few quick questions:

1. What's the primary goal?
   A. [Goal 1]  B. [Goal 2]  C. [Goal 3]  D. Other

2. Who's the target user?
   A. [User type 1]  B. [User type 2]  C. All users  D. Other

3. What's the scope?
   A. MVP only  B. Full-featured  C. Backend only  D. Frontend only

Reply with: "1A, 2C, 3B" (or type your own answers)
```

### Step 1.3: Generate PRD

Create `tasks/prd-[feature-name].md`:

```markdown
# PRD: [Feature Name]

## Overview

Brief description of the feature and its value.

## Goals

- Specific, measurable objective 1
- Specific, measurable objective 2

## Non-Goals

- Explicitly what this feature will NOT do
- Scope boundaries

## User Stories

### US-001: [Title]

**Description:** As a [user type], I want [capability] so that [benefit].

**Acceptance Criteria:**

- [ ] Specific, verifiable criterion
- [ ] Another testable criterion
- [ ] Typecheck passes
- [ ] Tests pass (if applicable)

### US-002: [Title]

...

## Technical Approach

- Key architectural decisions
- Integration points with existing code
- Dependencies between stories

## Success Metrics

How we'll know this feature is working correctly.
```

**Story Sizing Rules:**

| Right-sized (1 iteration)         | Too big (split)             |
| --------------------------------- | --------------------------- |
| Add a database column             | Build entire dashboard      |
| Create single API endpoint        | Add authentication system   |
| Add UI component to existing page | Refactor the API            |
| Add filter dropdown               | Complete feature end-to-end |

**Rule:** If you can't describe the change in 2-3 sentences, split it.

### Step 1.4: Get Approval

```
I've created the PRD at `tasks/prd-[feature-name].md`.

Summary:
- [N] user stories identified
- Estimated complexity: [Low/Medium/High]
- Key dependencies: [list]

Please review the PRD. Reply with:
- "approved" - Convert to prd.json and begin implementation
- "edit [story]" - Modify a specific story
- "add [story]" - Add a new story
- "questions" - Ask me anything about the approach
```

---

## Phase 2: JSON Conversion

**Goal:** Convert approved PRD to machine-readable `prd.json`.

### Step 2.1: Archive Previous Run (if needed)

```bash
# If prd.json exists with different branch
if [ -f prd.json ]; then
  BRANCH=$(jq -r '.branchName' prd.json)
  if [ "$BRANCH" != "current-branch-name" ]; then
    mkdir -p archive/$(date +%Y-%m-%d)-$BRANCH
    mv prd.json progress.md archive/$(date +%Y-%m-%d)-$BRANCH/
  fi
fi
```

### Step 2.2: Create Feature Branch

```bash
git checkout -b feature/[feature-name]
```

### Step 2.2a: Create Feature Worktree (Optional)

If `.worktree-scaffold.json` exists in the project root, create an isolated worktree for this feature. This keeps development separate from the main working directory.

**When to use worktrees:**
- Large features with many stories
- Features that need isolation from other work
- Parallel feature development

**How to create:**

```bash
# Check if worktree-scaffold config exists
if [ -f .worktree-scaffold.json ]; then
  # Read config
  WORKTREE_DIR=$(jq -r '.worktreeDir // "../"' .worktree-scaffold.json)
  BRANCH_PREFIX=$(jq -r '.branchPrefix // "feature/"' .worktree-scaffold.json)

  # Feature name without prefix
  FEATURE_NAME="${BRANCH_NAME#${BRANCH_PREFIX}}"
  WORKTREE_PATH="${WORKTREE_DIR}${FEATURE_NAME}"

  # Create worktree
  git worktree add "$WORKTREE_PATH" "$BRANCH_NAME"

  # Run scaffolding if configured
  SCAFFOLD_TYPE=$(jq -r '.defaultScaffold // "default"' .worktree-scaffold.json)
  # Generate scaffold files based on config templates

  echo "Worktree created at: $WORKTREE_PATH"
  echo "Continuing autonomous loop in worktree..."
  cd "$WORKTREE_PATH"
fi
```

**Store worktree info in prd.json:**

```json
{
  "worktree": {
    "enabled": true,
    "path": "../feature-name",
    "mainRepoPath": "/original/repo/path"
  }
}
```

### Step 2.3: Generate prd.json

```json
{
  "project": "[Project Name]",
  "branchName": "feature/[feature-name]",
  "description": "[Feature description]",
  "createdAt": "2024-01-15T10:00:00Z",
  "delegation": {
    "enabled": false,
    "fallbackToDirect": true
  },
  "delegationMetrics": {
    "totalStories": 0,
    "delegatedCount": 0,
    "directCount": 0,
    "successRate": 0,
    "avgAttempts": 0,
    "byAgent": {},
    "byType": {},
    "detectionAccuracy": null
  },
  "optimization": {
    "progressSummary": {
      "enabled": true,
      "recentStoriesCount": 3,
      "maxLearnings": 15,
      "autoGenerate": true
    }
  },
  "userStories": [
    {
      "id": "US-001",
      "title": "[Title]",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": [
        "Specific criterion 1",
        "Typecheck passes",
        "Tests pass"
      ],
      "priority": 1,
      "dependsOn": [],
      "passes": false,
      "attempts": 0,
      "notes": "",
      "detectedType": null,
      "delegatedTo": null
    }
  ]
}
```

**Delegation Configuration:**

- `delegation.enabled`: Set to `true` to enable smart delegation to specialized agents
- `delegation.fallbackToDirect`: If `true`, falls back to direct implementation when delegation fails
- `detectedType`: Automatically populated with story type (frontend, api, database, devops, fullstack, general)
- `delegatedTo`: Records which agent implemented the story (e.g., "frontend-agent", "api-agent", or null for direct implementation)

**Delegation Metrics:**

Automatically tracked performance metrics:

- `totalStories`: Total number of stories completed
- `delegatedCount`: Number of stories delegated to agents
- `directCount`: Number of stories implemented directly
- `successRate`: Percentage of stories that passed on first attempt (0-100)
- `avgAttempts`: Average number of attempts per story
- `byAgent`: Breakdown by agent type:
  ```json
  {
    "frontend-agent": { "count": 3, "successRate": 100, "avgAttempts": 1.0 },
    "api-agent": { "count": 2, "successRate": 50, "avgAttempts": 1.5 },
    "direct": { "count": 1, "successRate": 100, "avgAttempts": 1.0 }
  }
  ```
- `byType`: Breakdown by detected story type:
  ```json
  {
    "frontend": 3,
    "api": 2,
    "database": 1,
    "general": 1
  }
  ```
- `detectionAccuracy`: Manual validation results (optional, set by user review)

**Optimization Configuration:**

Token optimization settings for reducing context size:

- `optimization.progressSummary.enabled`: Enable progress.md summarization (default: `true`)
- `optimization.progressSummary.recentStoriesCount`: How many recent stories to include in full detail (default: `3`)
- `optimization.progressSummary.maxLearnings`: Maximum extracted learnings to include (default: `15`)
- `optimization.progressSummary.autoGenerate`: Auto-regenerate summary after each story (default: `true`)

**Disable Progress Summarization:**

To load full progress.md instead of summary:

```json
{
  "optimization": {
    "progressSummary": {
      "enabled": false
    }
  }
}
```

**Updating Metrics:**

After each story completion, update delegationMetrics:

```javascript
function updateDelegationMetrics(prd, story) {
  const metrics = prd.delegationMetrics || {
    totalStories: 0, delegatedCount: 0, directCount: 0,
    successRate: 0, avgAttempts: 0, byAgent: {}, byType: {}
  };

  // Update totals
  metrics.totalStories++;
  if (story.delegatedTo) {
    metrics.delegatedCount++;
  } else {
    metrics.directCount++;
  }

  // Update by-agent breakdown
  const agentKey = story.delegatedTo || 'direct';
  if (!metrics.byAgent[agentKey]) {
    metrics.byAgent[agentKey] = { count: 0, successRate: 0, avgAttempts: 0 };
  }
  metrics.byAgent[agentKey].count++;

  // Update by-type breakdown
  if (story.detectedType) {
    metrics.byType[story.detectedType] = (metrics.byType[story.detectedType] || 0) + 1;
  }

  // Calculate overall success rate (first attempt pass)
  const allStories = prd.userStories.filter(s => s.passes);
  const firstAttemptSuccess = allStories.filter(s => s.attempts === 1).length;
  metrics.successRate = Math.round((firstAttemptSuccess / allStories.length) * 100);

  // Calculate average attempts
  const totalAttempts = allStories.reduce((sum, s) => sum + s.attempts, 0);
  metrics.avgAttempts = (totalAttempts / allStories.length).toFixed(2);

  prd.delegationMetrics = metrics;
}
```

**Querying Metrics with jq:**

```bash
# Overall delegation rate
jq '.delegationMetrics | "Delegation: \(.delegatedCount)/\(.totalStories) (\((.delegatedCount/.totalStories*100)|round)%)"' prd.json

# Agent performance
jq '.delegationMetrics.byAgent | to_entries | .[] | "\(.key): \(.value.count) stories, \(.value.successRate)% success"' prd.json

# Most common story types
jq '.delegationMetrics.byType | to_entries | sort_by(-.value) | .[] | "\(.key): \(.value)"' prd.json

# Success rate trend
jq '.delegationMetrics | "Success rate: \(.successRate)% | Avg attempts: \(.avgAttempts)"' prd.json
```

### Step 2.4: Initialize Progress File

Create `progress.md`:

```markdown
# Progress Log: [Feature Name]

Branch: `feature/[feature-name]`
Started: [Date]

---
```

### Step 2.5: Detect Verification Commands

Scan the codebase to find the right commands:

```bash
# Check for common patterns
grep -l "typecheck\|tsc\|type-check" package.json 2>/dev/null
grep -l "test\|jest\|vitest\|pytest" package.json pyproject.toml 2>/dev/null
grep -l "lint\|eslint" package.json 2>/dev/null
```

Store in prd.json:

```json
{
  "verification": {
    "typecheck": "npm run typecheck",
    "test": "npm run test",
    "lint": "npm run lint",
    "build": "npm run build"
  }
}
```

---

## Phase 3: Autonomous Loop

**Goal:** Implement one story per iteration until complete.

### Step 3.0: Load Context (with Token Optimization)

At the start of EVERY iteration, load context efficiently:

```bash
# Read current state
cat prd.json
cat AGENTS.md 2>/dev/null

# Use optimized progress loading (see below)
```

**Optimized Progress Loading:**

To reduce token usage, prefer `progress-summary.md` over full `progress.md`:

```javascript
function loadProgressContext(prd) {
  const optimization = prd.optimization?.progressSummary ?? { enabled: true };

  // 1. Check if summary exists and optimization is enabled
  if (optimization.enabled && fileExists('progress-summary.md')) {
    const summaryMtime = getModifiedTime('progress-summary.md');
    const progressMtime = getModifiedTime('progress.md');

    // Use summary if it's fresh (newer than progress.md)
    if (summaryMtime >= progressMtime) {
      return readFile('progress-summary.md');  // ~400-800 tokens
    }
  }

  // 2. Fallback: Extract recent entries from progress.md
  const recentCount = optimization.recentStoriesCount || 3;
  return extractRecentEntries(readFile('progress.md'), recentCount);
}
```

**Token Savings:**

| Stories | Full progress.md | progress-summary.md | Savings |
|---------|------------------|---------------------|---------|
| 5 | ~1,500 tokens | ~500 tokens | 67% |
| 10 | ~3,000 tokens | ~700 tokens | 77% |
| 20 | ~6,000 tokens | ~900 tokens | 85% |

The summary grows logarithmically (learnings deduplicate) while full log grows linearly.

**Load cross-codebase learnings:**

```
# Query memory for relevant insights based on detected tech stack
mcp__memory__search_nodes({ query: "[detected-framework]" })  # e.g., "nextjs", "fastapi"
mcp__memory__search_nodes({ query: "mistake" })               # Avoid past mistakes
mcp__memory__search_nodes({ query: "pattern" })               # Apply known patterns
```

**Stack detection -> memory queries:**

| Detected Stack | Memory Queries                         |
| -------------- | -------------------------------------- |
| Next.js        | `nextjs`, `react`, `server-components` |
| Supabase       | `supabase`, `rls`, `postgres`          |
| FastAPI        | `fastapi`, `python`, `api`             |
| React          | `react`, `hooks`, `state-management`   |

Find the next story: first `passes: false` ordered by `priority`, respecting `dependsOn`.

### Step 3.0a: Analyze Story Type (Smart Delegation)

Before implementing, detect the story type to enable smart delegation.

**Story Type Detection:**

Analyze the story to determine its primary type:

```javascript
function detectStoryType(story) {
  const fullText = [
    story.title,
    story.description,
    ...story.acceptanceCriteria,
    story.notes || ''
  ].join(' ').toLowerCase();

  const signals = {
    frontend: 0,
    backend: 0,
    api: 0,
    database: 0,
    devops: 0,
    fullstack: 0
  };

  // Frontend patterns
  const frontendPatterns = [
    /\b(component|ui|page|form|button|modal|dropdown|layout|widget)\b/,
    /\b(react|vue|angular|svelte|next\.js|nuxt)\b/,
    /\b(css|style|theme|responsive|mobile|desktop)\b/,
    /\b(click|hover|animation|transition|render)\b/,
    /\/(components|pages|app|views|layouts)\//,
    /\.(tsx|jsx|vue|svelte)$/
  ];

  // API patterns
  const apiPatterns = [
    /\b(endpoint|route|api|rest|graphql)\b/,
    /\b(get|post|put|delete|patch)\s+(request|endpoint)/,
    /\b(middleware|authentication|authorization)\b/,
    /\b(controller|service|handler)\b/,
    /\/(api|routes|controllers|services)\//,
    /\b(express|fastapi|flask|django|nestjs)\b/
  ];

  // Database patterns
  const databasePatterns = [
    /\b(database|schema|migration|table|column|index)\b/,
    /\b(query|sql|postgres|mysql|mongodb|supabase)\b/,
    /\b(orm|prisma|drizzle|sequelize|mongoose)\b/,
    /\b(rls|row level security|foreign key|constraint)\b/,
    /\/(migrations|schema|models|entities)\//,
    /\b(create table|alter table|add column)\b/
  ];

  // DevOps patterns
  const devopsPatterns = [
    /\b(deploy|deployment|ci\/cd|docker|kubernetes|container)\b/,
    /\b(github actions|gitlab ci|jenkins|vercel|railway)\b/,
    /\b(environment variable|config|secrets|env)\b/,
    /\b(build|bundle|webpack|vite|rollup)\b/,
    /\.(dockerfile|yaml|yml|\.github\/workflows)$/,
    /\b(nginx|apache|load balancer|cdn)\b/
  ];

  // Fullstack patterns (touches multiple layers)
  const fullstackPatterns = [
    /\b(end.to.end|e2e|full.stack|complete feature)\b/,
    /\b(authentication system|oauth flow|signup flow)\b/,
    /\b(frontend.*backend|backend.*frontend)\b/,
    /\b(database.*ui|ui.*database)\b/
  ];

  // Score each category
  frontendPatterns.forEach(p => { if (p.test(fullText)) signals.frontend++; });
  apiPatterns.forEach(p => { if (p.test(fullText)) signals.api++; });
  databasePatterns.forEach(p => { if (p.test(fullText)) signals.database++; });
  devopsPatterns.forEach(p => { if (p.test(fullText)) signals.devops++; });
  fullstackPatterns.forEach(p => { if (p.test(fullText)) signals.fullstack++; });

  // API is subset of backend
  if (signals.api > 0) signals.backend = signals.api;

  // Determine primary type
  const maxScore = Math.max(...Object.values(signals));

  if (signals.fullstack >= 2) return 'fullstack';
  if (maxScore === 0) return 'general'; // No clear signals

  // Return highest scoring type (priority order if tied)
  const priority = ['database', 'api', 'backend', 'frontend', 'devops'];
  for (const type of priority) {
    if (signals[type] === maxScore) {
      return type;
    }
  }

  return 'general';
}
```

**Detection Implementation:**

When Step 3.0a runs during autonomous loop execution:

1. **Run Detection:**
   ```javascript
   const detectedType = detectStoryType(currentStory);
   ```

2. **Log to Console:**
   ```
   Story type detected: api
   Detection signals: { api: 3, backend: 3, frontend: 0, database: 0, devops: 0 }
   ```

3. **Store in prd.json:**
   ```javascript
   currentStory.detectedType = detectedType;
   savePRD(prd);
   ```

4. **Update progress.md:**
   ```markdown
   ## Story Analysis

   - Detected type: api
   - Confidence signals: { api: 3, backend: 3, frontend: 0 }
   ```

**Important:** Detection runs automatically but does **NOT** trigger delegation unless `delegation.enabled = true` in prd.json. This allows testing detection accuracy before enabling delegation.

**Example Output:**

```
## Starting: US-003 - Add user profile API endpoint

Story type detected: api
Detection signals: { api: 3, backend: 3, frontend: 0, database: 0, devops: 0 }

**Goal:** Create GET /api/users/:id endpoint

**Acceptance Criteria:**
- [ ] Returns user object with id, name, email
- [ ] Returns 404 if not found
- [ ] Returns 401 if not authenticated
- [ ] Typecheck passes
- [ ] Tests pass

**Approach:** Create new API route handler in app/api/users/[id]/route.ts...
```

### Step 3.1: Announce Task

```
## Starting: US-[XXX] - [Title]

**Goal:** [One-line description]

**Acceptance Criteria:**
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Typecheck passes

**Approach:** [2-3 sentences on how you'll implement this]
```

### Step 3.2: Implement Code (with Smart Delegation)

**Check Delegation Status:**

```javascript
const delegationEnabled = prd.delegation?.enabled === true;
const fallbackToDirect = prd.delegation?.fallbackToDirect !== false;
```

**Option A: Delegation Enabled**

If `delegationEnabled === true`:

1. **Select Specialized Agent:**

   ```javascript
   // Agent type mapping: story type → specialized agent skill
   const AGENT_MAP = {
     'frontend': 'frontend-agent',      // UI/component work
     'backend': 'backend-agent',        // Server-side logic (alias for api)
     'api': 'api-agent',                // REST/GraphQL endpoints
     'database': 'database-agent',      // Schema, migrations, queries
     'devops': 'devops-agent',          // CI/CD, deployment, infrastructure
     'fullstack': 'orchestrator-fullstack', // Multi-layer features
     'general': 'general-purpose'       // Catch-all for unclear stories
   };

   const storyType = detectStoryType(story); // From Step 3.0a
   const agentType = AGENT_MAP[storyType] || 'general-purpose';

   // Note: Agent availability is checked when Task tool is invoked
   // If agent skill is not installed, Task will fail and trigger fallback
   ```

   **Log agent selection:**
   ```
   Detected story type: ${storyType}
   Selected agent: ${agentType}

   Delegating to ${agentType}...
   ```

2. **Generate Subagent Context:**

   Create a detailed prompt for the subagent:

   ```markdown
   # Story Implementation Task

   You are implementing a single user story for the autonomous-dev orchestrator.

   ## Scope Constraints
   **ONLY implement this specific story.** Do not:
   - Implement other stories from the PRD
   - Refactor unrelated code
   - Add features beyond acceptance criteria
   - Create unnecessary abstractions
   - Create documentation unless explicitly required by acceptance criteria

   ## Story Details
   **ID:** ${story.id}
   **Title:** ${story.title}
   **Priority:** ${story.priority}

   **Description:**
   ${story.description}

   **Acceptance Criteria:**
   ${story.acceptanceCriteria.map(c => `- [ ] ${c}`).join('\n')}

   ## Project Context
   **Tech Stack:** ${detectStack()}
   **Branch:** ${prd.branchName}
   **Working Directory:** ${process.cwd()}

   **Verification Commands:**
   ${Object.entries(prd.verification || {})
     .map(([type, cmd]) => `- ${type}: \`${cmd}\``)
     .join('\n')}

   ## Repository Patterns
   ${readFile('AGENTS.md') || 'No documented patterns yet'}

   ## Recent Implementation Context
   ${extractRecentProgress(3)} // Last 3 entries from progress.md

   ## Memory Insights
   Patterns to apply:
   ${queryMemoryPatterns(detectStack())}

   Mistakes to avoid:
   ${queryMemoryMistakes()}

   ## Dependencies from Previous Stories
   ${story.dependsOn.map(id => `- ${id}: ${getPreviousStoryNotes(id)}`).join('\n')}

   ## Your Task
   1. Read relevant existing code
   2. Implement ONLY what's needed for this story
   3. Run verification commands
   4. Report structured results

   ## Required Output Format
   ```
   RESULT: [SUCCESS|FAILURE]

   Files changed:
   - path/to/file1.ts (new/modified)
   - path/to/file2.ts (modified)

   Verification:
   - Typecheck: [PASS|FAIL]
   - Tests: [PASS|FAIL - X/Y passed]
   - Lint: [PASS|FAIL]

   Implementation notes:
   [2-3 sentences describing key decisions]

   Learnings:
   [Patterns discovered or issues encountered]
   ```
   ```

3. **Invoke Subagent:**

   ```javascript
   const result = await Task({
     subagent_type: agentType,
     description: `Implement ${story.id}: ${story.title}`,
     prompt: subagentPrompt
   });
   ```

4. **Parse Subagent Result:**

   ```javascript
   function parseSubagentResult(output) {
     // Extract RESULT line
     const resultMatch = output.match(/RESULT:\s*(SUCCESS|FAILURE)/i);

     // Extract files changed
     const filesMatch = output.match(/Files changed:\n((?:- .+\n?)+)/);
     const filesChanged = filesMatch?.[1]
       ?.split('\n')
       .filter(l => l.trim())
       .map(l => l.replace(/^- /, '').trim()) || [];

     // Extract verification results
     const verificationMatch = output.match(/Verification:\n((?:- .+\n?)+)/);
     const verification = {};
     if (verificationMatch) {
       verificationMatch[1].split('\n').forEach(line => {
         const match = line.match(/- (\w+): (PASS|FAIL)/i);
         if (match) verification[match[1].toLowerCase()] = match[2].toUpperCase();
       });
     }

     // Extract notes
     const notesMatch = output.match(/Implementation notes:\n(.+?)(?=\n\n|Learnings:|$)/s);
     const notes = notesMatch?.[1]?.trim() || '';

     const learningsMatch = output.match(/Learnings:\n(.+?)$/s);
     const learnings = learningsMatch?.[1]?.trim() || '';

     return {
       success: resultMatch?.[1]?.toUpperCase() === 'SUCCESS',
       filesChanged,
       verification,
       notes,
       learnings
     };
   }

   const parsed = parseSubagentResult(result);
   ```

   **Validate Parsed Result:**

   ```javascript
   function validateSubagentResult(parsed, story) {
     const errors = [];

     // 1. Check required fields present
     if (parsed.success === undefined) {
       errors.push('Missing RESULT status');
     }

     if (!parsed.filesChanged || parsed.filesChanged.length === 0) {
       errors.push('No files changed reported');
     }

     if (!parsed.verification || Object.keys(parsed.verification).length === 0) {
       errors.push('No verification results reported');
     }

     // 2. Validate verification results format
     for (const [key, value] of Object.entries(parsed.verification)) {
       if (value !== 'PASS' && value !== 'FAIL') {
         errors.push(`Invalid verification status for ${key}: ${value}`);
       }
     }

     // 3. Check files changed are reasonable
     const suspiciousFiles = parsed.filesChanged.filter(file =>
       file.includes('node_modules/') ||
       file.includes('.git/') ||
       file.includes('package-lock.json') ||
       file.match(/\.(env|secret|key)$/)
     );

     if (suspiciousFiles.length > 0) {
       errors.push(`Suspicious files modified: ${suspiciousFiles.join(', ')}`);
     }

     // 4. Validate file paths exist or are new
     for (const file of parsed.filesChanged) {
       const isNew = file.includes('(new)');
       const filePath = file.replace(/\s*\(new\|modified\)/, '').trim();
       // Note: File existence check would happen here
       // if (!isNew && !fileExists(filePath)) {
       //   errors.push(`File not found: ${filePath}`);
       // }
     }

     return {
       valid: errors.length === 0,
       errors
     };
   }

   function allVerificationsPassed(verification) {
     return Object.values(verification).every(status => status === 'PASS');
   }

   // Validate result
   const validation = validateSubagentResult(parsed, story);

   if (!validation.valid) {
     console.error('⚠ Subagent result validation failed:');
     validation.errors.forEach(err => console.error(`  - ${err}`));
     // Treat as delegation failure
     parsed.success = false;
   }
   ```

   **Error Handling for Malformed Output:**

   ```javascript
   try {
     const parsed = parseSubagentResult(result);
     const validation = validateSubagentResult(parsed, story);

     if (!validation.valid) {
       throw new Error(`Validation failed: ${validation.errors.join('; ')}`);
     }
   } catch (error) {
     console.error(`✗ Failed to parse subagent output: ${error.message}`);

     // Log raw output for debugging
     console.log('Raw subagent output:');
     console.log(result.substring(0, 500)); // First 500 chars

     // Trigger fallback
     if (fallbackToDirect) {
       console.log('⚠ Falling back to direct implementation...');
       // Proceed to Option B
     } else {
       throw error;
     }
   }
   ```

5. **Handle Delegation Result:**

   If delegation **succeeds**:
   ```javascript
   if (parsed.success && allVerificationsPassed(parsed.verification)) {
     // Update story in prd.json
     story.passes = true;
     story.delegatedTo = agentType;
     story.completedAt = new Date().toISOString();

     // Log success
     console.log(`✓ ${story.id} completed via ${agentType}`);

     // Continue to Step 3.3 (verification)
   }
   ```

   If delegation **fails** and `fallbackToDirect === true`:

   **Common failure reasons:**
   - Agent skill not installed/available
   - Agent returned FAILURE result
   - Verification commands failed
   - Task tool error

   ```
   ⚠ Delegation to ${agentType} failed.
   Reason: ${getFailureReason(result)}

   Falling back to direct implementation...
   ```
   → Proceed to Option B (Direct Implementation)

   **Note:** The fallback mechanism provides automatic recovery when:
   - Selected agent is not installed (`general-purpose` always available as ultimate fallback)
   - Agent fails to implement the story correctly
   - Verification fails after delegation

   If delegation **fails** and `fallbackToDirect === false`:
   ```
   ✗ Delegation failed and fallback is disabled.

   Options:
   1. Enable fallback: Set delegation.fallbackToDirect = true
   2. Try different agent (manual override)
   3. Skip this story
   4. Pause autonomous mode

   What would you like to do?
   ```

   **Fallback Tracking:**

   When fallback occurs, story metadata reflects direct implementation:
   ```json
   {
     "id": "US-007",
     "delegatedTo": null,
     "notes": "Delegation to api-agent failed: agent not available. Fell back to direct implementation."
   }
   ```

   **Error Scenario Examples:**

   **Scenario 1: Agent Not Available**
   ```
   ## Starting: US-002 - Add user profile endpoint

   Story type detected: api
   Selected agent: api-agent

   Delegating to api-agent...
   ✗ Error: Skill 'api-agent' not found
   ⚠ Delegation to api-agent failed.
   Reason: Agent skill not installed

   Falling back to direct implementation...
   ```

   **Scenario 2: Agent Returns FAILURE**
   ```
   ## Starting: US-003 - Add email column to users table

   Delegating to database-agent...

   RESULT: FAILURE

   Verification:
   - Migration up: FAIL - Syntax error on line 12

   ⚠ Delegation to database-agent failed.
   Reason: Agent returned FAILURE result

   Falling back to direct implementation...
   ```

   **Scenario 3: Verification Fails**
   ```
   ## Starting: US-001 - Add dark mode toggle

   Delegating to frontend-agent...

   RESULT: SUCCESS

   Verification:
   - Typecheck: FAIL - Type error in ThemeToggle.tsx

   ⚠ Delegation to frontend-agent failed.
   Reason: Verification commands failed (typecheck: FAIL)

   Falling back to direct implementation...
   ```

   **Scenario 4: Malformed Output**
   ```
   ## Starting: US-005 - Set up CI/CD

   Delegating to devops-agent...

   [Agent output doesn't match expected format]

   ✗ Failed to parse subagent output: Missing RESULT status
   Raw subagent output:
   [First 500 characters shown...]

   ⚠ Falling back to direct implementation...
   ```

**Option B: Direct Implementation (Default)**

If `delegationEnabled === false` OR delegation failed with fallback:

1. Read relevant existing files first
2. Follow patterns from `AGENTS.md` and existing code
3. Write code for ONLY this user story
4. Keep changes minimal and focused

**Implementation Checklist:**

- [ ] Read existing code patterns first
- [ ] Make minimal necessary changes
- [ ] Add tests if acceptance criteria requires them
- [ ] Don't refactor unrelated code

### Step 3.3: Run Verification

Execute verification commands from prd.json:

```bash
# Run typecheck
npm run typecheck

# Run tests (if applicable to this story)
npm run test

# Run lint (optional but recommended)
npm run lint
```

### Step 3.4: Handle Results

**If verification passes:**

1. Update `prd.json`:

   ```json
   {
     "passes": true,
     "attempts": 1,
     "completedAt": "2024-01-15T11:30:00Z"
   }
   ```

2. Commit the work:

   ```bash
   git add -A
   git commit -m "feat(US-XXX): [Title]

   - [What was implemented]
   - [Key decisions made]"
   ```

3. Update `progress.md`:

   ```markdown
   ## [Timestamp] - US-XXX: [Title]

   **Implementation:**

   - [What was done]
   - [Files changed]

   **Learnings:**

   - [Patterns discovered]
   - [Gotchas encountered]

   ---
   ```

4. **Generate progress-summary.md** (Token Optimization):

   After updating progress.md, regenerate the compact summary:

   ```javascript
   function generateProgressSummary(prd, progressMd) {
     const optimization = prd.optimization?.progressSummary ?? { enabled: true };
     if (!optimization.enabled) return;

     const recentCount = optimization.recentStoriesCount || 3;
     const maxLearnings = optimization.maxLearnings || 15;

     const summary = `# Progress Summary: ${prd.project}

Branch: \`${prd.branchName}\`
Started: ${prd.createdAt?.split('T')[0] || 'Unknown'}
Last updated: ${new Date().toISOString().split('T')[0]}

## Completion Status

${generateCompletionStatus(prd)}

## Story Status

${generateStoryTable(prd)}

## Key Learnings (Extracted)

${extractKeyLearnings(progressMd, maxLearnings)}

## Recent Context (Last ${recentCount} Stories)

${extractRecentEntries(progressMd, recentCount)}

---

*Auto-generated from progress.md. Full history preserved in progress.md.*
`;

     writeFile('progress-summary.md', summary);
   }
   ```

   **Helper Functions:**

   ```javascript
   function generateCompletionStatus(prd) {
     const total = prd.userStories.length;
     const complete = prd.userStories.filter(s => s.passes).length;
     const inProgress = prd.userStories.find(s => !s.passes && s.attempts > 0);
     const pct = Math.round((complete / total) * 100);

     return `Stories: ${complete}/${total} complete (${pct}%)
Current: ${inProgress ? `${inProgress.id} (attempt ${inProgress.attempts})` : 'None'}
Blocked: None`;
   }

   function generateStoryTable(prd) {
     const header = '| ID | Title | Status | Agent | Attempts |\n|----|-------|--------|-------|----------|';
     const rows = prd.userStories.map(s => {
       const status = s.passes ? '✓' : (s.attempts > 0 ? '→' : '○');
       const agent = s.delegatedTo || '-';
       const title = s.title.length > 30 ? s.title.slice(0, 27) + '...' : s.title;
       return `| ${s.id} | ${title} | ${status} | ${agent} | ${s.attempts} |`;
     });
     return [header, ...rows].join('\n') + '\n\nLegend: ✓ complete, → in progress, ○ pending';
   }

   function extractKeyLearnings(progressMd, maxLearnings) {
     const learnings = { patterns: [], gotchas: [], dependencies: [] };

     // Extract from **Learnings:** sections
     const learningBlocks = progressMd.match(/\*\*Learnings:\*\*\n((?:- .+\n?)+)/g) || [];
     learningBlocks.forEach(block => {
       const lines = block.match(/- .+/g) || [];
       lines.forEach(line => {
         const text = line.replace(/^- /, '').trim();
         if (/gotcha|warning|careful|must|don't|avoid/i.test(text)) {
           learnings.gotchas.push(text);
         } else {
           learnings.patterns.push(text);
         }
       });
     });

     // Deduplicate and limit
     const uniquePatterns = [...new Set(learnings.patterns)].slice(0, maxLearnings);
     const uniqueGotchas = [...new Set(learnings.gotchas)].slice(0, 5);

     let output = '### Repository Patterns\n';
     output += uniquePatterns.map(p => `- ${p}`).join('\n') || '- No patterns extracted yet';
     output += '\n\n### Gotchas & Warnings\n';
     output += uniqueGotchas.map(g => `- ${g}`).join('\n') || '- No gotchas recorded';

     return output;
   }

   function extractRecentEntries(progressMd, count) {
     // Split by story headers (## YYYY-MM-DD or ## timestamp)
     const entries = progressMd.split(/(?=^## \d{4}-\d{2}-\d{2}|\n---\n\n## )/m);

     // Take last N entries (skip header)
     const recent = entries.filter(e => e.match(/^## \d/)).slice(-count);

     if (recent.length === 0) return 'No completed stories yet.';

     // Compress each entry (keep first 10 lines max)
     return recent.map(entry => {
       const lines = entry.trim().split('\n');
       if (lines.length <= 12) return entry.trim();
       return lines.slice(0, 12).join('\n') + '\n[...]';
     }).join('\n\n');
   }
   ```

   **When to Generate:**
   - After each story completion (Step 3.4)
   - When `status summarize` command is run
   - Automatically on next iteration if progress.md is newer

5. **Extract and save learnings to Memory:**

   After each successful story, evaluate if any learnings are broadly applicable:

   ```
   # Ask yourself:
   # 1. Did I discover a pattern that would help in other projects?
   # 2. Did I make a mistake that should be avoided elsewhere?
   # 3. Did I learn something about a framework/tool?

   # If yes, save to memory:
   mcp__memory__create_entities({
     entities: [{
       name: "pattern:descriptive-name",
       entityType: "pattern",
       observations: [
         "What the pattern is",
         "When to apply it",
         "Applies to: [frameworks/languages]"
       ]
     }]
   })
   ```

   **Learning extraction criteria:**

   | Save as                 | When                                      |
   | ----------------------- | ----------------------------------------- |
   | `pattern`               | Solution worked well and is reusable      |
   | `mistake`               | Made an error, had to fix it              |
   | `tech-insight`          | Learned something about a specific tool   |
   | `architecture-decision` | Made a structural choice that proved good |

   **Skip saving if:**
   - Learning is project-specific (put in AGENTS.md instead)
   - Already exists in memory (check first)
   - Too trivial to be useful

5. Check completion and continue:

   ```
   US-XXX complete. [N] stories remaining.

   Continuing to next story...
   ```

**If verification fails:**

1. Increment attempts: `"attempts": N+1`

2. Analyze failure:

   ```
   Verification failed for US-XXX (attempt N).

   Error: [error message]

   Analysis: [what went wrong]

   Fix: [what I'll change]
   ```

3. If `attempts < 3`: Fix and retry step 3.3

4. **After fixing a failure, save the mistake to memory:**

   ```
   mcp__memory__create_entities({
     entities: [{
       name: "mistake:descriptive-name",
       entityType: "mistake",
       observations: [
         "What went wrong: [description]",
         "How to avoid: [prevention strategy]",
         "Applies to: [frameworks/languages]",
         "Severity: [low/medium/high/critical]"
       ]
     }]
   })
   ```

5. If `attempts >= 3`:

   ```
   US-XXX failed after 3 attempts.

   Last error: [message]

   Options:
   1. Split this story into smaller pieces
   2. Get user help with the blockers
   3. Skip and continue with next story
   4. Pause autonomous mode

   What would you like to do?
   ```

### Step 3.5: Completion Check

After each story:

```javascript
const remaining = stories.filter((s) => !s.passes);
if (remaining.length === 0) {
  // All done!
  output("COMPLETE");
} else {
  // Continue to next story
  continueLoop();
}
```

**When all stories complete:**

```
===== FEATURE COMPLETE =====

Branch: feature/[name]
Stories completed: [N]
Total commits: [M]

Summary:
- [Key accomplishments]
- [Files changed]

Next steps:
1. Review the changes: git log --oneline feature/[name]
2. Run full test suite: npm test
3. Create PR when ready: gh pr create

Would you like me to:
A. Create a pull request
B. Show a detailed summary
C. Continue with more features
D. Clean up worktree (if used)
```

### Step 3.6: Worktree Cleanup (If Used)

If the feature was developed in a worktree, offer cleanup:

```bash
# Check if worktree was used
if [ -n "$(jq -r '.worktree.path // empty' prd.json)" ]; then
  WORKTREE_PATH=$(jq -r '.worktree.path' prd.json)
  MAIN_REPO=$(jq -r '.worktree.mainRepoPath' prd.json)

  echo "Feature developed in worktree: $WORKTREE_PATH"
  echo ""
  echo "Cleanup options:"
  echo "1. Keep worktree (for future reference)"
  echo "2. Remove worktree, keep branch"
  echo "3. Remove worktree and merge branch to main"
fi
```

**To remove worktree:**

```bash
# From main repo
cd "$MAIN_REPO"
git worktree remove "$WORKTREE_PATH"
git worktree prune
```

---

## Error Recovery

### Story Breaks Previous Functionality

```
Detected: Tests that passed before are now failing.

Affected tests: [list]

Options:
1. Rollback this story: git reset --hard HEAD~1
2. Fix the regression before continuing
3. Mark as known issue and continue
```

### Context Overflow Prevention

If a story is taking too many tokens:

1. Commit partial progress
2. Log current state to `progress.md`
3. Suggest splitting the story

### Stuck on Dependencies

If a story needs something not yet implemented:

1. Check if dependency story exists
2. If not, suggest adding it as US-00X
3. Reorder priorities if needed

---

## Key Files Reference

| File                        | Purpose                          | Created            |
| --------------------------- | -------------------------------- | ------------------ |
| `tasks/prd-*.md`            | Human-readable PRD               | Phase 1            |
| `prd.json`                  | Machine-readable task list       | Phase 2            |
| `progress.md`               | Append-only learnings (full)     | Phase 2+           |
| `progress-summary.md`       | Compact context (auto-generated) | Phase 3 (auto)     |
| `AGENTS.md`                 | Long-term repo patterns          | Anytime            |
| `archive/`                  | Previous completed PRDs          | Before new feature |
| `.worktree-scaffold.json`   | Worktree config (optional)       | User creates       |

---

## Worktree Integration

The autonomous-agent integrates with the `worktree-scaffold` skill for parallel development.

**Setup worktree support:**

1. Create `.worktree-scaffold.json` in project root (run `/worktree-scaffold` → `init worktree config`)
2. The agent will detect this file in Phase 2 and offer worktree creation
3. Each feature gets its own isolated workspace

**Benefits:**
- Isolate feature development from main repo
- Work on multiple features in parallel
- Keep main directory clean during development

**See also:** `/worktree-scaffold` skill for standalone worktree management.

---

## AGENTS.md Patterns to Document

When you discover patterns, add them to AGENTS.md:

```markdown
# Repository Patterns

## API Conventions

- All endpoints use REST conventions
- Error responses follow format: { error: string, code: number }

## Component Patterns

- UI components in src/components/
- Server actions in src/actions/

## Database

- Migrations in db/migrations/
- Schema in db/schema.ts

## Testing

- Unit tests alongside source files
- Integration tests in tests/

## Gotchas

- Must run db:generate after schema changes
- Server actions need revalidatePath for cache
```

---

## Quick Commands

| Command         | What it does                              |
| --------------- | ----------------------------------------- |
| "status"        | Show current progress and next story      |
| "skip"          | Skip current story, move to next          |
| "pause"         | Stop autonomous mode, wait for input      |
| "split [story]" | Break a story into smaller pieces         |
| "retry"         | Retry the current story                   |
| "complete"      | Force-mark current story as done          |
| "summarize"     | Regenerate progress-summary.md from full log |

### Summarize Command

The `summarize` command regenerates `progress-summary.md` from the full `progress.md`:

```
> summarize

Regenerating progress-summary.md...

Summary generated:
- Stories: 12/14 complete (86%)
- Learnings extracted: 15 patterns, 4 gotchas
- Recent context: Last 3 stories included
- Token savings: ~3,200 tokens (78% reduction)

Summary written to progress-summary.md
```

Use this command to:
- Force regenerate after manual edits to progress.md
- View token savings statistics
- Verify extracted learnings are correct

---

## Enabling Delegation (Beta)

**⚠️ BETA FEATURE:** Smart delegation is currently in beta. Automatic fallback to direct implementation ensures reliable operation.

### How to Enable

1. **Update prd.json:**
   ```json
   {
     "delegation": {
       "enabled": true,
       "fallbackToDirect": true
     }
   }
   ```

2. **Install specialized agents** (optional but recommended):
   ```bash
   # Frontend agent for UI/component work
   git clone https://github.com/org/frontend-agent ~/.claude/skills/frontend-agent

   # API agent for endpoint implementation
   git clone https://github.com/org/api-agent ~/.claude/skills/api-agent

   # Database agent for schema/migrations
   git clone https://github.com/org/database-agent ~/.claude/skills/database-agent

   # DevOps agent for CI/CD and deployment
   git clone https://github.com/org/devops-agent ~/.claude/skills/devops-agent
   ```

   **Note:** If agents aren't installed, delegation falls back to direct implementation automatically.

3. **Run autonomous-dev as normal:**
   - Story type detection runs automatically (Step 3.0a)
   - Delegation attempts if enabled and agent available (Step 3.2)
   - Falls back to direct implementation on any failure
   - Tracks metrics in `prd.json` (delegationMetrics)

### What to Expect

**With delegation enabled:**
- Each story is analyzed to detect type (frontend, api, database, etc.)
- Appropriate specialized agent is selected
- Agent implements the story with domain-specific expertise
- Fallback to direct implementation if agent unavailable or fails
- Metrics tracked for performance analysis

**Logging output:**
```
## Starting: US-003 - Add dark mode toggle

Story type detected: frontend
Selected agent: frontend-agent

Delegating to frontend-agent...

✓ Created app/components/ThemeToggle.tsx
✓ Typecheck passed
✓ Tests passed (2/2)

RESULT: SUCCESS

✓ US-003 complete (attempt 1)
  Implemented by: frontend-agent
```

### How to Disable

Set `delegation.enabled` to `false` in prd.json:
```json
{
  "delegation": {
    "enabled": false
  }
}
```

The autonomous loop continues with direct implementation.

### Beta Testing Checklist

- [ ] Enable delegation in a test PRD (non-production project)
- [ ] Run autonomous loop on 5-10 stories
- [ ] Review delegationMetrics in prd.json
- [ ] Check which agents performed well vs. needed fallback
- [ ] Review progress.md for delegation logs
- [ ] Verify fallback mechanism works (try without agents installed)
- [ ] Analyze metrics with jq queries (see examples.md)
- [ ] Report any issues or unexpected behavior

### Troubleshooting

**Issue: "Agent not found" error**
- **Cause:** Specialized agent skill not installed
- **Solution:** Either install the agent or rely on automatic fallback
- **Expected:** Delegation falls back to direct implementation automatically

**Issue: Delegation fails repeatedly for specific story type**
- **Cause:** Agent may not handle this pattern well
- **Solution:** Review progress.md logs, consider disabling delegation for that agent type
- **Temporary fix:** Implement story directly, file issue with agent maintainer

**Issue: Detection classifies story incorrectly**
- **Cause:** Story description lacks clear technical keywords
- **Solution:** Add specific keywords (e.g., "component", "endpoint", "migration")
- **Alternative:** Set `detectedType` manually in prd.json before running

**Issue: Metrics not updating**
- **Cause:** delegationMetrics object missing from prd.json
- **Solution:** Add empty metrics object (see schema in Phase 2 documentation)

**Issue: Want to force direct implementation for one story**
- **Solution:** Set `delegation.enabled = false` temporarily, or
- **Better:** Let delegation attempt and fallback if needed (no manual intervention required)

### Migration Guide: Enabling in Existing Projects

If you have an existing prd.json without delegation:

1. **Add delegation configuration:**
   ```json
   {
     "delegation": {
       "enabled": true,
       "fallbackToDirect": true
     },
     "delegationMetrics": {
       "totalStories": 0,
       "delegatedCount": 0,
       "directCount": 0,
       "successRate": 0,
       "avgAttempts": 0,
       "byAgent": {},
       "byType": {},
       "detectionAccuracy": null
     }
   }
   ```

2. **Add story-level fields** (optional - will be populated automatically):
   ```json
   {
     "id": "US-001",
     "detectedType": null,
     "delegatedTo": null
   }
   ```

3. **Run autonomous loop** - delegation activates automatically for remaining stories

### See Also

- [Detection validation](references/detection-validation.md) - Detection accuracy testing
- [Agent prompts](references/agent-prompts.md) - Subagent prompt templates
- [Examples](references/examples.md) - Complete delegation flow examples
- [Design doc](references/smart-delegation-design.md) - Architecture details
- [Progress summarization](references/progress-summarization-design.md) - Token optimization design

---

## Examples

See [references/examples.md](references/examples.md) for:

- Story splitting patterns
- Acceptance criteria templates
- Complete prd.json examples
- progress.md and progress-summary.md formats
