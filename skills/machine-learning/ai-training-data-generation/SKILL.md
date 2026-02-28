---
name: ai-training-data-generation
description: Generate high-quality training datasets from documents, text corpora, and structured content. Use when creating AI training data from dictionaries, documents, or when generating examples for machine learning models. Optimized for low-resource languages and domain-specific knowledge extraction.
---

# AI Training Data Generation

## Overview

A comprehensive skill for automatically generating high-quality training datasets from documents, text corpora, and structured content. Optimized for low-resource languages, dictionary content, and domain-specific knowledge extraction.

## Capabilities

- **Multi-strategy Generation**: Dictionary pairs, contextual definitions, completion tasks, classification examples
- **Quality Filtering**: Confidence scoring, duplicate removal, and content validation
- **Format Flexibility**: Support for multiple AI training formats (JSONL, HuggingFace, Ollama, OpenAI)
- **Language Awareness**: Multi-language support with special handling for accented characters
- **Scalable Processing**: Generate thousands of examples from large documents
- **Balance Management**: Ensure dataset diversity and prevent category imbalance

## Core Strategies

### 1. Dictionary Pair Extraction

Extract word-definition pairs from structured and semi-structured text.

**Detection Patterns**:

- Separator-based: `word – definition`, `term: meaning`
- Linguistic indicators: `means`, `is defined as`, `refers to`
- Structural cues: Indentation, formatting, list structures
- Context analysis: Surrounding text for validation

### 2. Implementation Pattern

```python
from .ai_training_generator import AITrainingDataGenerator

# Initialize generator
generator = AITrainingDataGenerator(min_confidence=0.7)

# Generate comprehensive training data
training_data = generator.generate_comprehensive_training_data(
    parsed_document,
    target_count=10000
)

# Export in multiple formats
files = generator.export_training_data(
    training_data,
    output_dir="training_output",
    format_type="ollama"
)
```

## Output Format Examples

### JSONL Format (Standard)

```json
{"input": "What does 'ááfengen' mean?", "output": "very good, excellent", "type": "dictionary_pair", "confidence": 0.95}
```

### Ollama Format

```json
{"prompt": "Translate this Chuukese word: ngang", "response": "fish", "system": "You are a Chuukese-English translator."}
```

### HuggingFace Format

```json
{"text": "### Instruction:\nWhat does 'chomong' mean in Chuukese?\n\n### Response:\nto help, assist"}
```

### OpenAI Fine-tuning Format

```json
{"messages": [{"role": "user", "content": "Define: kúún"}, {"role": "assistant", "content": "to go, to leave"}]}
```

## Quality Assurance

- **Content validity**: Does the example make linguistic sense?
- **Pattern matching**: Does it follow expected language patterns?
- **Context appropriateness**: Is the context relevant and helpful?
- **Uniqueness**: Avoid repetitive or duplicate content

## Best Practices

1. **Multiple validation passes**: Automated and manual quality checks
2. **Confidence thresholds**: Adjust based on use case requirements
3. **Human review sampling**: Periodic manual validation of generated examples
4. **Balance management**: Ensure even distribution across categories

## Dependencies

- `re`: Regular expression pattern matching
- `json`: Data serialization and export
- `hashlib`: Duplicate detection and content hashing
- `collections`: Data structure utilities and counting
