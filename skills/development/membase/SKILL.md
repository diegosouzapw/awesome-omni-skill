---
name: membase
description: "Git-native project memory system for storing and retrieving project decisions, implementation details, and documentation across coding sessions. Use when starting work on a topic, making design decisions, implementing features, or debugging issues that might have prior context. Claude should proactively query memories before implementing features or making architectural decisions to maintain consistency."
---

# Membase: Git-Native Project Memory

**TL;DR:** Store project decisions with `scripts/mb add`, retrieve with `scripts/mb query`, maintain consistency across sessions.

Membase is a git-native project memory system that stores project knowledge in simple text files (`.membase/`). It uses multi-dimensional tagging (topic + phase) to organize information and enables efficient context retrieval across sessions.

**Core principle:** Store decisions and context so they're never lost between sessions. Query before implementing to avoid contradicting earlier decisions.

## Quick Start

```bash
# Initialize membase
scripts/mb init

# Add a topic for your project
scripts/mb dims add topic authentication

# Store a decision
scripts/mb add \
  -s "JWT auth with 24h expiry" \
  -c "Using JWT for stateless authentication. Tokens expire in 24h. Refresh tokens valid 7 days." \
  topic=authentication phase=decision

# Query before implementing
scripts/mb query --topic authentication --phase decision
```

## When to Use Membase

### Store memories when:
- **Design decisions are made** (architecture, technology choices, patterns)
- **Implementation approaches are chosen** (algorithms, data structures)
- **Gotchas are discovered** (edge cases, framework quirks, workarounds)
- **User states preferences** (requirements, coding style)
- **Key patterns are established** (file locations, naming conventions)

### Retrieve memories when:
- **Starting work on a topic** (check existing decisions first)
- **User asks about previous decisions** ("What did we decide about X?")
- **Implementing with potential context** (maintain consistency)
- **Debugging or troubleshooting** (check for known issues)

## Multi-Dimensional Tagging

Organize memories with two dimensions:

**topic** - Project-specific areas you define:
```bash
scripts/mb dims add topic authentication
scripts/mb dims add topic database
scripts/mb dims add topic api
```

**phase** - Workflow stage (pre-populated):
- `decision` - Architectural/design decisions ⭐ **Most important**
- `backend-implementation` - Backend code details
- `frontend-implementation` - Frontend code details
- `troubleshooting` - Known issues and solutions
- `planning`, `requirements`, `testing`, `documentation`, `deployment`

## Common Patterns

**Record a decision:**
```bash
scripts/mb add \
  -s "PostgreSQL for main database" \
  -c "Chose PostgreSQL over MySQL for JSON support. Using SQLAlchemy ORM." \
  topic=database phase=decision
```

**Query before implementing:**
```bash
scripts/mb query --topic database --phase decision
scripts/mb query --search "PostgreSQL"
```

**Document implementation:**
```bash
scripts/mb add \
  -s "User model in src/models/user.py" \
  -c "Fields: username, email, password_hash. Email is unique. Bcrypt with 12 rounds." \
  topic=authentication phase=backend-implementation
```

**Capture gotchas:**
```bash
scripts/mb add \
  -s "Redis rate limiting memory leak fix" \
  -c "IMPORTANT: Must call redis.expire() after incrementing counter to prevent leak." \
  topic=api phase=troubleshooting
```

## Best Practices

**Writing summaries** (≤100 chars):
- ✓ Specific: "JWT auth with 24h expiry and refresh tokens"
- ✗ Vague: "Authentication stuff"

**Writing content**:
- Include the "why" behind decisions
- Mention file paths and locations
- Note alternatives considered
- Add specific configuration values

**Tagging consistently**:
```bash
topic=authentication phase=decision         # Design decision
topic=authentication phase=backend-implementation  # Implementation
topic=authentication phase=troubleshooting  # Problem/solution
```

## Claude Code Integration

**For Claude:** Proactively use membase by:

1. **Query before implementing** - Check `scripts/mb query --topic <area> --phase decision`
2. **Store after deciding** - Suggest storing significant decisions
3. **Maintain consistency** - Reference stored decisions (mention memory ID)
4. **Update when needed** - Suggest edits if decisions change

**Example workflow:**
```
User: "Let's add user authentication"
Claude: "Let me check existing decisions..."
[Runs: scripts/mb query --topic authentication --phase decision]
Claude: "Found JWT auth decision (memory abc12345). I'll follow that pattern..."
```

## Essential Commands

```bash
# Initialize
scripts/mb init

# Add memory
scripts/mb add -s "summary" -c "content" topic=X phase=Y

# Query
scripts/mb query --topic X --phase Y
scripts/mb query --search "keyword"
scripts/mb query --id abc123
scripts/mb query --all

# Manage dimensions
scripts/mb dims                    # List all
scripts/mb dims add topic <name>   # Add topic value

# Stats
scripts/mb stats
```

## Advanced Usage

For detailed examples, advanced commands, workflow integration, and tips, see [REFERENCE.md](REFERENCE.md).

---

**Remember:** Query before implementing, store after deciding. Membase keeps context alive across sessions.
