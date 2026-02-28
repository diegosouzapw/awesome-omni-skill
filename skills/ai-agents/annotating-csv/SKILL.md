---
name: annotating-csv
description: Annotates CSV rows using OpenAI by applying a prompt to each row and adding a new column with AI-generated results. Use when the user wants to classify, extract, summarize, or add AI-generated annotations to CSV data. Triggers on mentions of CSV annotation, row-by-row AI processing, bulk classification, or adding AI columns to spreadsheet data.
---

# CSV Annotation with OpenAI

Adds AI-generated annotations to CSV files by processing each row with a prompt.

## Quick Start

```bash
uv run annotate-csv.py "Classify sentiment as positive/negative/neutral" reviews.csv
```

Output: `reviews_annotated.csv` with new `annotation` column.

Dependencies install automatically on first run.

## Core Usage

```bash
uv run annotate-csv.py <prompt> <csv_file> [options]
```

| Option | Description | Default |
|--------|-------------|---------|
| `-o, --output` | Output file path | `<input>_annotated.csv` |
| `-c, --column` | Annotation column name | `annotation` |
| `--context` | Columns to use (`all` or comma-separated) | `all` |
| `--model` | OpenAI model | `gpt-5-mini` |
| `--parallelism` | Parallel workers | `10` |
| `--id-column` | Column for progress display | - |

## Prompt Sources

Prompts can be inline or from a file:

```bash
# Inline prompt
python annotate-csv.py "Extract the main topic" data.csv

# From file (for complex prompts)
python annotate-csv.py prompt.txt data.csv
```

## Common Patterns

**Classification:**
```bash
python annotate-csv.py "Classify sentiment: positive, negative, or neutral" feedback.csv -c sentiment
```

**Extraction:**
```bash
python annotate-csv.py "Extract the product name mentioned" reviews.csv -c product --context "review_text"
```

**Summarization:**
```bash
python annotate-csv.py "Summarize in one sentence" articles.csv -c summary
```

**See [EXAMPLES.md](EXAMPLES.md) for more patterns and prompt templates.**

## Environment Setup

Requires `OPENAI_API_KEY` in `.env` file:
```
OPENAI_API_KEY=sk-...
```

Run with uv (recommended):
```bash
uv run annotate-csv.py "Your prompt" data.csv
```

Or install dependencies first:
```bash
uv pip install -r requirements.txt
python annotate-csv.py "Your prompt" data.csv
```

## Workflow

1. **Preview data**: Check CSV structure and columns
2. **Choose context**: Decide which columns the AI needs to see
3. **Write prompt**: Be specific about expected output format
4. **Test on subset**: Try on a few rows first if large dataset
5. **Run full annotation**: Process the complete file
6. **Validate results**: Spot-check annotation quality

## Writing Effective Prompts

- Be explicit about output format (single word, phrase, sentence)
- List valid categories for classification tasks
- Specify what to do with ambiguous cases
- Keep prompts focused on one task

**See [PROMPTS.md](PROMPTS.md) for prompt-writing guidance.**

## Troubleshooting

| Issue | Solution |
|-------|----------|
| API key error | Check `.env` file has `OPENAI_API_KEY` |
| Column not found | Verify column names match CSV exactly |
| Rate limits | Reduce `--parallelism` to 5 or lower |
| Empty annotations | Check context columns have data |
