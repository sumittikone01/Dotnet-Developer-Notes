# 🛡️ Bulkhead Pattern

## 📌 What is it?

The **Bulkhead Pattern** isolates parts of a system into separate pools of resources (threads, connections, memory) so that **failure or overload in one part doesn't take down the entire system**.

## 🌍 Real-world analogy

The name comes from **ship design**. A ship's hull is divided into watertight compartments (bulkheads). If one compartment floods, the doors seal it off — the rest of the ship stays afloat.

```
Without Bulkheads:            With Bulkheads:
┌─────────────────┐           ┌───────┬───────┬───────┐
│                 │           │       │       │       │
│   ONE FLOOD  →  │           │ Flood │  Dry  │  Dry  │
│   SHIP SINKS 💥 │           │ Room  │ Room  │ Room  │
│                 │           │       │       │       │
└─────────────────┘           └───────┴───────┴───────┘
                                  Ship stays afloat ✅
```

## 🤔 Why do we need it?

Without isolation, one slow/failing dependency can exhaust **shared resources** (like a thread pool) and cause the **entire application** to become unresponsive — even for requests that have nothing to do with the failing dependency.

### Example scenario

Your app calls 3 services: `Payments`, `Inventory`, `Recommendations`.
If `Recommendations` starts responding slowly and all incoming requests use a **shared thread pool**, every thread eventually gets stuck waiting on `Recommendations` — so `Payments` and `Inventory` calls can't get a thread either, even though those services are healthy. **One weak link breaks everything.**

## ⚙️ Internal Working

Bulkheads are implemented by giving each dependency/module its **own limited pool** of:

- Threads
- Connections
- Queue slots
- Memory/CPU (in container-level bulkheading)

```
                 ┌─────────────────────────────┐
  Requests  ───▶ │        API Gateway          │
                 └───────────┬─────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
 ┌───────────────┐   ┌───────────────┐    ┌───────────────┐
 │ Pool: Payments│   │ Pool: Inventory│   │Pool: Recommend │
 │  (10 threads) │   │  (10 threads)  │    │  (10 threads)  │
 └───────────────┘   └───────────────┘    └───────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
   Payments API          Inventory API        Recommendations API
                                                (slow/failing 🐌)
```

If `Recommendations` pool saturates (all 10 threads stuck), **only recommendation requests fail** — Payments and Inventory keep working normally.

## 📊 Two Common Implementations

| Type                                  | Description                                                               | Example Tooling                           |
| ------------------------------------- | ------------------------------------------------------------------------- | ----------------------------------------- |
| **Thread pool isolation**       | Each dependency gets its own dedicated thread pool                        | Netflix Hystrix, Resilience4j             |
| **Semaphore isolation**         | Lightweight — limits concurrent calls via a counter, no separate threads | Resilience4j (lower overhead)             |
| **Process/Container isolation** | Separate services/pods per dependency entirely                            | Kubernetes resource limits, microservices |

## 💻 Code Example (C# — Polly Bulkhead Isolation)

```csharp
var bulkheadPolicy = Policy.BulkheadAsync(
    maxParallelization: 10,      // max concurrent calls to this dependency
    maxQueuingActions: 5         // extra requests queued before rejecting
);

await bulkheadPolicy.ExecuteAsync(() => callRecommendationServiceAsync());
```

If the pool + queue are full, new calls fail **fast** with a `BulkheadRejectedException` instead of piling up and starving the rest of the app.

## ⚡ Performance Considerations

- Isolation adds a small overhead (extra pools/threads use more memory).
- Pool sizes must be tuned — too small causes false rejections under normal load; too large defeats the purpose.

## 🚨 Common Mistakes

- ❌ Sharing one global thread pool across all downstream dependencies (defeats the whole purpose).
- ❌ Setting bulkhead limits too high — no real isolation benefit.
- ❌ Forgetting to combine with **Circuit Breaker** — bulkhead limits *concurrency*, it doesn't *detect* failure.

## 💡 Best Practices

- Apply bulkheads **per external dependency**, not per request type.
- Combine with **Circuit Breaker** (stop calling) + **Retry** (transient recovery) + **Timeout** (bound wait time) — together these four form the core **Resilience patterns** stack.
- Monitor pool saturation metrics to tune sizes correctly.

## 🎤 Interview Questions

1. What real-world problem does Bulkhead Pattern solve that Circuit Breaker doesn't?
2. Explain thread-pool isolation vs semaphore isolation.
3. Why can one slow dependency crash an entire application without bulkheads?
4. How would you size a bulkhead's thread pool in production?

## 📝 30-second Revision Cheat Sheet

- Bulkhead = isolate resources per dependency so one failure doesn't sink the whole app.
- Named after ship compartments.
- Implemented via separate thread pools / semaphores / containers.
- Prevents **resource starvation** cascades, unlike Circuit Breaker (which prevents wasted calls to a known-failing service).
- Always pair with Retry, Timeout, and Circuit Breaker for full resilience.
