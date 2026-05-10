# Design a Chat System

> **Difficulty:** Hard — tests real-time communication, message delivery guarantees, presence, and storage at scale.

---

## Step 1 — Clarify Requirements

**Functional:**
- 1:1 messaging
- Group chat (up to 500 members)
- Message delivery status: sent, delivered, read
- Online presence indicator
- Message history

**Non-functional:**
- 1B DAU, avg 40 messages/day
- Latency < 100ms for message delivery
- At-least-once delivery (no message loss)
- Messages stored for 5 years

---

## Step 2 — Estimation

```
Messages/day: 1B users × 40 = 40B messages/day → 460,000 msg/s
Storage/message: ~100 bytes
Daily storage: 40B × 100 bytes = 4 TB/day
5-year storage: 4 TB × 365 × 5 ≈ 7 PB
```

---

## Step 3 — High-Level Design

### Why WebSocket?

HTTP is request-driven — server can't push to client without a request. Options:

| | Long Polling | SSE | WebSocket |
|---|---|---|---|
| Direction | Pull | Server → client | Bidirectional |
| Latency | High | Medium | Low |
| Use | Simple notifications | Feed updates | Chat ✅ |

**WebSocket:** persistent bidirectional connection. Client connects once, both sides can send at any time.

### Architecture

```
Client A ──WebSocket──► Chat Server 1 ──► Message Queue (Kafka)
                                       ──► Message DB (write)

Client B ──WebSocket──► Chat Server 2

Kafka consumer:
  → look up Client B's connection → Chat Server 2
  → Chat Server 2 pushes to Client B via WebSocket
```

### Components

**Chat Servers:** maintain WebSocket connections. Stateless — each server tracks which users are connected to it.

**Presence Service:** tracks online/offline status in Redis.
```
Key: presence:{user_id}
Value: last_heartbeat_timestamp + server_id
TTL: 30s (renewed every 10s by client heartbeat)
```

**Message Queue (Kafka):** decouples sending from delivery. If recipient's server is down, message waits.

**Service Discovery:** when routing a message to a recipient, need to know which Chat Server they're connected to.
```
Key: user_session:{user_id}
Value: chat_server_id
```

### API

```
WebSocket /ws               → establish persistent connection
POST /messages              → send message (fallback if WS drops)
GET  /messages/{chat_id}    → message history (paginated)
GET  /users/{id}/presence   → online status
```

### Data Model

**messages**
```
message_id    BIGINT (Snowflake — sortable by time)
chat_id       BIGINT
sender_id     BIGINT
content       TEXT
type          ENUM(text, image, file)
created_at    TIMESTAMP
```

**chats**
```
chat_id       BIGINT
type          ENUM(direct, group)
created_at    TIMESTAMP
```

**chat_members**
```
chat_id       BIGINT
user_id       BIGINT
joined_at     TIMESTAMP
```

---

## Step 4 — Deep Dive

### Message Delivery Flow

```
1. Client A sends message via WebSocket to Chat Server 1
2. Chat Server 1 assigns Snowflake ID, saves to DB
3. Publishes to Kafka topic: messages
4. Kafka consumer checks: is Client B online?
   a. Online → find their Chat Server → push via WebSocket
   b. Offline → store as undelivered → push when they reconnect
5. Client B receives, ACKs → update delivery status
```

### Message Ordering

Use Snowflake IDs as message IDs — they encode timestamps, so sorting by ID = chronological order. No need for `ORDER BY created_at`.

### Delivery Guarantees

**At-least-once:** Kafka retains messages until consumer ACKs. If consumer crashes mid-delivery, message is re-delivered.

**Deduplication:** client tracks seen message IDs. Duplicate delivered = silently ignored.

**Undelivered messages:** when user comes online, server queries `undelivered_messages` table → pushes all pending → marks delivered.

### Group Chat Fan-out

For a 500-member group chat:
1. Save one message record
2. Fan-out: for each member, look up their Chat Server → push
3. At 500 members, fan-out is fast (in-memory, parallel)
4. For very large groups (>10K), use pub/sub: each group = a Kafka topic, members subscribe

### Message Storage

40B messages/day → use **sharding** by `chat_id`. All messages in a chat land on the same shard → fast retrieval without scatter-gather.

For history queries older than 30 days → archive to cold storage (S3). Recent messages stay in DB.

### Presence at Scale

1B users × heartbeat every 10s = 100M heartbeats/s → too much for one Redis.

Solution: shard presence data. Each user hashes to one of N presence Redis instances.

---

## Step 5 — Wrap-up

- **Monitoring:** WebSocket connection count, message delivery latency p99, Kafka consumer lag, undelivered message queue depth
- **Missing:** end-to-end encryption, message editing/deletion (soft delete), reactions, file/image transfer (pre-signed S3 URLs)
- **Edge case:** Chat Server crash → all connected users reconnect (thundering herd) → add exponential backoff + jitter to reconnect logic
- **Scale:** millions of concurrent WebSocket connections → each Chat Server handles ~50K connections. 1B DAU × 10% online = 100M connections → ~2,000 Chat Servers
