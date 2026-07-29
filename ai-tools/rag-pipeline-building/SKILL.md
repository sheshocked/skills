---
name: rag-pipeline-building
description: Build RAG pipelines in Python, chunking text, indexing embeddings in vector databases, and using cross-encoder rerankers.
category: ai-tools
tags: [rag, vector-search, sentence-transformers, qdrant, reranking, python]
---

# Rag Pipeline Building

## When to Use
Use to build question-answering systems over custom documentation bases (like manuals, logs, source code) to ground LLM responses and avoid hallucinations.

## Prerequisites
- Python 3.10+.
- Packages: `qdrant-client`, `sentence-transformers`, `numpy`.

## Workflow
1. Chunk source documents using recursive token-based splitting.
2. Generate vector embeddings using sentence-transformer models.
3. Index vectors in a vector database (e.g. Qdrant).
4. Perform hybrid search (semantic + keyword) and apply cross-encoder reranking before prompt injection.

## Key Patterns

### Python RAG Implementation (rag_core.py)
```python
from qdrant_client import QdrantClient
from sentence_transformers import SentenceTransformer, CrossEncoder

# Load embedding and reranking models
embed_model = SentenceTransformer("all-MiniLM-L6-v2")
rerank_model = CrossEncoder("mixedbread-ai/mxbai-rerank-large-v1")
client = QdrantClient(host="localhost", port=6333)

def query_rag(query_text: str, collection_name: str, limit: int = 10):
    # 1. Embed query
    query_vector = embed_model.encode(query_text).tolist()
    
    # 2. Perform Vector Search
    hits = client.search(
        collection_name=collection_name,
        query_vector=query_vector,
        limit=limit
    )
    
    # 3. Rerank results
    documents = [hit.payload["text"] for hit in hits]
    pairs = [[query_text, doc] for doc in documents]
    scores = rerank_model.predict(pairs)
    
    # Sort documents by reranker scores
    reranked = [doc for _, doc in sorted(zip(scores, documents), reverse=True)]
    
    # 4. Construct context
    context = "\n---\n".join(reranked[:3])
    return context
```

## Pitfalls
- **Generic chunking limits:** Fixed-character chunking splits sentences midway, breaking contextual coherence. Always use overlapping semantic dividers (e.g. paragraph limits).
- **Ignoring out-of-domain queries:** RAG will always return top-K hits even if irrelevant. Enforce score thresholds (`score > 0.6`) to reject bad context.

## Verification
- Test query results: verify retrieved chunks align semantically with queries.
- Monitor index memory usage under load.

