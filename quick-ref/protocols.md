# API Protocols

---

## Quick Decision

| Scenario | Protocol |
|---|---|
| User-facing request-response | **REST** |
| Fast internal service-to-service | **gRPC** |
| Server-to-client push only | **SSE** |
| Two-way real-time | **WebSocket** |
| Flexible client data fetching | **GraphQL** |

---

## Protocol Details

### REST
Default for user-facing APIs. HTTP methods on resource URLs. Simple, stateless, widely understood.

### gRPC
Binary protocol (protobuf), faster than REST. Use for internal service-to-service calls. Supports streaming.

### GraphQL
Client specifies exact data shape. Avoids over/under-fetching. Use for flexible mobile API calls. Adds complexity — use only when needed.

### SSE (Server-Sent Events)
Server pushes to client, **one-directional**. Use for live feeds, alerts, progress updates. Simpler than WebSocket but one-way only.

### WebSocket
**Bidirectional**, persistent connection. Use for chat, live comments, anything needing two-way real-time communication.

---

## Common Architecture Combo

> REST (mobile → API gateway) + gRPC (service-to-service) + SSE or WebSocket (real-time features)

---

## Pub/Sub vs Message Queue

| Type | How | Use When |
|---|---|---|
| **Message Queue** (point-to-point) | One message → one consumer. Work is distributed. | Task distribution, background jobs |
| **Pub/Sub** (fan-out) | One message → ALL subscribers get a copy. | Broadcasting events to multiple services |
| **Kafka does both** | Within a consumer group = message queue. Across consumer groups = pub/sub. | Most production use cases |

> *"Kafka gives us fan-out across services via consumer groups, and work distribution within each service via partitions."*

---

## API Gateway

Single entry point between client and microservices. Handles:
- Routing
- JWT authentication
- Rate limiting
- Request transformation (REST → gRPC)
- Response aggregation

> Without it, the client needs to know every service address and handle auth separately.

Tools: Kong, AWS API Gateway, Nginx
