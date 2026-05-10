# Design a URL Shortener

> **Difficulty:** Medium — great first practice problem. Tests hashing, DB design, caching, and redirects.

---

## Step 1 — Clarify Requirements

**Functional:**
- Given a long URL, generate a short URL (e.g. `short.ly/abc123`)
- Redirect short URL → original URL
- Custom aliases (optional)
- URL expiry (optional)
- Analytics: click count, referrer (optional)

**Non-functional:**
- 100M URLs created/day, 10:1 read/write ratio → 1B redirects/day
- Latency < 10ms for redirects
- High availability (redirect service — users will notice)
- URLs are permanent unless explicitly expired

---

## Step 2 — Estimation

```
Writes: 100M/day ÷ 100,000 = 1,000 QPS avg → 5,000 QPS peak
Reads:  1B/day   ÷ 100,000 = 10,000 QPS avg → 50,000 QPS peak

Storage per URL: ~500 bytes (long URL + metadata)
Daily storage: 100M × 500 bytes = 50 GB/day
5-year storage: 50 GB × 365 × 5 ≈ 90 TB
```

---

## Step 3 — High-Level Design

### API

```
POST /urls              body: { long_url, custom_alias?, expires_at? }
                        returns: { short_url }

GET  /{short_code}      → 301 redirect to long URL
GET  /urls/{id}/stats   → click count, referrers
```

> Use **301 (permanent redirect)** for caching at browser/CDN. Use **302 (temporary)** if you need accurate analytics (each redirect hits your server).

### Short Code Generation

**Option A — MD5/SHA256 hash:**
1. Hash the long URL → 128-bit hash
2. Take first 7 characters of base62 encoding
3. Check DB for collision → retry with offset if collision

**Option B — Base62 counter (Recommended):**
1. Auto-increment ID in DB (or distributed ID generator like Snowflake)
2. Encode ID to base62 (`[0-9a-zA-Z]`) → 7 chars supports 62^7 ≈ 3.5 trillion URLs
3. No collision possible

```
ID: 2009215674938
Base62: zn9edcu
```

### Data Model

**urls**
```
id           BIGINT PRIMARY KEY  (auto-increment or Snowflake ID)
short_code   VARCHAR(8) UNIQUE
long_url     TEXT
user_id      BIGINT
created_at   TIMESTAMP
expires_at   TIMESTAMP (nullable)
```

**clicks** (for analytics)
```
id           BIGINT
short_code   VARCHAR(8)
clicked_at   TIMESTAMP
referrer     TEXT
user_agent   TEXT
```

### Architecture

```
Client → API Gateway → URL Service → PostgreSQL (write)
                                   → Redis (cache short_code → long_url)

Client → CDN/LB → Redirect Service → Redis (cache hit? → 301 redirect)
                                   → PostgreSQL (cache miss → Redis → 301 redirect)

Click event → Kafka → Analytics Consumer → ClickHouse / DynamoDB
```

---

## Step 4 — Deep Dive

### Redirect Performance

Cache `short_code → long_url` in Redis with TTL = 24h. Redirect service checks Redis first — PostgreSQL only on cache miss.

With 10K QPS reads and ~90% cache hit rate → ~1K QPS to PostgreSQL. Easily handled.

### Collision Handling (hash approach)

1. Hash long URL → take 7 chars
2. If collision in DB, append counter and rehash
3. Store original + hash in `url_hash` column to dedup identical long URLs

### Custom Aliases

- Store in `short_code` field like any other
- Add `is_custom BOOLEAN` flag
- Validate uniqueness before saving

### URL Expiry

- Store `expires_at` in DB
- Redirect service checks expiry → return 410 Gone if expired
- Cleanup job: daily batch DELETE WHERE expires_at < NOW()

### Analytics at Scale

Don't write analytics synchronously on redirect — it adds latency on the hot path.

Instead: publish click event to Kafka → async consumer writes to ClickHouse (columnar, great for aggregations).

---

## Step 5 — Wrap-up

- **Monitoring:** redirect latency p99, cache hit rate, error rate, Kafka consumer lag
- **Missing:** rate limiting per user (prevent abuse), geo-based analytics, QR code generation
- **Edge case:** what if someone shortens the short URL itself? Detect and reject.
- **Scale:** CDN in front of redirect service — short URLs are cacheable, put them at the edge
