# Observability

The three pillars: **Metrics**, **Logs**, **Traces**. You can't fix what you can't see.

---

## The Three Pillars

| Pillar | Answers | Tools |
|---|---|---|
| **Metrics** | Is the system healthy right now? | Prometheus, Datadog, CloudWatch |
| **Logs** | What happened and when? | ELK stack, Loki, CloudWatch Logs |
| **Traces** | Why is this request slow? | Jaeger, Zipkin, Datadog APM |

> Interview phrase: *"I'd instrument with the three pillars — metrics for alerting, structured logs for debugging, and distributed traces for latency analysis."*

---

## Metrics

### RED Method (for services)

| Metric | Meaning | Alert On |
|---|---|---|
| **Rate** | Requests per second | Sudden drop (traffic loss) |
| **Errors** | Error rate (%) | > 1% for critical paths |
| **Duration** | Latency (p50, p95, p99) | p99 > SLA threshold |

> Use **percentiles**, not averages. p99 = 99% of requests are faster than this. Averages hide tail latency.

### USE Method (for resources)

| Metric | Meaning |
|---|---|
| **Utilization** | % time resource is busy (CPU, disk) |
| **Saturation** | Queue depth, wait time |
| **Errors** | Error count from the resource |

### Key Metrics by Component

| Component | Watch |
|---|---|
| **App servers** | Request rate, error rate, p99 latency, CPU |
| **Database** | Query latency, connection pool usage, replication lag |
| **Redis** | Hit rate, eviction rate, memory usage |
| **Kafka** | Consumer lag, throughput, partition leader count |
| **Load balancer** | Active connections, 5xx rate, backend health |

---

## Logging

### Structured Logging

Always use **structured (JSON) logs**, not plain text strings.

```json
// ❌ Bad
"User 123 failed to purchase coupon 456 at 2026-05-10"

// ✅ Good
{
  "level": "error",
  "timestamp": "2026-05-10T10:30:00Z",
  "service": "coupon-service",
  "user_id": "123",
  "coupon_id": "456",
  "error": "quota_exceeded",
  "request_id": "abc-def-123",
  "duration_ms": 42
}
```

### What to Log

| Always log | Never log |
|---|---|
| Request ID (for tracing) | Passwords, secrets |
| User ID (not PII like email) | Full credit card numbers |
| Error type + message | Raw auth tokens |
| Latency | Excessive debug noise in prod |
| Outcome (success/failure) | |

### Log Levels

| Level | When |
|---|---|
| `DEBUG` | Detailed internal state — dev only |
| `INFO` | Normal operations (request received, job completed) |
| `WARN` | Unexpected but recoverable (retry succeeded, cache miss) |
| `ERROR` | Failure that needs investigation |
| `FATAL` | System cannot continue |

---

## Distributed Tracing

One request spans multiple services. Tracing follows it end-to-end.

### Concepts

| Term | Meaning |
|---|---|
| **Trace** | The full journey of one request across all services |
| **Span** | One unit of work within a trace (e.g., one DB query) |
| **Trace ID** | Unique ID propagated through all services in a request |
| **Span ID** | Unique ID for each individual span |
| **Parent span** | The span that triggered this one |

### How It Works

```
Client Request
└── API Gateway (span 1)        [trace_id: abc123]
    ├── Auth Service (span 2)
    ├── Order Service (span 3)
    │   ├── DB Query (span 4)
    │   └── Redis lookup (span 5)
    └── Notification Service (span 6)
```

Propagate `trace_id` in request headers between services. Each service creates a child span and records timing.

> Interview phrase: *"I'd use OpenTelemetry for instrumentation — it's vendor-neutral and exports to Jaeger, Datadog, or any backend."*

---

## Alerting

### Alert on Symptoms, Not Causes

| ❌ Don't alert on | ✅ Alert on |
|---|---|
| CPU > 80% | Error rate > 1% |
| Memory > 70% | p99 latency > 500ms |
| Disk > 60% | Availability < 99.9% |

> CPU high is a cause. Users experiencing errors is a symptom. Alert on what users feel.

### SLA / SLO / SLI

| Term | Meaning | Example |
|---|---|---|
| **SLA** (Agreement) | Contract with customer | "99.9% uptime or refund" |
| **SLO** (Objective) | Internal target | "99.95% uptime" |
| **SLI** (Indicator) | Metric being measured | "% of successful requests in 30 days" |

Set internal SLOs tighter than external SLAs. SLO = 99.95%, SLA = 99.9% → buffer for incidents.

---

## Health Checks

Every service should expose:

```
GET /health       → { "status": "ok" }           # Is the service alive?
GET /ready        → { "status": "ready" }         # Is it ready to serve traffic?
```

- **Liveness:** is the process running? Kubernetes restarts failed liveness checks.
- **Readiness:** is the service ready for traffic? Kubernetes removes from LB rotation if readiness fails.
