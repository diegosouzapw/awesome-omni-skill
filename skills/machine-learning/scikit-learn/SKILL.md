---
name: scikit-learn
description: The industry standard library for machine learning in Python. Provides simple and efficient tools for predictive data analysis, covering classification, regression, clustering, dimensionality reduction, model selection, and preprocessing.
version: 1.4
license: BSD-3-Clause
---

# scikit-learn - Machine Learning in Python

A robust library for classical machine learning. It features a uniform API: all objects share the same interface for fitting, transforming, and predicting.

## When to Use

- **Classification**: Detecting categories (Spam vs. Ham, Disease diagnosis).
- **Regression**: Predicting continuous values (House prices, Stock trends).
- **Clustering**: Grouping similar objects (Market segmentation, Image compression).
- **Dimensionality Reduction**: Reducing feature count while keeping info (PCA, Visualization).
- **Model Selection**: Comparing models and tuning hyperparameters (Cross-validation, Grid search).
- **Preprocessing**: Transforming raw data into features (Scaling, Encoding, Imputation).

## Reference Documentation

**Official docs**: https://scikit-learn.org/stable/  
**User Guide**: https://scikit-learn.org/stable/user_guide.html  
**Search patterns**: `sklearn.pipeline.Pipeline`, `sklearn.model_selection`, `sklearn.ensemble`, `sklearn.preprocessing`

## Core Principles

### The "Estimator" Interface

- **Estimators**: Implement `fit(X, y)`. They learn from data.
- **Transformers**: Implement `transform(X)` (and `fit_transform(X)`). They modify data.
- **Predictors**: Implement `predict(X)`. They provide estimates for new data.

### Use scikit-learn For

- Tabular data (Excel-like, CSVs).
- Traditional ML (Random Forests, SVMs, Linear Models).
- Feature engineering and pipeline automation.
- Small to medium-sized datasets.

### Do NOT Use For

- Deep Learning / Neural Networks (use PyTorch or TensorFlow).
- Natural Language Processing at scale (use spaCy or HuggingFace).
- Large-scale "Big Data" (use Spark MLlib or Dask-ML).
- Real-time streaming predictions (consider specialized inference engines).

## Quick Reference

### Installation

```bash
pip install scikit-learn
```

### Standard Imports

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.metrics import classification_report, mean_squared_error
```

### Basic Pattern - Train/Predict

```python
from sklearn.ensemble import RandomForestClassifier

# 1. Prepare data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 2. Instantiate and fit
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# 3. Predict and evaluate
predictions = model.predict(X_test)
print(classification_report(y_test, predictions))
```

## Critical Rules

### ✅ DO

- **Split before anything** - Always use `train_test_split` before looking at data properties.
- **Use Pipelines** - Combine preprocessing and modeling to prevent data leakage.
- **Scale your data** - Models like SVM, KNN, and Linear Regression require feature scaling.
- **Check for Imbalance** - Use `stratify=y` in `train_test_split` for classification.
- **Cross-Validate** - Don't trust a single train/test split; use `cross_val_score`.
- **Handle Missing Values** - Use `SimpleImputer` or similar before fitting models.
- **Standardize Categories** - Use `OneHotEncoder` for nominal or `OrdinalEncoder` for ordinal data.

### ❌ DON'T

- **Fit on test data** - Never call `.fit()` or `.fit_transform()` on the test set.
- **Use Categorical data as-is** - Scikit-learn requires numerical input; encode strings first.
- **Ignore Class Imbalance** - Accuracy is misleading for imbalanced datasets; use F1-score or AUC.
- **Overfit** - Don't keep tuning hyperparameters until the test score is perfect.
- **Ignore Random State** - Set `random_state` for reproducibility during experiments.

## Anti-Patterns (NEVER)

```python
# ❌ BAD: Data Leakage (Fitting scaler on the whole dataset)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X) # Data from "future" test set leaks into training!
X_train, X_test, y_train, y_test = train_test_split(X_scaled, y)

# ✅ GOOD: Fit scaler only on training data
X_train, X_test, y_train, y_test = train_test_split(X, y)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test) # Use training mean/std

# ❌ BAD: Repeating preprocessing manually
# (Error-prone and hard to maintain)

# ✅ GOOD: Use Pipelines (Automates everything safely)
pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('model', RandomForestClassifier())
])
pipe.fit(X_train, y_train)
```

## Preprocessing (sklearn.preprocessing)

### Scaling and Encoding

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, OneHotEncoder

# Scaling numerical data
scaler = StandardScaler()
X_num_scaled = scaler.fit_transform(X_numeric)

# Encoding categorical data
encoder = OneHotEncoder(sparse_output=False, handle_unknown='ignore')
X_cat_encoded = encoder.fit_transform(X_categorical)

# Handling missing values
from sklearn.impute import SimpleImputer
imputer = SimpleImputer(strategy='median')
X_imputed = imputer.fit_transform(X_with_nan)
```

### Column Transformer (The Pro Way)

```python
from sklearn.compose import ColumnTransformer

numeric_features = ['age', 'salary']
categorical_features = ['city', 'job_type']

preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), numeric_features),
        ('cat', OneHotEncoder(), categorical_features)
    ])

# Now use this in a pipeline
pipeline = Pipeline([
    ('prep', preprocessor),
    ('clf', LogisticRegression())
])
```

## Classification

### Common Algorithms

```python
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.ensemble import GradientBoostingClassifier

# Logistic Regression (Baseline)
log_reg = LogisticRegression(max_iter=1000)

# Support Vector Machine
svm = SVC(kernel='rbf', probability=True)

# Gradient Boosting
gb = GradientBoostingClassifier(n_estimators=100, learning_rate=0.1)
```

## Regression

### Common Algorithms

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.ensemble import RandomForestRegressor

# Regularized Linear Models
ridge = Ridge(alpha=1.0) # L2
lasso = Lasso(alpha=0.1) # L1

# Non-linear Regression
rf_reg = RandomForestRegressor(n_estimators=100, max_depth=10)
```

## Model Evaluation

### Metrics

```python
from sklearn.metrics import accuracy_score, f1_score, roc_auc_score, r2_score, mean_absolute_error

# Classification
acc = accuracy_score(y_true, y_pred)
f1 = f1_score(y_true, y_pred, average='weighted')

# Regression
r2 = r2_score(y_true, y_pred)
mae = mean_absolute_error(y_true, y_pred)
```

### Cross-Validation

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(pipeline, X, y, cv=5, scoring='f1_macro')
print(f"Mean F1: {scores.mean():.4f} (+/- {scores.std():.4f})")
```

## Hyperparameter Tuning

### Grid Search and Randomized Search

```python
from sklearn.model_selection import GridSearchCV

param_grid = {
    'clf__n_estimators': [50, 100, 200],
    'clf__max_depth': [None, 10, 20],
    'clf__min_samples_split': [2, 5]
}

grid_search = GridSearchCV(pipeline, param_grid, cv=3, scoring='accuracy', n_jobs=-1)
grid_search.fit(X_train, y_train)

print(f"Best params: {grid_search.best_params_}")
best_model = grid_search.best_estimator_
```

## Dimensionality Reduction

### PCA (Principal Component Analysis)

```python
from sklearn.decomposition import PCA

# Reduce to 2 components for visualization
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

print(f"Explained variance ratio: {pca.explained_variance_ratio_}")
```

## Clustering

### K-Means and DBSCAN

```python
from sklearn.cluster import KMeans, DBSCAN

# K-Means (Requires specifying K)
kmeans = KMeans(n_clusters=3, n_init='auto')
clusters = kmeans.fit_predict(X)

# DBSCAN (Density-based, finds K automatically)
dbscan = DBSCAN(eps=0.5, min_samples=5)
clusters = dbscan.fit_predict(X)
```

## Practical Workflows

### 1. End-to-End Classification Pipeline

```python
def build_and_train_model(X, y):
    # 1. Identify types
    num_cols = X.select_dtypes(include=['int64', 'float64']).columns
    cat_cols = X.select_dtypes(include=['object', 'category']).columns

    # 2. Setup Preprocessing
    preprocessor = ColumnTransformer([
        ('num', StandardScaler(), num_cols),
        ('cat', OneHotEncoder(handle_unknown='ignore'), cat_cols)
    ])

    # 3. Create Pipeline
    clf = Pipeline([
        ('preprocessor', preprocessor),
        ('classifier', RandomForestClassifier(random_state=42))
    ])

    # 4. Train
    X_train, X_test, y_train, y_test = train_test_split(X, y, stratify=y)
    clf.fit(X_train, y_train)
    
    return clf, X_test, y_test

# model, X_test, y_test = build_and_train_model(df.drop('target', axis=1), df['target'])
```

### 2. Custom Feature Engineering (Transformer)

```python
from sklearn.base import BaseEstimator, TransformerMixin

class LogTransformer(BaseEstimator, TransformerMixin):
    def __init__(self, columns=None):
        self.columns = columns
    
    def fit(self, X, y=None):
        return self
    
    def transform(self, X):
        X_copy = X.copy()
        for col in self.columns:
            X_copy[col] = np.log1p(X_copy[col])
        return X_copy
```

## Performance Optimization

### Using n_jobs

```python
# Use all CPU cores for training/tuning
model = RandomForestClassifier(n_jobs=-1)
grid = GridSearchCV(model, param_grid, n_jobs=-1)
```

### Working with Large Data (partial_fit)

```python
from sklearn.linear_model import SGDClassifier

# Online learning (incremental fit)
model = SGDClassifier()
for X_chunk, y_chunk in data_stream:
    model.partial_fit(X_chunk, y_chunk, classes=np.unique(y_all))
```

## Common Pitfalls and Solutions

### Imbalanced Classes

```python
# ❌ Problem: Model predicts only the majority class
# ✅ Solution: Adjust class weights
model = RandomForestClassifier(class_weight='balanced')
# OR use SMOTE from imbalanced-learn library
```

### Convergence Warnings

```python
# ❌ Problem: "ConvergenceWarning: Liblinear failed to converge"
# ✅ Solution: Increase max_iter or scale data
model = LogisticRegression(max_iter=2000)
# Often solved by applying StandardScaler first!
```

### Categorical Values in Test Set not in Train

```python
# ❌ Problem: ValueError when unseen categories appear in test
# ✅ Solution: Use handle_unknown in OneHotEncoder
encoder = OneHotEncoder(handle_unknown='ignore')
```

Scikit-learn is the backbone of Python ML. Its API is so successful that many other libraries (XGBoost, LightGBM) mimic it.
