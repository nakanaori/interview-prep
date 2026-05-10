# Building Blocks

The components you'll assemble in every system design answer.

---

## Day 5 — Caching

### Strategies

| Strategy | How | When to Use |
|---|---|---|
| **Cache-aside** (default) | Check Redis → miss → query DB → store in cache. On write: update DB → delete Redis key. | Most things |
| **Write-through** | Write to cache + DB together. Cache always fresh. | Stale reads unacceptable (current balance) |
| **Write-behind** | Write to cache only, async flush to DB. Fast but risky. | Never for financial data |

### Cache Invalidation

- **TTL** — auto-expire after N seconds
- **Event-based** — delete key on write
- Use both: event-based for known writes, TTL as safety net

### Cache Eviction (when Redis memory is full)

- **LRU** (default) — evicts keys not accessed longest
- **LFU** — evicts least popular keys
- Set TTLs on individual keys too

### Cache Stampede

Popular key expires → 1000 requests hit DB simultaneously.

**Solutions:**
- **SETNX locking** — first request wins, others wait and retry
- **Staggered TTL** — random expiry times
- **Background refresh** — re-populate before expiry

**Key learning:** distinguish "caching a query result" vs "maintaining a live counter/set" in Redis

**Key learning:** if same data is read multiple times in a short window → cache it, even per-user data

---

## Day 6 — Message Queues & Async Processing

> Core question: "Does the user need to wait for this?" → sync if yes, async if no

### Kafka Concepts

| Concept | Description |
|---|---|
| **Topics** | Named channels, one per domain |
| **Producers** | Write messages |
| **Consumers** | Read messages |
| **Consumer groups** | Multiple instances share work; each message processed once |
| **Partitions** | Parallelism; same key = same partition = ordering guaranteed |

Kafka persists messages to disk — if consumer is down, messages wait. No data lost.

### Patterns

**Saga** — when multiple services must all succeed or all roll back. Each step has a compensating action (decrement quota → undo: increment back).

| Pattern | How | Best For |
|---|---|---|
| **Choreography** | Each service listens for events, no coordinator | Simple flows |
| **Orchestration** | Central coordinator manages flow | Payment flows, easier to monitor |
| **2PC** | Coordinator asks all participants to prepare then commit | Within one DB only — not across microservices |

> When NOT to use saga: nothing to roll back (KYC fails → just reject). Use simple retries + status tracking instead.

- **Parallel processing:** independent checks (KYC + credit) can run simultaneously
- **Dead letter queue (DLQ):** failed messages after max retries go here for ops to investigate
- **Retry with exponential backoff** for unreliable external APIs

---

## Day 7 — Database Scaling

### Scaling Ladder (walk through in order during interviews)

1. Optimize (indexes, queries, connection pooling)
2. Caching
3. Read replicas
4. Vertical partitioning
5. Sharding

### Key Concepts

**Connection pooling (PgBouncer):** reuse connections, reduce DB load

**Read replicas:** 80-90% reads offloaded. Replication lag = brief window where replica has stale data.
- **Read-your-own-writes:** after a user writes, force THAT user's reads to primary for a few seconds. Everyone else reads replicas normally.

**Vertical partitioning:** separate databases per service (users DB, transactions DB). Scale independently.

**Sharding:** split data across DBs by shard key. `user_id` is usually the right key for payment systems.

| Shard Key Quality | Example |
|---|---|
| ✅ Good | `user_id` — even distribution, co-locates related data |
| ❌ Bad | `created_date` — hot spot on today |
| ❌ Bad | `merchant_id` — Starbucks gets all traffic |

> Queries **with** shard key → fast (one shard). **Without** → slow (scatter-gather). Cross-shard joins = avoid.

**Consistent hashing:** avoids massive data migration when adding/removing shards. Adding 1 shard only moves ~20% of data instead of ~80%.

### Analytics Patterns

- **Pre-computed summary tables** — aggregate on write, not on read
- **CDC (Change Data Capture)** — watches PostgreSQL WAL, streams every change (PostgreSQL → CDC → Kafka → Elasticsearch)
- **CQRS** — writes to one store (PostgreSQL), reads from another (Elasticsearch)

### Elasticsearch vs PostgreSQL for filtering

| Use | When |
|---|---|
| PostgreSQL | Fixed known filters, indexable columns, <100M rows |
| Elasticsearch | Full-text search, unpredictable filter combos (15+ facets), fuzzy matching, autocomplete |

Start simple: PostgreSQL + indexes first. Add CDC + Elasticsearch only when filtering is too varied.

---

## Day 8 — Consistency & Reliability

### CAP Theorem

During a network partition, choose consistency **or** availability — not both.

| Type | Use For | Tools |
|---|---|---|
| **Strong consistency (CP)** | Credit limits, quota decrements, balances | PostgreSQL |
| **Eventual consistency (AP)** | Recommendation cache, analytics, notifications | Redis, Elasticsearch, Kafka |

> Interview phrase: *"The transactional core is strongly consistent. Everything else is eventually consistent with bounded staleness."*

### Idempotency

Same operation multiple times = same result. Use deterministic keys: `hash(user_id + resource_id + order_id)`

- Store with `UNIQUE` constraint on data table — PostgreSQL catches duplicates even if Redis misses
- Redis as fast-path cache; unique constraint is the safety net
- Matters for: payments, redemptions, transfers. Not needed for: reads, analytics logging.

### Retries

- Exponential backoff + jitter: wait 1s, 2s, 4s, 8s + random
- Only retry 5xx and timeouts — never 4xx
- Max 3-5 attempts then DLQ
- **Jitter:** without it, 1000 requests retry at the same time → adds randomness to spread them out
- **Hot path:** fail fast and fall back (don't retry). Background/async: retry with backoff.

### Circuit Breaker

One per downstream dependency.

```
CLOSED (normal) → OPEN (5 failures → fail fast) → HALF-OPEN (test one request after 30s)
```

> Interview rule: every external service call → mention timeout + circuit breaker.

### Timeouts

Every external call needs a timeout. No exceptions. Without one, a hung service causes cascading failure.

| Dependency | Timeout |
|---|---|
| Redis | 50-100ms |
| DB | 500ms-2s |
| Internal services | 1-3s |
| External APIs | 5-10s |

### Transactional Outbox

Write event to outbox table inside the same DB transaction — guarantees event + data always in sync.

Background worker reads outbox:
- **Polling** — query every 200-500ms for unpublished rows (simpler)
- **CDC on outbox table** — near real-time, no polling (lower latency, upgrade path)
