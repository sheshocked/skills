---
name: automl-tabular
description: 
category: ai-tools
tags: [automl-tabular]
---

## When to Use
Build tabular ML: feature engineering, gradient boosting, AutoML, SHAP explainability.

## Quick Start (LightGBM)
```python
import lightgbm as lgb
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

train_data = lgb.Dataset(X_train, label=y_train)
params = {"objective": "binary", "metric": "auc", "num_leaves": 31}
model = lgb.train(params, train_data, num_boost_round=100)

# SHAP explainability
import shap
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)
```

## AutoML Options
- **AutoGluon**: Best for quick baselines
- **H2O**: Enterprise-grade AutoML
- **TPOT**: Genetic algorithm-based

## Pitfalls
- **Data leakage**: Don't use future data for training
- **Feature engineering**: Domain knowledge beats AutoML
- **Class imbalance**: Use appropriate metrics (AUC, F1)

## Verification
- Cross-validate with proper CV splits
- Check feature importance
- Test on holdout set