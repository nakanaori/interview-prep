# Mock Interview

Worked example using the 5-Step Framework.

**Prompt:** *"Design a real-time transaction notification system."*

---

## Step 1 — Clarify Requirements (3-5 min)

Use the [7 Questions Framework](../quick-ref/framework.md) to extract requirements.

**Functional:**
1. Notifications for all transaction types: approved, declined, refund
2. Customizable preferences: toggle per type, quiet hours
3. Notification history screen
4. Tap notification → see transaction details

**Non-functional:**
- 5M transactions/day ≈ 50 QPS avg, 250 QPS peak
- Latency < 5 seconds end-to-end
- At-least-once delivery (better to send twice than miss)
- Availability > consistency

---

## Step 2 — Estimation (3-5 min)

> Quick math: total per day ÷ 100,000 = average QPS. Multiply by 5-10x for peak.

- 50 QPS average, 250 QPS peak → not heavy, no extreme scaling needed

---

## Step 3 — High-Level Design (15-20 min)

### API

| Endpoint | Method | Notes |
|---|---|---|
| `transaction_completed` | Kafka event | Internal trigger, not REST |
| `/notifications/history` | GET | Paginated, cursor-based |
| `/preferences/notifications` | GET + PUT | Per-user settings |

### Data Model

**notifications**
```
id, user_id, transaction_id, type, merchant_name, amount,
remaining_limit, is_read, created_at
```

**notification_preferences**
```
user_id, preferences (JSONB)
```
> JSONB because: no WHERE clause needed, app-level filtering, no schema change for new preference types

**device_tokens**
```
id, user_id, token, platform, is_active
```

### Architecture Flow

```
Transaction Service
  → Kafka "transaction_completed" event
  → Notification Service consumes
    → Redis (check preferences, cache-aside)
    → if not eligible: skip
    → PostgreSQL (save to notification history)
    → Redis (get device tokens)
    → FCM / APNs (send push)

User opens history:
  → API Server → PostgreSQL (query by user_id, cursor-based pagination)

User updates preferences:
  → API Server → PostgreSQL (update) → invalidate Redis cache
```

**Key learning:** if DB write fails, send push anyway (availability first), retry DB write async via retry queue

**Key learning:** if you don't know something (like device token format), reason from first principles — *"I need to reach a specific phone, so I need some device identifier"* — state assumption and move on

---

## Step 4 — Deep Dive (15-20 min)

**Scenario: flash sale causes 5x traffic spike, FCM responds slowly**

| Problem | Pattern | Solution |
|---|---|---|
| External API slow/failing | Circuit breaker | Fail fast, queue retries |
| Traffic spike | Horizontal scaling | Add more Kafka consumers |
| Too many HTTP calls | Batching | FCM supports 500 messages/request |
| Not all notifications urgent | Priority lanes | Separate Kafka topics for high/low priority |

### Mental Model: "What if X?" → Pattern

| Problem Signal | Pattern |
|---|---|
| "slow" / "latency" | Caching (Redis) or batching |
| "down" / "fails" | Circuit breaker + timeout + retry with backoff |
| "crashes mid-operation" | Idempotency + TTL on intermediate states |
| "halfway" / "partial failure" | Saga or transactional outbox |
| "at the same time" / "race condition" | Locking (atomic, pessimistic, or optimistic) |
| "twice" / "duplicate" / "retry" | Idempotency key + unique constraint |
| "spike" / "flash sale" | Horizontal scaling + Kafka + caching + rate limiting |
| "millions of rows" | Scaling ladder: index → cache → read replicas → sharding |

---

## Step 5 — Wrap-up (5 min)

> Template: *"With more time I'd add monitoring for [key metrics], [one missing feature], and [one edge case]."*

**Monitoring:** delivery rate, latency SLA, FCM error rates, Kafka consumer lag

**Analytics:** open rates, engagement by type, opt-out rates

**Missing features:** SMS/email fallback, rich notifications with merchant logos, localization

**Safety:** rate limiting per user, fraud alert escalation
