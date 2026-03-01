---
name: analyzing-text-patterns
description: Extract and analyze recurring patterns from log messages, span names, and event names using punctuation-based template discovery. Use when you need to understand log diversity, identify common message structures, detect unusual formats, or prepare for log parser development. Works by removing variable content and preserving structural markers.
---

# Analyzing Text Patterns

Extract recurring patterns from textual data by removing variable content (usernames, IDs, timestamps) and preserving structural markers (punctuation, spacing). This reveals common message templates, helps identify log diversity, and detects unusual or anomalous formats.

Use when you need to:
- Understand what types of logs/messages exist in your system
- Find recurring message templates for parser development
- Detect unusual or rare log formats (anomaly detection)
- Analyze log diversity across services
- Identify which services generate which log formats
- Prepare for structured log extraction

## Key Concepts

### Pattern Extraction vs Log Parsing

**Pattern Extraction** (this skill):
- Discovers structural templates
- Groups messages by shape/structure
- Ignores actual content (IDs, names, values)
- Fast, no ML required
- Example: `"User alice logged in"` and `"User bob99 logged in"` → same pattern

**Log Parsing** (different skill):
- Extracts field values from logs
- Requires known pattern/regex
- Captures actual content
- Example: Parse username from `"User {username} logged in"`

### How Pattern Extraction Works

**The Technique**:
1. Remove all word characters (letters, digits, underscore)
2. Replace whitespace with underscores
3. Result: Only punctuation and structure remain

**Example**:
```
Original: "2025-11-15 [INFO] User alice logged in at 10:30"
Step 1:   "-- [] User alice logged in at :"
Step 2:   "--_[]___logged_in_at_:"
```

This "punctuation pattern" represents the log template structure.

### When It Works Best

✅ **Ideal for**:
- Mixed log sources (multiple services, different formats)
- Discovering log templates in unfamiliar systems
- Understanding log diversity
- Finding rare/unusual log formats
- Analyzing unstructured logs

❌ **Not ideal for**:
- Homogeneous single-service logs (little diversity)
- When numbers are meaningful (error codes, IDs)
- Semantic grouping (grouping by meaning, not structure)
- Already-structured JSON logs (structure is explicit)

## Pattern 1: Basic Structure Extraction

**Concept**: Extract structural template by removing all variable content

**When to use**:
- Analyzing mixed log sources
- Understanding log diversity
- Finding common templates

**Query**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(string(body), /\w/, ""), /\s+/, "_")
make_col pattern_id:encode_base64(string(hash(punct)))
statsby count:count(), sample:any(body), group_by(pattern_id, punct)
sort desc(count)
make_col "%":round(100*count/window(sum(count), group_by()),1)
```

**How it works**:
1. `replace_regex(string(body), /\w/, "")` - Remove word characters (letters, digits, _)
2. `replace_regex(..., /\s+/, "_")` - Normalize whitespace to single underscore
3. `hash(punct)` - Create unique ID for each pattern
4. `encode_base64(...)` - Make pattern ID readable
5. `statsby count:count()` - Count occurrences per pattern
6. `sample:any(body)` - Keep one example message
7. Calculate `%` of total logs

**Example result**:
```
pattern_id                         | count | %    | sample
-----------------------------------|-------|------|------------------------------------------
LTcwOTA5MDA2MTQ0NDM3NTA2ODA=      | 3405  | 13.0 | 2025-11-15T21:59:38.985Z info MetricsExporter...
MzYzODU1NzM5OTY2NjQyMDU0NQ==      | 2877  | 11.0 | [2025-11-15 21:59:55,877] INFO Deleted log...
```

**Interpretation**:
- 13% of logs match Pattern 1 (JSON metrics exporter format)
- 11% match Pattern 2 (Kafka deletion logs)
- Shows clear log diversity

**Pros**:
- Fast and simple
- Works on any text field
- No configuration needed

**Cons**:
- Loses numeric patterns (error codes)
- Sensitive to formatting changes
- No semantic understanding

## Pattern 2: Preserve Numeric Patterns

**Concept**: Keep numbers to distinguish error codes, status codes, counts

**When to use**:
- Error code analysis (404 vs 500)
- Status code patterns
- When counts/IDs are structurally significant

**Query**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(string(body), /[a-zA-Z_]/, ""), /\s+/, "_")
make_col pattern_id:encode_base64(string(hash(punct)))
statsby count:count(), sample:any(body), group_by(pattern_id, punct)
sort desc(count)
make_col "%":round(100*count/window(sum(count), group_by()),1)
```

**Key difference**: `/[a-zA-Z_]/` removes only letters, **preserves digits**

**Example differentiation**:
```
Original Pattern 1:
  "Error 404: Not found" → "_:__"
  "Error 500: Server error" → "_:__"
  (Same pattern - can't distinguish!)

With Preserved Digits (Pattern 2):
  "Error 404: Not found" → "_404:__"
  "Error 500: Server error" → "_500:__"
  (Different patterns - can distinguish!)
```

**When this helps**:
- HTTP status code analysis
- Error code frequency
- Version number patterns
- Timestamp patterns (year, hour, etc.)

**Tradeoff**:
- More unique patterns (less grouping)
- Timestamps create per-second patterns
- May be too granular

## Pattern 3: Hybrid Approach

**Concept**: Combine structural and numeric patterns for maximum precision

**When to use**:
- Need both structure and number patterns
- Building comprehensive log taxonomy
- Maximum differentiation needed

**Query**:
```opal
limit 100000
make_col struct:replace_regex(replace_regex(string(body), /\w/, ""), /\s+/, "_")
make_col numeric:replace_regex(replace_regex(string(body), /[a-zA-Z_]/, ""), /\s+/, "_")
make_col pattern_id:encode_base64(string(hash(struct + "|" + numeric)))
statsby count:count(), sample:any(body), group_by(pattern_id, struct, numeric)
sort desc(count)
make_col "%":round(100*count/window(sum(count), group_by()),1)
```

**Combines**:
- `struct`: Punctuation pattern (from Pattern 1)
- `numeric`: Numeric pattern (from Pattern 2)
- Both patterns together create unique signature

**Example**:
```
Message: "2025-11-15 [ERROR] Request failed with status 500"
struct:  "--_[]___with_"
numeric: "2025-11-15_500"
pattern: Combination of both
```

**Pros**:
- Most precise differentiation
- Captures both structure and numbers
- Best for complex analysis

**Cons**:
- More complex query
- May create too many unique patterns
- Harder to interpret results

## Pattern 4: Apply to Different Fields

**For Span Names**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(span_name, /\w/, ""), /\s+/, "_")
make_col pattern_id:encode_base64(string(hash(punct)))
statsby count:count(), sample:any(span_name), group_by(pattern_id, punct)
sort desc(count)
make_col "%":round(100*count/window(sum(count), group_by()),1)
```

**Example results**:
```
28.2%: "" (no punctuation) - HTTP methods: "POST", "GET"
26.4%: "_" (single space) - "router frontend egress"
12.5%: "./" - gRPC: "oteldemo.CartService/GetCart"
```

**Use cases**:
- Identify naming conventions across services
- Find services using different protocols
- Understand span taxonomy

**For Event Names**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(event_name, /\w/, ""), /\s+/, "_")
make_col pattern_id:encode_base64(string(hash(punct)))
statsby count:count(), sample:any(event_name), group_by(pattern_id, punct)
sort desc(count)
make_col "%":round(100*count/window(sum(count), group_by()),1)
```

**Use cases**:
- Categorize span event types
- Find common event patterns
- Identify event naming consistency

## Common Use Cases

### Use Case 1: Log Diversity Analysis

**Goal**: Understand what types of logs exist in your system

**Query**: Basic pattern extraction (Pattern 1)

**Workflow**:
1. Run pattern extraction on log dataset
2. Review top 10-20 patterns
3. Identify dominant log types
4. Look for unexpected patterns

**Example**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(string(body), /\w/, ""), /\s+/, "_")
statsby count:count(), sample:any(body), group_by(punct)
sort desc(count)
limit 20
```

**Interpretation**:
- If top pattern is >80%: Very homogeneous logging
- If top 10 patterns <50%: High diversity, many log types
- Rare patterns (<1%): Unusual events or errors

### Use Case 2: Anomaly Detection (Rare Patterns)

**Goal**: Find unusual or one-off log patterns

**Query**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(string(body), /\w/, ""), /\s+/, "_")
statsby count:count(), sample:any(body), group_by(punct)
sort asc(count)  # Sort by rarest first
filter count < 10  # Only rare patterns
limit 20
```

**Use cases**:
- Identify malformed logs
- Find new/unexpected error messages
- Detect logging bugs (incorrect format)
- Security: Find unusual access patterns

**Example findings**:
```
count: 1 | sample: "FATAL: Database connection pool exhausted"
count: 2 | sample: "CRITICAL: Disk space at 99%"
count: 3 | sample: "<?xml malformed payload"
```

### Use Case 3: Service Identification by Pattern

**Goal**: Map log patterns to services

**Query**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(string(body), /\w/, ""), /\s+/, "_")
make_col svc:string(resource_attributes."k8s.deployment.name")
statsby count:count(), sample:any(body), group_by(punct, svc)
sort desc(count)
limit 30
```

**Use cases**:
- Understand service-specific logging styles
- Identify cross-service patterns
- Find which services need log standardization

**Example insights**:
- Pattern `[--_::,]_INFO_...` → Only Kafka service
- Pattern `--::.Z_info_...` → Only OpenTelemetry collector
- Pattern `::_=_[]_` → Phoenix/Elixir services

### Use Case 4: Log Parser Development

**Goal**: Build regex parsers for each log template

**Workflow**:
1. Run pattern extraction
2. Review top patterns (cover 80%+ of logs)
3. For each pattern:
   - Review sample message
   - Write regex to extract fields
   - Test on sample
4. Prioritize by frequency (%)

**Query**:
```opal
limit 100000
make_col punct:replace_regex(replace_regex(string(body), /\w/, ""), /\s+/, "_")
statsby count:count(), sample:any(body), group_by(punct)
sort desc(count)
make_col "%":round(100*count/window(sum(count), group_by()),1)
make_col cumulative_pct:window(sum("%"), group_by(), order_by(desc(count)))
filter cumulative_pct <= 80  # Patterns covering first 80% of logs
```

**Example**:
```
Pattern 1 (40%): "--_::,_INFO_..."
  Sample: "[2025-11-15 23:16:06,906] INFO Kafka message"
  Regex: /\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}),\d+\] (\w+) (.+)/

Pattern 2 (25%): "--T::.Z_info_..."
  Sample: "2025-11-15T23:16:12.525Z\tinfo\tMetricsExporter..."
  Regex: /(\d{4}-\d{2}-\d{2}T[\d:.]+Z)\t(\w+)\t(.+)/
```

## OPAL Syntax Key Points

### Regex Functions

**replace_regex**:
```opal
replace_regex(string, pattern, replacement)
```

**Common patterns**:
- `/\w/` - Word characters (letters, digits, underscore)
- `/[a-zA-Z_]/` - Letters and underscore only (preserves digits)
- `/[a-z0-9_]/` - Lowercase and digits (preserves uppercase)
- `/\s+/` - One or more whitespace characters

### Hash and Encoding

**Create pattern ID**:
```opal
make_col pattern_id:encode_base64(string(hash(punct)))
```

- `hash(punct)` - Creates numeric hash of pattern
- `string(...)` - Converts hash to string
- `encode_base64(...)` - Makes ID human-readable

**Why use hash?**
- Deterministic (same pattern = same ID)
- Short and readable with base64
- Efficient for grouping

### Window Functions for Percentages

**Calculate % of total**:
```opal
make_col "%":round(100*count/window(sum(count), group_by()),1)
```

- `window(sum(count), group_by())` - Sum of all counts (total logs)
- `count/window(...)` - Fraction of total
- `100*...` - Convert to percentage
- `round(..., 1)` - One decimal place

**Cumulative percentages**:
```opal
make_col cumulative_pct:window(sum("%"), group_by(), order_by(desc(count)))
```

- Sums percentages in descending count order
- Shows "top N patterns cover X% of logs"

### Limit Best Practices

**Always use limit**:
```opal
limit 100000  # At start of query
```

**Why**:
- Prevents analyzing millions of logs (expensive)
- 100K is usually enough for pattern discovery
- Faster query execution

**Adjust based on diversity**:
- Homogeneous logs: 10K may be enough
- High diversity: 100K or more
- Start small, increase if needed

## Troubleshooting

### Issue: Too many unique patterns

**Cause**: High log diversity or timestamps creating unique patterns

**Solutions**:
1. **Increase grouping tolerance**: Use basic pattern (removes all numbers)
2. **Focus on top patterns**: Add `limit 20` to results
3. **Filter by minimum count**: `filter count > 10`
4. **Remove timestamps first**:
   ```opal
   make_col cleaned:replace_regex(string(body), /\d{4}-\d{2}-\d{2}/, "DATE")
   make_col punct:replace_regex(replace_regex(cleaned, /\w/, ""), /\s+/, "_")
   ```

### Issue: Not enough differentiation

**Cause**: Logs have similar structure but different meanings

**Solutions**:
1. **Preserve digits**: Use Pattern 2 (preserve numbers)
2. **Use hybrid approach**: Pattern 3 (structure + numbers)
3. **Add semantic fields**: Group by service/level too:
   ```opal
   statsby count:count(), sample:any(body), group_by(punct, severity)
   ```

### Issue: Patterns not meaningful

**Cause**: Field doesn't have enough structure (too simple or too complex)

**Example failures**:
- Simple: `event_name = "message"` (44% of events)
- Complex: Random JSON blobs

**Solutions**:
1. **Try different field**:
   - Instead of `body`, try `event_name`
   - Instead of full message, extract first 50 chars
2. **Pre-filter**:
   ```opal
   filter not contains(body, "{")  # Skip JSON logs
   ```
3. **Consider alternative approach**: Structured extraction for JSON

### Issue: Performance slow with large dataset

**Cause**: Processing too many logs

**Solutions**:
1. **Reduce limit**: Start with `limit 10000`
2. **Add time filter**: Reduce time range (1h instead of 24h)
3. **Sample randomly**:
   ```opal
   filter random() < 0.1  # Sample 10% of logs
   limit 100000
   ```

### Issue: Punctuation pattern empty or same

**Cause**: Logs have no punctuation or all same punctuation

**Example**:
```
"User logged in" → "_logged_in" (no punctuation)
"Server started" → "_" (no punctuation)
```

**Solutions**:
1. **Not a good dataset for this technique** - Try parsing instead
2. **Use preserve-case variation**:
   ```opal
   make_col punct:replace_regex(replace_regex(string(body), /[a-z0-9_]/, ""), /\s+/, "_")
   ```
   Keeps uppercase letters as structure markers

3. **Combine with length patterns**:
   ```opal
   make_col words:array_length(split(string(body), " "))
   statsby count:count(), sample:any(body), group_by(words)
   ```
   Group by word count instead

## Key Takeaways

1. **Pattern extraction discovers structure, not content**
   - Groups messages by template/shape
   - Ignores variable data (usernames, IDs, timestamps)
   - Fast and deterministic

2. **Three pattern variations available**
   - Basic: Remove all word characters (best for general use)
   - Preserve digits: Keep numbers (error codes, status codes)
   - Hybrid: Combine both (maximum precision)

3. **Works on any text field**
   - Log messages (`body`)
   - Span names (`span_name`)
   - Event names (`event_name`)
   - Choose field based on what you're analyzing

4. **Best for heterogeneous data**
   - Mixed services with different formats
   - Understanding log diversity
   - Finding unusual patterns
   - Less useful for homogeneous single-service logs

5. **Always use limit**
   - `limit 100000` prevents overwhelming results
   - Sample size sufficient for pattern discovery
   - Adjust based on dataset size and diversity

6. **Combine with other dimensions**
   - Group by service, severity, namespace
   - Map patterns to sources
   - Identify service-specific formats

7. **Not the same as log parsing**
   - Pattern extraction: Find templates
   - Log parsing: Extract field values
   - Use pattern extraction first, then build parsers

8. **Review sample messages**
   - Always include `sample:any(body)` in statsby
   - Sample shows what pattern represents
   - Critical for interpretation

9. **Percentages show distribution**
   - Top pattern >80%: Homogeneous
   - Top 10 <50%: High diversity
   - Use to prioritize parser development

10. **Rare patterns indicate anomalies**
    - `filter count < 10` finds unusual logs
    - Useful for error detection
    - Security monitoring

## When to Use This Skill

Use analyzing-text-patterns skill when:
- Exploring unfamiliar log datasets
- Building log parsers (need to know patterns first)
- Analyzing log diversity across services
- Detecting unusual or malformed logs
- Understanding message structure
- Identifying service-specific logging styles
- Finding rare error patterns
- Preparing for structured log extraction

Cross-references:
- investigating-textual-data (for log parsing and field extraction)
- filtering-event-datasets (for basic log filtering)
- detecting-anomalies (for statistical anomaly detection)
