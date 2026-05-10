# Component Cheat Sheet

Quick lookup: which component to reach for and when.

---

## Core Components

| Component | Use When |
|---|---|
| **Redis** | Caching, rate limiting, counters, sessions, fast-reject layer |
| **Kafka** | Async processing, event-driven architecture, decoupling services |
| **Load Balancer** | Multiple app servers, distribute traffic |
| **Read Replicas** | Read-heavy workload (offload 80-90% of reads) |
| **Sharding** | Write-heavy, massive data that won't fit one DB |
| **JSONB (PostgreSQL)** | Flexible schema within a relational model |
| **Elasticsearch** | Full-text search, 15+ filter combinations, autocomplete |
| **CDN** | Static assets, global users, reduce origin load |
| **API Gateway** | Single entry point, JWT auth, rate limiting, routing |

---

## Pagination

| Type | How | When |
|---|---|---|
| **Offset-based** | `LIMIT 20 OFFSET 40` | Simple; degrades at high offsets; risk of skips on insert |
| **Cursor-based** | `WHERE created_at < cursor LIMIT 20` | Always fast; can't jump to page N; best for feeds |
| **Keyset** | Cursor with unique sortable field (ID) | Avoids duplicate timestamp issues |

> **Interview default: cursor-based.** *"Performs consistently regardless of how far back the user scrolls."*

---

## Deployment Strategies

| Strategy | How | Use When |
|---|---|---|
| **Blue-Green** | Two identical environments; switch traffic; instant rollback | Zero-downtime deploys |
| **Canary** | Route 5% → monitor → gradually increase to 100% | Critical services; catch regressions early |

> *"For critical services, I'd use canary deployments — 5% first, monitor, gradually increase. Instant rollback if anything looks wrong."*

---

## Containers & Kubernetes

| Concept | Description |
|---|---|
| **Container (Docker)** | Packages app + dependencies into portable unit |
| **Pod** | Smallest K8s unit, usually 1 container |
| **Deployment** | "Run N pods" |
| **Service** | Stable network address + load balancing |
| **HPA** | Auto-scale based on CPU/metrics |

> *"I'd scale horizontally using Kubernetes HPA — adds pods when CPU or queue depth exceeds threshold, scales down when load drops."*

---

## Advanced Patterns

| Pattern | When |
|---|---|
| **Service Mesh** (Linkerd/Istio) | 50+ microservices; sidecar handles retries, circuit breaking, mTLS automatically |
| **Leader Election** (Redis SETNX) | Scheduled job should run on only one instance |
| **Backpressure** | Producer sends faster than consumer processes → buffer in Kafka, scale consumers |
| **API Versioning** (URL path) | `/v1/resource` → `/v2/resource`; API gateway routes to correct version |
