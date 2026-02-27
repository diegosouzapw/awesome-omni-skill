---
name: data-designer
description: Generate high-quality synthetic datasets using statistical samplers and Claude's native LLM capabilities. Use when users ask to create synthetic data, generate datasets, create fake/mock data, generate test data, training data, or any data generation task. Supports CSV, JSON, JSONL, Parquet output. Adapted from NVIDIA NeMo DataDesigner (Apache 2.0).
---

# Data Designer

Generate synthetic datasets combining statistical samplers with Claude's LLM capabilities. No external API keys required.

## Workflow

1. **Clarify requirements** - Ask about purpose, columns, size, format
2. **Create schema** - Write `dataset_schema.json` defining columns
3. **Generate preview** - Run `batch_generator.py` for 3-5 rows
4. **Iterate** - Refine based on feedback
5. **Generate full dataset** - Batch generate, then merge
6. **Deliver** - Export to requested format

## Column Types

### Statistical Samplers (No LLM)

| Type | Description | Key Params |
|------|-------------|------------|
| `category` | Weighted random choice | `values`, `weights` |
| `subcategory` | Hierarchical (parent-based) | `mapping`, `category` |
| `uniform` | Uniform distribution | `low`, `high`, `dtype` |
| `gaussian` | Normal distribution | `mean`, `std`, `min_val`, `max_val` |
| `bernoulli` | Binary probability | `p`, `true_value`, `false_value` |
| `poisson` | Poisson distribution | `mean` |
| `datetime` | Random dates | `start`, `end`, `format` |
| `person` | Synthetic personas | `fields`, `age_range`, `locale` |
| `uuid` | Unique IDs | `prefix`, `format` |

### LLM Columns (Claude generates)

| Type | Description |
|------|-------------|
| `llm_text` | Free-form text |
| `llm_code` | Code with syntax validation |
| `llm_structured` | JSON matching schema |
| `llm_judge` | Quality scoring |

## Schema Format

Create `dataset_schema.json`:

```json
{
  "name": "dataset_name",
  "seed": 42,
  "columns": [
    {"name": "category", "type": "category", "params": {"values": ["A","B"], "weights": [0.6,0.4]}},
    {"name": "text", "type": "llm_text", "prompt": "Write about {{ category }}.", "depends_on": ["category"]}
  ],
  "output": {"format": "csv", "filename": "output"}
}
```

For full schema reference: [references/schema.md](references/schema.md)

## Jinja2 Templating

Reference columns in prompts:

```
Write a {{ rating }}-star review for {{ product_name }} by {{ customer.first_name }}.
```

Supports: `{{ var }}`, `{{ obj.field }}`, `{% if %}`, filters

## Scripts

### Generate Data

```bash
# Preview
python scripts/batch_generator.py --schema schema.json --rows 5 --output preview.json --preview

# Full generation
python scripts/batch_generator.py --schema schema.json --rows 100 --batch-size 20 --output batches/
```

### Merge & Export

```bash
python scripts/merger.py --input batches/ --output dataset.csv --flatten
```

Formats: `csv`, `json`, `jsonl`, `parquet`

## Generation Strategy

1. **Sampler columns first** - Python scripts, fast
2. **LLM columns in dependency order** - Topological sort by `depends_on`
3. **Batch processing** - Generate in batches of 20-50 for large datasets

For LLM columns, Claude generates directly:
- Render Jinja2 prompt with row data
- Generate content
- Validate if configured
- Retry on failure (max 3)

## Examples

**Simple:**
> "Generate 50 product reviews with ratings 1-5"

**Complex:**
> "Create 200 support tickets with: ticket_id (UUID), customer (name, email), category (billing/technical/general), priority (1-5 gaussian), description (LLM)"

**Code:**
> "Generate 100 Python functions with description, code (validated), tests"

## Tips

- Use `seed` for reproducibility
- Preview first, then scale
- Keep LLM prompts specific
- Use `subcategory` for correlated data

## Attribution

Adapted from [NVIDIA NeMo DataDesigner](https://github.com/NVIDIA-NeMo/DataDesigner) (Apache 2.0).
