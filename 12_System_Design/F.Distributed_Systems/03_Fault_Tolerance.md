# 03_Fault_Tolerance

> **Fault Tolerance** = a system's ability to **keep working correctly** even when some of its components (servers, disks, networks) fail.

> Where `01_CAP_Theorem.md` was about the *theoretical* trade-off when nodes can't communicate, Fault Tolerance is the *practical engineering* discipline of making sure a failure doesn't take your whole system down. Where `02_Consistent_Hashing.md` minimized disruption when servers are added/removed *intentionally*, this chapter covers *unintentional* failures.

## 📌 What is it?

No single server, disk, or network link is 100% reliable — hardware fails, processes crash, networks drop packets. **Fault tolerance** is the set of design techniques that let a system **absorb these failures** without the end user noticing (or with only minor, graceful degradation).

## 🤔 Why do we need it?

| Reality                                    | Consequence without fault tolerance                     |
| ------------------------------------------ | ------------------------------------------------------- |
| Servers crash / restart                    | One crash = total outage                                |
| Disks fail                                 | Data loss                                               |
| Networks partition (`01_CAP_Theorem.md`) | Requests time out or hang forever                       |
| A downstream dependency slows down         | The slowness cascades and takes everything down with it |

A fault-tolerant system assumes failure is **normal**, not exceptional, and designs for it from day one.

## 🌍 Real-world analogy

An **airplane with multiple engines**. It's designed so that if one engine fails mid-flight, the plane doesn't crash — it can still fly (perhaps at reduced performance) on the remaining engines and land safely. The failure is anticipated and planned for, not treated as a catastrophic surprise.

## 📊 Core Fault Tolerance Techniques

| Technique                      | What it does                                                              | Where you've already seen it                                                  |
| ------------------------------ | ------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **Redundancy**           | Run multiple copies of a component so one failing doesn't stop the system | Replicas (`01_Replication.md`), multiple app servers behind a load balancer |
| **Replication**          | Keep copies of data on multiple nodes                                     | `01_Replication.md`                                                         |
| **Failover**             | Automatically switch to a backup component when the primary fails         | Replica promoted to primary if the original primary dies                      |
| **Health Checks**        | Continuously probe components to detect failure early                     | Load balancers pinging servers (`C.Load_Balancing/03_Health_Checks.md`)     |
| **Timeouts & Retries**   | Don't wait forever for a failed dependency; retry transient failures      | HTTP client timeouts, retry policies                                          |
| **Circuit Breaker**      | Stop calling a dependency that's clearly failing, to prevent pile-up      | Covered in depth in`J.Reliability_Patterns/01_Circuit_Breaker.md`           |
| **Graceful Degradation** | Serve a reduced/partial experience instead of a full failure              | E.g., show cached data if the live DB is unreachable                          |

## ⚙️ Internal working — Failover flow

```
Normal operation:
   Client → Load Balancer → Primary DB (healthy)

Primary fails:
   Client → Load Balancer → Primary DB  ✗ (health check fails)
                              │
                    Failover triggered
                              ▼
   Client → Load Balancer → Promoted Replica (new Primary)

Client experience: a brief blip (or none, if failover is fast) — NOT a full outage
```

## 🖼 Redundancy — Single Point of Failure (SPOF) vs Fault-Tolerant design

```
SPOF (bad):                          Redundant (good):

Client → [ONE Server ] → DB          Client → [LB] → [Server A]
              ✗                                       [Server B]  → DB (replicated)
        (crashes = total outage)                       [Server C]
                                          (one crashes → other two absorb the load)
```

A **Single Point of Failure (SPOF)** is any one component whose failure takes down the entire system. Fault tolerance is largely the practice of **hunting down and eliminating SPOFs** — no single server, disk, or network path should be able to bring everything down alone.

## 💻 Code examples

### Basic — timeout + retry with exponential backoff

```csharp
public async Task<Product> GetProductWithRetryAsync(int id)
{
    int maxRetries = 3;
    int delayMs = 200;

    for (int attempt = 1; attempt <= maxRetries; attempt++)
    {
        try
        {
            using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(2)); // don't wait forever
            return await _productService.GetProductByIdAsync(id, cts.Token);
        }
        catch (Exception ex) when (attempt < maxRetries)
        {
            // Transient failure — wait a bit longer each time before retrying (exponential backoff)
            await Task.Delay(delayMs);
            delayMs *= 2;
        }
    }

    // All retries exhausted — fail gracefully rather than crash the whole request
    throw new ServiceUnavailableException("Product service unavailable after retries.");
}
```

### Intermediate — graceful degradation with a cache fallback

```csharp
public async Task<Product> GetProductWithFallbackAsync(int id)
{
    try
    {
        // Try the primary, live path first
        return await _dal.GetProductByIdAsync(id);
    }
    catch (SqlException)
    {
        // DB is down — fall back to whatever we last cached, even if slightly stale
        // (ties directly into D.Caching's Cache-Aside pattern)
        if (_cache.TryGetValue($"product:{id}", out Product cachedProduct))
        {
            return cachedProduct; // degraded, but the user still gets an answer
        }

        throw; // truly nothing available — only now does this become a hard failure
    }
}
```

### Practical — a simple health check endpoint (used by a load balancer)

```csharp
// Program.cs
app.MapGet("/health", async (AppDbContext dbCheck) =>
{
    try
    {
        // Lightweight check — just confirm the DB connection is alive
        await dbCheck.Database.CanConnectAsync();
        return Results.Ok("Healthy");
    }
    catch
    {
        return Results.StatusCode(503); // load balancer sees this and stops routing traffic here
    }
});
```

## ⚡ Performance considerations

- Retries must be used carefully — retrying too aggressively during a real outage can make things **worse** (a "retry storm" that overwhelms an already-struggling dependency). Exponential backoff exists specifically to prevent this.
- Health checks add small continuous overhead but are essential — the alternative (a load balancer sending traffic to a dead server) is far worse.
- Fault tolerance techniques generally trade a bit of **complexity and resource usage** (extra replicas, retry logic, health check infrastructure) for a large gain in **reliability**.

## 🚨 Common mistakes

- ❌ Retrying forever with no backoff or max-attempt limit — can turn a small blip into a cascading overload.
- ❌ No timeouts on external calls — a single slow/hung dependency can tie up threads/connections until the whole app grinds to a halt.
- ❌ Designing redundancy for servers but forgetting other SPOFs — a single load balancer, a single DNS entry, or a single network switch can just as easily be the thing that takes everything down.
- ❌ Treating "we have replicas" as automatically meaning "we're fault tolerant" — failover still needs to be automatic and tested, or a failure just becomes a slower outage instead of an instant one.

## 💡 Best practices

- ✅ Eliminate every Single Point of Failure you can find — servers, load balancers, network paths, even DNS.
- ✅ Always set timeouts on network/DB calls — never wait indefinitely.
- ✅ Use retries with exponential backoff, and a maximum attempt cap.
- ✅ Design for graceful degradation — a slightly-stale cached answer is almost always better than a hard error.
- ✅ Regularly test failover (don't just assume it works because it's configured — practice actual failure drills, sometimes called "chaos engineering").

## 🎤 Interview Quick-Fire Q&A

| Question                                                  | Answer                                                                                                                     |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| What is a Single Point of Failure (SPOF)?                 | Any single component whose failure would take down the entire system                                                       |
| Name three core techniques for achieving fault tolerance. | Redundancy, Failover, Health Checks (also valid: Retries with backoff, Circuit Breakers, Graceful Degradation)             |
| Why is exponential backoff important in retry logic?      | Prevents overwhelming an already-struggling dependency with a burst of immediate retries ("retry storm")                   |
| What is graceful degradation?                             | Serving a reduced or partial response (e.g., cached/stale data) instead of a hard failure when a dependency is unavailable |
| Why are timeouts critical for fault tolerance?            | Without them, a single slow/hung dependency can tie up resources indefinitely and cascade into a full outage               |

## 📝 30-second Revision Cheat Sheet

- Fault Tolerance = keep working correctly despite component failures — assume failure is normal.
- Core techniques: Redundancy, Replication, Failover, Health Checks, Timeouts + Retries (with backoff), Circuit Breakers, Graceful Degradation.
- Eliminate Single Points of Failure (SPOFs) everywhere — not just at the DB layer.
- Retries need a backoff strategy and a max-attempt cap, or they can make outages worse.
- Test failover in practice — configured redundancy that's never tested is not proven fault tolerance.
