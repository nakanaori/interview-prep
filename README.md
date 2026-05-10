# Interview Prep

Personal reference for software engineering interviews — system design, self-introduction, classic problems, and quick-reference cheat sheets.

---

## 📝 Self-Introduction

- [Tell Me About Yourself](self-intro.md) — script, structure, and tips

---

## 🏗️ System Design

Core concepts, organized by topic.

| Section | Topics |
|---|---|
| [Foundations](system-design/foundations.md) | HTTP, load balancing, SQL, NoSQL, Redis |
| [Networking](system-design/networking.md) | DNS, CDN, HTTP versions, L4 vs L7, TLS |
| [Building Blocks](system-design/building-blocks.md) | Caching, Kafka, DB scaling, CAP theorem |
| [Observability](system-design/observability.md) | Metrics (RED/USE), logging, distributed tracing |
| [Payment Systems](system-design/payment-systems.md) | Auth flow, ledger, saga, reconciliation |
| [Mock Interview](system-design/mock-interview.md) | Full worked example with the 5-step framework |

---

## 💡 Classic Problems

Worked designs using the 5-step framework. Read once, then redo from memory.

| Problem | Key Concepts |
|---|---|
| [URL Shortener](examples/url-shortener.md) | Hashing, redirects, caching, analytics |
| [Social Feed](examples/social-feed.md) | Fan-out, ranking, pagination, Redis sorted sets |
| [Chat System](examples/chat-system.md) | WebSocket, delivery guarantees, presence, sharding |

---

## ⚡ Quick Reference

Fast lookup during review sessions.

| Cheat Sheet | Use For |
|---|---|
| [5-Step Framework](quick-ref/framework.md) | Structuring any system design answer |
| [Numbers to Know](quick-ref/numbers.md) | Latency, storage, QPS estimation |
| [SQL vs NoSQL](quick-ref/sql-nosql.md) | Choosing the right database |
| [Component Cheat Sheet](quick-ref/building-blocks.md) | When to use Redis, Kafka, LB, etc. |
| [Auth & Rate Limiting](quick-ref/auth.md) | JWT, OAuth, token bucket |
| [API Protocols](quick-ref/protocols.md) | REST, gRPC, GraphQL, SSE, WebSocket |
| [Problem → Solution Patterns](quick-ref/patterns.md) | "What if X?" → answer instantly |
