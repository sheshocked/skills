---
name: data-cleaning-pipelines
description: 
category: general
tags: [data-cleaning-pipelines]
---

## When to Use
Clean messy data: deduplication, normalization, validation, ETL pipelines.

## Python Cleaning
```python
import pandas as pd

# Load
df = pd.read_csv("messy.csv")

# Remove duplicates
df = df.drop_duplicates()

# Normalize text
df["name"] = df["name"].str.strip().str.lower()

# Fill missing values
df["age"] = df["age"].fillna(df["age"].median())

# Validate
df = df[df["age"].between(0, 150)]

# Export
df.to_csv("clean.csv", index=False)
```

## Common Issues
| Issue | Solution |
|---|---|
| Duplicates | drop_duplicates() or fuzzy matching |
| Missing values | fillna(), dropna(), or imputation |
| Inconsistent text | str.strip(), str.lower() |
| Wrong types | astype(), pd.to_numeric() |
| Outliers | IQR method or domain rules |

## Pitfalls
- **Over-cleaning**: Don't remove valid edge cases
- **Data loss**: Keep original copy before cleaning
- **Validation**: Define rules before cleaning
- **Reproducibility**: Script the pipeline

## Verification
- Compare row counts before/after
- Check for remaining nulls
- Validate against known good data