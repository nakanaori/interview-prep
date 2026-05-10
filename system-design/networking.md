# Networking

Core networking concepts that come up in system design interviews.

---

## DNS (Domain Name System)

Translates domain names → IP addresses.

### Record Types

| Record | Purpose | Example |
|---|---|---|
| **A** | Domain → IPv4 address | `api.example.com → 1.2.3.4` |
| **AAAA** | Domain → IPv6 address | |
| **CNAME** | Alias → another domain | `www → example.com` |
| **MX** | Mail server | |
| **TXT** | Verification, SPF | |

### TTL (Time-To-Live)

How long DNS resolvers cache the record. Short TTL (60s) = fast failover, more DNS lookups. Long TTL (86400s) = fewer lookups, slow failover.

> For load balancing: use **DNS-based routing** to direct users to the nearest region. Each region gets its own IP via the same domain name.

---

## CDN (Content Delivery Network)

A globally distributed network of servers that caches content close to users.

### Push vs Pull

| Type | How | Use When |
|---|---|---|
| **Push CDN** | You upload content to CDN manually. | Small sites, infrequently updated content |
| **Pull CDN** | CDN fetches from origin on first request, caches it. | Large sites, frequently updated content |

> Default choice: **Pull CDN**. Set a sensible TTL on cacheable assets.

### What to Cache

- Static assets: JS, CSS, images, fonts ✅
- API responses (read-only, low-change) ✅
- HTML (if not user-specific) ✅
- User-specific data ❌
- Real-time data ❌

### CDN + Origin Flow

```
User → CDN edge (cache hit?) → return cached
                (cache miss?) → fetch from origin → cache → return
```

---

## HTTP Versions

| Version | Key Feature | When to Use |
|---|---|---|
| **HTTP/1.1** | One request per connection (with keep-alive). Head-of-line blocking. | Legacy, avoid |
| **HTTP/2** | Multiplexing — multiple requests over one connection. Header compression. Binary. | Default today |
| **HTTP/3** | Built on QUIC (UDP). No head-of-line blocking. Better on lossy networks. | Mobile-heavy apps |

---

## TCP vs UDP

| | TCP | UDP |
|---|---|---|
| **Connection** | Handshake required | Connectionless |
| **Reliability** | Guaranteed delivery, ordered | Best-effort, no ordering |
| **Speed** | Slower | Faster |
| **Use when** | Accuracy matters (APIs, DB, file transfer) | Speed matters, loss OK (video streaming, gaming, DNS) |

---

## Load Balancer: L4 vs L7

| | L4 (Transport) | L7 (Application) |
|---|---|---|
| **Operates on** | TCP/UDP | HTTP/HTTPS |
| **Routing by** | IP + port | URL path, headers, cookies |
| **Content-aware** | No | Yes |
| **SSL termination** | No | Yes |
| **Use when** | Maximum throughput, simple routing | API routing, A/B testing, auth |
| **Examples** | AWS NLB | AWS ALB, Nginx, Envoy |

> **Interview default: L7 load balancer.** It enables path-based routing (`/api → service A`, `/static → CDN`), SSL termination, and header inspection.

### Load Balancing Algorithms

| Algorithm | How | Best For |
|---|---|---|
| **Round-robin** | Requests distributed evenly in sequence | Uniform server capacity |
| **Weighted round-robin** | Heavier servers get more requests | Mixed server capacity |
| **Least connections** | Route to server with fewest active connections | Long-lived connections |
| **IP hash** | Same IP always hits same server | Stateful sessions (avoid — design stateless instead) |

---

## WebSocket vs SSE vs Long Polling

| | Long Polling | SSE | WebSocket |
|---|---|---|---|
| **Direction** | Client pulls | Server → client | Bidirectional |
| **Protocol** | HTTP | HTTP | WS |
| **Complexity** | Low | Low | Medium |
| **Use when** | Simple updates, low frequency | Dashboards, notifications | Chat, gaming, collaboration |

---

## HTTPS / TLS

- **TLS handshake:** client + server negotiate cipher, exchange certificates, establish session key
- **SSL termination at LB:** decrypt once at the load balancer, use HTTP internally. Faster (no re-encryption per service).
- **mTLS (mutual TLS):** both client and server present certificates. Used in service mesh for service-to-service auth.

> Interview phrase: *"SSL terminates at the API gateway. Internal traffic uses mTLS via the service mesh."*
