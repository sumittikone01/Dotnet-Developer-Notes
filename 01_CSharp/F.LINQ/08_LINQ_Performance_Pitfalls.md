
# 08_LINQ_Performance_Pitfalls

> A dedicated round-up of LINQ's most common, real-world performance mistakes — pulling together and expanding on warnings scattered across `01`–`07`, into one focused reference for writing genuinely fast LINQ code.

> Closes out **F.LINQ** for real this time (01–08). Everything here is a "gotcha" — code that looks fine, compiles fine, and produces the correct RESULT, but does so far less efficiently than it should.

## 📌 What is it?

LINQ's declarative style can hide the actual COST of an operation — a one-line LINQ chain might quietly do multiple full passes over a collection, or repeatedly hit a database, without any visual sign in the code that it's expensive. This chapter is the checklist for spotting that.

## 🚨 Pitfall #1 — Multiple Enumeration (recap + the most common offender)

```csharp
// ❌ BAD — 'query' is enumerated TWICE, running the full pipeline both times
var query = products.Where(p => p.IsActive);
Console.WriteLine(query.Count());
foreach (var p in query) { ProcessProduct(p); }
```

```csharp
// ✅ GOOD — materialize once, reuse the concrete List
var activeProducts = products.Where(p => p.IsActive).ToList();
Console.WriteLine(activeProducts.Count);      // List<T>.Count — O(1), no re-execution
foreach (var p in activeProducts) { ProcessProduct(p); }
```

> Full detail in `07_Deferred_vs_Immediate_Execution.md` — this is simply the #1 most common real-world LINQ performance bug, worth restating as pitfall #1.

## 🚨 Pitfall #2 — Filtering AFTER Projecting (wrong order)

```csharp
// ❌ BAD — projects EVERY element into a DTO first, THEN filters — wasted work on discarded elements
var result = products
    .Select(p => new ProductDto { Name = p.Name, Price = p.Price })
    .Where(dto => dto.Price > 100)
    .ToList();

// ✅ GOOD — filter FIRST (fewer elements), THEN project only the survivors
var result = products
    .Where(p => p.Price > 100)
    .Select(p => new ProductDto { Name = p.Name, Price = p.Price })
    .ToList();
```

```
BAD order:   [1000 products] → Select (1000 DTOs created) → Where (990 DISCARDED — wasted work!)
GOOD order:  [1000 products] → Where (990 discarded EARLY, cheaply) → Select (only 10 DTOs created)
```

## 🚨 Pitfall #3 — `Count() > 0` instead of `Any()`

```csharp
// ❌ BAD — Count() must potentially enumerate the ENTIRE sequence to get an exact count
if (products.Count(p => p.IsActive) > 0) { ... }

// ✅ GOOD — Any() stops at the FIRST match — doesn't need to scan the rest
if (products.Any(p => p.IsActive)) { ... }
```

```
Count(predicate) > 0:   scans potentially ALL elements to compute the exact total, THEN compares
Any(predicate):          stops IMMEDIATELY at the first match — often far fewer elements checked
```

> For a collection where a match is likely near the start, `Any()` can be dramatically faster. Even in the worst case (no matches at all), `Any()` is never slower than `Count() > 0` — there's no reason to prefer `Count() > 0` for an existence check.

## 🚨 Pitfall #4 — Recomputing the same LINQ expression repeatedly inside a loop

```csharp
// ❌ BAD — re-filters the ENTIRE products list on every single iteration of the outer loop
foreach (var order in orders)
{
    var product = products.FirstOrDefault(p => p.Id == order.ProductId); // O(n) EVERY time — O(n*m) overall!
    ...
}

// ✅ GOOD — build a lookup ONCE (O(n)), then O(1) lookups inside the loop
var productLookup = products.ToDictionary(p => p.Id);
foreach (var order in orders)
{
    productLookup.TryGetValue(order.ProductId, out var product); // O(1) — dramatically faster
    ...
}
```

```
BAD:   for each of M orders → scan ALL N products → O(N × M) total
GOOD:  build ONE dictionary (O(N)) → for each of M orders, O(1) lookup → O(N + M) total
```

> This is one of the highest-impact fixes in this whole list — turning an accidental O(N×M) nested scan into a fast O(N+M) with `ToDictionary`/`ToLookup`.

## 🚨 Pitfall #5 — Pulling everything into memory before filtering/paginating (recap from earlier chapters)

```csharp
// ❌ BAD — loads the ENTIRE table into memory, THEN filters/paginates in LINQ
List<Product> allProducts = _dal.GetAllProducts(); // pulls potentially MILLIONS of rows
var page = allProducts.Where(p => p.Category == "Electronics").Skip(20).Take(10).ToList();

// ✅ GOOD — push filtering AND pagination into the stored procedure/SQL itself
var page = _dal.GetProductsPaged(category: "Electronics", pageNumber: 3, pageSize: 10);
// (SQL does the filtering + OFFSET/FETCH — only 10 rows ever cross the network)
```

> Ties directly to `12_System_Design/I.API_Design/04_Pagination_and_Filtering.md` — the exact same principle: don't do at the application layer what the database can do far more efficiently with indexes.

## 🚨 Pitfall #6 — Using `OrderBy` when you only need `Min`/`Max`

```csharp
// ❌ BAD — sorts the ENTIRE collection (O(n log n)) just to grab the first element
var cheapest = products.OrderBy(p => p.Price).First();

// ✅ GOOD — Min/Max find the answer in a SINGLE pass (O(n)), no sorting needed
decimal cheapestPrice = products.Min(p => p.Price);
Product cheapestProduct = products.MinBy(p => p.Price); // .NET 6+ — returns the ELEMENT, not just the value
```

```
OrderBy(...).First():  O(n log n) — sorts EVERYTHING just to look at the first item
Min() / MinBy(...):     O(n) — single pass, no sorting required
```

## 🚨 Pitfall #7 — Repeated aggregate calls that could be combined into one pass

```csharp
// ❌ LESS EFFICIENT — 3 separate full passes over 'orders'
int count = orders.Count();
decimal total = orders.Sum(o => o.Amount);
decimal max = orders.Max(o => o.Amount);

// ✅ MORE EFFICIENT (for very large collections) — ONE pass, computing all three together
var (count2, total2, max2) = orders.Aggregate(
    (Count: 0, Total: 0m, Max: decimal.MinValue),
    (acc, o) => (acc.Count + 1, acc.Total + o.Amount, Math.Max(acc.Max, o.Amount)));
```

> This is a genuine trade-off: the combined `Aggregate` version is noticeably less readable. Only reach for it when profiling shows the multiple passes are actually a measured bottleneck on a very large collection — for typical business-app data sizes, the three separate, clearer calls are usually the better choice.

## 📊 Summary Table — Pitfall, Fix, and Impact

| Pitfall                                                 | Fix                                                    | Typical impact                                            |
| ------------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------------- |
| Multiple enumeration of the same query                  | `.ToList()` once, reuse                              | Avoids re-running the whole pipeline (or re-hitting a DB) |
| Filter after project                                    | Filter, THEN project                                   | Avoids projecting elements that get discarded anyway      |
| `Count() > 0` for existence checks                    | Use`Any()`                                           | Stops at first match instead of scanning everything       |
| Re-scanning a list inside a loop                        | `ToDictionary`/`ToLookup` once                     | O(N×M) → O(N+M) — often the biggest win on this list   |
| Filtering/paginating in-memory after loading everything | Push filtering/paging into SQL                         | Avoids pulling unnecessary rows across the network at all |
| `OrderBy(...).First()` for a single min/max           | Use`Min`/`Max`/`MinBy`/`MaxBy`                 | O(n log n) → O(n)                                        |
| Multiple separate aggregate passes                      | Combine with`Aggregate` (only if profiled as needed) | Fewer passes, at the cost of readability                  |

## 💻 Code examples

### Practical — the "before and after" for a real BAL method

```csharp
// ❌ BEFORE — several pitfalls stacked together
public List<ProductDto> GetTopExpensiveElectronics()
{
    var allProducts = _dal.GetAllProducts();                        // pitfall #5: pulls everything
    var electronics = allProducts.Where(p => p.Category == "Electronics");
    if (electronics.Count() > 0)                                     // pitfall #3: Count() > 0
    {
        var sorted = electronics.OrderByDescending(p => p.Price);    // fine here, since we need TOP N, not just one
        var dtos = sorted.Select(p => new ProductDto { Name = p.Name, Price = p.Price }); // ok — after filter
        return dtos.Take(5).ToList();
    }
    return new List<ProductDto>();
}

// ✅ AFTER — fixed
public List<ProductDto> GetTopExpensiveElectronics()
{
    // Push category filter + "top 5 by price" into the stored procedure directly
    return _dal.GetTopExpensiveProductsByCategory("Electronics", topN: 5)
        .Select(p => new ProductDto { Name = p.Name, Price = p.Price })
        .ToList();
}
```

## ⚡ General performance principle

- LINQ's readability can hide cost — always ask **"how many times does this actually iterate the data, and over how much data?"** for any LINQ chain touching a large collection.
- Most of these pitfalls matter far more at **scale** (large collections, hot code paths called frequently) — don't prematurely over-optimize simple, small, one-off queries at the expense of readability.

## 🚨 Common mistakes (meta-level)

- ❌ Applying every optimization in this chapter everywhere, even on small, rarely-called collections — adds complexity for no measurable benefit.
- ❌ Optimizing LINQ code without ever profiling/measuring first — some "pitfalls" here are genuinely negligible at small scale.
- ❌ Forgetting that the BEST fix for large-dataset LINQ performance problems is usually "push the work into SQL," not "write cleverer LINQ."

## 💡 Best practices

- ✅ Reserve these optimizations for LINQ chains that run over genuinely large collections, or that run very frequently (hot paths) — profile before micro-optimizing elsewhere.
- ✅ Default to `Any()` over `Count() > 0` as a habit — there's no downside, ever.
- ✅ Default to `ToDictionary`/`ToLookup` the moment you notice a LINQ lookup happening inside a loop — this is the highest-impact, easiest-to-spot fix on this list.
- ✅ Remember the overarching theme from `12_System_Design/D.Caching` and `E.Database_Scaling`: whenever possible, let the DATABASE do filtering/sorting/aggregation work at scale — LINQ-to-Objects is for shaping already-reasonably-sized, in-memory data.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                                                                     | Answer                                                                                                                                  |
| ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| Why is`Any()` generally preferred over `Count() > 0` for existence checks?                                               | `Any()` stops at the first match; `Count()` must potentially scan the entire sequence to produce an exact total                     |
| What's the performance problem with`products.FirstOrDefault(...)` called repeatedly inside a loop over another collection? | It turns an O(n) lookup into an O(n×m) nested scan; building a`Dictionary`/`Lookup` once first reduces this to O(n+m)              |
| Why filter before projecting, rather than after?                                                                             | Filtering first discards unwanted elements early, avoiding the cost of projecting/transforming elements that will just be thrown away   |
| Why is`OrderBy(...).First()` inefficient for finding a minimum?                                                            | It sorts the entire sequence (O(n log n)) just to look at the first element;`Min()`/`MinBy()` find the same answer in one O(n) pass |
| What's generally the biggest lever for LINQ performance on large datasets?                                                   | Pushing filtering, sorting, and aggregation into the database/SQL layer instead of pulling everything into memory for LINQ to process   |

## 📝 30-second Revision Cheat Sheet

- Materialize (`.ToList()`) once if a query will be enumerated more than once — avoids re-running the whole pipeline.
- Filter BEFORE projecting; use `Any()` instead of `Count() > 0`; use `Min`/`Max`/`MinBy`/`MaxBy` instead of `OrderBy(...).First()`.
- Building a `Dictionary`/`Lookup` before a loop turns O(N×M) nested scans into O(N+M) — often the single biggest real-world win.
- For large datasets, push filtering/sorting/pagination into SQL — don't pull everything into memory for LINQ to process.
- Optimize where it's actually measured to matter — don't sacrifice readability on small, infrequent queries for no real benefit.

---

✅ **F.LINQ chapter now fully complete** (01–08). Back to **G.Modern_CSharp** — 01 done, next up 02–03.
