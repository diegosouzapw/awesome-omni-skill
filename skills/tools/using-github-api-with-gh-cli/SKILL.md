---
name: Using GitHub API with gh CLI
description: This skill teaches how to access GitHub's REST and GraphQL APIs via gh api for inline PR comments, review threads, and data not available through standard gh commands. Use this when you need PR review comments, thread status, or other GitHub data that gh pr view doesn't provide.
allowed-tools: [Bash, Read, Write, Edit]
---

# Using GitHub API with gh CLI

The `gh` CLI doesn't have first-class support for inline/in-code pull request review comments (the line/thread comments on the "Files changed" tab). This skill shows how to access these and other GitHub data programmatically using `gh api`.

## What This Skill Does

- Access inline PR review comments
- Get review thread status (resolved/unresolved)
- Reply to inline review comments
- Resolve and unresolve review threads
- Use GitHub's REST API via gh api
- Use GitHub's GraphQL API via gh api
- Extract owner/repo/PR number from URLs
- Filter and query PR review data

## When You Need This Skill

Use this skill when:
- Standard `gh pr view` doesn't show the data you need
- You need inline/line-specific review comments from "Files changed" tab
- You need to check if review threads are resolved/unresolved
- You need to programmatically process review data

For day-to-day gh usage (viewing PRs, creating PRs, finding branch names), see the gh CLI section in CLAUDE.md.

## Prerequisites

1. **gh CLI installed** - Already available as `gh`
2. **Authenticated with GitHub** - Run `gh auth status` to verify
3. **jq** - For parsing JSON output (optional but recommended)

## Core Concepts

### What gh Commands Show vs Don't Show

**`gh pr view` shows:**
- PR conversation comments
- Review summaries
- Overall PR metadata

**`gh pr view` does NOT show:**
- Inline review comments (line-specific comments)
- Review thread resolved/unresolved status
- Code diff context for comments

**Use `gh api` when you need** the data that `gh pr view` doesn't provide.

### REST API vs GraphQL API

**REST API:**
- Simple for basic inline comment retrieval
- Returns arrays of comment objects
- Good for straightforward queries

**GraphQL API:**
- Better for complex queries
- Returns thread structure with resolved status
- Can fetch everything in one query
- Provides `isResolved` and `isOutdated` fields

## REST API Usage

### Get All Inline Review Comments for a PR

```bash
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments \
  -q '.[] | {id, path, line, body: .body, user: .user.login, created_at}'
```

**Endpoint:** `GET /repos/{owner}/{repo}/pulls/{pull_number}/comments`

**Returns:** Array of comment objects with:
- `id` - Comment ID
- `path` - File path
- `line` - Line number
- `body` - Comment text
- `user.login` - Author username
- `created_at` - Timestamp

**Reference:** [REST API endpoints for pull request review comments](https://docs.github.com/en/rest/pulls/comments)

### Get Inline Comments for a Specific Review

```bash
# First get reviews for the PR to find REVIEW_ID
gh api repos/OWNER/REPO/pulls/PR_NUMBER/reviews -q '.[].id'

# Then list the inline comments for one review
gh api repos/OWNER/REPO/pulls/PR_NUMBER/reviews/REVIEW_ID/comments \
  -q '.[] | {id, path, line, body: .body, author: .user.login}'
```

**Endpoints:**
- `GET /repos/{owner}/{repo}/pulls/{pull_number}/reviews` (list reviews)
- `GET /repos/{owner}/{repo}/pulls/{pull_number}/reviews/{review_id}/comments` (list comments)

**Reference:** [REST API endpoints for pull request reviews](https://docs.github.com/en/rest/pulls/reviews)

### Get PR Conversation Comments (Non-Inline)

```bash
gh api repos/OWNER/REPO/issues/PR_NUMBER/comments \
  -q '.[] | {id, body: .body, author: .user.login}'
```

**Note:** PR conversation comments use the Issues API endpoint, not Pulls.

**Endpoint:** `GET /repos/{owner}/{repo}/issues/{issue_number}/comments`

**Reference:** [REST API endpoints for issue comments](https://docs.github.com/en/rest/issues/comments)

## GraphQL API Usage

### Get All Review Threads with Resolved Status

```bash
gh api graphql -f query='
  query($owner:String!, $repo:String!, $number:Int!, $n:Int=100) {
    repository(owner:$owner, name:$repo) {
      pullRequest(number:$number) {
        reviewThreads(first:$n) {
          nodes {
            id
            isResolved
            isOutdated
            comments(first:$n) {
              nodes {
                databaseId
                bodyText
                path
                diffHunk
                author { login }
                createdAt
                url
              }
            }
          }
        }
      }
    }
  }' -F owner=OWNER -F repo=REPO -F number=PR_NUMBER
```

**Key Fields:**
- `isResolved` - Whether the thread is marked as resolved
- `isOutdated` - Whether the thread is outdated due to code changes
- `diffHunk` - The code context for the comment
- `path` - File path
- `bodyText` - Comment content

**Reference:** [GraphQL API documentation](https://docs.github.com/en/graphql)

## When to Use REST vs GraphQL

### Use REST when:

- You just need inline comments (body, path/line, author, timestamps)
- You want simplicity and speed
- You're fetching data from a single PR
- You don't need resolved/unresolved status

### Use GraphQL when:

- You need thread structure and resolved/unresolved status
- You need `isOutdated` flags
- You want to fetch everything in one query
- REST doesn't expose the metadata you need
- You're building complex queries across related data

## Responding to Review Comments

### Reply to an Inline Comment

```bash
# Get comment IDs first
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments --jq '.[] | "\(.id) \(.path):\(.line)"'

# Reply to a specific comment
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments/COMMENT_ID/replies \
  -X POST -f body="Done. Fixed in latest commit."
```

**Endpoint:** `POST /repos/{owner}/{repo}/pulls/{pull_number}/comments/{comment_id}/replies`

**Reference:** [Create a reply for a review comment](https://docs.github.com/en/rest/pulls/comments#create-a-reply-for-a-review-comment)

### Resolve a Review Thread

Review threads can only be resolved via GraphQL. First get the thread ID, then resolve it:

```bash
# Get thread IDs (note: these start with PRRT_)
gh api graphql -f query='
query {
  repository(owner: "OWNER", name: "REPO") {
    pullRequest(number: PR_NUMBER) {
      reviewThreads(first: 10) {
        nodes {
          id
          isResolved
          comments(first: 1) {
            nodes {
              body
              path
            }
          }
        }
      }
    }
  }
}'

# Resolve the thread
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "PRRT_kwDOxxxxxx"}) {
    thread {
      isResolved
    }
  }
}'
```

**Note:** The `threadId` is the GraphQL node ID (starts with `PRRT_`), not the REST API comment ID.

### Unresolve a Review Thread

```bash
gh api graphql -f query='
mutation {
  unresolveReviewThread(input: {threadId: "PRRT_kwDOxxxxxx"}) {
    thread {
      isResolved
    }
  }
}'
```

### Reply and Resolve in Sequence

A common workflow is to reply to feedback explaining what you did, then resolve the thread:

```bash
# 1. Reply to the comment (REST)
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments/COMMENT_ID/replies \
  -X POST -f body="Done. Created shared BigQuery bean."

# 2. Resolve the thread (GraphQL)
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "PRRT_kwDOxxxxxx"}) {
    thread { isResolved }
  }
}'
```

## Common Use Cases

### List All Unresolved Review Threads

```bash
gh api graphql -f query='
  query($owner:String!, $repo:String!, $number:Int!) {
    repository(owner:$owner, name:$repo) {
      pullRequest(number:$number) {
        reviewThreads(first:100) {
          nodes {
            isResolved
            comments(first:1) {
              nodes {
                path
                bodyText
                author { login }
              }
            }
          }
        }
      }
    }
  }' -F owner=OWNER -F repo=REPO -F number=PR_NUMBER \
  | jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

### Get All Comments by a Specific Reviewer

```bash
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments \
  | jq '.[] | select(.user.login == "username") | {path, line, body: .body}'
```

### Count Total Inline Comments

```bash
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments | jq '. | length'
```

### Get Comments with Specific Text

```bash
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments \
  | jq '.[] | select(.body | contains("TODO")) | {path, line, body: .body}'
```

## Extracting URL Components

### From PR URL

```bash
# From: https://github.com/owner/repo/pull/123
PR_URL="https://github.com/owner/repo/pull/123"

# Extract owner
OWNER=$(echo "$PR_URL" | cut -d'/' -f4)

# Extract repo
REPO=$(echo "$PR_URL" | cut -d'/' -f5)

# Extract PR number
PR_NUMBER=$(echo "$PR_URL" | cut -d'/' -f7)
```

### Using gh to Parse URL

```bash
gh pr view "$PR_URL" --json number,repository \
  -q '{number: .number, owner: .repository.owner.login, repo: .repository.name}'
```

This is more robust as it handles different URL formats.

## Examples

### Example 1: Get All Inline Comments for a PR

```bash
# User: "Show me all inline review comments on PR #123"

OWNER="food-truck"
REPO="mono"
PR_NUMBER="123"

gh api repos/${OWNER}/${REPO}/pulls/${PR_NUMBER}/comments \
  -q '.[] | "[\(.path):\(.line)] @\(.user.login): \(.body)"'
```

### Example 2: Find Unresolved Comments

```bash
# User: "What review comments are still unresolved on that PR?"

PR_URL="https://github.com/food-truck/mono/pull/123"

# Parse URL
OWNER=$(echo "$PR_URL" | cut -d'/' -f4)
REPO=$(echo "$PR_URL" | cut -d'/' -f5)
PR_NUMBER=$(echo "$PR_URL" | cut -d'/' -f7)

# Get unresolved threads
gh api graphql -f query='
  query($owner:String!, $repo:String!, $number:Int!) {
    repository(owner:$owner, name:$repo) {
      pullRequest(number:$number) {
        reviewThreads(first:100) {
          nodes {
            isResolved
            comments(first:1) {
              nodes {
                path
                bodyText
                author { login }
              }
            }
          }
        }
      }
    }
  }' -F owner=${OWNER} -F repo=${REPO} -F number=${PR_NUMBER} \
  | jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false) | .comments.nodes[0] | "[\(.path)] @\(.author.login): \(.bodyText)"'
```

### Example 3: Get My Comments on a PR

```bash
# User: "What comments did I leave on PR #123?"

MY_USERNAME=$(gh api user -q '.login')

gh api repos/food-truck/mono/pulls/123/comments \
  | jq --arg user "$MY_USERNAME" '.[] | select(.user.login == $user) | {path, line, body: .body}'
```

### Example 4: Summary of Review Activity

```bash
# User: "Give me a summary of review activity on PR #123"

PR_NUMBER="123"
OWNER="food-truck"
REPO="mono"

# Total comments
TOTAL=$(gh api repos/${OWNER}/${REPO}/pulls/${PR_NUMBER}/comments | jq '. | length')

# Comments by reviewer
echo "Total inline comments: $TOTAL"
echo ""
echo "By reviewer:"
gh api repos/${OWNER}/${REPO}/pulls/${PR_NUMBER}/comments \
  | jq -r '.[] | .user.login' \
  | sort | uniq -c | sort -rn
```

## Verification

### Check gh Authentication

```bash
# Verify you're authenticated
gh auth status

# Expected output shows authenticated user and scopes
```

### Test API Access

```bash
# Test REST API access
gh api user -q '.login'

# Should return your GitHub username
```

### Verify Repository Access

```bash
# Check if you can access a repo
gh api repos/OWNER/REPO -q '.name'

# Should return the repo name
```

## Troubleshooting

### Issue: "Could not resolve to a Repository"

**Cause**: Incorrect owner/repo, or no access to repository

**Solution**:
```bash
# Verify owner and repo names
gh repo view OWNER/REPO

# Check if you have access
gh api repos/OWNER/REPO -q '.name'

# Verify authentication
gh auth status
```

### Issue: "Resource not accessible by integration"

**Cause**: Missing scopes in authentication token

**Solution**:
```bash
# Re-authenticate with required scopes
gh auth login --scopes repo,read:org

# Check current scopes
gh auth status
```

### Issue: Empty results for inline comments

**Cause**: PR has no inline comments, or querying wrong endpoint

**Solution**:
```bash
# Verify PR has inline comments by viewing in browser

# Check you're using pulls endpoint, not issues
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments

# For conversation comments, use issues endpoint
gh api repos/OWNER/REPO/issues/PR_NUMBER/comments
```

### Issue: GraphQL query syntax error

**Cause**: Malformed GraphQL query

**Solution**:
```bash
# Test query structure with simpler query first
gh api graphql -f query='
  query {
    viewer { login }
  }'

# Add complexity incrementally
# Check GitHub GraphQL Explorer for query validation
```

### Issue: Rate limiting

**Cause**: Too many API requests

**Solution**:
```bash
# Check rate limit status
gh api rate_limit

# Wait for rate limit reset
# Or use GraphQL (higher rate limits)
```

## Best Practices

1. **Parse URLs with gh when possible**
   - `gh pr view URL --json` is more robust than string parsing
   - Handles different URL formats automatically

2. **Use GraphQL for complex queries**
   - Single query vs multiple REST calls
   - Better for resolved/unresolved status
   - Higher rate limits

3. **Cache results locally**
   - API calls count against rate limits
   - Save results to file for repeated analysis

4. **Use jq for filtering**
   - Filter on client side to reduce API calls
   - Build up complex jq queries incrementally

5. **Check authentication first**
   - Always verify `gh auth status` before debugging
   - Saves time troubleshooting permission issues

6. **Use -q flag with gh api**
   - Provides cleaner output
   - Can extract specific fields directly

## Quick Reference

```bash
# Get inline PR comments (REST)
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments

# Get comment IDs with file/line info
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments --jq '.[] | "\(.id) \(.path):\(.line)"'

# Reply to a comment (REST)
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments/COMMENT_ID/replies -X POST -f body="Done."

# Get thread IDs and resolved status (GraphQL)
gh api graphql -f query='query { repository(owner:"OWNER", name:"REPO") { pullRequest(number:PR_NUMBER) { reviewThreads(first:10) { nodes { id isResolved } } } } }'

# Resolve a thread (GraphQL)
gh api graphql -f query='mutation { resolveReviewThread(input:{threadId:"PRRT_xxx"}) { thread { isResolved } } }'

# Unresolve a thread (GraphQL)
gh api graphql -f query='mutation { unresolveReviewThread(input:{threadId:"PRRT_xxx"}) { thread { isResolved } } }'

# Parse PR URL
gh pr view URL --json number,repository

# Count comments
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments | jq '. | length'

# Filter by author
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments | jq '.[] | select(.user.login == "username")'

# Check rate limits
gh api rate_limit
```

## References

- [GitHub CLI Manual](https://cli.github.com/manual/)
- [GitHub REST API Quickstart](https://docs.github.com/rest/quickstart)
- [GitHub GraphQL API Documentation](https://docs.github.com/en/graphql)
- [gh cli Discussion #3993](https://github.com/cli/cli/discussions/3993) - Community discussion on retrieving PR reviews

## Related Skills

- **Git Workflow** - Understanding git operations for PR management
- **JSON Processing with jq** - Essential for parsing gh api output
