# 🔁 Retry Pattern

## 📌 What is it?

The **Retry Pattern** is a reliability pattern where a failed operation (usually a network call) is **automatically re-attempted** a certain number of times before giving up, instead of failing immediately on the first error.

## 🤔 Why do we need it?

Most failures in distributed systems are **transient** — they resolve themselves within milliseconds to seconds:

- A brief network blip
- A server temporarily overloaded
- A momentary DB connection timeout
- A load balancer routing to an instance mid-restart

Without retries, a single transient hiccup causes a full request failure — even though the exact same request would have succeeded a moment later.

## 🌍 Real-world analogy

You call a friend and the call drops. You don't give up forever — you **call again**. If it fails 5 times in a row, *then* you assume something's really wrong and stop trying (that's where Circuit Breaker comes in 👉 see `01_Circuit_Breaker.md`).

## 🧠 Intuition

> Retry = "Maybe it was just bad luck. Let's try again."

But retrying blindly and instantly can make things **worse**, not better (see 🚨 below). So retries need **strategy**, not just repetition.

## ⚙️ Internal Working — Retry Strategies

| Strategy                               | How it works                                        | When to use                                     |
| -------------------------------------- | --------------------------------------------------- | ----------------------------------------------- |
| **Immediate Retry**              | Retry instantly, no delay                           | Only for extremely rare, ultra-transient errors |
| **Fixed Delay**                  | Wait a constant time (e.g. 2s) between retries      | Simple, predictable systems                     |
| **Exponential Backoff**          | Wait time doubles each retry (1s → 2s → 4s → 8s) | Most common in production                       |
| **Exponential Backoff + Jitter** | Add randomness to backoff to avoid retry storms     | ⭐ Industry best practice                       |

### 🖼 Exponential Backoff Flow

```
Attempt 1 fails ──▶ wait 1s ──▶ Attempt 2 fails ──▶ wait 2s ──▶ Attempt 3 fails ──▶ wait 4s ──▶ Attempt 4 (success ✅)
```

### Why Jitter matters

If 1000 clients all fail at the same time and all retry after exactly 2s, they'll **all hit the server again at the same instant** → causes a second wave of failure (a "retry storm" / "thundering herd"). Jitter randomizes the wait so retries spread out.

```
Without Jitter:  |----2s----|----2s----|  (all clients retry together 💥)
With Jitter:     |--1.3s--| |--2.7s--| |--1.9s--|  (spread out ✅)
```

## 💻 Code Examples (C# — Polly library, common in ASP.NET Core)

**Basic Retry:**

```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .Retry(3);
```

**Exponential Backoff + Jitter:**

```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(
        retryCount: 5,
        sleepDurationProvider: retryAttempt =>
            TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
            + TimeSpan.FromMilliseconds(new Random().Next(0, 1000))
    );

await retryPolicy.ExecuteAsync(() => httpClient.GetAsync(url));
```

## 🚨 Common Mistakes

- ❌ Retrying **non-idempotent** operations (e.g. "charge credit card") → can cause duplicate charges/double processing.
- ❌ No backoff → hammering an already-struggling service, making the outage worse.
- ❌ No max retry limit → infinite retry loops.
- ❌ No jitter → retry storms.
- ❌ Retrying errors that will **never** succeed (e.g. `400 Bad Request`, `401 Unauthorized`) — only retry **transient** errors (timeouts, `503`, `429`, connection resets).

## 💡 Best Practices

- Only retry **idempotent** operations, or make operations idempotent (e.g. using idempotency keys).
- Always use **exponential backoff with jitter** in production.
- Set a **max retry count** and a **max total wait time**.
- Combine with **Circuit Breaker** — stop retrying once a service is clearly down.
- Log every retry attempt for observability.

## 🎤 Interview Questions

1. Why is exponential backoff preferred over fixed-delay retry?
2. What is a "retry storm" and how does jitter solve it?
3. Should you retry a `POST` request that creates a payment? Why/why not?
4. How does Retry Pattern relate to Circuit Breaker?

## 📝 30-second Revision Cheat Sheet

- Retry = re-attempt failed transient operations.
- Use **exponential backoff + jitter**, not fixed delay.
- Only retry **idempotent** or transient-safe operations.
- Always cap max attempts.
- Pairs naturally with Circuit Breaker for full resilience.
