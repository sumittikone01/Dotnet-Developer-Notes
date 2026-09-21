# 03_Write_Through

> **Write-Through** = every write goes to the cache **and** the database together, synchronously, before the write is considered complete.

> Builds on `02_Cache_Aside.md` — instead of invalidating the cache on write and letting the next read rebuild it, Write-Through keeps the cache fresh immediately.

## 📌 What is it?

In this pattern, the **cache sits in front of the write path too**, not just reads. The app writes to the cache, and the cache (or the app, acting like the cache) immediately writes through to the database. The write is only "done" once both are updated.

## 🤔 Why do we need it?

- Guarantees the cache is **never stale** — data is written to both places at once.
- Great for data that's **read again immediately after being written** (e.g., a user updates their profile and expects to see it instantly).
- Removes the "cold start miss" problem that Cache-Aside has right after invalidation.

## 🌍 Real-world analogy

A **bank passbook update**. When you deposit money, the teller updates the bank's main ledger (DB) **and** your passbook (cache) at the same time, in the same transaction. You never walk away with a passbook that disagrees with the ledger.

## ⚙️ Internal working

```
WRITE Path:
  1. App sends write request
  2. Write goes to Cache
  3. Cache immediately writes through to DB (synchronously)
  4. Only after DB confirms → write is acknowledged as complete

READ Path:
  Same as before — check cache first, it's always fresh, so hits are common
```

```
        write            write
App ─────────► Cache ─────────► DB
                 │                │
                 └──── ack only after DB confirms ────┘
```

Compare to Cache-Aside's write path, which deletes the key and waits for the *next read* to repopulate — Write-Through repopulates **immediately, as part of the write itself**.

## 📊 Cache-Aside vs Write-Through vs Write-Back (preview of `04_Write_Back.md`)

| Aspect                      | Cache-Aside                         | Write-Through                    | Write-Back                                        |
| --------------------------- | ----------------------------------- | -------------------------------- | ------------------------------------------------- |
| Cache freshness after write | Stale until next read (invalidated) | Always fresh immediately         | Fresh in cache, DB lags behind                    |
| Write latency               | Fast (only DB touched)              | Slower (cache + DB, synchronous) | Fastest (only cache touched)                      |
| Risk of data loss           | None (DB is always current)         | None (DB always current)         | Possible (if cache crashes before flushing to DB) |
| Unused data cached?         | No                                  | Yes, every write is cached       | Yes                                               |

## 💻 Code examples

### Basic — Write-through service method

```csharp
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly ProductDAL _dal;

    public void UpdateProduct(Product product)
    {
        string cacheKey = $"product:{product.Id}";

        // 1. Write to DB first (source of truth) — via stored procedure/ADO.NET
        _dal.UpdateProduct(product);

        // 2. Immediately write the SAME fresh data into cache — no invalidation, no waiting
        _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));

        // Any read right after this will be a HIT with fresh data
    }
}
```

### Intermediate — Write-through on Create

```csharp
public Product CreateProduct(Product product)
{
    // 1. Insert into DB, get back generated Id
    int newId = _dal.InsertProduct(product);
    product.Id = newId;

    // 2. Populate cache immediately with the newly created entity
    string cacheKey = $"product:{newId}";
    _cache.Set(cacheKey, product, TimeSpan.FromMinutes(5));

    return product;
}
```

> Notice the difference from Cache-Aside: there is **no `_cache.Remove(...)`** call anywhere. We proactively `Set` fresh data instead of deleting and waiting.

## ⚡ Performance considerations

- Every write pays the **cost of updating two systems** (cache + DB) synchronously → higher write latency than Cache-Aside.
- Not ideal for **write-heavy** workloads with data that's rarely re-read — you pay the cache-write cost for data nobody asks for again.
- Best suited for **read-heavy + read-immediately-after-write** scenarios.

## 🚨 Common mistakes

- ❌ Using Write-Through for write-heavy, rarely-read data — wastes cache writes for no benefit.
- ❌ Forgetting that write latency increases (some devs assume caching always makes things faster — writes actually get slower here).
- ❌ Not handling partial failure (DB write succeeds but cache write fails) — decide up front which system is the "source of truth" if they disagree (usually DB).

## 💡 Best practices

- ✅ Use Write-Through when reads-after-write are frequent (profile pages, settings screens, dashboards).
- ✅ Treat the DB as the ultimate source of truth — if cache write fails after DB write succeeds, log it and let TTL/expiration eventually self-correct.
- ✅ Combine with a reasonable TTL anyway, as a safety net against silent cache/DB drift.

## 🎤 Interview Quick-Fire Q&A

| Question                                                       | Answer                                                                                                           |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| What's the core idea of Write-Through?                         | Every write updates cache and DB together, synchronously, before being acknowledged                              |
| How is it different from Cache-Aside on write?                 | Cache-Aside deletes the key and waits for next read; Write-Through updates the cache immediately with fresh data |
| What's the main trade-off?                                     | Slower writes (two systems updated) in exchange for always-fresh cache and no cold-start misses                  |
| When is Write-Through the right choice?                        | When data is read again immediately/frequently after being written                                               |
| What happens if the cache write fails after DB write succeeds? | DB (source of truth) remains correct; cache is briefly stale until next explicit set or TTL expiry               |

## 📝 30-second Revision Cheat Sheet

- Write-Through = write to cache + DB together, synchronously.
- No invalidation needed — cache is always fresh right after write.
- Slower writes than Cache-Aside, but no cold-start miss on read.
- Best for read-heavy, read-immediately-after-write scenarios.
- DB remains the source of truth if the two ever disagre
