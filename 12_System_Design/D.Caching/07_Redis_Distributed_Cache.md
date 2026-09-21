# 07_Redis_Distributed_Cache

> **Redis** = an in-memory, key-value data store used as a **distributed cache** — a single shared cache that every server in your app talks to, instead of each server keeping its own separate copy.

> Closes the loop opened in `01_Cache_Basics.md` ("in-memory cache isn't shared across servers") and `05_Cache_Invalidation.md` (the load-balanced invalidation problem) — Redis is the standard fix for both.

## 📌 What is it?

Redis (**RE**mote **DI**ctionary **S**erver) is a standalone, network-accessible, in-memory data store. Instead of each ASP.NET Core server having its **own** `IMemoryCache`, all servers connect to **one Redis instance (or cluster)**, so they all see the same cached data.

```
Local (in-memory) cache:              Distributed cache (Redis):

Server A → [Cache A]                  Server A ─┐
Server B → [Cache B]  (different!)    Server B ─┼──► [ Redis ]  (one shared cache)
Server C → [Cache C]                  Server C ─┘
```

## 🤔 Why do we need it?

- Solves the **multi-server consistency problem**: one server's write is instantly visible to all servers.
- Survives **app restarts/deploys** — cache isn't wiped just because your ASP.NET Core process restarted (unlike `IMemoryCache`, which lives inside the process).
- Offloads memory pressure from your web servers — Redis runs as its own service, scaled independently.
- Supports more than just key-value: lists, sets, sorted sets, pub/sub, expirations — a genuine toolkit, not just a dictionary.

## 🌍 Real-world analogy

Going back to the **library front desk** analogy from `01_Cache_Basics.md`: local in-memory cache is like each library branch keeping its **own** private front-desk notes — branch A doesn't know what branch B put out. Redis is like a **single shared front desk system** all branches check — update it once, every branch sees the same answer.

## ⚙️ Internal working (typical flow with ASP.NET Core)

```
Client Request
      │
      ▼
ASP.NET Core App (any server behind load balancer)
      │
      ▼
IDistributedCache.GetAsync("product:5")
      │
   ┌──┴───┐
   │ Redis │  ← single shared store, reachable by ALL app servers
   └──┬───┘
      │ MISS
      ▼
  DAL → Stored Procedure → SQL Server
      │
      ▼
IDistributedCache.SetAsync("product:5", data)
      │
      ▼
  Return to client
```

Same Cache-Aside logic as `02_Cache_Aside.md` — the only thing that changed is **where** the cache lives (a shared external service instead of process memory).

## 📊 In-Memory Cache vs Redis (Distributed Cache)

| Aspect                 | `IMemoryCache` (local)                    | Redis (`IDistributedCache`)                           |
| ---------------------- | ------------------------------------------- | ------------------------------------------------------- |
| Shared across servers? | ❌ No — each server has its own            | ✅ Yes — one shared store                              |
| Survives app restart?  | ❌ No — lives in process memory            | ✅ Yes — separate process/service                      |
| Speed                  | Fastest (no network hop)                    | Slightly slower (network round-trip) — still very fast |
| Setup complexity       | None — built into ASP.NET Core             | Requires a Redis server/instance                        |
| Best for               | Single-server apps, per-request memoization | Multi-server / load-balanced production apps            |

## 💻 Code examples

### Basic — configuring Redis in ASP.NET Core

```csharp
// Program.cs
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = builder.Configuration.GetConnectionString("Redis"); // e.g. "localhost:6379"
    options.InstanceName = "MyApp_";
});
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "Redis": "localhost:6379"
  }
}
```

### Intermediate — using `IDistributedCache` (works with strings/byte arrays — needs serialization)

```csharp
public class ProductService
{
    private readonly IDistributedCache _cache;
    private readonly ProductDAL _dal;

    public ProductService(IDistributedCache cache, ProductDAL dal)
    {
        _cache = cache;
        _dal = dal;
    }

    public async Task<Product> GetProductByIdAsync(int id)
    {
        string cacheKey = $"product:{id}";

        // IDistributedCache only stores strings/bytes — must serialize
        string? cachedJson = await _cache.GetStringAsync(cacheKey);
        if (cachedJson != null)
        {
            return JsonSerializer.Deserialize<Product>(cachedJson)!; // HIT
        }

        // MISS — go through BAL → DAL → Stored Procedure
        var product = _dal.GetProductById(id);

        if (product != null)
        {
            var options = new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            };
            await _cache.SetStringAsync(cacheKey, JsonSerializer.Serialize(product), options);
        }

        return product!;
    }
}
```

### Practical — invalidation with Redis (same pattern as Cache-Aside, now shared instantly across servers)

```csharp
public async Task UpdateProductAsync(Product product)
{
    _dal.UpdateProduct(product);                          // 1. Update DB
    await _cache.RemoveAsync($"product:{product.Id}");    // 2. Invalidate — EVERY server sees this immediately
}
```

> Compare to local `IMemoryCache.Remove(...)` — that only clears the cache on the **one server handling this request**. `IDistributedCache.RemoveAsync(...)` against Redis clears it for **every server** at once. This is the exact fix for the distributed-invalidation problem described in `05_Cache_Invalidation.md`.

## ⚡ Performance considerations

- A Redis call involves a **network round trip** — slower than local memory access, but still typically sub-millisecond on a local network.
- Serialization/deserialization (JSON) adds a small CPU cost — consider a faster binary serializer (e.g., MessagePack) for very hot paths.
- Redis itself can be clustered/replicated for high availability — becomes its own small piece of infrastructure to monitor (ties into `Infrastructure` and `Logging_and_Monitoring` chapters later).

## 🚨 Common mistakes

- ❌ Treating Redis as "just like `IMemoryCache`" and forgetting it requires **serialization** (you can't store raw C# objects directly).
- ❌ No fallback if Redis is temporarily unreachable — app should degrade gracefully (fetch from DB) rather than crash.
- ❌ Not setting any expiration on Redis keys — unlike process memory, a forgotten Redis cache can silently grow for months.

## 💡 Best practices

- ✅ Always set an expiration on Redis entries — treat it the same as local cache in that regard.
- ✅ Wrap Redis calls in try/catch with a DB fallback — a cache should never be a single point of failure for your whole app.
- ✅ Use a consistent `InstanceName`/key prefix to avoid key collisions if multiple apps share one Redis instance.
- ✅ For read-heavy, multi-server ASP.NET Core apps (the typical production setup), Redis should be the **default** choice over local `IMemoryCache`.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                       | Answer                                                                                       |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| What problem does Redis solve that`IMemoryCache` can't?                      | Sharing cached data consistently across multiple app server instances behind a load balancer |
| Does data in Redis survive an app restart?                                     | Yes — Redis runs as a separate process/service, independent of your app's lifecycle         |
| What must you do before storing a C# object in Redis via`IDistributedCache`? | Serialize it (e.g., to JSON) —`IDistributedCache` only stores strings/byte arrays         |
| What should happen if Redis becomes unreachable?                               | The app should gracefully fall back to querying the DB directly, not crash                   |
| When is local`IMemoryCache` still preferable over Redis?                     | Single-server apps, or extremely hot per-request data where even a network hop is too slow   |

## 📝 30-second Revision Cheat Sheet

- Redis = shared, external, in-memory cache used by all app servers together.
- Fixes the two big local-cache problems: no cross-server sharing, and data lost on app restart.
- `IDistributedCache` only stores strings/bytes → must serialize objects (JSON).
- Same Cache-Aside logic as before — just swap `IMemoryCache` for `IDistributedCache`.
- Always set expiration + a DB fallback for when Redis is unreachable.
- Default choice for real, multi-server, load-balanced production apps.

---

✅ **D.Caching chapter complete** (01–07). Next up: **E.Database_Scaling**.
