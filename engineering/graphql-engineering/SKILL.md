---
name: graphql-engineering
description: - Building APIs for mobile apps that need flexible data fetching
category: engineering
tags: [graphql-engineering]
---

## When to Use

- Building APIs for mobile apps that need flexible data fetching
- Replacing multiple REST endpoints with a unified graph
- Frontend teams need to iterate on data requirements without backend changes
- Complex data relationships that flatten poorly in REST

## Core Concepts

- **Schema-First Design**: Define your GraphQL schema as the source of truth. Generate resolvers from it.
- **Resolver Tree**: Each field resolves independently. `Query.user.posts` means user resolver runs first, then posts resolver.
- **N+1 Problem**: Fetching a list then querying related data per item. Solve with DataLoader batching.
- **Queries vs Mutations vs Subscriptions**: Queries read, mutations write (serial), subscriptions push real-time updates.
- **Persisted Queries**: Pre-registered query hashes sent instead of full query strings. Reduces bandwidth and enables whitelisting.

## Workflow

1. **Design schema** — SDL-first, define types, inputs, enums
2. **Implement resolvers** — one per field, keep them thin (delegate to services)
3. **Add DataLoader** — batch and cache DB queries within a single request
4. **Auth directives** — `@auth(requires: ADMIN)` on sensitive fields
5. **Rate limiting** — query complexity analysis to prevent abuse
6. **Monitor** — per-field resolver performance tracing

## Key Patterns

```graphql
# Schema definition (schema.graphql)
type Query {
  user(id: ID!): User
  users(filter: UserFilter, first: Int, after: String): UserConnection!
}

type Mutation {
  createOrder(input: CreateOrderInput!): CreateOrderPayload!
}

type User {
  id: ID!
  name: String!
  email: String! @auth(requires: SELF)
  posts(first: Int, after: String): PostConnection!
}

input CreateOrderInput {
  productId: ID!
  quantity: Int! @constraint(min: 1, max: 1000)
  idempotencyKey: String! @constraint(minLength: 16)
}

# Connection pattern for pagination (Relay-style)
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  endCursor: String
}
```

```python
# DataLoader for N+1 prevention
from typing import List
from graphql import GraphQLResolveInfo
from strawberry.dataloader import DataLoader

async def load_users(ids: List[str]) -> List[User]:
    rows = await db.fetch("SELECT * FROM users WHERE id = ANY($1)", ids)
    by_id = {r["id"]: r for r in rows}
    return [by_id.get(i) for i in ids]

user_loader = DataLoader(load_fn=load_users)

# Resolver using DataLoader
async def resolve_user_posts(
    user: User, info: GraphQLResolveInfo, first: int = 20, after: str = None
):
    return await post_loader.load(user.id)  # batched, not N+1

# Query complexity limiter
from graphql import GraphQLSchema
from query_depth_limiter import depth_limit_validator
from graphql_extensions import cost_analysis

schema = GraphQLSchema(...)
# Limit query depth to prevent deeply nested abuse
schema.validation_rules.add(depth_limit_validator(max_depth=7))
# Cost analysis: each connection = cost 10, each scalar = 1
schema.validation_rules.add(cost_analysis(max_cost=500))
```

```python
# Persisted queries with automatic generation
import hashlib
import json

PERSISTED_QUERIES = {}

def register_query(query_str: str) -> str: