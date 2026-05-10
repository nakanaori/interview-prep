# Numbers Every Engineer Should Know

Memorize these. They come up in every estimation question.

---

## Latency Numbers

| Operation | Latency | Notes |
|---|---|---|
| L1 cache reference | 0.5 ns | |
| L2 cache reference | 7 ns | 14× L1 |
| RAM reference | 100 ns | 20× L2 |
| Read 1 MB from RAM | 250 µs | |
| SSD random read | 100 µs | |
| Read 1 MB from SSD | 1 ms | 4× RAM |
| Round trip within same datacenter | 500 µs | |
| Read 1 MB from HDD | 20 ms | |
| Round trip CA → Netherlands | 150 ms | |

> **Rule of thumb:** RAM is fast, disk is slow, network is variable. Cache everything that can be cached.

### Simplified Mental Model

```
L1/L2 cache   → nanoseconds  (ignore)
RAM           → microseconds (fast)
SSD           → milliseconds (ok)
Network/HDD   → milliseconds (slow)
Cross-region  → 100+ ms      (very slow)
```

---

## Powers of 2

| Power | Approx. Value | Name |
|---|---|---|
| 2^10 | 1 thousand | 1 KB |
| 2^20 | 1 million | 1 MB |
| 2^30 | 1 billion | 1 GB |
| 2^40 | 1 trillion | 1 TB |
| 2^50 | 1 quadrillion | 1 PB |

---

## Availability & Nines

| Availability | Downtime/year | Downtime/month |
|---|---|---|
| 99% (2 nines) | 3.65 days | 7.2 hours |
| 99.9% (3 nines) | 8.7 hours | 43 minutes |
| 99.99% (4 nines) | 52 minutes | 4.3 minutes |
| 99.999% (5 nines) | 5 minutes | 26 seconds |

> Most production services target **99.9% to 99.99%**. Five nines requires extreme engineering effort.

---

## Back-of-Envelope Formulas

### QPS

```
Average QPS  = total requests per day ÷ 100,000
Peak QPS     = average QPS × 5 (or × 10 for spiky traffic)
```

> Why 100,000? There are 86,400 seconds in a day ≈ 100K for easy math.

### Storage

```
1 character   = 1 byte
1 integer     = 4 bytes
1 UUID        = 16 bytes
1 timestamp   = 8 bytes
1 image (avg) = 300 KB
1 video (avg) = 50 MB
```

### Common Benchmarks

| System | Typical throughput |
|---|---|
| PostgreSQL (simple queries) | ~10,000 QPS per instance |
| Redis | ~100,000 QPS |
| Kafka | ~1,000,000 msg/s per partition group |
| Nginx | ~50,000 req/s |

---

## Estimation Template

```
1. Daily active users (DAU): X million
2. Requests per user per day: Y
3. Total requests/day: X million × Y
4. Average QPS: ÷ 100,000
5. Peak QPS: × 5
6. Storage per record: Z bytes
7. Daily storage: total records × Z
8. 5-year storage: daily × 365 × 5
```
