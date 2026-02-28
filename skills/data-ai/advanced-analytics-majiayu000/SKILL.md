---
name: advanced-analytics
description: Advanced analytics including machine learning, predictive modeling, and big data techniques
version: "2.0.0"
sasmp_version: "2.0.0"
bonded_agent: 06-advanced-analytics-specialist
bond_type: PRIMARY_BOND

# Skill Configuration
config:
  atomic: true
  retry_enabled: true
  max_retries: 3
  backoff_strategy: exponential
  model_training_timeout: 3600

# Parameter Validation
parameters:
  skill_level:
    type: string
    required: true
    enum: [intermediate, advanced, expert]
    default: intermediate
  focus_area:
    type: string
    required: false
    enum: [regression, classification, clustering, timeseries, feature_engineering, all]
    default: all
  deployment_target:
    type: string
    required: false
    enum: [notebook, api, batch, realtime]
    default: notebook

# Observability
observability:
  logging_level: info
  metrics: [model_accuracy, training_time, prediction_latency, feature_importance]
  model_versioning: true
---

# Advanced Analytics Skill

## Overview
Master advanced analytics techniques including machine learning, predictive modeling, and big data processing for sophisticated data analysis.

## Core Topics

### Machine Learning Fundamentals
- Supervised vs unsupervised learning
- Classification algorithms (logistic regression, decision trees, random forest)
- Regression algorithms (linear, polynomial, ensemble methods)
- Clustering (K-means, hierarchical, DBSCAN)

### Predictive Analytics
- Time series forecasting (ARIMA, exponential smoothing)
- Customer segmentation and RFM analysis
- Churn prediction models
- A/B testing and experimentation

### Big Data Technologies
- Introduction to Spark and PySpark
- Data lakes and data mesh concepts
- Cloud analytics platforms (AWS, GCP, Azure)
- Real-time analytics with streaming data

### Advanced Techniques
- Feature engineering best practices
- Model validation and cross-validation
- Hyperparameter tuning
- Model deployment considerations

## Learning Objectives
- Build and validate machine learning models
- Implement predictive analytics solutions
- Work with big data technologies
- Apply advanced statistical techniques

## Error Handling

| Error Type | Cause | Recovery |
|------------|-------|----------|
| Overfitting | Model too complex | Add regularization, reduce features |
| Underfitting | Model too simple | Add features, increase complexity |
| Data leakage | Target info in features | Review feature engineering pipeline |
| Class imbalance | Skewed target | Use SMOTE, class weights, or resampling |
| Convergence failure | Poor hyperparameters | Grid search, adjust learning rate |

## Related Skills
- statistics (for foundational statistical knowledge)
- programming (for ML implementation)
- databases-sql (for big data querying)
