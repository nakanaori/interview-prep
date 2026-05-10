# Design a Social Feed

> **Difficulty:** Hard — tests fan-out strategies, ranking, pagination, and scale. Classic Twitter/Instagram question.

---

## Step 1 — Clarify Requirements

**Functional:**
- Users can post (text, images)
- Users follow other users
- Home feed: posts from people you follow, reverse-chronological (or ranked)
- Infinite scroll / pagination

**Non-functional:**
- 500M DAU, avg user follows 200 people, posts 2x/day
- Feed load < 200ms
- Eventual consistency OK (slight delay in feed update is fine)
- Read-heavy: 100:1 read/write ratio

---

## Step 2 — Estimation

```
Writes: 500M users × 2 posts/day = 1B posts/day → 12,000 write QPS
Reads:  500M users × 10 feed views/day = 5B reads/day → 60,000 read QPS

Storage per post: ~1 KB text + image ref
Daily storage: 1B × 1KB = 1 TB/day (+ object storage for images)
```

---

## Step 3 — High-Level Design

### API

```
POST /posts                 → create post
GET  /feed                  → paginated home feed (cursor-based)
GET  /users/{id}/posts      → user's own posts
POST /users/{id}/follow     → follow a user
```

### Core Design Decision: Fan-out Strategy

This is the key question. When user A posts, how do followers see it?

#### Fan-out on Write (Push model)
When a post is created, immediately write it to every follower's feed cache.

```
Post created → Fan-out service → for each follower → append post_id to feed list in Redis
User loads feed → read from Redis feed list (pre-built)
```

✅ Feed reads are instant (Redis)
❌ Write amplification: 1 post × 1M followers = 1M writes
❌ Celebrities with millions of followers make fan-out very slow

#### Fan-out on Read (Pull model)
When a user loads their feed, fetch recent posts from all followed users.

```
User loads feed → get followed user IDs → fetch posts from each → merge + sort → return
```

✅ No write amplification
❌ Slow reads (N DB queries per feed load, N = number of follows)
❌ Popular users queried repeatedly

#### Hybrid (Recommended)

- **Normal users (< 1M followers):** fan-out on write → feed pre-built in Redis
- **Celebrities (> 1M followers):** fan-out on read → fetched and merged at read time

```
User loads feed
→ Read pre-built feed from Redis (normal followees)
→ Fetch recent posts from celebrity followees directly
→ Merge + return
```

### Data Model

**posts**
```
id          BIGINT (Snowflake — encodes timestamp, avoids sort)
user_id     BIGINT
content     TEXT
image_url   TEXT
created_at  TIMESTAMP
```

**follows**
```
follower_id   BIGINT
followee_id   BIGINT
created_at    TIMESTAMP
PRIMARY KEY (follower_id, followee_id)
```

**feed cache (Redis)**
```
Key: feed:{user_id}
Type: Sorted Set
Score: post timestamp (for ordering)
Value: post_id
Max size: 500 most recent post IDs per user
```

---

## Step 4 — Deep Dive

### Feed Pagination

Use cursor-based pagination — offset degrades badly at scale.

```
GET /feed?cursor=<last_post_id>&limit=20
→ WHERE id < cursor ORDER BY id DESC LIMIT 20
```

Return `next_cursor` in response. Client passes it on next request.

### Feed Ranking (beyond chronological)

Rank by engagement score:

```
score = recency_weight × time_decay + engagement_weight × (likes + comments×2 + shares×3)
```

- Compute scores async (Kafka consumer) when likes/comments happen
- Store score in Redis sorted set
- Recompute periodically for old posts

### Image Storage

Never store images in DB. Use object storage (S3):
1. Client requests pre-signed upload URL from API
2. Client uploads directly to S3 (no traffic through your servers)
3. Post stores S3 URL
4. Serve via CDN for fast global reads

### Handling Celebrities

Keep a `is_celebrity` flag (follower count > threshold). Fan-out service skips them. Read service fetches celebrity posts separately and merges.

---

## Step 5 — Wrap-up

- **Monitoring:** feed load latency p99, fan-out queue depth, cache hit rate, Redis memory
- **Missing:** feed filtering (mute, block), content ranking ML model, stories (TTL-based posts)
- **Edge case:** user unfollows → invalidate feed cache. User deletes post → remove from all feed caches.
- **Scale:** shard follows table by `follower_id`. Shard posts table by `user_id`.
