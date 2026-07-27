---
name: rag-pipeline
description: 
category: ai-tools
tags: [rag-pipeline]
---

## When to Use
Build Retrieval-Augmented Generation pipelines: chunking, embeddings, vector storage, retrieval, reranking.

## Architecture
```
Documents → Chunking → Embedding → Vector DB
                                         ↓
User Query → Embedding → Similarity Search → Top-K chunks
                                                    ↓
                                        LLM with context → Answer
```

## Chunking Strategies
1. **Fixed-size**: 512 tokens with 50-token overlap
2. **Semantic**: Split at paragraph/section boundaries
3. **Recursive**: Split by headings, then paragraphs, then sentences
4. **Document-aware**: Respect markdown/HTML structure

## Key Pipeline
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Qdrant

# Chunk
splitter = RecursiveCharacterTextSplitter(
    chunk_size=512, chunk_overlap=50,
    separators=["\\n## ", "\\n### ", "\\n\\n", "\\n", " "]
)
chunks = splitter.split_documents(docs)

# Embed + Store
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Qdrant.from_documents(chunks, embeddings, location=":memory:")

# Retrieve
results = vectorstore.similarity_search(query, k=5)

# Rerank (optional)
from sentence_transformers import CrossEncoder
reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
reranked = reranker.rank(query, [r.page_content for r in results])
```

## Pitfalls
- **Chunk size**: Too small loses context, too large dilutes relevance
- **Embedding model**: text-embedding-3-small for cost, large for accuracy
- **Overlap**: 10-20% overlap prevents context loss at boundaries
- **Metadata**: Store source, page, section for citation
- **Dedup**: Remove near-identical chunks

## Verification
- Test retrieval with known queries
- Measure hit rate (relevant chunk in top-5)
- Verify answer quality with/without RAG
- Check latency (retrieval + generation)