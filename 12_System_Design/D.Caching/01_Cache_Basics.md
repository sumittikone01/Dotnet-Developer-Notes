# 01_Cache_Basics

> **Caching** = storing a copy of expensive-to-fetch data somewhere faster, so future requests skip the expensive path.

## 📌 What is it?

A cache sits **between your application and the "true" source of data** (a database, an API, a file, a computed result). It holds a copy of frequently-accessed data in a fast-access layer (RAM) so you don't repeat slow work.

```
Without cache:
Client → App → SQL Server (slow: disk I/O, query planning, network)

With cache:
Client → App → Cache (RAM, microseconds) → SQL Server (only on miss)
```

## 🤔 Why do we need it?

| Problem without caching                         | How caching helps                |
| ----------------------------------------------- | -------------------------------- |
| Every request hits SQL Server → high load      | Repeated reads served from RAM   |
| Same expensive computation runs again and again | Result computed once, reused     |
| High latency for users far from DB              | Cache can sit closer to the user |
| DB becomes a bottleneck under load              | Cache absorbs most read traffic  |

Rule of thumb: **cache anything that is read often but changes rarely.**

## 🧠 Intuition

Think of caching as a **notebook of recent answers**. Instead of re-solving a math problem every time someone asks, you write the answer once and hand out the same page next time — until the answer changes and you have to erase and re-solve.

## 🌍 Real-world analogy

A **librarian's front desk**. Popular books are kept at the front desk (cache) instead of the back warehouse (database). Most requests are satisfied instantly at the desk. Rare books still require a trip to the warehouse (cache miss → DB hit).

## ⚙️ Internal working (Cache Hit / Miss flow)

```
Request for key "user:101"
        │
        ▼
   Is key in cache? ──Yes──► Return cached value  (CACHE HIT)
        │
        No
        ▼
   Fetch from DB (SQL Server)
        │
        ▼
   Store result in cache
        │
        ▼
   Return value to caller   (CACHE MISS)
```

- **Cache Hit** — data found in cache → fast return.
- **Cache Miss** — data not in cache → fallback to source, then populate cache.
- **Hit Ratio** = Hits / (Hits + Misses) → higher is better. Below ~80% often means your caching strategy needs tuning.

## 📊 Types of caching (by location)

| Type              | Where it lives         | Example                            | Notes                                                  |
| ----------------- | ---------------------- | ---------------------------------- | ------------------------------------------------------ |
| In-memory (local) | Inside the app process | `MemoryCache` in ASP.NET Core    | Fastest, but not shared across servers                 |
| Distributed       | Separate service       | Redis,`IDistributedCache`        | Shared across all app instances, survives app restarts |
| Client-side       | Browser                | HTTP cache headers, localStorage   | Reduces network round-trips entirely                   |
| Database-level    | DB engine itself       | SQL Server plan cache, buffer pool | Managed by SQL Server, not your code                   |
| CDN               | Edge servers           | Static files, images               | Covered later in Infrastructure chapter                |

## 💻 Code examples

### Basic — In-memory cache in ASP.NET Core

```csharp
// Program.cs
builder.Services.AddMemoryCache(); // registers IMemoryCache

// In your BAL/Service class
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly ProductDAL _dal;

    public ProductService(IMemoryCache cache, ProductDAL dal)
    {
        _cache = cache;
        _dal = dal;
    }

    public Product GetProductById(int id)
    {
        string cacheKey = $"product:{id}";

        // Try to get from cache first
        if (_cache.TryGetValue(cacheKey, out Product cachedProduct))
        {
            return cachedProduct; // CACHE HIT
        }

        // CACHE MISS — go to DAL (which calls a stored procedure via ADO.NET)
        Product product = _dal.GetProductById(id);

        // Store in cache for 5 minutes
        _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));

        return product;
    }
}
```

### Intermediate — with sliding + absolute expiration

```csharp
var cacheOptions = new MemoryCacheEntryOptions
{
    SlidingExpiration = TimeSpan.FromMinutes(2),   // resets timer on each access
    AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10) // hard cutoff regardless of access
};

_cache.Set(cacheKey, product, cacheOptions);
```

- **Sliding expiration** — "stays alive if actively used."
- **Absolute expiration** — "dies no matter what, after X time." Prevents stale data living forever.
- Best practice: **combine both** so hot data doesn't get evicted too soon, but nothing lives forever unchecked.

## ⚡ Performance considerations

- In-memory cache is **per-server** — in a load-balanced setup (multiple IIS instances), each server has its own copy → inconsistent data across servers. This is why **Distributed caching (Redis)** exists (next chapter references: `07_Redis_Distributed_Cache.md`).
- Caching too much = memory pressure on your server.
- Caching too little = defeats the purpose.

## 🚨 Common mistakes

- ❌ Caching data that changes every second (defeats the purpose, adds staleness risk).
- ❌ No expiration set → memory leak over time ("cache grows forever").
- ❌ Not invalidating cache after an UPDATE/DELETE stored procedure call → users see stale data.
- ❌ Using in-memory cache in a multi-server (load-balanced) app expecting consistency.

## 💡 Best practices

- ✅ Always set an expiration (sliding, absolute, or both).
- ✅ Use a consistent, predictable **key naming convention** (`entity:id`, e.g. `product:101`).
- ✅ Invalidate/update the cache entry right after a DB write (see `02_Cache_Aside.md`).
- ✅ Monitor your hit ratio — low hit ratio means your caching keys/strategy need rethinking.

## 🎤 Interview Quick-Fire Q&A

| Question                                                   | Answer                                                                                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| What is a cache hit vs miss?                               | Hit = found in cache; Miss = not found, fetched from source and then cached                                  |
| Why not just cache everything?                             | Memory is limited; stale/rarely-used data wastes space and risks staleness                                   |
| Difference between sliding and absolute expiration?        | Sliding resets on access; absolute expires at a fixed point regardless of access                             |
| Why doesn't in-memory cache work well with load balancers? | Each server instance has its own separate memory — no shared state → use distributed cache (Redis) instead |
| What metric tells you if caching is effective?             | Hit ratio (hits / total requests)                                                                            |

## 📝 30-second Revision Cheat Sheet

- Cache = fast copy of slow data, sits between app and source (DB).
- Hit = served from cache; Miss = fetched from DB + cached.
- Types: In-memory (local, per-server), Distributed (Redis, shared), Client-side, CDN.
- Always set expiration — sliding (resets on use) + absolute (hard cutoff).
- In-memory cache ≠ safe for multi-server apps → use Redis there.
- Track hit ratio to judge effectiveness.
