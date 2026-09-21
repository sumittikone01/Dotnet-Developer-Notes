# 04_Write_Back

> **Write-Back** (a.k.a. **Write-Behind**) = the write goes to the cache immediately and is acknowledged right away; the database is updated **later, asynchronously**.

> Builds on `03_Write_Through.md` — flips the ordering: instead of writing to DB first (or together), Write-Back writes to cache first and defers the DB write.

## 📌 What is it?

The app writes only to the cache. The cache holds the write and flushes it to the database **later** — either after a delay, in batches, or via a background worker. The caller doesn't wait for the DB at all.

## 🤔 Why do we need it?

- **Fastest possible writes** — caller only waits on an in-memory operation, not a disk-based DB write.
- Great for **write-heavy** workloads (e.g., view counters, analytics events, high-frequency sensor data) where individual DB writes would be too slow/costly.
- Allows **batching** many writes into fewer, more efficient DB operations (e.g., 1000 view-count increments flushed as a single `UPDATE` every 10 seconds instead of 1000 separate round trips).

## 🌍 Real-world analogy

A **restaurant order pad**. The waiter (cache) jots your order down immediately and tells you "got it!" right away — they don't run to the kitchen (DB) for every single item. Periodically, they batch several orders and hand them to the kitchen together. Fast for you, efficient for the kitchen — but if the waiter's notepad gets lost before reaching the kitchen, that order is gone.

## ⚙️ Internal working

```
WRITE Path:
  1. App writes to Cache
  2. Cache immediately acknowledges the write — App moves on (FAST)
  3. Cache (or a background process) later flushes the write to DB
     — could be: after a delay, on a schedule, or when a batch threshold is hit

READ Path:
  Cache always has the latest value (even before DB is updated) → reads are always fresh from cache
```

```
App ──write──► Cache ──ack immediately──► App continues

                Cache
                  │  (async, batched, delayed)
                  ▼
                  DB
```

## 📊 Full comparison: Cache-Aside vs Write-Through vs Write-Back

| Aspect            | Cache-Aside                      | Write-Through                | Write-Back                                        |
| ----------------- | -------------------------------- | ---------------------------- | ------------------------------------------------- |
| Write path        | App → DB, then invalidate cache | App → Cache → DB (sync)    | App → Cache only (async to DB)                   |
| Write speed       | Fast                             | Slow (2 systems, sync)       | Fastest (1 system, async)                         |
| Cache freshness   | Stale until next read            | Always fresh                 | Always fresh                                      |
| Risk of data loss | None                             | None                         | **Yes** — if cache crashes before flushing |
| Best for          | General-purpose reads            | Read-immediately-after-write | Write-heavy, high-throughput scenarios            |

## 💻 Code examples

### Basic — conceptual write-back with a background flush

```csharp
public class ViewCountService
{
    private readonly IMemoryCache _cache;
    private readonly ConcurrentQueue<int> _pendingFlushIds = new();

    public void IncrementViewCount(int productId)
    {
        string cacheKey = $"views:{productId}";

        // 1. Update cache immediately — this is the "write"
        int current = _cache.TryGetValue(cacheKey, out int val) ? val : 0;
        _cache.Set(cacheKey, current + 1, TimeSpan.FromHours(1));

        // 2. Queue this id for a later DB flush — DO NOT hit the DB now
        _pendingFlushIds.Enqueue(productId);

        // Caller gets control back immediately — no DB wait
    }
}
```

### Intermediate — background worker flushing batched writes

```csharp
public class ViewCountFlushWorker : BackgroundService
{
    private readonly ViewCountService _viewCountService;
    private readonly ProductDAL _dal;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await Task.Delay(TimeSpan.FromSeconds(10), stoppingToken);

            // Pull all pending counts from cache and flush as ONE batched DB call
            var batch = _viewCountService.DrainPendingCounts();
            if (batch.Count > 0)
            {
                _dal.BulkUpdateViewCounts(batch); // single stored-procedure call for the whole batch
            }
        }
    }
}
```

> This is the key win: **1000 increments → 1 DB call every 10 seconds**, instead of 1000 individual `UPDATE` statements.

## ⚡ Performance considerations

- Massive write-throughput improvement — ideal for counters, logs, telemetry, "likes"/"views".
- **Data loss window exists**: if the cache process crashes before the scheduled flush, those writes are gone forever (unless the cache itself is persisted, e.g. Redis with AOF/RDB persistence).
- Adds complexity: you need a background flushing mechanism, retry logic, and monitoring for flush failures.

## 🚨 Common mistakes

- ❌ Using Write-Back for **critical, non-recoverable data** (financial transactions, orders) — the data-loss risk is unacceptable there.
- ❌ No monitoring on the flush worker — if it silently stops, cache and DB drift apart indefinitely.
- ❌ Forgetting that reads must go through the **same cache**, not the DB directly, or they'll see outdated values (DB hasn't caught up yet).

## 💡 Best practices

- ✅ Reserve Write-Back for **high-volume, non-critical, tolerant-of-small-loss data**: view counts, analytics, logging, telemetry.
- ✅ Use a cache technology with persistence (e.g., Redis with AOF) to minimize the data-loss window.
- ✅ Always have monitoring/alerting on the background flush process.
- ✅ Never use Write-Back for financial or otherwise "must not lose" data — use Write-Through or plain synchronous writes there instead.

## 🎤 Interview Quick-Fire Q&A

| Question                                     | Answer                                                                           |
| -------------------------------------------- | -------------------------------------------------------------------------------- |
| What's another name for Write-Back?          | Write-Behind                                                                     |
| What makes Write-Back writes so fast?        | The caller only waits on the cache write; DB write happens later, asynchronously |
| What's the biggest risk of Write-Back?       | Data loss if the cache crashes before the delayed/batched write reaches the DB   |
| When should you avoid Write-Back?            | For critical data that must never be lost, e.g. financial transactions           |
| What's a real-world use case for Write-Back? | View counters, analytics events, telemetry/logging — high volume, loss-tolerant |

## 📝 30-second Revision Cheat Sheet

- Write-Back = write to cache only, DB updated later (async/batched).
- Fastest writes of the three patterns — but carries a **data-loss risk** if cache crashes before flush.
- Great for high-volume, loss-tolerant data (counters, logs, telemetry).
- Never use for critical data (payments, orders) — use Write-Through/Cache-Aside there.
- 
- Needs a background flush mechanism + monitorin
