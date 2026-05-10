# Foundations

Core infrastructure concepts every system design interview assumes you know.

---

## Day 1 — Client to Server: What Actually Happens

- **DNS → TCP → HTTP request → response**
- HTTP methods: GET, POST, PUT, DELETE
- Status codes: 200, 400, 404, 500
- REST API basics — endpoints, request/response format

**Exercise:** Design the REST API for a coupon service.

---

## Day 2 — How Servers Handle Traffic

- **Single server limits** → what breaks first: CPU/memory → DB connections → single point of failure
- **Load balancer** — distributes requests (round-robin, least connections, IP hash)
- **LB as SPOF:** use redundant LBs — active-passive (standby via heartbeat) or active-active cluster behind DNS. Cloud LBs (AWS ALB) handle this automatically.
- **Horizontal scaling** (more servers) vs **vertical scaling** (bigger server)
- **Stateless vs stateful servers** — always design stateless

**Exercise:** "50x flash sale traffic — what breaks and how to fix it?"

**Key learning:** scale app servers + add Redis cache for reads + Redis fast-reject for quota writes

---

## Day 3 — SQL Databases

### Indexes

- **B-tree** (default): equality + range queries
- **Composite**: order matters — leftmost column first
- **GIN**: for JSONB, heavy writes but fine for config tables
- Index columns in WHERE, JOIN, ORDER BY — don't over-index write-heavy tables

### Transactions and ACID

| Property | Meaning |
|---|---|
| **Atomicity** | All or nothing (BEGIN/COMMIT) |
| **Consistency** | DB enforces rules (foreign keys, constraints) |
| **Isolation** | Concurrent transactions don't interfere |
| **Durability** | Committed data survives crashes |

> Fintech rule: financial operations always need ACID → PostgreSQL as default

### Locking Strategies

| Strategy | When to Use | How |
|---|---|---|
| **Atomic update** | Simplest, low contention | `UPDATE SET quota = quota - 1 WHERE quota > 0` |
| **Pessimistic** | High contention (1000 users, same coupon) | `SELECT FOR UPDATE` locks the row |
| **Optimistic** | Low contention (users updating own profiles) | Read with version, write with `WHERE version = N`, retry on conflict |

> **Contention** = multiple requests competing for the same resource at the same time.

**Key learning:** junction table + JSONB for flexible rules; filter in 2 stages (SQL narrows, app evaluates)

**Key learning:** Redis = bouncer (fast-reject), PostgreSQL = source of truth — both decremented every time

---

## Day 4 — NoSQL + Redis + When to Use What

### Redis Data Structures

| Type | Use Case |
|---|---|
| Strings | Counters |
| Sets | Unique collections |
| Sorted sets | Feeds |
| Hashes | Objects |
| TTL | Auto-expiry |

### NoSQL Options

| DB | Use When |
|---|---|
| **DynamoDB** | Predictable access pattern, partition key + sort key, massive scale, no joins |
| **Cassandra** | Extreme write throughput, time-series, petabyte scale |
| **MongoDB** | Document-shaped data — but PostgreSQL + JSONB usually covers this |

**Key learning:** almost every scenario = PostgreSQL (source of truth) + Redis (speed layer) + object storage (files)

**Key learning:** DynamoDB when access pattern is locked in and simple. Cassandra only at extreme write scale.

**Default:** PostgreSQL + Redis. Mention DynamoDB only for clear fits (feeds, sessions, event logs).
