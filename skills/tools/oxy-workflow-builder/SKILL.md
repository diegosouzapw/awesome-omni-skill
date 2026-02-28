---
name: oxy-workflow-builder
description: Build Oxy workflows, SQL queries, and agents following best practices. Use when the user asks to create data pipelines, queries, or analysis agents. Enforces hierarchy - semantic queries first, then SQL/workflows, then agents.
---

# Oxy Workflow Builder

You are an expert at building Oxy data workflows, SQL queries, and AI agents. Your role is to help users extract insights from data using the right tool for the job, following a clear hierarchy of approaches.

## The Oxy Hierarchy (CRITICAL)

When solving data analysis problems, ALWAYS follow this hierarchy:

### 1. Semantic Queries (PREFERRED)
**Use semantic queries whenever possible** - they're the most maintainable and business-friendly approach.

- **When to use**: The semantic layer has views/topics covering the needed data
- **How**: Use natural language queries against the semantic engine
- **Why preferred**:
  - No SQL knowledge required
  - Automatic joins across views
  - Business-friendly terminology
  - Maintained centrally in semantic layer

**Before writing SQL or agents, ALWAYS check if semantic layer views exist that can answer the question.**

### 2. SQL Queries & Workflows (FALLBACK)
**Use SQL when semantic layer doesn't cover your needs** - you need custom logic or the data isn't in semantic layer yet.

- **When to use**:
  - Data not yet in semantic layer
  - Complex transformations or calculations
  - ETL/data pipeline operations
  - Need parameterized queries

- **SQL Files** (`*.sql`):
  - Single query execution
  - Support Jinja2 templating for parameters
  - Can be dry-run tested

- **Workflow Files** (`*.workflow.yml`):
  - Multi-step data pipelines
  - Orchestrate multiple queries
  - Transform and load data

```bash
# Run SQL query
oxy run query.sql

# Run with parameters
oxy run query.sql -v year=2024 -v month=12

# Dry run to test
oxy run query.sql --dry-run

# Run workflow
oxy run pipeline.workflow.yml
```

### 3. AI Agents (LAST RESORT)
**Use agents only when you need AI reasoning** - they're the most flexible but least deterministic.

- **When to use**:
  - Need natural language understanding
  - Complex analysis requiring reasoning
  - Exploratory data analysis
  - Dynamic query generation based on data

- **Agent Files** (`*.agent.yml`):
  - Require a question/prompt to run
  - Can access databases and tools
  - Use LLMs for reasoning

```bash
# Run agent with a question
oxy run analysis.agent.yml "What are the trends in customer behavior?"
```

## Decision Tree

Use this decision tree when the user asks for data analysis:

```
Does semantic layer have the data?
├─ YES → Use semantic queries (#1)
└─ NO → Is this a deterministic query/pipeline?
    ├─ YES → Use SQL/Workflow (#2)
    └─ NO → Need AI reasoning?
        ├─ YES → Use Agent (#3)
        └─ NO → Build semantic layer views first, then use semantic queries
```

## Essential Commands

```bash
# Validation
oxy validate                    # Validate all YAML configs (agents, workflows, apps)
oxy build                       # Validate semantic layer

# Semantic queries
oxy semantic-engine --dev-mode  # Start semantic engine for queries

# SQL execution
oxy run query.sql               # Run SQL file
oxy run query.sql --dry-run     # Test without executing
oxy run query.sql -v key=value  # Run with variables

# Workflows
oxy run pipeline.workflow.yml   # Run workflow

# Agents
oxy run agent.agent.yml "question"  # Run agent with prompt

# Discovery
find . -name "*.sql" -not -path "*/.*"
find . -name "*.workflow.yml"
find . -name "*.agent.yml"
find semantics/views -name "*.view.yml"
find semantics/topics -name "*.topic.yml"
```

## SQL File Structure

SQL files support Jinja2 templating for dynamic queries:

```sql
-- query.sql
-- Description: Brief description of what this query does
-- Variables:
--   - start_date: Start date for the date range
--   - end_date: End date for the date range

SELECT
    date,
    customer_id,
    SUM(amount) as total_amount
FROM {{ databases.clickhouse.schema }}.orders
WHERE date BETWEEN '{{ start_date }}' AND '{{ end_date }}'
GROUP BY date, customer_id
ORDER BY total_amount DESC;
```

### SQL Best Practices

1. **Add header comments** with description and required variables
2. **Use Jinja2 variables** for parameterization: `{{ variable_name }}`
3. **Reference databases** via context: `{{ databases.db_name.schema }}.table`
4. **Test with dry-run** before executing: `oxy run query.sql --dry-run`
5. **Keep queries focused** - one clear purpose per file
6. **Name descriptively** - `monthly_revenue_by_restaurant.sql` not `query1.sql`

### Common SQL Patterns

**Date filtering with variables:**
```sql
WHERE created_at >= '{{ start_date }}'
  AND created_at < '{{ end_date }}'
```

**Dynamic schema references:**
```sql
FROM {{ databases.clickhouse.restaurant_analytics }}.orders
```

**Conditional logic:**
```sql
{% if include_cancelled %}
WHERE status IN ('completed', 'cancelled')
{% else %}
WHERE status = 'completed'
{% endif %}
```

## Workflow File Structure

Workflows orchestrate multi-step data operations:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/workflow.json

name: my_workflow
description: "What this workflow accomplishes"

tasks:
  - name: step_1
    type: execute_sql
    database: my_database
    sql_query: |
      SELECT * FROM source_table
      WHERE condition = true

  - name: step_2
    type: execute_sql
    database: my_database
    sql_query: |
      SELECT
        column1,
        SUM(column2) as total
      FROM {{ step_1 }}
      GROUP BY column1

  - name: step_3
    type: execute_sql
    database: my_database
    sql_query: |
      INSERT INTO destination_table
      SELECT * FROM {{ step_2 }}
```

### Workflow Best Practices

1. **Name tasks descriptively** - clear purpose for each task
2. **Add descriptions** - explain what each task accomplishes
3. **Reference previous task outputs** - `{{ task_name }}`
4. **Keep workflows focused** - single pipeline per file
5. **Test incrementally** - validate each task's SQL separately first
6. **Document dependencies** - note what data/tables are required

### Common Workflow Patterns

**ETL Pipeline:**
```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/workflow.json

tasks:
  - name: extract
    type: execute_sql
    database: my_database
    sql_query: SELECT * FROM source_table WHERE date = '{{ date }}'

  - name: transform
    type: execute_sql
    database: my_database
    sql_query: |
      SELECT
        TRIM(name) as name,
        CAST(amount as DECIMAL(10,2)) as amount
      FROM {{ extract }}

  - name: load
    type: execute_sql
    database: my_database
    sql_query: |
      INSERT INTO analytics.processed_data
      SELECT * FROM {{ transform }}
```

**Aggregation Pipeline:**
```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/workflow.json

tasks:
  - name: daily_totals
    type: execute_sql
    database: my_database
    sql_query: |
      SELECT date, SUM(amount) as total
      FROM transactions
      GROUP BY date

  - name: moving_average
    type: execute_sql
    database: my_database
    sql_query: |
      SELECT
        date,
        total,
        AVG(total) OVER (
          ORDER BY date
          ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) as moving_avg_7d
      FROM {{ daily_totals }}
```

## Agent File Structure

Agents use AI for analysis requiring reasoning:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/agent.json

name: my_agent
description: "What this agent analyzes"

model: "claude-3-5-sonnet-20241022"  # Or other supported models

system_instructions: |
  You are a data analyst specializing in [domain].

  Your role is to:
  - Analyze the provided data
  - Identify patterns and insights
  - Provide actionable recommendations

  Be concise and focus on business impact.

tools:
  - type: database
    database: clickhouse
    description: "Access to transactional database"

  - type: python
    description: "For calculations and data manipulation"

# Optional: Pre-load context
context:
  - type: sql
    query: |
      SELECT * FROM summary_table
      WHERE date >= CURRENT_DATE - INTERVAL 30 DAY
```

### Agent Best Practices

1. **Clear system prompt** - define the agent's expertise and role
2. **Specific tools** - only include tools the agent needs
3. **Focused purpose** - one type of analysis per agent
4. **Pre-load context** - provide relevant data upfront when possible
5. **Test with real questions** - validate with actual use cases
6. **Document expected questions** - add examples in description

### Common Agent Patterns

**Trend Analysis Agent:**
```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/agent.json

name: trend_analyzer
description: "Analyzes trends and forecasts future patterns"

system_instructions: |
  You are a trend analysis expert. Given time-series data:
  1. Identify significant trends (growth, decline, seasonality)
  2. Highlight anomalies or outliers
  3. Provide forecasts when appropriate
  4. Explain business implications

tools:
  - type: database
    database: clickhouse
  - type: python
    description: "For statistical analysis"
```

**Customer Insights Agent:**
```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/agent.json

name: customer_insights
description: "Analyzes customer behavior and segments"

system_instructions: |
  You are a customer analytics expert. Analyze customer data to:
  - Segment customers by behavior
  - Identify high-value customers
  - Spot churn risks
  - Recommend retention strategies

tools:
  - type: database
    database: clickhouse
```

## Building Process

### Before Starting: Check the Hierarchy

1. **Check semantic layer first**:
   ```bash
   # List available views
   find semantics/views -name "*.view.yml"

   # List available topics
   find semantics/topics -name "*.topic.yml"

   # If views exist, use semantic queries!
   oxy semantic-engine --dev-mode
   ```

2. **If semantic layer insufficient**:
   - Missing data? Consider building views first (use oxy-semantic-layer skill)
   - Custom logic needed? Proceed to SQL/workflow

3. **If SQL/workflow insufficient**:
   - Need AI reasoning? Proceed to agents
   - Otherwise, keep using SQL

### Step 1: Understand Requirements

- What data is needed?
- Is it available in semantic layer?
- Is the query deterministic or needs reasoning?
- Are there variables/parameters?
- Is it a one-off query or repeatable pipeline?

### Step 2: Choose the Right Tool

Based on the hierarchy:
1. **Semantic query** - if data is in semantic layer
2. **SQL file** - if it's a single deterministic query
3. **Workflow** - if it's a multi-step pipeline
4. **Agent** - if AI reasoning is required

### Step 3: Build Incrementally

**For SQL:**
1. Write the query with parameter placeholders
2. Add header comments (description, variables)
3. Test with `--dry-run`
4. Run with actual parameters
5. Verify results

**For Workflows:**
1. Design the pipeline (extract → transform → load)
2. Write SQL for each step
3. Test each step's SQL separately first
4. Combine into workflow file
5. Run the complete workflow
6. Verify final output

**For Agents:**
1. Define the agent's purpose and expertise
2. Write a clear system prompt
3. Add necessary tools (database, python, etc.)
4. Test with sample questions
5. Refine based on results

### Step 4: Validate and Test

```bash
# Validate all YAML configs (ALWAYS run after creating/editing YAML)
oxy validate

# Or validate a single file
oxy validate --file=my_workflow.workflow.yml

# For SQL: dry-run first
oxy run query.sql --dry-run

# Then run for real
oxy run query.sql -v param=value

# For workflows: run and monitor
oxy run workflow.workflow.yml

# For agents: test with real questions
oxy run agent.agent.yml "Your test question"
```

## Quality Guidelines

### Naming Conventions

- Use `snake_case` for all file names
- Be descriptive and specific
- Include the purpose: `monthly_revenue_report.sql`, `customer_etl_pipeline.workflow.yml`
- Avoid generic names: `query1.sql`, `agent.agent.yml`

### Documentation

- **SQL files**: Header comments with description and variables
- **Workflows**: Description for workflow and each step
- **Agents**: Clear description of what the agent analyzes

### Testing

- **Always validate generated YAML first**: `oxy validate` (or `oxy validate --file=<path>` for a single file)
- **SQL files**: Use `--dry-run` before executing
- **Test incrementally**: One step at a time for workflows
- **Real questions**: Test agents with actual use cases

### Organization

- Group related files in directories
- Separate by domain: `revenue/`, `customers/`, `operations/`
- Keep semantic layer separate: `semantics/views/`, `semantics/topics/`

## Common Patterns by Use Case

### Reporting (Use Semantic Queries!)
If views exist, use semantic engine:
```bash
oxy semantic-engine --dev-mode
# Then query: "Show me total revenue by month this year"
```

If views don't exist, create them first (oxy-semantic-layer skill), then use semantic queries.

### Data Pipeline (Use Workflow)
```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/workflow.json

name: daily_aggregation_pipeline
tasks:
  - name: extract_daily_data
    type: execute_sql
    database: my_database
    sql_query: SELECT * FROM raw_data WHERE date = '{{ date }}'
  - name: aggregate
    type: execute_sql
    database: my_database
    sql_query: SELECT category, SUM(amount) FROM {{ extract_daily_data }} GROUP BY category
  - name: load
    type: execute_sql
    database: my_database
    sql_query: INSERT INTO aggregated_data SELECT * FROM {{ aggregate }}
```

### Parameterized Query (Use SQL)
```sql
-- monthly_report.sql
-- Variables: year, month
SELECT
    customer_id,
    COUNT(*) as order_count,
    SUM(amount) as total_amount
FROM orders
WHERE YEAR(date) = {{ year }}
  AND MONTH(date) = {{ month }}
GROUP BY customer_id;
```

### Exploratory Analysis (Use Agent)
```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/oxy-hq/oxy/refs/heads/main/json-schemas/agent.json

name: data_explorer
description: "Explores data to find insights"
system_instructions: |
  You are a data explorer. Examine the data and:
  1. Summarize key statistics
  2. Identify interesting patterns
  3. Suggest areas for deeper analysis
tools:
  - type: database
    database: clickhouse
  - type: python
```

## Troubleshooting

### SQL File Issues

**Error: Variable not defined**
- Cause: Missing variable in command
- Fix: Add `-v variable_name=value` to command

**Error: Table not found**
- Cause: Incorrect table reference
- Fix: Check schema in `.databases/` directory

**Error: SQL syntax error**
- Cause: Invalid SQL
- Fix: Test with `--dry-run`, check SQL syntax for your database

### Workflow Issues

**Error: Task result not found**
- Cause: Reference to non-existent task
- Fix: Check task names match exactly: `{{ task_name }}`

**Error: Task fails partway**
- Cause: SQL error in one task
- Fix: Test each task's SQL separately first

### Agent Issues

**Error: Agent requires prompt**
- Cause: No question provided
- Fix: Add question: `oxy run agent.agent.yml "Your question"`

**Error: Tool not available**
- Cause: Referenced tool not configured
- Fix: Check database names in config.yml, verify tool types

## Remember the Hierarchy!

When a user asks for data analysis:

1. **Check semantic layer first** - Can we answer with semantic queries?
2. **Use SQL/workflow if needed** - Is it deterministic logic?
3. **Use agents only when necessary** - Does it require AI reasoning?

The best solution is the simplest solution that works. Don't use agents when SQL will do. Don't use SQL when semantic queries will do.

## DeepWiki Fallback

For Oxy features not covered here, query DeepWiki with:

"I am a user of this project, not its maintainer. Please prioritize looking at the project docs, examples and json-schemas to answer my question: [your question]"

Only search `oxy-hq/oxy` repository.

## Documentation Links

- Oxy Documentation: https://docs.oxy.tech/
- Workflows: https://docs.oxy.tech/learn-about-oxy/automations
- SQL Queries: https://docs.oxy.tech/learn-about-oxy/queries
- Agents: https://docs.oxy.tech/learn-about-oxy/agents
- Semantic Layer: https://docs.oxy.tech/learn-about-oxy/semantic-layer
