# 05_Cache_Invalidation

> **Cache Invalidation** = the process of removing or marking stale data in a cache so it doesn't get served after the underlying source has changed.

> Ties together `02_Cache_Aside.md` (invalidate-on-write), `03_Write_Through.md`, and `04_Write_Back.md` (all had to deal with staleness) — this chapter is the deep dive on the "hardest problem in computer science" joke.

## 📌 What is it?

Cache invalidation is how you answer: **"the data changed — now what happens to the old cached copy?"** There's no single universal answer; it's a set of strategies, each with trade-offs.

> 💬 Famous quote (Phil Karlton): *"There are only two hard things in Computer Science: cache invalidation and naming things."*

## 🤔 Why do we need it?

Without invalidation, a cache is just a bug generator — it will happily keep serving data that's no longer true. Every caching strategy (Cache-Aside, Write-Through, Write-Back) needs a clear invalidation story, or users will see **stale data**.

## 🌍 Real-world analogy

A **"Sale Price" sticker** on a store shelf. When the sale ends, if nobody removes the sticker (invalidates it), customers keep expecting the sale price at checkout — causing confusion and mismatches with the actual register price (DB).

## 📊 Invalidation Strategies — Comparison

| Strategy                                          | How it works                                                                  | Pros                                      | Cons                                                 |
| ------------------------------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------- | ---------------------------------------------------- |
| **TTL (Time-To-Live)**                      | Cache entry auto-expires after a fixed duration                               | Simple, no manual tracking                | Data can be stale for up to the TTL window           |
| **Explicit invalidation (delete-on-write)** | App deletes the key immediately when the source changes                       | Immediate consistency                     | Must remember to do it everywhere data changes       |
| **Write-Through update**                    | App updates the cache entry directly on write (see`03_Write_Through.md`)    | Cache never goes stale                    | More write complexity                                |
| **Version/ETag-based**                      | Cache key includes a version number; bump version on change                   | Old versions naturally "fall off"         | Slightly harder key management                       |
| **Event-driven invalidation**               | A message/event (e.g., pub/sub) tells other servers "key X changed, evict it" | Works across multiple distributed servers | Needs messaging infrastructure (e.g., Redis pub/sub) |

## ⚙️ Internal working — TTL vs Explicit, side by side

```
TTL-based:
  Set key, expires_at = now + 5 min
  ... time passes ...
  Read after 5 min → auto-expired → MISS → rebuild from DB

Explicit invalidation:
  Set key (no fixed expiry, or long expiry as safety net)
  DB UPDATE happens → App explicitly calls cache.Remove(key)
  Read immediately after → MISS → rebuild from DB (fresh right away, not after a wait)
```

**Key insight:** TTL alone reacts *eventually*; explicit invalidation reacts *immediately*. Production systems usually use **both** — explicit invalidation for immediacy, TTL as a safety net in case an invalidation call is ever missed.

## 🖼 The distributed-cache invalidation problem

```
Server A (updates Product #5, invalidates its LOCAL in-memory cache)
Server B (load-balanced sibling — still has stale Product #5 in ITS OWN in-memory cache!)
Server C (same problem)

           ┌─────────┐
Request → │  Load    │ → could hit Server A, B, or C — different answers!
           │ Balancer │
           └─────────┘
```

This is exactly why **distributed caching (Redis)** exists — one shared cache means invalidating once invalidates it for every server. In-memory (local) caches can't solve this cleanly without an event-driven "tell every server" mechanism.

## 💻 Code examples

### Basic — explicit invalidation on update (recap from Cache-Aside)

```csharp
public void UpdateProduct(Product product)
{
    _dal.UpdateProduct(product);              // 1. Update source of truth
    _cache.Remove($"product:{product.Id}");   // 2. Invalidate immediately
}
```

### Intermediate — invalidating related/derived keys too

```csharp
public void UpdateProduct(Product product)
{
    _dal.UpdateProduct(product);

    // Invalidate the specific product...
    _cache.Remove($"product:{product.Id}");

    // ...AND any cached lists/aggregates that included it!
    _cache.Remove("products:all");
    _cache.Remove($"products:category:{product.CategoryId}");
}
```

> 🚨 This is the #1 real-world invalidation bug: developers invalidate the single-item cache but forget the **list/summary caches** that also contain that item.

### Practical — version-based key (avoids explicit deletes entirely)

```csharp
// Store a "version" counter separately
int version = _cache.GetOrCreate("product:5:version", entry => 1);
string cacheKey = $"product:5:v{version}";

// On update, just bump the version — old key silently becomes unreachable and eventually expires via TTL
public void UpdateProduct(Product product)
{
    _dal.UpdateProduct(product);
    int currentVersion = _cache.Get<int>($"product:{product.Id}:version");
    _cache.Set($"product:{product.Id}:version", currentVersion + 1);
    // No need to hunt down and delete the old cache entry — it's simply never referenced again
}
```

## ⚡ Performance considerations

- Explicit invalidation adds a tiny bit of latency to every write (extra cache call) — negligible compared to the cost of serving stale data.
- Event-driven invalidation (pub/sub across servers) adds infrastructure complexity but is **necessary** once you scale beyond one server with local caching.

## 🚨 Common mistakes

- ❌ Invalidating the single-entity key but forgetting derived/list caches that also contain that entity.
- ❌ Relying only on TTL for data that changes unpredictably — users can see stale data for the entire TTL window.
- ❌ Assuming invalidating your own server's in-memory cache invalidates it everywhere (it doesn't, in a load-balanced setup).

## 💡 Best practices

- ✅ Use **explicit invalidation for immediacy + TTL as a safety net** — belt and suspenders.
- ✅ Track and invalidate *all* related keys (single item + lists/aggregates), not just the obvious one.
- ✅ For multi-server apps, either move to a distributed cache (Redis) or implement event-driven invalidation (pub/sub).

## 🎤 Interview Quick-Fire Q&A

| Question                                                                        | Answer                                                                                                                  |
| ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Why is cache invalidation considered "hard"?                                    | Multiple valid strategies exist, each with trade-offs, and it's easy to miss invalidating related/derived cache entries |
| What are the two most common invalidation strategies?                           | TTL (time-based expiry) and explicit invalidation (delete-on-write)                                                     |
| Why combine TTL with explicit invalidation?                                     | Explicit gives immediate consistency; TTL is a safety net if an invalidation call is ever missed                        |
| Why doesn't invalidating a local in-memory cache work in a load-balanced setup? | Each server has its own separate cache copy — invalidating one doesn't affect the others                               |
| What's a common real bug in invalidation logic?                                 | Invalidating the single-item key but forgetting cached lists/aggregates that also include that item                     |

## 📝 30-second Revision Cheat Sheet

- Invalidation = removing/expiring stale cache entries after the source changes.
- Two main strategies: TTL (auto-expiry) and explicit invalidation (delete-on-write) — use both together.
- Don't forget to invalidate **derived/list caches**, not just the single entity.
- Local in-memory caches don't sync across load-balanced servers — need Redis or pub/sub events for that.
- "Two hard things in CS: cache invalidation and naming things
