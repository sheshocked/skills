---
name: search-engineering
description: - Full-text search across documents, products, or content
category: engineering
tags: [search-engineering]
---

## When to Use

- Full-text search across documents, products, or content
- Autocomplete/typeahead that needs sub-100ms response times
- Faceted search (filter by category, price, rating simultaneously)
- Log aggregation and analytics requiring fast query over time ranges

## Core Concepts

- **Inverted Index**: Maps terms to documents. "cat" → [doc1, doc3, doc5]. Enables O(1) term lookup.
- **Relevance Scoring**: BM25 (term frequency × inverse document frequency) is the standard. TF-IDF is the classic.
- **Analyzers**: Tokenize, normalize, filter text. Custom analyzers handle synonyms, stemming, language-specific rules.
- **Sharding**: Split index across nodes. Each shard holds a subset. Queries fan out to all shards, results merged.
- **Index Design**: Schema matters more in search than in relational DBs. Denormalize aggressively.

## Workflow

1. **Define search requirements** — what fields, what ranking, what filters
2. **Design index mapping** — field types, analyzers, tokenizers
3. **Implement indexing pipeline** — batch + real-time indexing
4. **Build query layer** — combine full-text, filters, sorting, pagination
5. **Tune relevance** — boost fields, adjust BM25 parameters, add synonyms
6. **Monitor** — query latency, index size, relevance quality (click-through rate)

## Key Patterns

```python
# Elasticsearch index mapping with custom analyzer
from elasticsearch import Elasticsearch

es = Elasticsearch(["http://localhost:9200"])

index_body = {
    "settings": {
        "number_of_shards": 3,
        "number_of_replicas": 1,
        "analysis": {
            "analyzer": {
                "product_analyzer": {
                    "type": "custom",
                    "tokenizer": "standard",
                    "filter": ["lowercase", "asciifolding", "edge_ngram_filter"],
                }
            },
            "filter": {
                "edge_ngram_filter": {
                    "type": "edge_ngram",
                    "min_gram": 2,
                    "max_gram": 20,
                }
            }
        },
    },
    "mappings": {
        "properties": {
            "name": {
                "type": "text",
                "analyzer": "product_analyzer",
                "fields": {
                    "exact": {"type": "keyword"},  # For exact match filtering
                },
            },
            "description": {"type": "text", "analyzer": "standard"},
            "category": {"type": "keyword"},
            "price": {"type": "scaled_float", "scaling_factor": 100},
            "tags": {"type": "keyword"},
            "popularity_score": {"type": "float"},
            "created_at": {"type": "date"},
        }
    },
}

es.indices.create(index="products", body=index_body)
```

```python
# Search query with filters, sorting, and pagination
def search_products(
    query: str = None,
    category: str = None,
    min_price: float = None,
    max_price: float = None,
    sort: str = "relevance",
    page: int = 1,
    page_size: int = 20,
):
    must = []
    filters = []

    # Full-text search
    if query:
        must.append({
            "multi_match": {
                "query": query,
                "fields": ["name^3", "description", "tags^2"],  # Boost name and tags
                "type": "best_fields",
                "fuzziness": "AUTO",  # Handle typos
            }
        })

    # Filters
    if category:
        filters.append({"term": {"category": category}})
    if min_price is not None or max_price is not None:
        price_range = {}
        if min_price is not None:
            price_range["gte"] = min_price * 100  # Scaled float
        if max_price is not None:
            price_range["lte"] = max_price * 100
        filters.append({"range": {"price": price_range}})

    # Sorting
    sort_config = {
        "relevance": "_score",
        "price_asc": {"price": {"order": "asc"}},
        "price_desc": {"price": {"order": "desc"}},
        "newest": {"created_at": {"order": "desc"}},
        "popular": {"popularity_score": {"order": "desc"}},
    }

    body = {
        "query": {
            "bool": {
                "must": must or [{"match_all": {}}],
                "filter": filters,
            }
        },
        "sort": [sort_config.get(sort, "_score")],
        "from": (page - 1) * page_size,
        "size": page_size,
        "highlight": {
            "fields": {
                "name": {},
                "description": {"fragment_size": 150, "number_of_fragments": 3},
            }
        },
    }

    result = es.search(index="products", body=body)
    return {
        "hits": result["hits"]["hits"],
        "total": result["hits"]["total"]["value"],
        "page": page,
        "page_size": page_size,
    }
```

```python
# Autocomplete with edge ngrams
def autocomplete(prefix: str, max_results: int = 10):
    body = {
        "query": {
            "bool": {
                "must": [
                    {"prefix": {"name": prefix.lower()}},
                ],
                "filter": [
                    {"range": {"popularity_score": {"gte": 0.1}}},
                ],
            }
        },
        "size": max_results,
        "_source": ["name", "category", "price"],
        "highlight": {"fields": {"name": {}}},
    }
    result = es.search(index="products", body=body)
    return [hit["_source"] for hit in result["hits"]["hits"]]
```

## Pitfalls

- **Treating search like SQL**: `WHERE name LIKE '%term%'` is slow and misses relevance. Use inverted indexes.
- **Ignoring analyzer mismatch**: Index with one analyzer, query with another = no matches. Ensure consistency.
- **Over-sharding**: 100 shards for 10GB index = overhead. Aim for 10-50GB per shard.
- **N+1 indexing**: Indexing relationships separately. Denormalize into the document for single-query retrieval.
- **Missing synonyms**: Users search "laptop" but products are called "notebook". Add synonym mappings.

## Verification

- Query latency p95 < 200ms for full-text search, <50ms for autocomplete
- Relevance quality: check top-10 results are relevant (manually or via click data)
- Index health: all shards green, no unassigned replicas
- Synonym test: search "laptop" and "notebook" return similar results
- Load test: simulate 1000 concurrent searches, verify no degradation