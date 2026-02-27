---
name: PRD-README v1 Standard
description: This skill should be used when the user asks to "write a readme", "create readme", "generate readme", "improve readme", "audit readme", "review readme", "fix readme", "readme best practices", "readme standard", "perfect readme", or mentions README quality, documentation standards, or developer experience documentation.
version: 0.1.0
---

# PRD-README v1 Standard

Product-Ready Documentation README standard for creating documentation that reliably produces correct adoption decisions, fast first success, low support burden, sustained trust, and low drift.

## Core Definition

A README is "perfect" when it achieves:

1. **Correct adoption decision** - Reader quickly knows if project fits their needs
2. **Fast first success** - Copy/paste path to working result in under 5 minutes
3. **Low support burden** - Answers top questions upfront
4. **Sustained trust** - Clear status and quality signals
5. **Low drift** - Stays accurate via process/automation

## The 9-Step Build Process

Follow these steps in order when creating or improving a README.

### Step 0: Specify Target Reader and Job-to-be-Done

Before writing any README content, define these three lines:

```
Primary user: [e.g., "Python developer who wants an HTTP client library"]
Primary job: [e.g., "install → call API → handle auth"]
Success in 5 minutes: [e.g., "send one request and see JSON response"]
```

Failure to define these results in either a manifesto or an API dump.

### Step 1: Engineer the Top Section (Above the Fold)

Goal: 15-30 seconds to decide whether to continue reading.

Include in this exact order:
1. `# ProjectName`
2. One-sentence value proposition: what it does + for whom + differentiator
3. Status line (mandatory): `Status: active | beta | stable | deprecated | unmaintained`
4. Primary links: docs, releases/changelog, issues/discussions/support
5. Badges (optional, limited): only build, version, license
6. One visual if product is visual (screenshot/GIF)

**Rules:**
- If first screen is 70% badges → conversion problem
- If status is unclear → trust problem

### Step 2: Provide Executable Quickstart

Goal: First working result in under 5 minutes (TTFS - Time To First Success).

Quickstart must contain:
1. Prerequisites (explicit versions)
2. Install (one-liner if possible)
3. Run/Use (minimal example)
4. Expected output or "what success looks like"

**Quality bar:** A brand-new user copies and pastes commands with zero edits. If secrets required, point to `.env.example` with minimum required variables.

### Step 3: Add Real Usage (2-4 Common Workflows)

Goal: Enable meaningful use beyond hello-world.

For each workflow include:
- Short explanation (1-3 sentences)
- Code/CLI block
- Pitfall note if common error exists

Structure:
- Common workflow 1 (most frequent)
- Common workflow 2 (second most frequent)
- Optional: Advanced workflow (link out if long)

### Step 4: Progressive Disclosure

Goal: Keep README readable and maintainable.

README includes only the 80/20 (most common paths). Move deep details to:
- `/docs/*`
- `CONFIGURATION.md`
- `ARCHITECTURE.md`
- `API.md`
- Wiki/documentation site

Link to these clearly from README.

### Step 5: Contributor Path

If accepting contributions:
- Add Contributing section linking to `CONTRIBUTING.md`
- Include dev setup: clone → install → test
- Mention code style/lint/test requirements

If not accepting contributions:
- State explicitly
- Explain how to report bugs or request changes

### Step 6: Support Routes

Answer these questions:
- Where to ask questions (Issues vs Discussions vs chat)
- What info to include when filing a bug
- Response expectations (optional)

### Step 7: Legal and Security

Make non-ambiguous:
- **License**: Name it and link to `LICENSE`
- **Security**: Link to `SECURITY.md` or provide reporting instructions

Mandatory for production adoption in organizations.

### Step 8: Accessibility Requirements

Apply these checks:
- Headings follow strict hierarchy (# → ## → ###, no skipping)
- Every image has meaningful alt text
- Links are descriptive ("Contributing guidelines" not "click here")
- Short paragraphs and lists to reduce cognitive load
- Define acronyms on first use
- Avoid sarcasm/idioms that harm global readability

### Step 9: Prevent Documentation Rot

Treat README like code with automation:
- Markdown lint (format consistency)
- Link checker (prevents link rot)
- Snippet testing (execute README code blocks when feasible)
- Quickstart CI smoke test (runs commands in clean environment)

Minimum: Monthly README validation workflow or validation on each release.

## Standard Section Order

Use this skeleton unless there's a strong reason not to:

```markdown
# Title

One-line value proposition

Status: [status] | [Docs](link) | [Releases](link) | [Support](link)

![optional screenshot](path)

## Why / Motivation (short)

## Quickstart
### Prerequisites
### Install
### Run
### Expected Result

## Usage
### Common Workflow 1
### Common Workflow 2

## Configuration

## Troubleshooting / FAQ

## Roadmap (optional)

## Contributing

## Support

## Security

## License

## Maintainers / Credits
```

## Acceptance Tests

Run these 10 checks against any README. All must pass for "perfect" status:

| # | Test | Pass Criteria |
|---|------|---------------|
| 1 | What is it? | New user answers in 10 seconds |
| 2 | Is it maintained? | New user answers in 10 seconds |
| 3 | Quickstart works | Works on clean machine in ≤5 minutes |
| 4 | Runnable example | At least one exists and is copy/pasteable |
| 5 | Expected output | Shown or described |
| 6 | Navigation | Find Install/Usage/Support via headings in ≤10 seconds |
| 7 | License | Explicit and linked |
| 8 | Contribution route | Clear (even if "not accepting") |
| 9 | Accessibility | Images have alt text, headings hierarchical |
| 10 | Scope | Doesn't try to be entire manual; deep info linked |

## Additional Resources

### Reference Files

For detailed guidance on specific aspects:
- **`references/codebase-analysis.md`** - How to analyze a codebase to auto-detect target reader and job-to-be-done
- **`references/audit-checklist.md`** - Detailed audit procedure with scoring

### Using This Skill

**For README generation:**
1. Analyze codebase to determine Step 0 values
2. Follow 9-step process in order
3. Validate against 10 acceptance tests
4. Output README following standard section order

**For README auditing:**
1. Run 10 acceptance tests
2. Score each (pass/fail)
3. Identify specific issues
4. Provide prioritized fixes
5. Offer to auto-fix when requested
