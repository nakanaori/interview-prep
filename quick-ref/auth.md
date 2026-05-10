# Auth & Rate Limiting

---

## JWT (JSON Web Token)

Self-contained token proving who the user is. Stateless — any server can verify, no session store needed.

### Structure: `header.payload.signature`

| Part | Contents |
|---|---|
| **Header** | Algorithm used to sign (e.g. HS256) |
| **Payload** | user_id, role, email, issued_at, expires_at. Base64-encoded, **not encrypted** — never put passwords/secrets here |
| **Signature** | HMAC hash of (header + payload + secret_key). Tampered payload = signature mismatch = rejected |

**Advantage:** stateless, no DB lookup per request.

**Disadvantage:** can't revoke before expiry.

**Solution:** short expiry (15 min) + refresh tokens.

---

## OAuth 2.0

Protocol for "Login with Google/GitHub." User never gives your app their password.

**Flow:**
1. User redirected to identity provider
2. Logs in at provider
3. Provider gives your server an auth code
4. Exchange auth code for access token
5. Get user profile → create your own JWT

---

## Interview Phrase

> *"API Gateway verifies JWT tokens. Users authenticate via OAuth 2.0 or credentials, receive JWT with short expiry + refresh token."*

---

## Rate Limiting

### Algorithms

| Algorithm | How | Best For |
|---|---|---|
| **Token Bucket** | Bucket fills at steady rate; each request takes a token; empty = rejected. Allows controlled bursts. | Default choice. Used by AWS, Stripe. |
| **Sliding Window** | Count requests in rolling time window. More accurate, no burst. | Strict per-minute limits |
| **Leaky Bucket** | Requests enter queue, processed at fixed rate. Queue full = dropped. Smooths traffic. | External API calls |

> **Interview answer:** *"I'd use token bucket with Redis."* Explain algorithm only if asked.

### Redis Implementation

```
INCR user:{id}:requests
EXPIRE user:{id}:requests 60
// reject if count > threshold
```
