# SQL vs NoSQL

## Decision Framework

Ask two questions:

1. Do I need **relationships or transactions**? → **SQL (PostgreSQL)**
2. Is the **schema unpredictable**, or is the **access pattern simple key-value at scale**? → **NoSQL**

> Most real systems use **both**.

---

## Quick Decision

| Scenario | Choice |
|---|---|
| Temporary / ephemeral data | Redis |
| Need ACID / joins | PostgreSQL |
| Simple key-value at massive scale | DynamoDB |
| Flexible schema, moderate scale | PostgreSQL + JSONB |
| Extreme write throughput | Cassandra |
| Full-text search | Elasticsearch |

---

## NoSQL Types — When to Use Each

### Redis (key-value)
Caching, counters, rate limiting, sessions, fast-reject layers. In-memory = sub-ms reads.
- For ephemeral/hot data, not permanent large datasets

### DynamoDB (document)
Predictable access pattern with partition key + sort key. Feeds, sessions, event logs.
- ⚠️ Trap: once you pick partition key, you're stuck. New query pattern = GSI (extra cost).

### Cassandra (wide-column)
Extreme write throughput, time-series, petabyte scale. Rarely needed — just know it exists.

### MongoDB (document)
Flexible querying, index any field. But **PostgreSQL + JSONB usually covers the same use cases** — skip MongoDB.

### Elasticsearch
Full-text search, complex filtering across any field. Use alongside primary DB via CDC.

---

## Default Stack

> **PostgreSQL** (source of truth) + **Redis** (speed layer) + **object storage** (files, blobs)

Add DynamoDB only for clear fits. Add Elasticsearch only for full-text search or 15+ facet combinations.
