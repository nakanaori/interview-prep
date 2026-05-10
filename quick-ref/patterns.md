# Problem → Solution Patterns

When the interviewer asks "what if...", match the signal to the pattern.

---

## Performance & Latency

| Signal | Pattern | Example |
|---|---|---|
| "slow" / "latency" | **Caching (Redis)** or **batching** | Cache user preferences with TTL |
| "stale data" / "out of date" | **TTL + event-based invalidation** | Invalidate Redis key on update, TTL as safety net |

---

## Failures & Reliability

| Signal | Pattern | Example |
|---|---|---|
| "down" / "fails" / "unavailable" | **Circuit breaker + timeout + retry with backoff** | FCM down → fail fast, queue retries |
| "crashes mid-operation" / "timeout" | **Idempotency + TTL on intermediate states** | Unique constraint catches retry after crash |
| "halfway" / "partial failure" / "rollback" | **Saga** (compensating actions) or **transactional outbox** | Restore quota if payment fails |
| "loses data" / "crashes" / "restarts" | **PostgreSQL as source of truth + replicas** | Redis crashes → fall back to PostgreSQL |

---

## Concurrency & Duplicates

| Signal | Pattern | Example |
|---|---|---|
| "at the same time" / "concurrent" / "race condition" | **Locking** (atomic, pessimistic, or optimistic) | `UPDATE WHERE quota > 0` |
| "twice" / "duplicate" / "retry" | **Idempotency key** + unique constraint | `hash(user_id + order_id)` |

---

## Scale & Traffic

| Signal | Pattern | Example |
|---|---|---|
| "spike" / "flash sale" / "10x traffic" | **Horizontal scaling + Kafka + caching + rate limiting** | Scale consumers, cache hot data |
| "too many requests" / "abuse" / "spam" | **Rate limiting** (Redis counter with TTL) | INCR with 60s TTL, cap at threshold |
| "millions of rows" / "table is huge" | **Scaling ladder:** index → cache → read replicas → sharding | Composite index first, shard if needed |

---

## Data & Access Patterns

| Signal | Pattern | Example |
|---|---|---|
| "different query pattern" / "analytics" | **Summary tables, CDC → Elasticsearch, CQRS** | Write by user_id, analytics by merchant |
| "flexible" / "varies" / "changes" | **JSONB** (PostgreSQL) or **DynamoDB** | Coupon eligibility rules in JSONB |
| "search" / "full-text" / "filter by anything" | **Elasticsearch** (+ CDC from primary DB) | Search comments by keyword |
| "audit" / "compliance" / "history" | **Immutable append-only log** (ledger) | Never delete ledger entries, add reversals |

---

## Architecture & Communication

| Signal | Pattern | Example |
|---|---|---|
| "real-time" / "live" / "instant" | **WebSocket + pub/sub** | Live comments via WebSocket + Redis Pub/Sub |
| "multiple services need to know" | **Kafka event-driven architecture** | Payment event → notification, analytics, rewards |
| "user doesn't need to wait" | **Async processing (Kafka)** | Send notification in background after payment |
| "global users" / "multiple regions" | **CDN + DNS-based routing + regional replicas** | Tokyo servers for Japan, Singapore for SEA |

---

## Security & Operations

| Signal | Pattern | Example |
|---|---|---|
| "secret" / "sensitive" / "PII" | **Encryption at rest + pre-signed URLs + access controls** | KYC docs encrypted in S3 |
| "scheduled" / "recurring" | **Job scheduler + message queue** | Monthly statements via cron + batch job on read replica |
