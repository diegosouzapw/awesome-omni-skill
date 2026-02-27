---
name: observe
description: Query and manage Observe using the Observe CLI. Use when the user wants to run OPAL queries, list datasets, manage objects, or interact with their Observe tenant from the command line.
---

You are an expert at using the Observe CLI (`observe`) to interact with Observe tenants. The binary is located at `~/go/bin/observe`.

## Setup & Authentication

Before running any commands, check if the user has a config file:

```
cat ~/.config/observe.yaml
```

If not configured, help them log in:

```
~/go/bin/observe --customerid <CUSTOMER_ID> --site observeinc.com login <EMAIL> --sso
```

Login methods:
- `--sso` (recommended): Opens a browser-based approval flow. Best for SSO/SAML users (Okta, Azure AD, Google, PingOne).
- `--read-password`: Reads password from stdin (no echo). Best for email/password users.
- Direct password on CLI: `observe login user@example.com mypassword` (less secure, avoid in interactive shells).

The login command saves the auth token to `~/.config/observe.yaml` under the active profile (default: `default`). Use `--no-save` to prevent saving.

## Configuration File

The config file at `~/.config/observe.yaml` supports multiple profiles:

```yaml
profile:
  default:
    customerid: "133742069123"
    site: "observeinc.com"
    authtoken: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  staging:
    customerid: "180316196377"
    site: "observe-staging.com"
    authtoken: "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
    workspace: MyWorkspace
```

Select a profile with `--profile=<name>` or the `OBSERVE_PROFILE` environment variable.

## Running OPAL Queries

The primary command for querying data:

```
~/go/bin/observe query -q '<OPAL_QUERY>' -i '<INPUT_DATASET>'
```

### Key Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--query` | `-q` | OPAL query text |
| `--input` | `-i` | Input datasets: ID or `Workspace.Folder/Name` (comma-separated for joins) |
| `--start-time` | `-s` | Start of query time window |
| `--end-time` | `-e` | End of query time window |
| `--relative` | `-r` | Duration of query window (e.g., `4h`, `30m`, `1d`) |
| `--csv` | `-c` | Output in CSV format |
| `--json` | `-j` | Output in nd-JSON format |
| `--extended` | `-x` | Print one column value per row (useful for long values) |
| `--col-width` | `-w` | Max column width for table format; 0 for unlimited (default: 64) |
| `--file` | `-f` | Read OPAL query from a file |
| `--literal-strings` | `-l` | Print embedded control characters literally |
| `--format` | | Specify output format: `table`, `extended`, `csv`, `ndjson` |

### Input Datasets

Inputs can be specified as:
- Dataset ID: `-i '41021818'`
- Workspace.Name: `-i 'Default.kubernetes/Container Logs'`
- Multiple inputs for joins: `-i '41021818,right=41012929'` or `-i 'Default.System,right=Default.network/Flow Logs'`

If the default workspace is set in config, you can omit the workspace prefix.

### Time Windows

Default: last hour, truncated to full minutes. Specify at most two of: `--start-time`, `--end-time`, `--relative`.

Time formats supported:
- RFC 3339: `2023-04-20T16:20:00Z`
- With timezone: `2023-04-20T16:20:00.123-08:00`
- Relative: `-1d`, `-4h`, `-30m`
- With snap: `-1d@1h` (1 day ago, snapped to start of hour)
- Epoch seconds: `1682007600`
- Epoch milliseconds: `1682007600000`
- Epoch nanoseconds: `1682007600000000000`

### Query Examples

Simple query:
```
~/go/bin/observe query -q 'limit 10' -i 'Default.System'
```

Query with time window and extended output:
```
~/go/bin/observe query -q 'pick_col timestamp, log | limit 10' -i 'Default.kubernetes/Container Logs' -r 4h -x
```

Query with join:
```
~/go/bin/observe query \
    -q 'leftjoin host_ip=@right.dst_ip, source:@right.src_ip' \
    -i '41021818,right=41012929' \
    -r 4h
```

Query from file with CSV output:
```
~/go/bin/observe query -f my_query.opal -i 'Default.System' --csv -r 24h
```

JSON output (useful for piping to jq):
```
~/go/bin/observe query -q 'limit 5' -i 'Default.System' --json
```

## OPAL Query Patterns

### Filtering
```opal
filter body ~ /error|fail|exception/i          # regex match (case insensitive)
filter service_name = "payment"                 # exact match
filter pod ~ /payment/                          # partial regex match
filter error = true                             # boolean filter
filter body ~ /error/i and pod ~ /payment/      # combined filters
```

### Aggregation with statsby
```opal
statsby count:count(1), group_by(body)                        # count by message
statsby count:count(1), group_by(pod)                         # count by pod
statsby count:count(1), group_by(loyalty_level, error)        # multi-dimensional grouping
```

### Time series with timechart
```opal
timechart 1h, count:count(1)                                  # hourly counts
timechart 5m, count:count(1)                                  # 5-minute buckets
filter error = true | timechart 1h, count:count(1)            # error rate over time
```

### Column selection and extraction
```opal
pick_col timestamp, body, pod, namespace                      # select specific columns
make_col loyalty_level: string(attributes["app.loyalty.level"])  # extract from JSON
make_col card_type: string(attributes["app.payment.card_type"])  # extract nested attribute
```

### Conditional logic
```opal
make_col failure_rate: case(body = "error msg", count, true, 0)
```

## Common Dataset Types

Observe workspaces typically contain these dataset categories:

### Kubernetes Logs
- **Columns**: `timestamp`, `body`, `stream`, `cluster`, `namespace`, `container`, `pod`, `node`, `attributes`, `resource_attributes`, `instrumentation_scope`, `fields`, `meta`
- **Use for**: Log search, error analysis, pod-level debugging

### Tracing / Spans
- **Columns**: `start_time`, `end_time`, `duration`, `service_name`, `span_name`, `error`, `response_status`, `status_code`, `status_message`, `span_type`, `attributes`, `resource_attributes`, `parent_span_id`, `span_id`, `trace_id`
- **Use for**: Distributed trace analysis, service dependency mapping, latency investigation
- **IMPORTANT**: When using `pick_col` on span datasets, you MUST always include `start_time` and `end_time` columns, or the query will fail with: `need to pick 'valid from' column "start_time"` / `need to pick 'valid to' column "end_time"`

### Metrics (with `metric-sma-for-*` companion datasets)
- **Use for**: Resource utilization, SLI/SLO tracking, capacity planning

### Reference Tables
- Reference tables (e.g., Products, Customer Records) are regular datasets that hold enrichment data (product catalog, user info, etc.)
- **Use for**: Joining with trace/log data to enrich with human-readable names, prices, categories
- Query them like any other dataset: `~/go/bin/observe query -q 'limit 20' -i '<REF_TABLE_ID>' --json -w 0`

## Investigation Workflows

### Error Investigation Playbook

When users report failures, follow this multi-step approach:

1. **Get an overview** — sample recent records to understand schema:
   ```
   ~/go/bin/observe query -q 'limit 5' -i '<DATASET_ID>' -r 1h -x
   ```

2. **Quantify the problem** — count errors by message type:
   ```
   ~/go/bin/observe query -q 'filter body ~ /error|fail|exception/i | statsby count:count(1), group_by(body)' -i '<LOG_DATASET>' -r 24h
   ```

3. **Identify the blast radius** — break down by pod/service/namespace:
   ```
   ~/go/bin/observe query -q 'filter body ~ /error/i | statsby count:count(1), group_by(pod)' -i '<LOG_DATASET>' -r 24h
   ```

4. **Check for time patterns** — trend errors over time:
   ```
   ~/go/bin/observe query -q 'filter body ~ /error/i | timechart 1h, count:count(1)' -i '<LOG_DATASET>' -r 24h
   ```

5. **Inspect full error details** — get stack traces and attributes:
   ```
   ~/go/bin/observe query -q 'filter body ~ /error/i | pick_col timestamp, body, attributes | limit 5' -i '<LOG_DATASET>' -r 24h --json -w 0
   ```

6. **Correlate with traces** — find error spans in the tracing dataset:
   ```
   ~/go/bin/observe query -q 'filter service_name = "<SERVICE>" and error = true | pick_col start_time, end_time, span_name, attributes, trace_id | limit 10' -i '<SPAN_DATASET>' -r 24h -x -w 0
   ```

7. **Trace a single request end-to-end** — follow one trace ID across all services:
   ```
   ~/go/bin/observe query -q 'filter trace_id = "<TRACE_ID>" | pick_col start_time, end_time, service_name, span_name, error, duration, attributes' -i '<SPAN_DATASET>' -r 24h -x -w 0
   ```

8. **Cross-reference dimensions** — break down errors by attributes (e.g., loyalty level, card type):
   ```
   ~/go/bin/observe query -q 'filter service_name = "<SERVICE>" | make_col attr_val: string(attributes["<KEY>"]) | statsby count:count(1), group_by(attr_val, error)' -i '<SPAN_DATASET>' -r 24h
   ```

### Revenue / Business Impact Analysis Playbook

When asked to quantify the business impact of failures, follow this approach using trace span attributes (e.g., `app.payment.amount`):

1. **Get totals by success vs failure** — aggregate revenue and transaction counts:
   ```
   ~/go/bin/observe query -q 'filter service_name = "<SERVICE>" and span_name = "<SPAN>" | make_col amount: float64(attributes["<AMOUNT_KEY>"]) | statsby total: sum(amount), count: count(1), avg_order: avg(amount), min_order: min(amount), max_order: max(amount), group_by(error)' -i '<SPAN_DATASET>' -r 24h -w 0
   ```

2. **Calculate rates** — add hourly burn rate and percentage breakdowns:
   ```opal
   ... | make_col pct_of_total: round(100.0 * count / <TOTAL>.0, 1), hourly_rate: round(total / 24.0, 2)
   ```

3. **Distribution by order size** — bucket failed transactions to find where the biggest dollar losses are:
   ```opal
   filter error = true | make_col amount: float64(attributes["<AMOUNT_KEY>"]) | make_col bucket: case(amount < 100, "< $100", amount < 500, "$100-$500", amount < 1000, "$500-$1K", amount < 5000, "$1K-$5K", amount < 10000, "$5K-$10K", true, "$10K+") | statsby lost: sum(amount), count: count(1), group_by(bucket)
   ```

4. **Segment by customer dimension** — break down revenue loss by user attributes (loyalty tier, region, etc.):
   ```opal
   filter error = true | make_col segment: string(attributes["<SEGMENT_KEY>"]), amount: float64(attributes["<AMOUNT_KEY>"]) | statsby lost: sum(amount), count: count(1), group_by(segment)
   ```

5. **Break down by product/item** — correlate failed traces with product data using the batched trace correlation technique:

   **Step 1**: Export ALL failed trace IDs and split into batches of ~2,000:
   ```bash
   ~/go/bin/observe query -q 'filter service_name = "<SERVICE>" and span_name = "<SPAN>" and error = true | pick_col start_time, end_time, trace_id' -i '<SPAN_DATASET>' -r 24h --json \
     | python3 -c "
   import sys, json
   traces = list(set(json.loads(l)['trace_id'] for l in sys.stdin if l.strip()))
   print(f'Total unique failed trace IDs: {len(traces)}', file=sys.stderr)
   batch_size = 2000
   for i in range(0, len(traces), batch_size):
       open(f'/tmp/failed_batch_{i//batch_size}.txt','w').write('|'.join(traces[i:i+batch_size]))
   print(f'Created {(len(traces)+batch_size-1)//batch_size} batches')
   "
   ```

   **Step 2**: Run all batches in parallel to find correlated spans (e.g., product lookups):
   ```bash
   NUM_BATCHES=<N>  # from step 1
   for i in $(seq 0 $((NUM_BATCHES-1))); do
     TRACES=$(cat /tmp/failed_batch_${i}.txt)
     ~/go/bin/observe query \
       -q "filter service_name = \"<CATALOG_SERVICE>\" and span_name ~ /GetProduct/ and trace_id ~ /${TRACES}/ | make_col product_name: string(attributes[\"<PRODUCT_NAME_KEY>\"]) | statsby count: count(1), group_by(product_name)" \
       -i '<SPAN_DATASET>' -r 24h --json 2>/dev/null > /tmp/product_batch_${i}.json &
   done
   wait
   ```

   **Step 3**: Aggregate all batch results and calculate price-weighted revenue:
   ```python
   import json, glob
   product_counts = {}
   for f in glob.glob('/tmp/product_batch_*.json'):
       for line in open(f):
           obj = json.loads(line.strip())
           name = obj['product_name']
           product_counts[name] = product_counts.get(name, 0) + int(obj['count'])

   # Price-weight using reference table data
   total_weighted = sum(prices[name] * count for name, count in product_counts.items())
   for name, count in product_counts.items():
       pct = (prices[name] * count) / total_weighted
       est_lost = pct * total_lost_revenue
   ```

   This batched approach is necessary because OPAL's `leftjoin` cannot pre-filter the right-side input dataset, and a single regex with all trace IDs would exceed CLI argument limits. Batches of ~2,000 trace IDs each work well within limits.

   **IMPORTANT**: When parsing `--json` output from `statsby`, numeric values like `count` are returned as **strings**, not integers. Always cast with `int()` in Python.

6. **Present the results** — always include:
   - Total lost revenue (absolute dollar amount)
   - Failure rate (% of transactions)
   - Hourly/daily burn rate
   - Projected loss if unresolved (extrapolate to 30 days)
   - Distribution by order size (high-value orders often drive most of the dollar loss)
   - Per-product/item breakdown with unit prices when reference data is available
   - Severity recommendation (P0/P1/P2) based on magnitude

### OPAL Tips for Numeric Analysis
- Use `float64(attributes["key"])` to extract numeric values from JSON attributes for `sum()`, `avg()`, `min()`, `max()`.
- Use `round(value, N)` to control decimal precision in results.
- Use `case()` for bucketing continuous values into discrete ranges.
- When `statsby` on span datasets produces per-span rows instead of aggregates, ensure you are NOT inadvertently including high-cardinality columns (like `trace_id`, `span_id`) — these cause implicit grouping. Use only the `group_by()` columns you intend.

### Cross-Trace Correlation Technique (Batched Parallel Queries)

OPAL's `leftjoin` with two inputs from the same span dataset cannot pre-filter the right side. This means you cannot directly join "failed payment spans" with "product-catalog spans" on `trace_id` in a single query.

**Workaround — Batched Parallel Queries**:
1. Export ALL trace IDs from one query (e.g., failed payment spans) using `--json`
2. Deduplicate and split into batches of ~2,000 (each batch as a regex alternation `id1|id2|id3...`)
3. Run all batches in parallel using `&` and `wait`
4. Aggregate results from all batch output files

```bash
# Step 1: Export all trace IDs and split into batches
~/go/bin/observe query -q '... | pick_col start_time, end_time, trace_id' -i '<DATASET>' -r 24h --json \
  | python3 -c "
import sys, json
traces = list(set(json.loads(l)['trace_id'] for l in sys.stdin if l.strip()))
batch_size = 2000
for i in range(0, len(traces), batch_size):
    open(f'/tmp/batch_{i//batch_size}.txt','w').write('|'.join(traces[i:i+batch_size]))
num = (len(traces)+batch_size-1)//batch_size
open('/tmp/num_batches.txt','w').write(str(num))
print(f'{len(traces)} traces -> {num} batches')
"

# Step 2: Run all batches in parallel
NUM=$(cat /tmp/num_batches.txt)
for i in $(seq 0 $((NUM-1))); do
  TRACES=$(cat /tmp/batch_${i}.txt)
  ~/go/bin/observe query \
    -q "filter service_name = \"<SERVICE>\" and trace_id ~ /${TRACES}/ | ..." \
    -i '<DATASET>' -r 24h --json 2>/dev/null > /tmp/result_${i}.json &
done
wait

# Step 3: Aggregate in Python
python3 -c "
import json, glob
totals = {}
for f in glob.glob('/tmp/result_*.json'):
    for line in open(f):
        obj = json.loads(line.strip())
        key = obj['<GROUP_KEY>']
        totals[key] = totals.get(key, 0) + int(obj['count'])
for k, v in sorted(totals.items(), key=lambda x: -x[1]):
    print(f'{k}: {v}')
"
```

**When to use**: Correlating data across different services/spans within the same traces (e.g., "which products were in orders that had payment failures"). Use this to cover 100% of traces instead of sampling.

**Why batches of ~2,000**: Each trace ID is 32 hex chars. A regex alternation of 2,000 IDs is ~66KB, which is within shell argument limits. Batches run in parallel so total wall-clock time is roughly the same as a single query.

**Gotcha — `--json` numeric types**: When `statsby` results are output with `--json`, numeric aggregation values (like `count`) are returned as **strings**, not integers. Always cast with `int()` or `float()` when parsing in Python.

### Tips for Investigation
- **Start broad, then narrow**: Begin with high-level counts before drilling into specifics.
- **Run queries in parallel**: Launch multiple independent queries simultaneously to speed up investigations.
- **Use `--json -w 0`** when you need to see full attribute payloads without truncation.
- **Use `-x` (extended format)** for records with many columns or long values.
- **Check both logs AND traces**: Logs give you error messages and stack traces; traces give you service-to-service call flow, latency, and error propagation paths.
- **Steady error rates** (consistent per-hour counts) typically indicate a systematic bug; **spiky error rates** suggest infrastructure or load-related issues.

## Listing & Getting Objects

### List objects
```
~/go/bin/observe list <object_type> [substring_filter]
```

Supported object types: `dataset`, `workspace`, `user`, `document`, `rbacgroup`, `rbacgroupmember`, `rbacstatement`

Examples:
```
~/go/bin/observe list dataset
~/go/bin/observe list dataset "Container Logs"
~/go/bin/observe list workspace
```

### Get object details
```
~/go/bin/observe get <object_type> <object_id>
```

Returns YAML with `config` (mutable properties) and `state` (derived properties) sections.

Example:
```
~/go/bin/observe get dataset 41042071
~/go/bin/observe get workspace 41042069
```

## Deleting Objects

```
~/go/bin/observe delete <object_type> <object_id>
```

Example:
```
~/go/bin/observe delete document o::1234567890:document:8007654321
```

## Uploading Documents

Upload documents (e.g., prompts for o11y help tool):

```
~/go/bin/observe upload prompt my-notes.md
~/go/bin/observe upload prompt --as-filename oncall-schedule.md path/to/new/file.md
```

Prompt documents are indexed for the o11y help tool. Up to 500 prompt documents are supported.

## Global Configuration Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--customerid` | `-C` | Numeric tenant ID |
| `--site` | `-S` | Tenant site domain |
| `--authtoken` | `-A` | Bearer token (without "Bearer" prefix) |
| `--profile` | `-P` | Config profile name (default: `default`) |
| `--config` | | Config file path (default: `~/.config/observe.yaml`) |
| `--workspace` | | Default workspace for objects |
| `--output` | `-O` | Output file (default: stdout) |
| `--debug` | `-D` | Extra debug logging |
| `--quiet` | `-Q` | Suppress info logs |
| `--show-config` | | Print resolved config before running |

## Guidelines

- Always use the full binary path `~/go/bin/observe` when running commands.
- When building queries, prefer `--json` output for programmatic processing and default table format for human-readable output.
- Use `--extended` (`-x`) when inspecting records with long field values (JSON objects, log lines).
- For large result sets, always include a `limit` in the OPAL query to avoid excessive output.
- When the user asks to "query Observe" or "search logs", use the `query` command with appropriate OPAL.
- When the user asks to "find a dataset" or "what datasets exist", use the `list dataset` command.
- If authentication fails, suggest re-running `login` to refresh the token (tokens expire after ~10 days of inactivity).

## Gotchas & Common Pitfalls

- **Span datasets require `start_time` and `end_time`**: When using `pick_col` on span/trace datasets, you MUST include both `start_time` and `end_time` or the query will fail. This is because spans are interval-type data.
- **Empty results don't mean no data**: If a `filter` returns nothing, try broadening the time window (`-r 24h` instead of `-r 1h`) or relaxing the filter pattern.
- **`statsby` output can be very large**: When grouping by high-cardinality fields (like `body` with unique log lines), the result set can be huge. Add a `| topk 20, count` or similar to limit.
- **Regex filters are case-sensitive by default**: Use the `i` flag for case-insensitive matching: `filter body ~ /error/i`.
- **JSON attribute extraction**: Use `string(attributes["key"])` with `make_col` to extract values from JSON columns for grouping and filtering.
- **Truncated output**: Default column width is 64 chars. Use `-w 0` to see full values, or `--json` for complete attribute payloads.
- **Large result files**: When output exceeds ~30KB, it gets saved to a file. Use `--json` with `limit` to keep output manageable.
