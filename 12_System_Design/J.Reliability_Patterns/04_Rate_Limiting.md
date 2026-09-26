# 🚦 Rate Limiting

## 📌 What is it?

**Rate Limiting** controls **how many requests** a client (user, IP, API key) can make to a system **within a given time window**, rejecting or delaying requests beyond that limit.

## 🤔 Why do we need it?

- Protects backend services from being **overwhelmed** (accidental or malicious).
- Ensures **fair usage** — one noisy client can't starve resources for everyone else.
- Prevents **abuse**: brute-force login attempts, scraping, DDoS-style traffic.
- Controls **cost** on paid downstream APIs (e.g. third-party services billed per call).

## 🌍 Real-world analogy

A **nightclub bouncer** only lets in a fixed number of people at a time. Once it's full, new people wait in line (queue) or are turned away (rejected) — this keeps the club from becoming dangerously overcrowded.

## 📊 Common Algorithms

| Algorithm                        | How it works                                                       | Pros                       | Cons                                      |
| -------------------------------- | ------------------------------------------------------------------ | -------------------------- | ----------------------------------------- |
| **Fixed Window**           | Count requests in fixed time blocks (e.g. per minute)              | Simple                     | Burst at window edges (2x limit possible) |
| **Sliding Window Log**     | Store timestamp of every request, count within rolling window      | Very accurate              | Memory-heavy at scale                     |
| **Sliding Window Counter** | Weighted average of current + previous fixed window                | Good accuracy, low memory  | Slightly approximate                      |
| **Token Bucket**           | Bucket refills tokens at fixed rate; each request consumes a token | Allows controlled bursts   | Slightly more complex                     |
| **Leaky Bucket**           | Requests queue and are processed at a constant fixed rate          | Smooths traffic completely | Can add latency under burst               |

### 🖼 Fixed Window Edge Problem

```
Limit: 5 req/minute

Window 1 [0:00 - 1:00]: ▓▓▓▓▓ (5 requests, all at 0:59)
Window 2 [1:00 - 2:00]: ▓▓▓▓▓ (5 requests, all at 1:00)

→ 10 requests actually landed within a 2-second span! 😬
```

### 🖼 Token Bucket (most widely used)

```
Bucket capacity: 5 tokens, refill 1 token/sec

[🪙🪙🪙🪙🪙]  ← full bucket, allows a burst of 5 instantly
Request arrives → consumes 1 token
[🪙🪙🪙🪙 ]  → refills over time back to 5
If bucket empty → request rejected (429) or queued
```

## ⚙️ Where Rate Limiting Is Applied

| Layer                                 | Example                              |
| ------------------------------------- | ------------------------------------ |
| **Client-side**                 | App throttles its own outgoing calls |
| **API Gateway / Reverse Proxy** | Nginx, Kong, AWS API Gateway         |
| **Application level**           | Middleware in ASP.NET Core           |
| **Database level**              | Connection pool limits               |

## 💻 Code Example (ASP.NET Core built-in Rate Limiting middleware)

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddTokenBucketLimiter("api", opt =>
    {
        opt.TokenLimit = 10;
        opt.TokensPerPeriod = 2;
        opt.ReplenishmentPeriod = TimeSpan.FromSeconds(1);
        opt.QueueLimit = 5;
    });
});

app.UseRateLimiter();

app.MapGet("/data", () => "Hello")
   .RequireRateLimiting("api");
```

Rejected requests typically return **HTTP 429 Too Many Requests**, often with a `Retry-After` header.

## ⚡ Performance Considerations

- Distributed systems need a **shared store** (Redis) to track counts across multiple servers — in-memory counters per server won't enforce a true global limit.
- Sliding Window Log is accurate but expensive at high scale; Token Bucket is the common production compromise.

## 🚨 Common Mistakes

- ❌ Rate limiting per-server instead of globally in a multi-instance deployment (each server allows the full limit → real limit = N × servers).
- ❌ Not returning `Retry-After` header — clients don't know when to retry.
- ❌ Same limit for all endpoints (login endpoint needs stricter limits than a read-only GET).
- ❌ Confusing rate limiting with **throttling of a single user's fair share** vs **DDoS protection** — they need different strategies.

## 💡 Best Practices

- Use **Token Bucket** for most APIs — allows natural bursts while controlling average rate.
- Store counters in **Redis** (or similar) for distributed enforcement.
- Return **429** with a `Retry-After` header.
- Apply **different limits per endpoint sensitivity** (e.g. stricter for `/login`).
- Combine with **API keys / user IDs**, not just IP (IPs can be shared behind NAT).

## 🎤 Interview Questions

1. Compare Token Bucket vs Leaky Bucket — when would you choose one over the other?
2. Why does Fixed Window allow bursts at window boundaries, and how does Sliding Window fix it?
3. How would you implement a global rate limiter across multiple server instances?
4. What HTTP status code and header should a rate-limited response return?

## 📝 30-second Revision Cheat Sheet

- Rate Limiting = cap requests per client per time window.
- **Token Bucket** = industry favorite (allows bursts, smooth average).
- **Fixed Window** = simple but has edge-burst problem.
- Distributed systems → use **Redis** for shared counting.
- Always respond with `429 Too Many Requests` + `Retry-After`.
