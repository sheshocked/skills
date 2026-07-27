---
name: vector-databases
description: 
category: ai-tools
tags: [vector-databases]
---

## When to Use
Choose and operate vector databases: Qdrant, pgvector, Weaviate, Milvus for similarity search and semantic retrieval.

## Comparison
| Feature | Qdrant | pgvector | Weaviate |
|---|---|---|---|
| Setup | Standalone | PostgreSQL extension | Standalone |
| Filtering | Rich payload filters | SQL WHERE | GraphQL |
| Scalability | Horizontal | Vertical | Horizontal |
| Best for | Production | Existing PG | Complex schemas |

## Qdrant Setup
```python
from qdrant_client import QdrantClient
from qdrant_client.models import VectorParams, Distance, PointStruct

client = QdrantClient(":memory:")

# Create collection
client.create_collection(
    collection_name="docs",
    vectors_config=VectorParams(size=1536, distance=Distance.COSINE),
)

# Insert
client.upsert(
    collection_name="docs",
    points=[
        PointStruct(id=1, vector=[0.1]*1536, payload={"text": "hello", "source": "doc1"}),
    ]
)

# Search
results = client.search(
    collection_name="docs",
    query_vector=[0.1]*1536,
    limit=5,
    query_filter={"must": [{"key": "source", "match": {"value": "doc1"}}]}
)
```

## Pitfalls
- **Index type**: HNSW for speed, IVF for memory
- **Distance metric**: Cosine for normalized embeddings, Euclidean for raw
- **Payload size**: Keep metadata small
- **Batch operations**: Use batch insert for >1000 vectors

## Verification
- Benchmark search latency
- Check recall rate vs brute force
- Test with realistic query patterns