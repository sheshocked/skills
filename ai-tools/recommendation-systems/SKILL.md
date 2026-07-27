---
name: recommendation-systems
description: 
category: ai-tools
tags: [recommendation-systems]
---

## When to Use
Build recommendation systems: collaborative filtering, two-tower models, ranking, cold start.

## Approaches
1. **Collaborative filtering**: User-item interactions
2. **Content-based**: Item features
3. **Hybrid**: Combine both
4. **Two-tower**: Neural embedding approach

## Quick Start (Surprise library)
```python
from surprise import Dataset, SVD
from surprise.model_selection import cross_validate

data = Dataset.load_builtin('ml-100k')
algo = SVD(n_factors=50)
cross_validate(algo, data, measures=['RMSE', 'MAE'], cv=5)
```

## Pitfalls
- **Cold start**: New users/items have no interaction data
- **Popularity bias**: Popular items dominate recommendations
- **Scalability**: Matrix factorization doesn't scale to billions

## Verification
- Offline metrics: RMSE, NDCG, MAP
- Online metrics: CTR, conversion rate
- A/B testing for production validation