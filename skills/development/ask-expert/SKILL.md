---
name: ask-expert
description: Creates expert consultation documents with code extraction, git diffs, and size tracking (125KB limit). Use when user asks to "create an expert consultation document", "prepare code for expert review", "gather architecture context", or needs comprehensive technical documentation for external analysis. Requires Node.js 18+.
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(node:*)
  - Bash(git:*)
  - Bash(wc:*)
  - Bash(ls:*)
  - Bash(cat:*)
user-invocable: true
context: fork
---

# Expert Consultation Document Creator

Create comprehensive technical consultation documents by extracting code, diffs, and architectural context within LLM token limits (125KB).

## Document Structure

Follow this proven structure:

### Part 1: Problem Context (~15-25 KB)
1. **Problem** - Issue, errors, test failures
2. **Our Solution** - What was implemented and why
3. **Concerns** - Code smells, coupling, architectural questions
4. **Alternatives** - Other approaches, trade-offs

### Part 2: Complete Architecture (~60-90 KB)
5. **Architecture Overview** - ASCII diagram, data flow, patterns
6. **Components** - Frontend, tests, controllers
7. **Services** - Implementation and interfaces
8. **Models** - Domain entities with relationships

### Part 3: Expert Request (~5-10 KB)
9. **Questions** - Specific technical questions
10. **Success Criteria** - Requirements and priorities

## Workflow

### Step 1: Write Problem Context

Create descriptive filename like `{topic}-consultation.md`:

```bash
cat > feature-consultation.md << 'EOF'
# Expert Consultation: [Feature Name]

## 1. Problem
[Describe the issue]

## 2. Our Solution
[What was implemented]

## 3. Concerns
[Technical concerns]

## 4. Alternatives
[Other approaches considered]

## 5. Architecture Overview
[ASCII diagram]

---
# Complete Architecture Context
EOF
```

### Step 2: Extract Code

Use the bundled extraction script with size tracking.

**💡 The script accepts multiple files in one call** - batch files for efficiency:

```bash
node scripts/extract-code.js \
  --track-size --output=doc.md \
  --section="Core Files" \
  file1.ts file2.ts file3.ts \
  --section="Tests" \
  test1.ts test2.ts
```

**File format options:**
- Full file: `src/Service.cs`
- Line ranges: `src/Service.cs:100-200` or `src/Service.cs:1-30,100-150`
- Git diff: `src/Service.cs:diff` or `src/Service.cs:diff=master..HEAD`

**Prefer FULL files over chunks** for better expert analysis. Use chunks only for very large files.

### Step 3: Add Expert Request

```bash
cat >> consultation.md << 'EOF'

---
# Expert Guidance Request

## Questions
1. [Specific question about architecture]
2. [Question about trade-offs]
3. [Question about refactoring approach]

## Success Criteria
- [Required constraints]
- [Priorities]

**Please answer in English**
EOF
```

### Step 4: Verify Size

```bash
wc -c consultation.md  # Should be 100-125 KB
```

DO NOT read the full file back (exceeds context).

## Code Extraction Examples

See [EXAMPLES.md](EXAMPLES.md) for detailed usage patterns.

**Basic extraction:**
```bash
node scripts/extract-code.js \
  --track-size --output=doc.md \
  src/Component.vue tests/Component.test.ts
```

**With sections:**
```bash
node scripts/extract-code.js \
  --track-size --output=doc.md \
  --section="What Changed" \
  src/Service.cs:diff \
  --section="Implementation" \
  src/Service.cs src/Model.cs
```

**Using config file:**
```bash
node scripts/extract-code.js \
  --config=extraction-plan.json
```

## Config File Format

Create reusable extraction plans:

```json
{
  "output": "consultation.md",
  "trackSize": true,
  "sections": [
    {
      "header": "What Changed",
      "files": ["src/Service.cs:diff"]
    },
    {
      "header": "Core Implementation",
      "files": ["src/Service.cs", "src/Model.cs"]
    }
  ]
}
```

See `scripts/extract-code-example.json` for complete example.

## Critical Rules

- ✅ Use `--track-size` to stay within 125 KB
- ✅ Batch multiple files in single command
- ✅ Use absolute path to script from any directory
- ✅ Include FULL files when possible
- ✅ Add architecture diagrams
- ✅ Include working AND failing tests
- ❌ Don't read completed file back
- ❌ Don't send only bug fix without context

## Troubleshooting

**Script not found:**
```bash
# Verify script exists
ls scripts/extract-code.js

# Show help
node scripts/extract-code.js --help
```

**Git diff errors:**
```bash
git status              # Verify git repo
git rev-parse master    # Verify branch exists
```

**Exceeding 125 KB:**
- Use line ranges instead of full files for large services
- Remove boilerplate and simple DTOs
- Focus on core interfaces and modified code
- Split into multiple consultations

## Code Inclusion Priority

**Must include:**
- Core interfaces/abstractions
- Modified/bug-fix code
- Domain models
- Key service methods
- Test examples

**Skip if tight on space:**
- Boilerplate
- Simple DTOs
- Repetitive test setups
