---
name: ai-engineering-guide
description: Practical guide for building production ML systems based on Chip Huyen's AI Engineering book. Use when users ask about model evaluation, deployment strategies, monitoring, data pipelines, feature engineering, cost optimization, or MLOps. Covers metrics, A/B testing, serving patterns, drift detection, and production best practices.
---

# AI Engineering Guide

## Overview

**What this skill covers**

- Designing and scaling AI applications, including system architecture and user feedback mechanisms.
- Building and maintaining high-quality datasets for model training and finetuning.
- Evaluating AI systems with appropriate criteria and methods.
- Implementing effective evaluation methodologies for open-ended AI systems.
- Deciding between prompting, RAG, and finetuning, and configuring finetuning methods.
- Selecting and deploying foundation models, and optimizing model outputs.
- Optimizing LLM inference systems for latency and cost efficiency.
- Designing effective prompts and implementing safety measures in prompt engineering.
- Implementing RAG systems and designing AI agents.

**When to use this skill**

- When designing, scaling, and monitoring AI applications.
- When building, verifying, and maintaining datasets for model training.
- When selecting or evaluating AI systems and defining success metrics.
- When designing evaluations for open-ended AI systems.
- When deciding on and configuring finetuning methods.
- When selecting foundation models and planning compute resources.
- When optimizing inference systems for performance and cost.
- When constructing prompts and managing context in AI applications.
- When implementing RAG systems and designing AI agents.

## Reference Files Guide

### ai-engineering-architecture-and-user-feedback.md

**When to use:** Use this file for queries related to designing, scaling, and monitoring AI applications, implementing system architectures, and collecting user feedback. It is particularly useful for questions about decision frameworks, architecture progression, and feedback mechanisms in AI systems.

**Key topics:**

- System Architecture: Progression and components
- Context Enhancement: RAG and tools
- Guardrails: Input and output safety
- Model Routing: Routers and gateways
- Caching: Latency and cost reduction

**Example queries:**

- "How do I decide when to add new components to my AI system architecture?"
- "What are best practices for implementing guardrails in AI applications?"
- "How can I optimize caching to reduce latency and cost in AI models?"
- "What strategies should I use for collecting and utilizing user feedback in AI systems?"
- "When should I introduce an orchestrator in my AI pipeline?"

### dataset-engineering.md

**When to use:** Use this file when you need guidance on building, verifying, and maintaining high-quality datasets for pretraining or finetuning models. It is particularly useful for decisions involving dataset composition, synthetic data use, and data quality verification.

**Key topics:**

- Dataset Composition: Coverage strategy
- Finetuning Approaches: PEFT vs full finetuning
- Synthetic Data: Best practices and risks
- Data Quality: Verification and filtering methods
- Annotation Guidelines: Best practices

**Example queries:**

- "How do I decide between PEFT and full finetuning?"
- "What are best practices for using synthetic data?"
- "How can I verify the quality of my dataset?"
- "What should I include in annotation guidelines?"
- "How do I ensure dataset coverage for multiple languages?"

### evaluate-ai-systems.md

**When to use:** Use this file when designing, selecting, or evaluating AI systems, particularly when needing guidance on evaluation criteria, methods, and workflows for AI models.

**Key topics:**

- Evaluation Criteria: Defining success metrics
- Automation: Verification and monitoring
- Model Selection: Build vs buy decisions
- Safety: Screening for harmful content
- Instruction-Following: Adherence checks

**Example queries:**

- "How do I evaluate the success of my AI model?"
- "What criteria should I use to select an AI system?"
- "How can I automate the verification of AI outputs?"
- "What are the best practices for monitoring AI models in production?"
- "How do I ensure my AI system follows instructions accurately?"

### evaluation-methodology.md

**When to use:** Use this reference when designing, implementing, or operating evaluations for open-ended AI systems, especially when selecting evaluation methods, computing language modeling metrics, or using AI as a judge.

**Key topics:**

- Evaluation Methods: Choosing techniques
- Language Modeling: Perplexity and metrics
- Functional Correctness: Execution accuracy
- Similarity Evaluation: Lexical and semantic
- AI-as-Judge: LLM-based evaluation

**Example queries:**

- "How do I choose the right evaluation method for my AI model?"
- "What is perplexity and how do I compute it?"
- "How can I evaluate the functional correctness of generated code?"
- "What are the best practices for using AI as a judge in evaluations?"
- "How do I measure semantic similarity for text outputs?"

### finetuning.md

**When to use:** Use this file when you need guidance on deciding between prompting, retrieval-augmented generation (RAG), and finetuning, or when configuring and deploying finetuning methods like LoRA and QLoRA.

**Key topics:**

- Finetuning decision-making
- Hardware sizing and memory
- Method selection (LoRA, QLoRA)
- Hyperparameter configuration
- Model merging techniques

**Example queries:**

- "Should I use RAG or finetuning for my model?"
- "How do I configure LoRA for finetuning?"
- "What hardware do I need for finetuning a large model?"
- "How can I merge multiple finetuned models?"
- "What are the best practices for hyperparameter tuning in finetuning?"

### foundation-models.md

**When to use:** Use this file when you need guidance on selecting and deploying foundation models, planning compute resources, curating training data, or optimizing model outputs for specific tasks and domains.

**Key topics:**

- Model selection criteria
- Compute resource planning
- Training data strategy
- Post-training alignment
- Sampling and decoding techniques

**Example queries:**

- "How do I choose a model for non-English text processing?"
- "What are the best practices for planning compute resources for model training?"
- "How can I curate training data for a domain-specific model?"
- "What techniques can I use to align model outputs with human preferences?"
- "How do I optimize sampling parameters for reliable model outputs?"

### inference-optimization.md

**When to use:** Use this file when optimizing LLM inference systems for lower latency and cost, diagnosing bottlenecks, or implementing specific optimization techniques for model serving.

**Key topics:**

- Optimization Order: Step-by-step process
- Core Metrics: Latency and throughput
- Bottleneck Diagnosis: Compute vs bandwidth
- High-ROI Optimizations: Quantization, batching
- Parallelism Strategies: Scaling techniques

**Example queries:**

- "How can I reduce latency in LLM inference?"
- "What are the best practices for optimizing model throughput?"
- "How do I diagnose bottlenecks in my inference pipeline?"
- "What quantization techniques should I use for LLMs?"
- "How can I implement parallelism to scale my model serving?"

### introduction.md

**When to use:** Use this reference file when you need guidance on building AI applications using foundation models, including decision-making frameworks, evaluation techniques, and deployment strategies. It is particularly useful for queries related to adapting foundation models, selecting AI techniques, and optimizing AI application performance.

**Key topics:**

- Decision Frameworks: AI application viability
- Technique Selection: Prompting vs RAG vs Finetuning
- Evaluation Essentials: Metrics and datasets
- Deployment Strategies: Internal vs external-facing
- Inference Optimization: Latency and cost reduction

**Example queries:**

- "How do I decide whether to build or buy an AI solution?"
- "What are the best practices for evaluating AI model performance?"
- "How can I optimize the latency of my AI application?"
- "When should I use RAG over finetuning for my AI model?"
- "What metrics should I track for AI deployment success?"

### prompt-engineering.md

**When to use:** Use this file for queries related to designing effective prompts, optimizing prompt structures, and implementing safety measures in prompt engineering. It is particularly useful for tasks involving prompt construction, context management, and defensive strategies against prompt injection.

**Key topics:**

- Prompt Anatomy: Core components
- Chat Templates: Correctness and guardrails
- In-Context Learning: Zero-shot vs few-shot
- Context Efficiency: Length and structure
- Defensive Engineering: Threats and defenses

**Example queries:**

- "How do I structure a prompt for few-shot learning?"
- "What are best practices for managing long context in prompts?"
- "How can I prevent prompt injection attacks?"
- "What is the correct chat template format for my model?"
- "How do I implement chain-of-thought reasoning in prompts?"

### rag-and-agents.md

**When to use:** Use this reference file when dealing with queries related to implementing Retrieval-Augmented Generation (RAG) systems, optimizing retrieval algorithms, or designing and deploying AI agents. It is particularly useful for questions about retrieval strategies, agent architectures, and memory management in AI systems.

**Key topics:**

- RAG Pipeline: Essential components
- Retrieval Algorithms: Hybrid search
- Chunking Strategy: Defaults and variants
- Vector Search: Index selection
- Agent Architecture: Planning and execution

**Example queries:**

- "How do I implement a RAG system to reduce hallucinations?"
- "What are the best practices for hybrid search using BM25 and vector search?"
- "How should I design an AI agent to handle multi-step tasks?"
- "What chunking strategy should I use for optimal retrieval performance?"
- "How can I manage memory effectively in a RAG system?"
