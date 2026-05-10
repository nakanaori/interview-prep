# Payment Systems

Core concepts for fintech and payment system design interviews.

---

## Payment Flow

```
Authorization → Capture → Settlement
```

| Phase | Description |
|---|---|
| **Authorization** | Real-time, holds funds. The critical hot path. Most interview questions focus here. |
| **Capture** | Merchant collects the held funds |
| **Settlement** | Money actually moves between accounts |

---

## Ledger (Double-Entry Bookkeeping)

- Every transaction = **debit + credit**
- Entries are **immutable** — never delete or update
- Refunds = new reversal entries, never delete originals

---

## Distributed Transactions

### Saga (preferred for microservices)

Each step has a compensating action for rollback.

| Pattern | How | Best For |
|---|---|---|
| **Choreography** | Each service listens for events, no coordinator | Simple flows, but hard to debug |
| **Orchestration** | Central coordinator manages flow | Payment flows, easier to monitor |

### 2PC (Two-Phase Commit)

Coordinator asks all participants to prepare, then commit. **Does not work** across service boundaries or external APIs — use saga instead.

---

## Idempotency in Payments

Deterministic key: `hash(user_id + order_id + action)`

1. Check Redis for existing result (fast path)
2. If miss, check DB with `UNIQUE` constraint on `idempotency_key`
3. PostgreSQL catches duplicates even if Redis cache misses

---

## Reconciliation

Daily job comparing records between systems to find discrepancies. Essential for detecting missed settlements, duplicate charges, or system drift.

---

## Key Design Rules

- Authorization is the hot path — optimize for latency and consistency
- Use strong consistency (PostgreSQL ACID) for all financial writes
- Saga over 2PC for anything crossing service boundaries
- Ledger entries are append-only — audit trail is permanent
