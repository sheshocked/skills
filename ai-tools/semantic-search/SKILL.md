---
name: semantic-search
description: 
category: ai-tools
tags: [semantic-search]
---

## When to Use
Build semantic search: embedding indexing, query expansion, re-ranking, analytics.

## Pipeline
```
Documents → Embedding → Vector Index
                            ↓
Query → Embedding → Similarity Search → Re-rank → Results
```

## Key Patterns
```python
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer('all-MiniLM-L6-v2')

# Index
doc_embeddings = model.encode(documents)

# Search
query_embedding = model.encode([query])
scores = np.dot(doc_embeddings, query_embedding.T)
top_k = np.argsort(scores[0])[-5:][::-1]
```

## Pitfalls
- **Embedding model choice**: Match model to your domain
- **Index updates**: Handle document additions/deletions
- **Query expansion**: Add synonyms for better recall
- **Re-ranking**: Use cross-encoder for precision

## Verification
- Measure recall@k
- Test with diverse queries
- Check latency under load