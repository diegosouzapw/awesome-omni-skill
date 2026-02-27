---
name: polars-bio
description: A high-performance skill for processing genomic data (VCF, FASTA, BED) using polars-bio. Features streaming VCF processing, interval joins, FASTA analysis, and variant annotation.
license: Apache-2.0
---

# Polars-Bio Skill

## Overview

This skill leverages `polars-bio` and `polars` to provide a robust toolkit for large-scale genomic data analysis. It is designed to handle datasets larger than memory using Polars' streaming engine and lazy evaluation.

## Capabilities

### 1. VCF Processing
*   **Lazy Loading:** Use `pb.scan_vcf()` to define execution plans without loading data into memory immediately.
*   **Streaming Conversion:** Efficiently convert VCFs to Parquet (`sink_parquet`) for faster downstream queries.
*   **Filtering & Cleaning:** Parse INFO fields and filter variants using Polars expressions.

### 2. Genomic Interval Operations
*   **Overlap Joins:** High-performance interval joins (finding variants within genomic regions like Cytobands or Genes) using `pb.overlap()`.

### 3. Sequence Analysis
*   **FASTA Processing:** Lazy reading of FASTA files (`pb.scan_fasta`) to calculate sequence metrics (e.g., GC content, sequence length).

### 4. Variant Annotation
*   **Database Integration:** Annotate VCF variants with external datasets (e.g., gnomAD, dbSNP) using efficient point-joins on `chrom`, `start`, `ref`, `alt`.

## Quick Start

### Basic VCF Loading & Interval Join

```python
import polars as pl
import polars_bio as pb

# 1. Lazy Load VCF
vcf_lf = pb.scan_vcf("data/clinvar.vcf.gz")

# 2. Load Regions (e.g., BED file)
regions_lf = pl.scan_csv("data/regions.bed", separator="\t", has_header=False, 
                         new_columns=["chrom", "start", "end", "name"])

# 3. Perform Interval Overlap
# Finds variants in VCF that overlap with regions
joined_lf = pb.overlap(vcf_lf, regions_lf)

# 4. Execute (Streaming)
result = joined_lf.collect(streaming=True)
print(result)
```

### Convert VCF to Parquet

```python
# Efficiently convert large VCF to Parquet without high memory usage
pb.scan_vcf("input.vcf").sink_parquet("output.parquet")
```

### FASTA GC Content Analysis

```python
lf = pb.scan_fasta("genome.fa")
lf = lf.with_columns(
    pl.col("sequence").str.count_matches("G|C").alias("gc_count"),
    pl.col("sequence").str.len_chars().alias("len")
)
print(lf.select(pl.col("gc_count").sum() / pl.col("len").sum()).collect())
```

## Benchmarks

The skill includes a benchmark comparing **Eager** vs. **Streaming** execution. Streaming mode has been shown to reduce memory usage by **~33%** on standard workloads. See `ANALYSIS_REPORT.md` for details.

## Dependencies

- `polars`
- `polars-bio`
- `pyarrow` (for Parquet)