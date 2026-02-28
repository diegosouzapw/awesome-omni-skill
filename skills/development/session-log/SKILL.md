---
name: session-log
description: 'Create structured session logs for AI coding/chat sessions. Use when starting a new development session, tracking decisions and context, documenting troubleshooting, or preserving conversation history for future reference.'
---

# Session Log Skill

This skill teaches agents to create well-structured session logs that capture the context, decisions, and outcomes of AI-assisted development sessions. Session logs serve as breadcrumbs for future developers (including future you) to understand *why* decisions were made.

## When to Use This Skill

Use this skill when:
- Starting a new coding/chat session with specific goals
- Making architectural or design decisions that need documentation
- Troubleshooting complex issues with multiple attempted solutions
- Learning something new that should be preserved for reference
- Wrapping up a session and wanting to capture what was accomplished
- Creating audit trails for important changes

## Core Principles

**Session logs are NOT application logs**: They capture human-AI collaboration context, not system events. Think "meeting notes" not "server logs".

**Focus on "why" not "what"**: Code diffs show *what* changed. Session logs explain *why* it changed, what alternatives were considered, and what was learned.

**Structure enables searchability**: Consistent format makes logs searchable and parseable. Use markdown + frontmatter for maximum compatibility.

**Context is king**: Future readers lack your current context. Include enough detail to reconstruct your thought process months later.

## Session Log File Structure

### Directory Location

Create session logs in: `.github/sessions/`

**Why this location?**
- `.github/` - Already used for repo metadata (workflows, agents, skills)
- `sessions/` - Clear purpose, separate from other .github/ content
- Included in git - Session logs are documentation, not secrets
- Searchable - Can grep across all sessions for patterns

### Filename Format

Use ISO 8601 timestamp format: `YYYY-MM-DDTHH-MM-SSZ.md`

**Examples**:
- `2026-02-08T15-30-00Z.md` (February 8, 2026, 3:30 PM UTC)
- `2026-02-08T09-15-30Z.md` (February 8, 2026, 9:15:30 AM UTC)

**Why this format?**
- **Sortable**: Alphabetical sorting = chronological order
- **Unambiguous**: ISO 8601 is universally understood
- **Collision-resistant**: Down-to-second precision prevents overwrites
- **Tool-friendly**: No spaces or special chars, easy to parse

**Generate timestamp** (in bash):
```bash
date -u +"%Y-%m-%dT%H-%M-%SZ"
```

### File Format Template

```markdown
---
session_id: <unique-identifier>
started: <ISO-8601-timestamp>
ended: <ISO-8601-timestamp-or-ongoing>
participants: [human, ai-agent-name]
tags: [feature-name, bug-fix, refactoring, learning, etc]
status: ongoing | completed | abandoned
---

# Session: <Brief descriptive title>

## Goal

<What you're trying to accomplish in 1-3 sentences>

## Context

### Starting State
- **Current situation**: <What exists now>
- **Problem/need**: <What prompted this session>
- **Prior attempts**: <What's been tried before, if any>

### Environment
- **Branch**: <git branch name>
- **Tools**: <Relevant tools/frameworks/versions>
- **Files involved**: <Key files to be modified/created>

## Conversation Log

### <Timestamp> - <Topic/Phase>

**Discussion**:
<What was discussed, questions asked, alternatives considered>

**Decision**:
<What was decided and why>

**Action**:
<What was done (code written, commands run, files created)>

**Outcome**:
<What happened as a result>

---

*(Repeat this pattern for each major phase of the session)*

## Key Decisions

### Decision 1: <Brief title>
- **Question**: <What needed deciding>
- **Options**: <What alternatives were considered>
- **Choice**: <What was chosen>
- **Rationale**: <Why this option was best>
- **Trade-offs**: <What was sacrificed>

*(Repeat for each major decision)*

## Challenges & Solutions

### Challenge 1: <Brief description>
- **Problem**: <What went wrong>
- **Attempted solutions**: <What didn't work>
- **Final solution**: <What worked>
- **Lesson learned**: <Key takeaway>

## Artifacts Created

- [ ] `path/to/file.ext` - <Purpose/description>
- [ ] `another/file.ext` - <Purpose/description>
- [ ] Agent/skill: `.github/agents/name.md` - <Purpose>

## Next Steps

- [ ] <Immediate follow-up task>
- [ ] <Testing needed>
- [ ] <Documentation to update>
- [ ] <Future improvements to consider>

## References

- Related sessions: `YYYY-MM-DDTHH-MM-SSZ.md`
- Issues/PRs: #123, #456
- Related Commits: [If branch information and commit history is readily available via git]
- Documentation: [Link to relevant docs]
- External resources: [Helpful articles/guides]

## Summary

<2-3 sentence summary of what was accomplished and learned>

---

**Total session time**: <X hours Y minutes>  
**Status**: <completed | ongoing | abandoned>  
**Outcome**: <success | partial | blocked>
```

## Session Log Best Practices

### DO ✅

**Include session metadata** in frontmatter:
- Unique session ID (UUID or timestamp)
- Start/end timestamps
- Participants (human username, AI agent name)
- Status tracking (ongoing/completed/abandoned)
- Tags for categorization

**Capture decision context**:
- What options were considered
- Why each option was rejected
- Trade-offs made
- Assumptions that influenced the decision

**Log errors and failures**:
- What didn't work
- Why it failed
- How you debugged it
- What you learned

**Track time spent**:
- Session duration
- Time on different phases
- Helps estimate future work

**Link related artifacts**:
- Code files created/modified
- PRs/issues opened
- Related session logs
- External documentation

**Write for future you**:
- Assume you'll forget all context in 6 months
- Explain acronyms and project-specific terms
- Include enough detail to reproduce decisions

**Update logs in real-time**:
- Don't wait until the end of the session
- Capture thoughts while fresh
- It's easier than reconstructing later

### DON'T ❌

**Don't log code directly**:
- Link to files/commits instead
- Code belongs in version control, not logs
- Exception: Small snippets (< 10 lines) to illustrate a point

**Don't log sensitive data**:
- No API keys, passwords, tokens
- No PII (personally identifiable information)
- No customer data
- Use placeholders: `<REDACTED>`, `<API_KEY>`

**Don't duplicate what git already tracks**:
- Commit messages explain what changed
- Session logs explain why and how you decided

**Don't write novels**:
- Be concise but complete
- Bullet points over paragraphs
- Focus on decisions and outcomes, not play-by-play

**Don't skip the "why"**:
- "Added feature X" is useless
- "Added feature X because Y users requested it and Z data showed need" is valuable

## Structured Logging for AI Sessions

When logging AI-assisted sessions specifically:

### Conversation Tracking

**Track the flow of ideas**:
```markdown
### 14:30 - Initial approach discussion

**Human**: How should we structure the authentication system?

**AI suggested**: Three approaches:
1. JWT with Redis sessions (fast, scalable)
2. Traditional server-side sessions (simple, proven)
3. Stateless JWT-only (minimal infrastructure)

**Human concern**: Redis adds operational complexity

**AI recommendation**: Option 2 for MVP, migrate to Option 1 when scale demands

**Decision**: Server-side sessions with cookie storage
**Rationale**: Prioritizing simplicity for initial launch, can optimize later

**Implementation**:
- Created `auth/session-manager.ts`
- Added session middleware to Express app
- Configured secure cookie settings
```

### Tool/MCP Usage

**Log tool invocations when they matter**:
```markdown
### 15:45 - Debugging deployment failure

**Command run**: `kubectl get pods -n production`
**Output**: 3/5 pods in CrashLoopBackOff
**Inference**: Recent config change broke pod startup

**Command run**: `kubectl logs <pod-name>`
**Key error**: `ECONNREFUSED connecting to Redis`
**Root cause**: Redis connection string format changed in v2.0

**Fix applied**: Updated `REDIS_URL` format in secrets
**Verification**: All pods healthy after rollout
```

### Learning Capture

**Document insights gained**:
```markdown
## Key Learnings

### TypeScript Generic Constraints
**Context**: Struggled with typing a utility function
**Discovery**: Can use `extends` to constrain generic types
**Example**: `function process<T extends Identifiable>(item: T)` 
**Source**: [TypeScript Handbook - Generics](https://...)
**Apply to**: Similar utility functions across codebase

### Docker Build Optimization
**Problem**: 8-minute build times
**Solution**: Multi-stage builds + layer caching
**Result**: Down to 90 seconds
**Key insight**: Order Dockerfile commands by change frequency
```

## Session Categories & Tags

Use consistent tags for easier searching:

**By Activity Type**:
- `feature-development` - Building new functionality
- `bug-fix` - Resolving defects
- `refactoring` - Restructuring existing code
- `debugging` - Investigating issues
- `learning` - Educational/exploratory session
- `architecture` - System design decisions
- `code-review` - Reviewing changes
- `documentation` - Writing/updating docs
- `deployment` - Release and operations work

**By Domain**:
- `frontend`, `backend`, `infrastructure`, `database`
- `authentication`, `payments`, `notifications`
- `performance`, `security`, `testing`

**By Outcome**:
- `completed` - Goals achieved
- `partial` - Some progress made
- `blocked` - Hit a blocker
- `research` - Gathered information

## Automation Helpers

### Quick Session Start

Create a helper function in your shell:

```bash
# Add to ~/.zshrc or ~/.bashrc
new-session() {
  local session_dir=".github/sessions"
  local timestamp=$(date -u +"%Y-%m-%dT%H-%M-%SZ")
  local session_file="$session_dir/$timestamp.md"
  
  mkdir -p "$session_dir"
  
  cat > "$session_file" << EOF
---
session_id: $timestamp
started: $(date -u +"%Y-%m-%dT%H:%M:%SZ")
ended: ongoing
participants: [human, copilot]
tags: []
status: ongoing
---

# Session: <Title>

## Goal

<What you're trying to accomplish>

## Context

### Starting State
- **Current situation**: 
- **Problem/need**: 
- **Prior attempts**: 

### Environment
- **Branch**: 
- **Tools**: 
- **Files involved**: 

## Conversation Log

### $(date -u +"%H:%M") - Initial setup

**Discussion**:

**Decision**:

**Action**:

**Outcome**:

EOF

  echo "Created session log: $session_file"
  ${EDITOR:-code} "$session_file"  # Or your editor of choice
}
```

### Session Summary Generator

Ask your AI agent to generate a summary:

```
@agent Generate a summary of this session based on our conversation. Include:
- What we were trying to accomplish
- Key decisions made and why
- What was built/changed
- Any blockers or open questions
- Next steps

Format it as a session log entry using the session-log skill template.
```

## Searching Session Logs

**Find sessions by tag**:
```bash
grep -r "tags:.*authentication" .github/sessions/
```

**Find sessions working on specific files**:
```bash
grep -r "auth/session-manager.ts" .github/sessions/
```

**Find all debugging sessions**:
```bash
grep -r "status: completed" .github/sessions/ | grep "debugging"
```

**List recent sessions**:
```bash
ls -lt .github/sessions/ | head -10
```

## Integration with Agents

When creating a session-log-aware agent:

```markdown
---
name: session-logger
description: Helps create and maintain session logs for development work
tools: ['edit', 'read']
---

You are a session logging specialist. When users start or wrap up sessions:

1. **Session start**: Create a new session log file using the template
2. **During session**: Help update the log with decisions, challenges, learnings
3. **Session end**: Generate summary, mark status as completed, calculate duration
4. **Session search**: Help find relevant past sessions by tag, file, or topic

Always use the session-log skill's template and best practices.
```

## Remember

Session logs are **investments in future productivity**. They take 5-10 minutes to maintain but save hours of "why did we do this?" confusion later.

The goal isn't perfect documentation - it's **just enough context** to make future work faster and better informed.