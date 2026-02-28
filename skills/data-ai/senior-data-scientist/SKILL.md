---

# === CORE IDENTITY ===
name: senior-data-scientist
title: Senior Data Scientist Skill Package
description: World-class data science skill for statistical modeling, experimentation, causal inference, and advanced analytics. Expertise in Python (NumPy, Pandas, Scikit-learn), R, SQL, statistical methods, A/B testing, time series, and business intelligence. Includes experiment design, feature engineering, model evaluation, and stakeholder communication. Use when designing experiments, building predictive models, performing causal analysis, or driving data-driven decisions.
domain: engineering
subdomain: data-engineering

# === WEBSITE DISPLAY ===
difficulty: advanced
time-saved: "TODO: Quantify time savings"
frequency: "TODO: Estimate usage frequency"
use-cases:
  - Designing data pipelines for ETL/ELT processes
  - Building data warehouses and data lakes
  - Implementing data quality and governance frameworks
  - Creating analytics dashboards and reporting

# === RELATIONSHIPS ===
related-agents: []
related-skills: []
related-commands: []
orchestrated-by: []

# === TECHNICAL ===
dependencies:
  scripts: []
  references: []
  assets: []
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack: [Python 3.8+, Markdown]

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for senior-data-scientist"
    output: "TODO: Add expected output"

# === ANALYTICS ===
stats:
  downloads: 0
  stars: 0
  rating: 0.0
  reviews: 0

# === VERSIONING ===
version: v1.0.0
author: Claude Skills Team
contributors: []
created: 2025-10-20
updated: 2025-11-23
license: MIT

# === DISCOVERABILITY ===
tags:
  - analysis
  - analytics
  - data
  - design
  - engineering
  - scientist
  - senior
  - testing
featured: false
verified: true
---


# Senior Data Scientist

World-class senior data scientist skill for production-grade AI/ML/Data systems.

## Overview

This skill provides world-class data science capabilities through three core Python automation tools and comprehensive reference documentation. Whether designing experiments, building predictive models, performing causal inference, or driving data-driven decisions, this skill delivers expert-level statistical modeling and analytics solutions.

Senior data scientists use this skill for A/B testing, experiment design, statistical modeling, causal inference, time series analysis, feature engineering, model evaluation, and business intelligence. Expertise covers Python (NumPy, Pandas, Scikit-learn), R, SQL, statistical methods, hypothesis testing, and advanced analytics techniques.

**Core Value:** Accelerate analytics and experimentation by 65%+ while improving model accuracy, statistical rigor, and business impact through proven methodologies and automated pipelines.

## Quick Start

### Main Capabilities

```bash
# Core Tool 1
python scripts/experiment_designer.py --input data/ --output results/

# Core Tool 2  
python scripts/feature_engineering_pipeline.py --target project/ --analyze

# Core Tool 3
python scripts/model_evaluation_suite.py --config config.yaml --deploy
```

## Core Capabilities

- **Experiment Design & A/B Testing** - Statistical power analysis, sample size calculation, multi-armed bandits, sequential testing
- **Statistical Modeling** - Regression, classification, time series, causal inference, Bayesian methods
- **Feature Engineering** - Automated feature generation, selection, transformation, interaction terms, dimensionality reduction
- **Model Evaluation** - Cross-validation, hyperparameter tuning, bias-variance tradeoff, model interpretation (SHAP, LIME)
- **Business Analytics** - Customer segmentation, churn prediction, lifetime value, attribution modeling, forecasting
- **Causal Inference** - Propensity score matching, difference-in-differences, instrumental variables, regression discontinuity

## Python Tools

### 1. Experiment Designer

Design statistically rigorous experiments with power analysis.

**Key Features:**
- A/B test design with sample size calculation
- Statistical power analysis
- Multi-variant testing setup
- Sequential testing frameworks
- Bayesian experiment design

**Common Usage:**
```bash
# Design A/B test
python scripts/experiment_designer.py --effect-size 0.05 --power 0.8 --alpha 0.05

# Multi-variant test
python scripts/experiment_designer.py --variants 4 --mde 0.03 --output experiment_plan.json

# Sequential testing
python scripts/experiment_designer.py --sequential --stopping-rule obf

# Help
python scripts/experiment_designer.py --help
```

**Use Cases:**
- Designing product experiments before launch
- Calculating required sample sizes
- Planning sequential testing strategies

### 2. Feature Engineering Pipeline

Automate feature generation, selection, and transformation.

**Key Features:**
- Automated feature generation (polynomial, interaction terms)
- Feature selection (mutual information, recursive elimination)
- Encoding (one-hot, target, frequency)
- Scaling and normalization
- Dimensionality reduction (PCA, t-SNE, UMAP)

**Common Usage:**
```bash
# Generate features
python scripts/feature_engineering_pipeline.py --input data.csv --generate --interactions

# Feature selection
python scripts/feature_engineering_pipeline.py --input data.csv --select --top-k 20

# Full pipeline
python scripts/feature_engineering_pipeline.py --input data.csv --pipeline full --output features.csv

# Help
python scripts/feature_engineering_pipeline.py --help
```

**Use Cases:**
- Preparing features for model training
- Reducing feature dimensionality
- Discovering important feature interactions

### 3. Model Evaluation Suite

Comprehensive model evaluation with interpretability.

**Key Features:**
- Cross-validation strategies (k-fold, stratified, time-series)
- Hyperparameter optimization (grid search, random search, Bayesian)
- Model interpretation (SHAP values, feature importance, partial dependence)
- Performance metrics (accuracy, precision, recall, F1, AUC, MAE, RMSE)
- Model comparison and statistical testing

**Common Usage:**
```bash
# Evaluate model
python scripts/model_evaluation_suite.py --model model.pkl --data test.csv --metrics all

# Hyperparameter tuning
python scripts/model_evaluation_suite.py --model sklearn.ensemble.RandomForestClassifier --tune --data train.csv

# Model interpretation
python scripts/model_evaluation_suite.py --model model.pkl --interpret --shap

# Help
python scripts/model_evaluation_suite.py --help
```

**Use Cases:**
- Comparing multiple model architectures
- Finding optimal hyperparameters
- Explaining model predictions to stakeholders

See [statistical_methods_advanced.md](references/statistical_methods_advanced.md) for comprehensive tool documentation and advanced examples.

## Core Expertise

This skill covers world-class capabilities in:

- Advanced production patterns and architectures
- Scalable system design and implementation
- Performance optimization at scale
- MLOps and DataOps best practices
- Real-time processing and inference
- Distributed computing frameworks
- Model deployment and monitoring
- Security and compliance
- Cost optimization
- Team leadership and mentoring

## Tech Stack

**Languages:** Python, SQL, R, Scala, Go
**ML Frameworks:** PyTorch, TensorFlow, Scikit-learn, XGBoost
**Data Tools:** Spark, Airflow, dbt, Kafka, Databricks
**LLM Frameworks:** LangChain, LlamaIndex, DSPy
**Deployment:** Docker, Kubernetes, AWS/GCP/Azure
**Monitoring:** MLflow, Weights & Biases, Prometheus
**Databases:** PostgreSQL, BigQuery, Snowflake, Pinecone

## Key Workflows

### 1. A/B Test Design and Analysis

**Time:** 2-3 hours for design, ongoing for analysis

1. **Define Hypothesis** - State null and alternative hypotheses, success metrics
2. **Design Experiment** - Calculate sample size, randomization strategy
   ```bash
   # Design A/B test with power analysis
   python scripts/experiment_designer.py --effect-size 0.05 --power 0.8 --alpha 0.05 --output test_plan.json
   ```
3. **Run Experiment** - Implement randomization, collect data
4. **Analyze Results** - Statistical significance testing, confidence intervals
5. **Report Findings** - Effect size, business impact, recommendations

See [experiment_design_frameworks.md](references/experiment_design_frameworks.md) for detailed methodology.

### 2. Predictive Model Development

**Time:** 1-2 days for initial model, ongoing refinement

1. **Exploratory Data Analysis** - Understand distributions, correlations, missing data
2. **Feature Engineering** - Generate and select features
   ```bash
   # Automated feature engineering
   python scripts/feature_engineering_pipeline.py --input data.csv --pipeline full --output features.csv
   ```
3. **Model Training** - Train multiple model types (linear, tree-based, neural nets)
4. **Model Evaluation** - Cross-validation, hyperparameter tuning
   ```bash
   # Evaluate and tune model
   python scripts/model_evaluation_suite.py --model sklearn.ensemble.RandomForestClassifier --tune --data train.csv
   ```
5. **Model Interpretation** - SHAP values, feature importance, business insights

### 3. Causal Inference Analysis

**Time:** 3-5 hours for setup and analysis

1. **Define Causal Question** - Treatment, outcome, confounders
2. **Select Method** - Propensity score matching, diff-in-diff, instrumental variables
3. **Implement Analysis** - Control for confounders, estimate treatment effect
4. **Validate Assumptions** - Check overlap, parallel trends, instrument validity
5. **Report Causal Estimates** - Average treatment effect, confidence intervals, sensitivity analysis

See [statistical_methods_advanced.md](references/statistical_methods_advanced.md) for causal inference techniques.

### 4. Time Series Forecasting

**Time:** 4-6 hours for model development

1. **Data Preparation** - Handle missing values, detect seasonality, stationarity tests
2. **Feature Engineering** - Lag features, rolling statistics, external variables
   ```bash
   # Generate time series features
   python scripts/feature_engineering_pipeline.py --input timeseries.csv --temporal --lags 7,14,30
   ```
3. **Model Selection** - ARIMA, Prophet, LSTM, XGBoost for time series
4. **Cross-Validation** - Time-series split, walk-forward validation
5. **Forecast & Monitor** - Generate forecasts, track accuracy over time

## Reference Documentation

### 1. Statistical Methods Advanced

Comprehensive guide available in `references/statistical_methods_advanced.md` covering:

- Advanced patterns and best practices
- Production implementation strategies
- Performance optimization techniques
- Scalability considerations
- Security and compliance
- Real-world case studies

### 2. Experiment Design Frameworks

Complete workflow documentation in `references/experiment_design_frameworks.md` including:

- Step-by-step processes
- Architecture design patterns
- Tool integration guides
- Performance tuning strategies
- Troubleshooting procedures

### 3. Feature Engineering Patterns

Technical reference guide in `references/feature_engineering_patterns.md` with:

- System design principles
- Implementation examples
- Configuration best practices
- Deployment strategies
- Monitoring and observability

## Production Patterns

### Pattern 1: Scalable Data Processing

Enterprise-scale data processing with distributed computing:

- Horizontal scaling architecture
- Fault-tolerant design
- Real-time and batch processing
- Data quality validation
- Performance monitoring

### Pattern 2: ML Model Deployment

Production ML system with high availability:

- Model serving with low latency
- A/B testing infrastructure
- Feature store integration
- Model monitoring and drift detection
- Automated retraining pipelines

### Pattern 3: Real-Time Inference

High-throughput inference system:

- Batching and caching strategies
- Load balancing
- Auto-scaling
- Latency optimization
- Cost optimization

## Best Practices

### Development

- Test-driven development
- Code reviews and pair programming
- Documentation as code
- Version control everything
- Continuous integration

### Production

- Monitor everything critical
- Automate deployments
- Feature flags for releases
- Canary deployments
- Comprehensive logging

### Team Leadership

- Mentor junior engineers
- Drive technical decisions
- Establish coding standards
- Foster learning culture
- Cross-functional collaboration

## Performance Targets

**Latency:**
- P50: < 50ms
- P95: < 100ms
- P99: < 200ms

**Throughput:**
- Requests/second: > 1000
- Concurrent users: > 10,000

**Availability:**
- Uptime: 99.9%
- Error rate: < 0.1%

## Security & Compliance

- Authentication & authorization
- Data encryption (at rest & in transit)
- PII handling and anonymization
- GDPR/CCPA compliance
- Regular security audits
- Vulnerability management

## Common Commands

```bash
# Development
python -m pytest tests/ -v --cov
python -m black src/
python -m pylint src/

# Training
python scripts/train.py --config prod.yaml
python scripts/evaluate.py --model best.pth

# Deployment
docker build -t service:v1 .
kubectl apply -f k8s/
helm upgrade service ./charts/

# Monitoring
kubectl logs -f deployment/service
python scripts/health_check.py
```

## Resources

- Advanced Patterns: `references/statistical_methods_advanced.md`
- Implementation Guide: `references/experiment_design_frameworks.md`
- Technical Reference: `references/feature_engineering_patterns.md`
- Automation Scripts: `scripts/` directory

## Senior-Level Responsibilities

As a world-class senior professional:

1. **Technical Leadership**
   - Drive architectural decisions
   - Mentor team members
   - Establish best practices
   - Ensure code quality

2. **Strategic Thinking**
   - Align with business goals
   - Evaluate trade-offs
   - Plan for scale
   - Manage technical debt

3. **Collaboration**
   - Work across teams
   - Communicate effectively
   - Build consensus
   - Share knowledge

4. **Innovation**
   - Stay current with research
   - Experiment with new approaches
   - Contribute to community
   - Drive continuous improvement

5. **Production Excellence**
   - Ensure high availability
   - Monitor proactively
   - Optimize performance
   - Respond to incidents
