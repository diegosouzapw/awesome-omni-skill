---
name: baton
description: Context management system for solving compaction problems. Provides TLDR summaries, conversation tracking, and efficient post-compaction recovery. Triggers on "/baton".
---

# Baton - Context Management Skill

Manages context across sessions and compactions using TLDR summaries and structured documentation.

## Implementation

When `/baton` is invoked, parse the subcommand and execute the corresponding action:

### Core Commands
- **No arguments or `load`**: Display current TLDR summary
- **`init`**: Initialize new conversation
- **`rename <title>`**: Set conversation title (max 60 chars)
- **`rename --suggest`**: Get AI-generated title suggestions
- **`save [note]`**: Save current state with optional note
- **`update [section]`**: Update SUMMARY.md sections automatically
- **`update --auto`**: Auto-generate updates from recent activity
- **`history`**: Show all conversations with titles
- **`status`**: Show token usage and save recommendation
- **`archive`**: Archive completed items to prevent file bloat

### Navigation & Switching
- **`switch <conv-id-or-title>`**: Switch to different conversation
- **`switch --recent`**: Show recent conversations to choose from

### Search & Discovery
- **`search <term>`**: Search across all conversations, bugs, decisions
- **`search --bugs <term>`**: Search only bugs
- **`search --decisions <term>`**: Search only decisions
- **`search --conversations <term>`**: Search only conversations
- **`context <topic>`**: Load relevant past context (smart search)

### Reporting & Analytics
- **`report [timeframe]`**: Generate work summary report
- **`report --today`**: Today's work summary
- **`report --week`**: Past week summary
- **`report --conversation <id>`**: Specific conversation report
- **`metrics`**: Show baton system effectiveness metrics
- **`stats`**: Alias for metrics

### Validation & Health
- **`validate`**: Check file structure integrity
- **`health`**: Alias for validate

### Configuration
- **`auto-save on|off|status`**: Configure auto-save triggers
- **`template create <name>`**: Create custom SUMMARY.md template
- **`template use <name>`**: Switch to custom template
- **`template list`**: Show available templates

### Git Integration
- **`git-link`**: Associate conversation with current git branch
- **`git-summary`**: Generate commit message from SUMMARY.md

## Implementation Details

Each command should be implemented by reading/writing the appropriate files in `.claude/`:

### For `/baton init`
```bash
CONV_ID="conv-$(date +%Y%m%d-%H%M%S)"
mkdir -p .claude/conversations/$CONV_ID
echo $CONV_ID > .claude/CURRENT_CONVERSATION_ID
# Create SUMMARY.md from template
# Initialize shared files if they don't exist
touch .claude/{BUGS.md,DECISIONS.md,CONVERSATION_HISTORY.md,ENHANCEMENTS.md,USER_FEEDBACK.md}
mkdir -p .claude/{archive,templates}
# Create default settings if missing
cat > .claude/settings.json <<EOF
{
  "autoSave": {
    "enabled": true,
    "thresholds": [70, 85, 95],
    "notifyOnSave": true
  },
  "archiveThresholds": {
    "conversations": 10,
    "bugs": 20,
    "decisions": 15
  },
  "defaultTemplate": "standard"
}
EOF
```

### For `/baton load`
- Read `.claude/CURRENT_CONVERSATION_ID`
- Display `.claude/conversations/{conv-id}/SUMMARY.md`
- Display filtered entries from `.claude/BUGS.md` (matching conversation ID)
- Display filtered entries from `.claude/DECISIONS.md` (matching conversation ID)
- Display current TODO list if available

### For `/baton rename <title>`
- Read `.claude/CURRENT_CONVERSATION_ID`
- Update `.claude/conversations/{conv-id}/SUMMARY.md` Title field
- Update `.claude/CONVERSATION_HISTORY.md` entry for this conversation
- Validate title length (max 60 chars for clean display)
- Used in standardized response format header
- Example: `/baton rename Self-Improving AI Chatbot`

### For `/baton rename --suggest`
- Analyze recent work: TODO items, file changes, recent conversation
- Generate 3-5 concise title suggestions (max 60 chars each)
- Display suggestions with numbers for easy selection
- Allow user to choose or provide custom title
- Example output:
  ```
  Suggested titles based on recent work:
  1. API Authentication System
  2. User Login Bug Fixes
  3. Database Schema Migration
  4. Payment Integration Setup
  5. Custom: [Enter your own]

  Choose (1-5) or press Enter for custom:
  ```

### For `/baton save [note]`
- Read current state (TODO list, recent changes)
- Update `.claude/conversations/{conv-id}/SUMMARY.md`
- Add timestamp and optional note
- Update Task Checklist, Key Files, State Snapshot sections
- Show confirmation: "Saved at 45% token usage"

### For `/baton update [section]`
**Auto-update specific sections:**
- `--auto`: Analyze recent activity and update all sections
- `tasks`: Update Task Checklist from TODO list
- `files`: Update Key Files from recent git changes
- `state`: Update State Snapshot from current file/line
- `context`: Update Context in 3 Lines from recent work

**Implementation:**
- Read recent TODO completions, file modifications
- Analyze conversation for decisions, bugs discovered
- Update relevant SUMMARY.md sections
- Show what was updated: "Updated: Task Checklist (3 completed), Key Files (2 modified)"

### For `/baton history`
- Display `.claude/CONVERSATION_HISTORY.md`
- Format with conversation IDs, titles, status, dates
- Show active vs completed conversations

### For `/baton status`
- Check current token usage (estimate from conversation length)
- Calculate percentage to 70% threshold
- Show time since last SUMMARY.md update
- Recommend save if > 70%
- Check file sizes and recommend archival if thresholds exceeded:
  - CONVERSATION_HISTORY.md > 10 conversations
  - BUGS.md > 20 bugs
  - DECISIONS.md > 15 decisions
- Display auto-save configuration status

### For `/baton archive`
- Count items in each file by status
- Move completed/fixed/implemented items to `.claude/archive/`
- Archive structure:
  - `archive/conversations/YYYY-MM.md` (completed conversations by month)
  - `archive/bugs/fixed-YYYY-MM.md` (fixed bugs by month)
  - `archive/decisions/implemented-YYYY-MM.md` (implemented decisions by month)
- Update main files to remove archived items
- Show summary: "Archived 5 conversations, 12 bugs, 8 decisions"
- Keep last 10 active conversations, all active/in-progress bugs, all proposed/accepted decisions

### For `/baton switch <conv-id-or-title>`
**Switch between conversations:**
- Accept either conversation ID or title (fuzzy match)
- Update `.claude/CURRENT_CONVERSATION_ID`
- Load new conversation's SUMMARY.md
- Show transition message: "Switched from 'API Redesign' → 'Bug Fixes'"
- Display new conversation's current state

**With `--recent` flag:**
- List 5-10 most recent conversations
- Show: ID, Title, Last Modified, Status
- Allow numbered selection
- Example:
  ```
  Recent conversations:
  1. [Active] API Authentication System (modified 2h ago)
  2. [Active] Database Migration (modified 5h ago)
  3. [Paused] Payment Integration (modified 2d ago)

  Choose conversation (1-3):
  ```

### For `/baton search <term>`
**Search across all baton files:**
- Search SUMMARY.md files in all conversations
- Search BUGS.md, DECISIONS.md, ENHANCEMENTS.md, USER_FEEDBACK.md
- Return matches with context (file, conversation ID, line)
- Highlight matching text
- Sort by relevance (exact match > partial match)

**Example output:**
```
Found 'authentication' in 5 locations:

CONVERSATIONS:
  conv-20251224-015859: API Authentication System
    - Line 12: "Implementing OAuth2 authentication"

  conv-20251223-140521: User Login Refactor
    - Line 8: "Fixed authentication token expiry bug"

DECISIONS:
  Decision #3 (conv-20251224-015859)
    - "Chose OAuth2 over email/password authentication"

BUGS:
  Bug #7 (conv-20251223-140521)
    - "Authentication fails after 24 hours"
```

**With specific filters:**
- `--bugs`: Search only BUGS.md
- `--decisions`: Search only DECISIONS.md
- `--conversations`: Search only SUMMARY.md files

### For `/baton context <topic>`
**Smart context loading:**
- Semantic search across past conversations for topic
- Load relevant decisions, bugs, failed attempts
- Show context from most relevant conversations
- Example: `/baton context authentication`

**Output:**
```
Loading context for 'authentication'...

Found 3 relevant conversations:

1. conv-20251224-015859: API Authentication System (Active)
   Decisions: OAuth2 implementation, JWT tokens
   Bugs: None currently
   Status: 85% complete

2. conv-20251223-140521: User Login Refactor (Completed)
   Decisions: Session timeout 24h
   Bugs: Fixed token expiry issue
   Failed: Email/password approach (too complex)

3. conv-20251220-093012: Security Audit (Completed)
   Decisions: Require 2FA for admin accounts

Key learnings:
- Avoid email/password (maintenance overhead)
- Use OAuth2 with established providers
- JWT tokens preferred over sessions
```

### For `/baton report [timeframe]`
**Generate work summary reports:**
- Aggregate work across conversations
- Count tasks completed, bugs fixed, decisions made
- List key files modified
- Perfect for standups, status updates

**Timeframes:**
- `--today`: Today's work
- `--week`: Past 7 days
- `--month`: Past 30 days
- `--conversation <id>`: Specific conversation
- No flag: Ask for timeframe

**Example output:**
```markdown
## Work Summary: Past 7 Days (Dec 18-25, 2025)

**Active Conversations:** 3
**Completed Conversations:** 2

### Tasks Completed: 24
- API authentication system (12 tasks)
- Database migration (8 tasks)
- Bug fixes (4 tasks)

### Bugs Fixed: 5
- Authentication token expiry
- Database connection timeout
- API rate limiting error
- UI rendering glitch
- Memory leak in cron job

### Decisions Made: 8
- OAuth2 for authentication
- PostgreSQL for main database
- Redis for caching
- Next.js for frontend
- ... (4 more)

### Files Modified: 47
Top 10:
- src/auth/oauth.ts (23 changes)
- src/db/schema.sql (18 changes)
- src/api/endpoints.ts (15 changes)
... (7 more)

### Key Achievements:
- Completed API authentication system
- Migrated database schema to v2
- Reduced bug count from 12 → 7
```

### For `/baton metrics` or `/baton stats`
**Show system effectiveness:**
- Total conversations tracked
- Token savings vs full conversation logs
- Compaction survival rate
- Average context restoration time
- File sizes and growth trends

**Example output:**
```
📊 Baton System Metrics

CONVERSATIONS:
  Total: 12 (3 active, 9 completed)
  Avg Duration: 3.2 hours
  Avg Compactions: 4.2 per conversation
  Longest: conv-20251215-081234 (12 compactions)

TOKEN EFFICIENCY:
  Full Conversation Logs: ~612,000 tokens
  TLDR Summaries: ~6,200 tokens
  Compression Ratio: 98.9% (99x reduction)
  Estimated Cost Savings: $18.36 (at $0.03/1K tokens)

CONTEXT RESTORATION:
  Success Rate: 100% (12/12 compactions survived)
  Avg Restoration Time: 1.2 seconds
  Avg Tokens Read: 1,150 per restoration

FILE HEALTH:
  CONVERSATION_HISTORY.md: 2.1 KB (12 entries)
  BUGS.md: 1.8 KB (7 active, 15 archived)
  DECISIONS.md: 2.4 KB (11 active, 8 implemented)
  Archive Size: 18.2 KB (47 archived items)

RECOMMENDATIONS:
  ✅ System healthy
  ℹ️ Consider archiving (BUGS.md has 22 total entries)
```

### For `/baton validate` or `/baton health`
**Check file integrity:**
- Verify `.claude/` directory structure exists
- Check all required files present
- Validate SUMMARY.md has required sections
- Detect missing conversation IDs, broken tags
- Check for orphaned files
- Validate settings.json syntax

**Example output:**
```
🔍 Baton Health Check

DIRECTORY STRUCTURE:
  ✅ .claude/ exists
  ✅ .claude/conversations/ exists
  ✅ .claude/archive/ exists
  ✅ .claude/templates/ exists

REQUIRED FILES:
  ✅ CONVERSATION_HISTORY.md (2.1 KB)
  ✅ BUGS.md (1.8 KB)
  ✅ DECISIONS.md (2.4 KB)
  ✅ CURRENT_CONVERSATION_ID (present)
  ✅ settings.json (valid JSON)
  ⚠️ ENHANCEMENTS.md (empty)
  ⚠️ USER_FEEDBACK.md (empty)

CONVERSATIONS:
  ✅ conv-20251224-015859: All sections present
  ✅ conv-20251223-140521: All sections present
  ❌ conv-20251220-093012: Missing "Failed Attempts" section

TAGS:
  ✅ All BUGS.md entries have Conv: tags
  ⚠️ 2 DECISIONS.md entries missing Conv: tags

ISSUES FOUND: 3
  [1] conv-20251220-093012/SUMMARY.md missing "Failed Attempts"
  [2] DECISIONS.md line 45 missing Conv: tag
  [3] DECISIONS.md line 67 missing Conv: tag

FIX COMMAND:
  /baton fix --auto   # Auto-fix common issues
```

### For `/baton auto-save on|off|status`
**Configure automatic saves:**
- `on`: Enable auto-save at configured thresholds
- `off`: Disable auto-save
- `status`: Show current configuration

**Auto-save behavior:**
- Monitor token usage continuously
- Save at thresholds: 70%, 85%, 95% (configurable)
- Show notification: "🔁 Auto-saved at 72% token usage"
- Never interrupt user work
- Settings stored in `.claude/settings.json`

**Example output:**
```
Auto-Save Configuration:
  Status: ✅ Enabled
  Thresholds: 70%, 85%, 95%
  Notify on save: Yes
  Last auto-save: 15 minutes ago (at 71%)

Current token usage: 68% (save at 70%)
```

### For `/baton template create <name>`
**Create custom templates:**
- Copy current SUMMARY.md structure
- Save to `.claude/templates/<name>.md`
- Allow editing/customization
- Templates stored for reuse

**Example:**
```bash
/baton template create research

Created template: research
Location: .claude/templates/research.md

Customize with additional sections:
- Research Questions
- Literature Review
- Methodology
- Data Sources
```

### For `/baton template use <name>`
**Switch templates:**
- Load template from `.claude/templates/<name>.md`
- Apply to new conversations
- Existing conversations keep their template

### For `/baton template list`
**Show available templates:**
```
Available Templates:
  ✓ standard (default) - Standard development template
  • research - Academic research template
  • devops - Infrastructure/deployment template
  • bugfix - Bug investigation template

Current conversation template: standard
```

### For `/baton git-link`
**Git integration:**
- Associate current conversation with git branch
- Store mapping in `.claude/git_links.json`
- Add conversation ID to commit template
- Tag commits with conversation context

**Example:**
```bash
/baton git-link

✅ Linked conversation to git:
   Conv: conv-20251224-015859
   Branch: feature/oauth-authentication

Commits on this branch will be tagged with conversation ID.
Use /baton git-summary to generate commit message.
```

### For `/baton git-summary`
**Generate commit message from SUMMARY:**
- Extract Task Checklist completed items
- Summarize key changes
- Include conversation ID for traceability

**Example output:**
```
feat: implement OAuth2 authentication system

- Add OAuth2 provider integration (Google, GitHub)
- Create JWT token generation and validation
- Implement user session management
- Add authentication middleware
- Write integration tests for auth flow

Related to conversation: conv-20251224-015859
```

## Problem Solved

During long autonomous sessions, Claude Code can go through 5-10 auto-compactions, losing critical technical details:
- Bug reproduction steps
- Failed approaches (leads to retry loops)
- Architecture decisions and rationale
- Exact file locations and current state
- What was tried and why it didn't work

## Solution

Two-tier context system with intelligent automation:
- **Tier 1 (TLDR)**: ~1,000 tokens - Always read after compaction
- **Tier 2 (Full details)**: ~50,000 tokens - Read on-demand only
- **Compression**: 25-100x token reduction
- **Auto-save**: Never lose work at compaction boundaries
- **Search**: Find past context instantly
- **Smart updates**: Reduce manual overhead

## Commands Reference

### Core Commands

**`/baton init`**
Initialize context management for new conversation

**`/baton load`** or **`/baton`**
Display current conversation TLDR

**`/baton save [note]`**
Manually save current state

**`/baton update [section]`** ⭐ NEW
Auto-update SUMMARY.md from recent activity

**`/baton rename <title>`**
Set conversation title (max 60 chars)

**`/baton rename --suggest`** ⭐ NEW
Get AI-generated title suggestions

**`/baton history`**
Show all conversations and status

**`/baton status`**
Check token usage, get save recommendations

**`/baton archive`**
Archive completed items to prevent bloat

---

### Navigation

**`/baton switch <conv-id-or-title>`** ⭐ NEW
Switch between conversations

**`/baton switch --recent`** ⭐ NEW
Show recent conversations menu

---

### Search & Discovery

**`/baton search <term>`** ⭐ NEW
Search across conversations, bugs, decisions

**`/baton search --bugs <term>`** ⭐ NEW
Search only bugs

**`/baton search --decisions <term>`** ⭐ NEW
Search only decisions

**`/baton context <topic>`** ⭐ NEW
Load relevant past context (smart search)

---

### Reporting

**`/baton report [timeframe]`** ⭐ NEW
Generate work summary report

**`/baton metrics`** ⭐ NEW
Show system effectiveness metrics

**`/baton stats`** ⭐ NEW
Alias for metrics

---

### Validation

**`/baton validate`** ⭐ NEW
Check file structure integrity

**`/baton health`** ⭐ NEW
Alias for validate

---

### Configuration

**`/baton auto-save on|off|status`** ⭐ NEW
Configure automatic saves

**`/baton template create <name>`** ⭐ NEW
Create custom SUMMARY.md template

**`/baton template use <name>`** ⭐ NEW
Switch to custom template

**`/baton template list`** ⭐ NEW
Show available templates

---

### Git Integration

**`/baton git-link`** ⭐ NEW
Associate conversation with git branch

**`/baton git-summary`** ⭐ NEW
Generate commit message from SUMMARY

---

## ENHANCEMENTS.md - Future Ideas Tracking

Track potential improvements and feature ideas:

```markdown
## Enhancement #1: Add Dark Mode
**Conv:** conv-20251224-015859
**Proposed:** 2025-12-24
**Priority:** High | Medium | Low
**Status:** Proposed | Accepted | In Progress | Implemented | Rejected
**Rationale:** Users requested dark mode for nighttime viewing
**Impact:** Affects all UI components, requires theme system
**Effort:** ~2 days
**Dependencies:** None
**Proposed By:** User | Claude
**Notes:** Consider CSS variables for easy theming
```

**When to add:**
- User mentions "we should..." or "it would be nice if..."
- Claude identifies optimization opportunities
- Discussing future improvements
- Brainstorming features

**Status progression:**
- Proposed → Accepted → In Progress → Implemented
- Or: Proposed → Rejected (with reason)

---

## USER_FEEDBACK.md - Questions Waiting for User

Critical for long autonomous sessions when user is away:

```markdown
## Feedback Request #1: Authentication Method
**Conv:** conv-20251224-015859
**Asked:** 2025-12-24 03:15
**Risk Level:** High | Medium | Low
**Status:** Pending | Auto-Decided | Answered | No Longer Needed
**Context:** Building user authentication system
**Question:** Should we use OAuth2 or email/password auth?
**Options:**
1. OAuth2 (Google + GitHub) - Easier for users, harder to implement
2. Email/Password - Traditional, requires password reset flow
3. Both - Best UX, most complex
**Blockers:** Can't proceed with auth implementation until decided
**Workaround:** Working on other features in the meantime
**Recommendation:** OAuth2 (most common for modern apps, aligns with best practices)
**Auto-Decision:** [If Low risk, Claude fills this in with choice made]
**User Validation:** [User confirms or requests redo]
```

**Risk Assessment Guide:**

**High Risk** - Block and wait for user (COSTS MONEY or irreversible):
- **Costs money**: Cloud provider choice (AWS vs GCP), paid services, API pricing
- **Expensive to redo**: 3+ days of rework if wrong choice
- Architecture decisions affecting entire system
- Breaking changes or data migration
- Security-critical choices (authentication, authorization, encryption)
- Irreversible decisions (data deletion, production deployments)
- User explicitly requested input on this type of decision
- **RULE: If choosing wrong costs money or >2 days rework → HIGH RISK**

**Medium Risk** - Auto-decide, document, validate (easy to redo):
- Framework/library choices (can swap in <1 day)
- UI/UX decisions (can redesign easily)
- Non-critical performance tradeoffs
- Feature prioritization
- **RULE: If wrong choice costs <1 day to fix → MEDIUM RISK**

**Low Risk** - Auto-decide, implement, validate later (trivial to redo):
- Variable naming conventions (<1 hour to refactor)
- File organization (can reorganize easily)
- Minor styling choices (quick CSS changes)
- Error message wording (find/replace)
- Default values (config change)
- Logging verbosity (config change)
- **RULE: If wrong choice costs <1 hour to fix → LOW RISK**

**When to add:**
- **High Risk**: Add to USER_FEEDBACK.md, block work on that feature
- **Medium/Low Risk**: Make decision, document in DECISIONS.md with status="Auto-Decided"

**Auto-Decision Workflow (Low/Medium Risk):**
1. Claude makes best-guess decision
2. Implements the feature
3. Documents in DECISIONS.md:
   ```markdown
   **Status:** Auto-Decided (Pending User Validation)
   **Chosen:** OAuth2
   **Rationale:** Modern apps prefer OAuth, easier UX
   **Alternatives:** Email/password, Both
   **Risk:** Low - Can swap auth provider without data loss
   **Redo Effort:** ~2 hours if user prefers different approach
   ```
4. At next session start, ask user: "I chose OAuth2 for auth (modern standard). Alternatives were email/password or both. Okay with this?"
5. User validates (70% chance) or requests redo (30% chance)

**Critical behavior:**
- **ALWAYS check USER_FEEDBACK.md at start of session**
- **Surface pending HIGH-RISK questions immediately**
- **For Auto-Decided items: Ask user for validation**
- **Be ready to redo work if user disagrees (~30% of time)**
- **Mark as "Validated" once user confirms**

---

## Auto-Behavior (No Command Needed)

Claude automatically follows these behaviors if `.claude/` directory exists:

**On Session Start:**
1. Check for `.claude/CURRENT_CONVERSATION_ID`
2. Read `CONVERSATION_HISTORY.md` (all conversations overview)
3. Read `conversations/{conv-id}/SUMMARY.md` (this conversation's TLDR)
4. Read `BUGS.md` and `DECISIONS.md` (filtered to this conversation)
5. **Read `USER_FEEDBACK.md` - Surface any pending questions immediately**
6. **Read `settings.json` - Load auto-save configuration**
7. Total: ~1,000-1,500 tokens for full context restoration

**During Work:**
- **Use standardized response format** (see CLAUDE.md) with Title, Request, Tasks, Summary
- **Monitor token usage for auto-save triggers**
- Update SUMMARY.md after significant actions
- Update conversation Title via `/baton rename` when conversation focus shifts
- Append to BUGS.md when discovering bugs
- Append to DECISIONS.md when making architecture choices
- **Append to ENHANCEMENTS.md when ideas are discussed**
- **Append to USER_FEEDBACK.md when user input needed but user away**
- Update CONVERSATION_HISTORY.md on major milestones
- **Auto-save at 70%, 85%, 95% token thresholds if enabled**

**After Compaction:**
- IMMEDIATELY run `/baton load` equivalent automatically
- Restore from TLDR (~1K tokens instead of 50K+ full log)
- Resume work with full context

**Auto-Save Triggers:**
- At 70% token usage: First checkpoint
- At 85% token usage: Second checkpoint
- At 95% token usage: Final checkpoint before compaction
- Show notification: "🔁 Auto-saved at 72% token usage"

---

## File Structure

```
.claude/
├── CONVERSATION_HISTORY.md          # All conversations (~200 tokens)
├── BUGS.md                          # All bugs, tagged with conv-id
├── DECISIONS.md                     # All decisions, tagged with conv-id
├── ENHANCEMENTS.md                  # Future enhancement ideas
├── USER_FEEDBACK.md                 # Questions waiting for user input
├── CURRENT_CONVERSATION_ID          # Current conversation ID
├── settings.json                    # Baton configuration
├── git_links.json                   # Git branch associations
├── conversations/
│   └── {conv-id}/
│       └── SUMMARY.md               # This conversation TLDR (~300 tokens)
├── templates/
│   ├── standard.md                  # Default template
│   ├── research.md                  # Research project template
│   └── devops.md                    # DevOps/infrastructure template
└── archive/
    ├── conversations/
    │   └── YYYY-MM.md               # Archived conversations by month
    ├── bugs/
    │   └── fixed-YYYY-MM.md         # Fixed bugs by month
    └── decisions/
        └── implemented-YYYY-MM.md   # Implemented decisions by month
```

---

## settings.json Configuration

```json
{
  "autoSave": {
    "enabled": true,
    "thresholds": [70, 85, 95],
    "notifyOnSave": true
  },
  "archiveThresholds": {
    "conversations": 10,
    "bugs": 20,
    "decisions": 15
  },
  "defaultTemplate": "standard",
  "gitIntegration": {
    "enabled": true,
    "tagCommits": true,
    "addConvIdToMessage": true
  },
  "search": {
    "caseSensitive": false,
    "maxResults": 50
  }
}
```

---

## SUMMARY.md Format

```markdown
# Conversation {conv-id} - TLDR

**Title:** [Brief conversation goal, set via /baton rename, max 60 chars]
**Status:** Active | Completed | Paused
**Started:** YYYY-MM-DD HH:MM
**Duration:** Xh
**Compactions:** N

## Context in 3 Lines
[High-level overview of what's happening]

## Task Checklist
- [x] Completed task
- [ ] Pending task

## Decisions Made
- Decision #N: What was decided and why

## Key Files Created/Modified
- path/to/file.ts (what changed)

## Failed Attempts (Don't Retry)
- Approach X: Why it failed

## Next Actions
1. First priority
2. Second priority

## State Snapshot
**Current file:** exact/path.ts
**Current line:** 42
**Current task:** Specific thing being worked on
**Blockers:** Any blockers
**Ready to:** Next immediate action
```

---

## Conversation ID Tagging

When adding to shared files (BUGS.md, DECISIONS.md):

```markdown
**Conv:** conv-20251223-225929
```

This enables:
- Multiple conversations working simultaneously
- Each conversation identifying their work
- Shared awareness across conversations
- No file conflicts

---

## Integration with CLAUDE.md

Add this to your project's CLAUDE.md:

```markdown
## Context Management Protocol

This project uses `.claude/` context management system.

**After compaction:** Automatically reads TLDR summaries
**Manual control:** Use `/baton` skill commands
**Token efficiency:** 25-100x compression (50K→1K tokens)
**Auto-save:** Enabled at 70%, 85%, 95% thresholds
```

See full protocol in CLAUDE.md for details.

---

## Portability

To use across multiple projects:

1. **Option A - Local per project:**
   ```bash
   cp -r .claude-code/skills/baton /path/to/other/project/.claude-code/skills/
   ```

2. **Option B - Global symlink:**
   ```bash
   # Move to central location
   mv .claude-code/skills/baton /mnt/foundry_project/Claude_skills/

   # Symlink from ~/.claude/skills/
   ln -s /mnt/foundry_project/Claude_skills/baton ~/.claude/skills/baton
   ```

3. **Option C - Copy to central location:**
   ```bash
   cp -r .claude-code/skills/baton /mnt/foundry_project/Claude_skills/
   ln -s /mnt/foundry_project/Claude_skills/baton ~/.claude/skills/baton
   ```

---

## Token Efficiency

- Full conversation log: 50,000+ tokens
- TLDR summary: 500-2,000 tokens
- Compression ratio: 25-100x
- Post-compaction read: ~1,000-1,500 tokens total
- Auto-save overhead: ~50 tokens per save
- Enables: Long autonomous sessions without context loss

---

## Benefits

✅ **For humans:**
- Quick scan of what happened
- Thread view of conversations
- Easy navigation
- Instant search across all work
- Weekly/monthly reports for standups

✅ **For Claude:**
- Efficient context restoration
- No retry of failed approaches
- Preserves technical details
- Enables multi-conversation work
- Auto-saves prevent context loss

✅ **For projects:**
- Portable across projects
- Standardized context management
- Reduced token costs (98%+ reduction)
- Better long-running session support
- Git integration for traceability

---

## Version

**Baton v2.0** - Enhanced with 11 new features (2025-12-25)

**New in v2.0:**
1. Auto-save triggers (prevent context loss)
2. Conversation switching (multi-conversation workflow)
3. Smart SUMMARY.md updates (reduce manual overhead)
4. Search across conversations (find past context)
5. Title auto-suggestions (AI-powered naming)
6. Validation & health check (data integrity)
7. Export & reporting (status summaries)
8. Conversation metrics (prove system value)
9. Smart context loading (relevance-based)
10. Git integration (branch association, commit messages)
11. Template customization (project-specific needs)
