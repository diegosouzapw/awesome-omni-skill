---
name: zotero-mcp
description: Interface with Zotero's MCP server to search and retrieve bibliographic data using advanced semantic search and multi-strategy approaches. Designed for output as a plain markdown formatted outline, suitable for pasting into Logseq. Also offers side-by-side translation of Chinese titles and abstracts for improved English language search within Logseq. Context-aware - uses agents in Claude Code, batched searches in Claude Desktop.
---

# Zotero MCP Search Skill

Interface with Zotero's MCP server to search and retrieve bibliographic data using advanced semantic search and multi-strategy approaches.

**Designed for plain markdown outline output:** This skill generates bibliographies formatted as plain markdown outlines suitable for pasting into Logseq with:
- Outline hierarchy (nested bullets, no markdown headers)
- Compact citation format (Author, Year. Title)
- Side-by-side Chinese-English translations for improved English language search
- Plain text (no bold styling)
- Zotero item links

**Context-Aware Skill:** This skill automatically adapts its strategy based on the environment:
- **Claude Code:** Uses autonomous agents for comprehensive multi-step searches
- **Claude Desktop:** Uses batched manual searches with safety limits

## Core Philosophy

**Search comprehensively, not narrowly.** Never settle for a single search attempt. Always:
- Use semantic search for conceptual discovery
- Try multiple search angles and variations
- Combine different search methods
- Iteratively refine based on results
- Ask clarifying questions when needed
- Format bibliographies for Logseq with proper outline structure

## Environment Detection

**Detect which environment you're running in:**
- **Claude Code:** Agents available (Task tool), no conversation crash risk
- **Claude Desktop:** No agents, conversation deletion risk with large responses

**How to tell:** Check if Task tool is available in your tool list.

---

# PART 1: Claude Desktop Strategy

## ⚠️ CRITICAL: Avoiding Claude Desktop Crashes ⚠️

**KNOWN BUG**: Claude Desktop has a critical bug where large MCP responses cause request timeouts that DELETE the entire conversation without warning or recovery.

### Safe Search Strategy: Batched Iteration

To get comprehensive results safely, use **iterative batched searches**:

**✅ SAFE: Multiple small searches**
- Each search limited to 10-15 results maximum
- Multiple searches with different angles/filters
- Combine results across searches

**❌ UNSAFE: Single large search**
- limit=50 or higher in one call
- Risks conversation deletion
- No recovery possible

### Recommended Approaches for Claude Desktop

#### Approach 1: Multi-Angle Coverage (Preferred)
Instead of one search with limit=50, do **5 searches** with different angles:
```
1. Semantic search: "main concept" (limit=10)
2. Semantic search: "related concept variation" (limit=10)
3. Keyword search: "specific terms" (limit=10)
4. Tag-based search: relevant tags (limit=10)
5. Notes/annotations search: "concept" (limit=10)
```
This gives ~50 results from different perspectives, safer than one large call.

#### Approach 2: Iterative Interactive
For user-guided exploration:
```
1. First batch: limit=10
2. Present results
3. Ask: "Would you like more results, or shall I search from a different angle?"
4. Next batch based on user feedback
```

### Default Limits for Claude Desktop

**Always use these safe defaults:**
- `limit=10` for initial searches
- `limit=15` if user explicitly needs more
- **Never exceed limit=20** in a single call
- For comprehensive results: use multiple searches, not larger limits

---

# PART 2: Claude Code Strategy

## Agent-First Approach

**When running in Claude Code, use autonomous agents** for comprehensive searches.

### When to Use Agents (Claude Code Only)

Use agents when:
- User needs comprehensive literature searches
- User asks exploratory questions ("What do I have about...")
- User needs multi-angle search coverage
- User wants thorough, curated results
- User's query is conceptual or thematic

Don't use agents when:
- User asks for a single specific paper by exact title
- User just wants to see recent additions
- User is browsing collections interactively

### Agent Task Patterns

#### Pattern 1: Comprehensive Topic Search

**User request:** "Find me papers about [topic]"

**Agent task:**
```
Search my Zotero library comprehensively for papers about [topic].

Use multiple search strategies autonomously:

1. SEMANTIC SEARCHES (5-6 variations)
   - Try natural language phrasings
   - Try different conceptual angles
   - Use synonyms and related terms
   - No limit restrictions - get comprehensive results

2. KEYWORD SEARCHES (3-4 variations)
   - Try exact terms and variations
   - Try broader/narrower terms
   - Try related methodology terms

3. TAG-BASED SEARCHES
   - First use zotero_get_tags to discover relevant tags
   - Then search by multiple relevant tags
   - Try tag combinations

4. FULL-TEXT & ANNOTATIONS
   - Use zotero_search_notes for the concept
   - Use zotero_get_annotations to find highlights
   - Search for related concepts in notes

5. SYNTHESIS
   - Combine all results and deduplicate
   - Identify the 20-30 most relevant papers
   - Group by theme/approach if applicable
   - Provide brief relevance explanations

Return a curated list with:
- Paper metadata (title, authors, year)
- Why each paper is relevant
- Thematic groupings if applicable
- Coverage note (which strategies found what)
```

#### Pattern 2: Author-Focused Search

**User request:** "What do I have by/about [author]?"

**Agent task:**
```
Find all items related to [author] in my Zotero library.

Search comprehensively:

1. DIRECT AUTHORSHIP
   - zotero_search_items with author="[author]"
   - zotero_advanced_search with author field

2. CITATIONS & MENTIONS
   - zotero_search_notes: "[author]" (cited in my notes)
   - zotero_get_annotations: "[author]" (mentioned in highlights)
   - zotero_semantic_search: "[author]'s main concepts/theories"

3. RELATED WORK
   - Search for concepts/theories associated with this author
   - Search for methodologies they use
   - Search for co-authors

4. SYNTHESIS
   - Organize by: (1) authored by, (2) cited in my notes, (3) related concepts
   - Include temporal overview if relevant
   - Note any thematic clusters

Return organized results with context about how each item relates to [author].
```

#### Pattern 3: Concept in Context

**User request:** "Research about X in context Y"

**Agent task:**
```
Find papers about [X] in the context of [Y].

Multi-angle search strategy:

1. COMBINED SEARCHES
   - Semantic: "X in Y" + variations
   - Semantic: "Y approaches to X"
   - Advanced: keyword=X AND keyword=Y
   - Advanced: keyword=X AND keyword=[Y synonyms]

2. SEPARATE THEN CROSS-REFERENCE
   - Find strong X papers
   - Find strong Y papers
   - Identify overlap and connections
   - Check if X papers mention Y in fulltext
   - Check if Y papers mention X in fulltext

3. TAG ANALYSIS
   - Get tags related to X
   - Get tags related to Y
   - Search items tagged with both domains
   - Find bridging concepts in tags

4. ANNOTATION MINING
   - Search notes for X+Y together
   - Search notes for X and Y separately
   - Check if you've annotated connections

5. SYNTHESIS
   - Papers directly about X in Y context
   - Papers about X that discuss Y
   - Papers about Y that discuss X
   - Papers that bridge both (even if not explicit)

Return results grouped by relevance strength and connection type.
```

#### Pattern 4: Exploratory Discovery

**User request:** "What's related to X?" or "Explore my library for X"

**Agent task:**
```
Explore my Zotero library to discover all material related to [X].

Comprehensive discovery approach:

1. DIRECT SEARCHES (Cast wide net)
   - Semantic search: multiple phrasings of X
   - Keyword search: X and synonyms
   - Tag search: X-related tags
   - Notes/annotations: X mentions

2. EXPANSION PHASE
   - Analyze top results to identify:
     * Related concepts and theories
     * Related methodologies
     * Related application domains
     * Related authors
   - Search for each of these expansions

3. DEEP EXPLORATION
   - For promising papers, check fulltext for related concepts
   - Look at tags on promising papers, search those tags
   - Check annotations for related ideas
   - Look for cited works mentioned in your notes

4. THEMATIC CLUSTERING
   - Group all findings by themes/approaches
   - Identify conceptual clusters
   - Note connections between clusters
   - Highlight surprising/unexpected connections

5. SYNTHESIS
   - Core papers directly about X
   - Related theoretical frameworks
   - Methodological connections
   - Application domains
   - Surprising/tangential connections worth noting

Return a thematic map of your library's coverage of this topic.
```

### How to Launch Agents (Claude Code Only)

Use the Task tool with `subagent_type="general-purpose"`:

```
Task(
  description="Comprehensive Zotero search for X",
  subagent_type="general-purpose",
  prompt="[Use one of the task patterns above]"
)
```

---

# PART 3: Universal Search Strategy

## Multi-Method Search Approach

**ALWAYS use multiple search methods in combination** (in both environments):

### A. Semantic Search (Primary for Conceptual Discovery)
- Use `zotero_semantic_search` for conceptual, thematic, or exploratory queries
- Try multiple phrasings of the same concept
- Use natural language descriptions, not just keywords
- Example variations:
  - "theories of embodied cognition"
  - "how body influences thought and reasoning"
  - "physical experience shapes cognitive processes"
- **Claude Desktop:** limit=10
- **Claude Code (agents):** no limit restriction

### B. Keyword Search (Complementary)
- Use `zotero_search_items` with multiple keyword variations
- Try synonyms, related terms, broader/narrower terms
- Use different word forms (singular/plural, verb/noun)
- Example variations for "reading comprehension":
  - "reading comprehension"
  - "text understanding"
  - "literacy"
  - "comprehension strategies"

### C. Advanced Search (Targeted)
- Use `zotero_advanced_search` for precise criteria
- Combine multiple fields (author + keyword, year + tag)
- Use for filtering after broader searches

### D. Tag & Collection Filtering
- Use `zotero_get_tags` to discover relevant tags
- Use `zotero_search_by_tag` with multiple tag variations
- Use `zotero_get_collections` and `zotero_get_collection_items` for organized searches

### E. Full-Text & Annotations
- Use `zotero_search_notes` to search annotations and highlights
- Use `zotero_get_item_fulltext` for content not in metadata
- Critical for finding concepts mentioned in text but not in titles/abstracts

## Common Search Patterns

### Pattern: "Find me papers about X"

**Claude Desktop:**
```
1. zotero_semantic_search: "X" (limit=10)
2. zotero_semantic_search: "X alternative phrasing" (limit=10)
3. zotero_search_items: keyword variations of X (limit=10)
4. zotero_get_tags: look for X-related tags
5. zotero_search_by_tag: if relevant tags found (limit=10)
6. zotero_search_notes: "X" (limit=10)
Result: ~50+ results safely retrieved
```

**Claude Code:**
```
Launch agent with Task tool (Pattern 1)
Agent performs comprehensive multi-strategy search
Returns curated results with synthesis
```

### Pattern: "What do I have on [author]?"

**Claude Desktop:**
```
1. zotero_search_items: author="[Author]" (limit=10)
2. zotero_advanced_search: author + recent years (limit=10)
3. zotero_search_notes: "[Author]" (limit=10)
4. zotero_semantic_search: "[Author]'s main concepts" (limit=10)
```

**Claude Code:**
```
Launch agent with Task tool (Pattern 2)
```

## Search Failure Recovery

If initial searches yield poor results:

1. **Ask clarifying questions:**
   - "Are you looking for theoretical or empirical work?"
   - "Any specific time period or authors?"
   - "Is this about methodology, findings, or theory?"

2. **Broaden search:**
   - Use more general terms
   - Remove filters
   - Try related fields/disciplines

3. **Check search database status:**
   - Use `zotero_get_search_database_status`
   - Suggest `zotero_update_search_database` if outdated

4. **Try alternative angles:**
   - If searching for method, search for problems it solves
   - If searching for theory, search for phenomena it explains
   - If searching for author, search for concepts they study

---

# PART 4: Bibliography Output Formatting

When the user requests a bibliography or formatted output of search results:

## Logseq Formatting Standards

**ALWAYS use outline hierarchy (nested bullet points), NEVER use markdown headers (`#`):**
**NO BOLD STYLING - use plain text for all content**

**Compact citation format:**
```
- Main Topic - Bibliography
	- A. Category Name
		- Author(s), Year. Title (English Translation if applicable)
			- Type: Article Type
			- Journal: Journal Name, Volume X, Issue Y, Pages Z
			- Zotero: [zotero://select/library/items/ITEM_KEY](zotero://select/library/items/ITEM_KEY)
			- DOI: [if available]
			- Abstract (Chinese): [if applicable]
			- Abstract (English): [translation or original]
```

**Format rules:**
- First line: Author(s), Year. Title (English Translation)
- For Chinese authors: 黃美金 (Huang Mei-Jin)
- For multiple authors: separate with semicolons
- For no date: use "n.d."
- For no author: use institutional name or "N/A"

## Translation Requirements

1. **Chinese Abstracts:**
   - Always provide BOTH Chinese original and English translation
   - Label clearly as `Abstract (Chinese):` and `Abstract (English):`
   - Translate comprehensively, not just summaries

2. **Chinese Titles:**
   - Provide English translation in parentheses after Chinese title
   - Format: `中文標題 (English Translation)`
   - If English title exists in metadata, use that; otherwise translate

3. **Author Names:**
   - Include both Chinese characters and romanization when available
   - Format: `黃美金 (Huang Mei-Jin)`

## Required Metadata Fields

Each bibliography entry must include (when available):
- Title (with translations as needed)
- Authors (with Chinese/romanization)
- Date (YYYY-MM-DD or YYYY)
- Type (Journal Article, Book Chapter, Thesis, etc.)
- Journal/Publication (with volume, issue, pages)
- Zotero link (format: `zotero://select/library/items/ITEM_KEY`)
- Abstract (Chinese and/or English as applicable)
- DOI (if available)

## What NOT to Include

- ❌ **No summary sections** at the end of bibliographies (no totals, no "organized by themes" recap)
- ❌ **No markdown headers** (`#`, `##`, etc.) - use nested bullets only
- ❌ **No bold styling** (`**text**`) - use plain text throughout
- ❌ **No item counts** or statistics sections at the end

## Bibliography Structure Example

```
- Indigenous Language Proficiency Certification (原住民族語言能力認證) - Bibliography
	- A. Core Certification Papers
		- 黃美金 (Huang Mei-Jin), 2003. 原住民族語言能力認證：回顧與展望 (Indigenous Language Proficiency Certification: Review and Prospects)
			- Type: Journal Article
			- Journal: 原住民教育季刊, Issue 9, Pages 5-27
			- Zotero: [zotero://select/library/items/W44VF3CG](zotero://select/library/items/W44VF3CG)
			- Abstract (Chinese): [full Chinese abstract]
			- Abstract (English): [full English translation]
	- B. Policy & Historical Context
		- [next paper...]
```

## File Handling

When creating bibliographies:
1. Save to `/Users/niyaro/Desktop/` with descriptive filename
2. Use `.md` extension
3. Open in BBEdit automatically (use `bbedit` command)
4. Confirm file creation and location to user

---

# PART 5: Tool Quick Reference

| Tool | Primary Use | Claude Desktop Limit | Claude Code (Agent) |
|------|-------------|---------------------|---------------------|
| `zotero_semantic_search` | Conceptual discovery | limit=10 | No limit, use liberally |
| `zotero_search_items` | Keyword matching | limit=10 | No limit |
| `zotero_advanced_search` | Precise filtering | limit=10 | No limit |
| `zotero_get_tags` | Discover tags | No limit needed | No limit needed |
| `zotero_search_by_tag` | Tag filtering | limit=10 | No limit |
| `zotero_search_notes` | Annotation search | limit=10 | No limit |
| `zotero_get_annotations` | Highlight retrieval | limit=10 | No limit |
| `zotero_get_item_fulltext` | Full-text access | N/A (single item) | N/A (single item) |
| `zotero_get_collections` | Collection discovery | No limit needed | No limit needed |
| `zotero_get_recent` | Recent additions | limit=10 | No limit |
| `zotero_update_search_database` | Update index | N/A | N/A |

---

# Critical Reminders

## For Both Environments
- **Never use just one search method**
- **Never try just one search term variation**
- **Always check tags before searching**
- **Always search both metadata and full-text/annotations**
- **Always explain your search path**
- **Always refine based on initial results**

## Claude Desktop Specific
- **🚨 SAFETY: Always use limit=10 (max 15-20) to prevent conversation deletion**
- **🚨 SAFETY: For comprehensive results, use multiple searches or agents, NOT large limits**

## Claude Code Specific
- **Default to agents** for non-trivial searches
- **Be comprehensive** - no crash limits
- **Let agents explore** - they can handle complexity autonomously
- **Synthesize results** - raw lists are not enough

## Bibliography Formatting
- **Format for Logseq** - use outline hierarchy, not headers; NO bold styling
- **Compact citation format** - Author(s), Year. Title on first line
- **Translate Chinese content** - provide both original and English versions

---

**Remember: This skill adapts to your environment. In Claude Code, leverage agents for comprehensive autonomous searches. In Claude Desktop, use careful batched searches with safety limits.**
