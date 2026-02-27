---
name: tc-review
description: Review QA test case documents (Markdown) against source code to find coverage gaps. Use when reviewing test cases, analyzing test coverage, suggesting missing scenarios, or learning codebase to extract testable scenarios. Supports project-based learning from multiple GitHub repos with dual memory (local + Dify RAG).
---

# Test Case Review

Review test case documents against source code knowledge. Learn codebases, identify coverage gaps, suggest missing test cases.

## Quick Start

```bash
# 1. Create project and add repos
python scripts/setup_project.py myproject https://github.com/org/repo.git

# 2. Review test cases
python scripts/review.py tests/checkout.md --project myproject
```

## Decision Tree

```
User task → What do they need?
    │
    ├─ Review test cases → Is project set up?
    │   ├─ No → Run: python scripts/setup_project.py <name> <repo-url>
    │   └─ Yes → Run: python scripts/review.py <testcase.md> --project <name>
    │
    ├─ Learn new codebase → Run: python scripts/learn.py <project>
    │
    ├─ Get suggestions → Run: python scripts/suggest.py <project>
    │
    └─ Search knowledge → Run: python scripts/search.py <project> "<query>"
```

## Available Scripts

**Always run with `--help` first** to see usage. Scripts handle complex workflows reliably.

| Script | Purpose |
|--------|---------|
| `scripts/setup_project.py` | Create project + add repo + learn (all-in-one) |
| `scripts/review.py` | Review test case file against knowledge |
| `scripts/learn.py` | Learn/re-learn source code from repos |
| `scripts/suggest.py` | Generate test case suggestions |
| `scripts/search.py` | Search project knowledge |
| `scripts/batch_review.py` | Review multiple files at once |

## Test Case Format

Test cases must be Markdown. See [references/testcase-format.md](references/testcase-format.md) for complete guide.

```markdown
# Test Suite: Payment Flow

## TC-001: Successful Payment
**Preconditions:**
- User logged in
- Cart has items

**Steps:**
1. Navigate to checkout
2. Enter valid card
3. Click Pay

**Expected Result:**
Payment success, order created
```

## Example Workflows

### Review Single File
```bash
python scripts/review.py tests/login.md --project auth-service
```

Output:
```
📊 Score: 65/100  Coverage: 65%

✅ Covered: Successful login, Invalid password
⚠️ Missing: Account locked, Rate limiting, Session timeout
💡 Suggested: TC-005 Account Lockout, TC-006 Rate Limit
```

### Batch Review
```bash
python scripts/batch_review.py "tests/**/*.md" --project myapp --min-score 70
```

### Quick Review (No Project Setup)
```bash
python scripts/setup_project.py temp-project https://github.com/org/repo.git
python scripts/review.py tests/feature.md --project temp-project
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes* | For Claude LLM |
| `OPENAI_API_KEY` | Yes* | For OpenAI/Codex |
| `LLM_PROVIDER` | No | `anthropic` or `openai` (default: anthropic) |
| `LLM_MODEL` | No | Model name |
| `DIFY_API_KEY` | No | Enable Dify RAG |
| `DIFY_DATASET_ID` | No | Dify dataset ID |

*One of ANTHROPIC_API_KEY or OPENAI_API_KEY required.

## Reference Files

- [testcase-format.md](references/testcase-format.md) - Complete test case writing guide
- [scenarios.md](references/scenarios.md) - Test scenario types and examples
- [troubleshooting.md](references/troubleshooting.md) - Common issues and solutions
