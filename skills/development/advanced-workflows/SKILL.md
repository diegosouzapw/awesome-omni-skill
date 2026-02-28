---
name: advanced-workflows
description: Multi-tool orchestration patterns for complex Bluera Knowledge operations. Teaches progressive library exploration, adding libraries with job monitoring, handling large result sets, multi-store searches, and error recovery workflows.
---

# Advanced Bluera Knowledge Workflows

Master complex multi-tool operations that combine multiple MCP tools for efficient knowledge retrieval and management.

## Progressive Library Exploration

When exploring a new library or codebase, use this pattern for efficient discovery:

### Workflow: Find Relevant Code in Unknown Library

```
1. list_stores()
   → See what's indexed, identify target store

2. get_store_info(store)
   → Get metadata: file paths, size, indexed files
   → Understand scope before searching

3. search(query, detail='minimal', stores=[target])
   → Get high-level summaries of relevant code
   → Review relevance scores (>0.7 = good match)

4. get_full_context(result_ids[top_3])
   → Deep dive on most relevant results only
   → Get complete code with full context
```

**Example:**

User: "How does Vue's computed properties work?"

```
list_stores()
→ Found: vue, react, pydantic

get_store_info('vue')
→ Path: .bluera/bluera-knowledge/repos/vue/
→ Files: 2,847 indexed

search("computed properties", intent='find-implementation', detail='minimal', stores=['vue'])
→ Result 1: packages/reactivity/src/computed.ts (score: 0.92)
→ Result 2: packages/reactivity/__tests__/computed.spec.ts (score: 0.85)
→ Result 3: packages/runtime-core/src/apiComputed.ts (score: 0.78)

get_full_context(['result_1_id', 'result_2_id'])
→ Full code for ComputedRefImpl class
→ Complete API implementation

Now explain with authoritative source code.
```

## Adding New Library with Job Monitoring

When adding large libraries, monitor indexing progress to know when search is ready:

### Workflow: Add Library and Wait for Index

```
1. create_store(url_or_path, name)
   → Returns: job_id
   → Background indexing starts

2. check_job_status(job_id)
   → Poll every 10-30 seconds
   → Status: 'pending' | 'running' | 'completed' | 'failed'
   → Progress: percentage, current file

3. When status='completed':
   list_stores()
   → Verify store appears in list

4. search(query, stores=[new_store], limit=5)
   → Test search works
   → Verify indexing quality
```

**Example:**

```
create_store('https://github.com/fastapi/fastapi', 'fastapi')
→ job_id: 'job_abc123'
→ Status: Indexing started in background

# Poll for completion (typically 30-120 seconds for medium repos)
check_job_status('job_abc123')
→ Status: running
→ Progress: 45% (processing src/fastapi/routing.py)

# ... wait 30 seconds ...

check_job_status('job_abc123')
→ Status: completed
→ Indexed: 487 files, 125k lines

# Verify and test
list_stores()
→ fastapi: 487 files, vector + FTS indexed

search("dependency injection", stores=['fastapi'], limit=3)
→ Returns relevant FastAPI DI patterns
→ Store is ready for use!
```

## Handling Large Result Sets

When initial search returns many results, use progressive detail to avoid context overload:

### Workflow: Progressive Detail Strategy

```
1. search(query, detail='minimal', limit=20)
   → Get summaries only (~100 tokens/result)
   → Review all 20 summaries quickly

2. Filter by relevance score:
   - Score > 0.8: Excellent match
   - Score 0.6-0.8: Good match
   - Score < 0.6: Possibly irrelevant

3. For top 3-5 results (score > 0.7):
   get_full_context(selected_ids)
   → Fetch complete code only for relevant items
   → Saves ~80% context vs fetching all upfront

4. If nothing relevant:
   search(refined_query, detail='contextual', limit=10)
   → Try different query with more context
   → Or broaden/narrow the search
```

**Example:**

```
# Initial broad search
search("authentication middleware", detail='minimal', limit=20)
→ 20 results, scores ranging 0.45-0.92
→ Total context: ~2k tokens (minimal)

# Filter by score
Top results (>0.7):
  - Result 3: auth/jwt.ts (score: 0.92)
  - Result 7: middleware/authenticate.ts (score: 0.85)
  - Result 12: auth/session.ts (score: 0.74)

# Get full code for top 3 only
get_full_context(['result_3', 'result_7', 'result_12'])
→ Complete implementations for relevant files only
→ Context: ~3k tokens (vs ~15k if we fetched all 20)

# Found what we needed! If not, would refine query and retry.
```

## Multi-Store Search with Ranking

When searching across multiple stores, use ranking to prioritize results:

### Workflow: Cross-Library Search

```
1. search(query, limit=10)
   → Searches ALL stores
   → Returns mixed results ranked by relevance

2. Review store distribution:
   - If dominated by one store: might narrow to specific stores
   - If balanced: good cross-library perspective

3. For specific library focus:
   search(query, stores=['lib1', 'lib2'], limit=15)
   → Search only relevant libraries
   → Get more results from target libraries
```

**Example:**

User: "How do different frameworks handle routing?"

```
# Search all indexed frameworks
search("routing implementation", intent='find-implementation', limit=15)
→ Result mix:
  - express (score: 0.91)
  - fastapi (score: 0.89)
  - hono (score: 0.87)
  - vue-router (score: 0.82)
  - ...

# All stores represented, good comparative view!

# If user wants deeper FastAPI focus:
search("routing implementation", stores=['fastapi', 'starlette'], limit=20)
→ More FastAPI/Starlette-specific results
→ Deeper exploration of Python framework routing
```

## Error Recovery

When operations fail, use these recovery patterns:

### Workflow: Handle Indexing Failures

```
1. create_store() fails or job_status shows 'failed'
   → Check error message
   → Common issues:
     - Git auth required (private repo)
     - Invalid URL/path
     - Disk space
     - Network timeout

2. Recovery actions:
   - Auth issue: Provide credentials or use HTTPS
   - Invalid path: Verify URL/path exists
   - Disk space: delete_store() unused stores
   - Network: Retry with smaller repo or use --shallow

3. Verify recovery:
   list_stores() → Check store appeared
   search(test_query, stores=[new_store]) → Verify searchable
```

**Example:**

```
create_store('https://github.com/private/repo', 'my-repo')
→ job_id: 'job_xyz'

check_job_status('job_xyz')
→ Status: failed
→ Error: "Authentication required for private repository"

# Recovery: Use authenticated URL or SSH
create_store('git@github.com:private/repo.git', 'my-repo')
→ job_id: 'job_xyz2'

check_job_status('job_xyz2')
→ Status: completed
→ Success!
```

## Combining Workflows

Real-world usage often combines these patterns:

```
User: "I need to understand how Express and Hono handle middleware differently"

1. list_stores() → check if both indexed
2. If not: create_store() for missing framework(s)
3. check_job_status() → wait for indexing
4. search("middleware implementation", stores=['express', 'hono'], detail='minimal')
5. Review summaries, identify key files
6. get_full_context() for 2-3 most relevant from each framework
7. Compare implementations with full context
```

This multi-step workflow is efficient, targeted, and conserves context.

## Best Practices

1. **Always start with detail='minimal'** - Get summaries first, full context selectively
2. **Monitor background jobs** - Don't search newly added stores until indexing completes
3. **Use intent parameter** - Helps ranking ('find-implementation' vs 'find-pattern' vs 'find-usage')
4. **Filter by stores when known** - Faster, more focused results
5. **Check relevance scores** - >0.7 is usually a strong match, <0.5 might be noise
6. **Progressive refinement** - Broad search → filter → narrow → full context

These workflows reduce token usage, minimize tool calls, and get you to the right answer faster.
