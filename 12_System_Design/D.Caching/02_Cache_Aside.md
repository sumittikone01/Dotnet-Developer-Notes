# 02_Cache_Aside

> **Cache-Aside** (a.k.a. "Lazy Loading") = the application code is responsible for loading data into the cache — the cache never talks to the database on its own.

> Builds on `01_Cache_Basics.md`'s hit/miss concept — this chapter formalizes that flow into a named, industry-standard pattern.

## 📌 What is it?

The most common caching pattern. The **application** — not the cache — decides when to read from DB and when to populate the cache. The cache is just a passive key-value store sitting beside the app.

## 🤔 Why do we need it?

- It's the **simplest pattern to implement** — no special cache infrastructure needed beyond a basic key-value store.
- Only requested data gets cached (no wasted memory on unused rows) — contrast with Write-Through, where every write is cached whether it's read again or not.
- Naturally resilient: if the cache is down or empty, the app still works (just slower, hitting DB directly).

## 🌍 Real-world analogy

You (the app) keep a **sticky note** on your desk with your friend's phone number (cache) so you don't have to call the directory service (DB) every time. If the sticky note isn't there, you call the directory, get the number, then **write your own sticky note** — the directory doesn't do that for you.

## ⚙️ Internal working

```
READ Path:
  1. App asks cache for key
  2. Cache MISS → App queries DB directly
  3. App writes the result back into cache
  4. App returns result to caller

  Next read for same key → CACHE HIT → cache returns directly, DB untouched

WRITE Path (the part people forget!):
  1. App updates DB (via stored procedure)
  2. App INVALIDATES (deletes) the corresponding cache key
     — does NOT update the cache directly (safer, avoids race conditions)
  3. Next read → cache MISS → fresh data pulled from DB → cache repopulated
```

```
        ┌────────┐   miss    ┌────────┐
Read →  │ Cache  │ ────────► │   DB   │
        └────────┘           └────────┘
             ▲                    │
             └────── populate ────┘

Write →  DB updated  →  DELETE cache key  →  (next read repopulates it)
```

## 📊 Cache-Aside vs Write-Through (preview — full comparison in `03_Write_Through.md`)

| Aspect                        | Cache-Aside                 | Write-Through                |
| ----------------------------- | --------------------------- | ---------------------------- |
| Who writes to cache           | Application, on read (lazy) | Cache/system, on every write |
| First read after data changes | Miss (rebuild from DB)      | Hit (already fresh)          |
| Unused data                   | Never cached                | Cached even if never read    |
| Complexity                    | Low                         | Slightly higher              |

## 💻 Code examples

### Basic — Read path (cache-aside)

```csharp
public Product GetProductById(int id)
{
    string cacheKey = $"product:{id}";

    if (_cache.TryGetValue(cacheKey, out Product product))
    {
        return product; // HIT
    }

    // MISS — go through BAL → DAL → Stored Procedure (ADO.NET)
    product = _dal.GetProductById(id);

    if (product != null)
    {
        _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));
    }

    return product;
}
```

### Intermediate — Write path with invalidation

```csharp
public void UpdateProduct(Product product)
{
    // 1. Update DB first (calls sp_UpdateProduct via ADO.NET SqlCommand)
    _dal.UpdateProduct(product);

    // 2. Invalidate the stale cache entry — DO NOT try to "update" the cache here
    string cacheKey = $"product:{product.Id}";
    _cache.Remove(cacheKey);

    // Next GetProductById(product.Id) call will MISS and rebuild from DB
}
```

### Practical — Kendo Grid scenario

```csharp
// Controller action backing a Kendo Grid update
[HttpPost]
public IActionResult UpdateProduct([DataSourceRequest] DataSourceRequest request, ProductViewModel model)
{
    if (ModelState.IsValid)
    {
        _productService.UpdateProduct(model.ToEntity()); // handles DB update + cache invalidation internally
    }

    return Json(new[] { model }.ToDataSourceResult(request, ModelState));
}
```

> Why invalidate instead of update-in-place? If two threads update the cache directly at the same time, one write can overwrite the other with stale data. Deleting and letting the **next read rebuild from the source of truth (DB)** avoids that race condition entirely.

## ⚡ Performance considerations

- First request after invalidation is always a **miss** (slightly slower) — this is called the **"cold start" cost** of cache-aside.
- Under high concurrency, many simultaneous misses for the same key can cause a **"thundering herd"** (many threads hitting DB at once for the same data) — mitigated with locks or request coalescing (advanced topic).

## 🚨 Common mistakes

- ❌ Forgetting to invalidate cache after an UPDATE/DELETE stored procedure — classic bug causing users to see stale data indefinitely.
- ❌ Updating the cache value directly on write instead of invalidating — risks race conditions under concurrent writes.
- ❌ No expiration as a safety net — if invalidation logic is ever missed, data would be stale forever without a TTL fallback.

## 💡 Best practices

- ✅ Always pair cache-aside with a **TTL (expiration)** as a safety net, even though invalidation should handle most cases.
- ✅ Invalidate (delete), don't overwrite, cache entries on write.
- ✅ Keep cache key naming consistent between read and write paths (`product:{id}` everywhere).

## 🎤 Interview Quick-Fire Q&A

| Question                                         | Answer                                                                               |
| ------------------------------------------------ | ------------------------------------------------------------------------------------ |
| What is Cache-Aside also known as?               | Lazy Loading                                                                         |
| Who is responsible for populating the cache?     | The application code, not the cache system itself                                    |
| What should happen to cache on a DB write?       | The corresponding cache key should be invalidated (deleted), not updated in place    |
| Why delete instead of update the cache on write? | Avoids race conditions from concurrent writes overwriting each other with stale data |
| What's a downside of cache-aside?                | First read after invalidation is always a cache miss ("cold start")                  |

## 📝 30-second Revision Cheat Sheet

- App manages the cache — cache is passive.
- Read: check cache → miss → DB → populate cache → return.
- Write: update DB → **delete** cache key (don't update it directly) → next read rebuilds it.
- Only requested data gets cached — memory-efficient.
- Still pair with TTL/expiration as a safety net.
- Downside: cold-start miss right after invalidatio
