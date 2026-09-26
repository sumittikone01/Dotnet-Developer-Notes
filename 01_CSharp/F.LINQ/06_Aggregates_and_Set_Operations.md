# 06_Aggregates_and_Set_Operations

> **Aggregates** = operators that reduce a sequence down to a single value (a count, sum, average, min/max). **Set Operations** = operators that treat sequences like mathematical sets (union, intersect, except) — comparing elements ACROSS two sequences rather than within one.

> Closes out **F.LINQ**. These are the operators that finally take you from "shaped sequence" (Filtering/Projection/Ordering/Grouping/Joining) to a **single concrete answer** — the natural last stop in a LINQ pipeline.

## 📌 What is it?

```
AGGREGATES: many elements → ONE value

products.Count()          → 42
products.Sum(p => p.Price) → 1249.50
products.Max(p => p.Price) → 199.99

SET OPERATIONS: two sequences → ONE combined/compared sequence

listA.Union(listB)         → all UNIQUE elements from BOTH
listA.Intersect(listB)     → only elements present in BOTH
listA.Except(listB)        → elements in listA that are NOT in listB
```

## 🤔 Why do we need them?

| Need                                                                | Operator family               |
| ------------------------------------------------------------------- | ----------------------------- |
| "How many products are in stock?"                                   | Aggregate (`Count`)         |
| "What's the total value of all orders today?"                       | Aggregate (`Sum`)           |
| "What's the most expensive product?"                                | Aggregate (`Max`)           |
| "Which customers are in BOTH our email list and our SMS list?"      | Set operation (`Intersect`) |
| "Which products were in last month's catalog but NOT this month's?" | Set operation (`Except`)    |

These are exactly the everyday questions a Kendo Grid's summary row, a dashboard tile, or a comparison report needs — all backed by a single LINQ call rather than manual loops with running totals.

## 🌍 Real-world analogy

**Aggregates** are like a calculator's totals button on a receipt — take every line item and boil it down to one number (subtotal, count of items, most expensive item). **Set Operations** are like comparing two guest lists for a merged event — who's on BOTH lists (`Intersect`), who's on EITHER list (`Union`), and who was invited to party A but NOT party B (`Except`).

## 📊 Aggregate Operators — Full Reference

| Operator                              | Returns      | Notes                                                                         |
| ------------------------------------- | ------------ | ----------------------------------------------------------------------------- |
| `Count()` / `Count(predicate)`    | `int`      | Total count, or count matching a condition                                    |
| `Sum(selector)`                     | numeric type | Adds up a numeric property across all elements                                |
| `Average(selector)`                 | numeric type | ⚠️ Throws`InvalidOperationException` on an EMPTY sequence                 |
| `Min(selector)` / `Max(selector)` | element type | ⚠️ Also throws on an empty sequence                                         |
| `Aggregate(seed, func)`             | custom       | A general-purpose "roll your own aggregate" — full custom accumulation logic |
| `Any()` / `Any(predicate)`        | `bool`     | "Does at least one element exist / match?"                                    |
| `All(predicate)`                    | `bool`     | "Do ALL elements match this condition?"                                       |

## 🖼 The Empty-Sequence Trap — a real, common bug

```csharp
List<Product> products = new List<Product>();  // EMPTY list

products.Count();              // ✅ returns 0 — safe
products.Sum(p => p.Price);    // ✅ returns 0 — safe
products.Average(p => p.Price); // ❌ THROWS InvalidOperationException — "Sequence contains no elements"
products.Max(p => p.Price);     // ❌ THROWS InvalidOperationException
products.First();               // ❌ THROWS InvalidOperationException
products.FirstOrDefault();      // ✅ returns null/default — SAFE alternative
```

> **Rule of thumb:** `Count`/`Sum` are always safe on empty sequences (return 0). `Average`/`Min`/`Max`/`First`/`Single` are NOT — always check `.Any()` first, or use the `OrDefault` variant where one exists.

## ⚙️ Internal working — `Aggregate` (the general-purpose accumulator)

```csharp
// Simulates Sum() manually, using Aggregate — shows what's happening "under the hood"
int total = numbers.Aggregate(0, (accumulator, current) => accumulator + current);

// Step by step, for numbers = [1, 2, 3, 4]:
// accumulator=0, current=1 → returns 1
// accumulator=1, current=2 → returns 3
// accumulator=3, current=3 → returns 6
// accumulator=6, current=4 → returns 10
// FINAL RESULT: 10
```

```
Aggregate("seed" = 0):

  0 ──(+1)──► 1 ──(+2)──► 3 ──(+3)──► 6 ──(+4)──► 10
  seed        after 1st   after 2nd   after 3rd   FINAL RESULT
              element     element     element
```

> `Aggregate` is genuinely powerful but rarely needed in everyday code — `Sum`/`Count`/`Average`/`Max`/`Min` already cover the vast majority of real use cases. Reach for `Aggregate` only when you need truly custom accumulation logic those built-ins can't express.

## 📊 Set Operators — Full Reference

| Operator             | Behavior                                                     | SQL-ish equivalent      |
| -------------------- | ------------------------------------------------------------ | ----------------------- |
| `Union(other)`     | All UNIQUE elements from BOTH sequences (duplicates removed) | `UNION`               |
| `Intersect(other)` | Only elements present in BOTH sequences                      | `INTERSECT`           |
| `Except(other)`    | Elements in the first sequence NOT present in the second     | `EXCEPT` / `NOT IN` |
| `Concat(other)`    | ALL elements from both sequences, KEEPING duplicates         | `UNION ALL`           |

```
listA = [1, 2, 3, 4]        listB = [3, 4, 5, 6]

Union:      [1, 2, 3, 4, 5, 6]   ← all UNIQUE values from both
Intersect:  [3, 4]                ← only values in BOTH
Except:     [1, 2]                ← values in A but NOT in B
Concat:     [1, 2, 3, 4, 3, 4, 5, 6]  ← EVERYTHING, duplicates kept
```

> ⚠️ `Union`/`Intersect`/`Except` compare elements using **default equality** — for custom objects (like a `Product` class), this means REFERENCE equality unless you override `Equals`/`GetHashCode`, or pass an `IEqualityComparer<T>` explicitly.

## 💻 Code examples

### Basic — everyday aggregates on DAL results

```csharp
List<Product> products = _dal.GetAllProducts();

int totalCount = products.Count();
int inStockCount = products.Count(p => p.Stock > 0);
decimal totalValue = products.Sum(p => p.Price);
decimal? averagePrice = products.Any() ? products.Average(p => p.Price) : null; // guard against empty!
Product? mostExpensive = products.Any() ? products.OrderByDescending(p => p.Price).First() : null;
```

### Intermediate — building a dashboard summary object

```csharp
public DashboardSummaryDto GetDashboardSummary()
{
    List<Order> orders = _orderDal.GetTodaysOrders();

    return new DashboardSummaryDto
    {
        TotalOrders = orders.Count(),
        TotalRevenue = orders.Sum(o => o.Amount),         // safe even if orders is empty (returns 0)
        AverageOrderValue = orders.Any() ? orders.Average(o => o.Amount) : 0, // guarded!
        LargestOrder = orders.Any() ? orders.Max(o => o.Amount) : 0,
        HasAnyPendingOrders = orders.Any(o => o.Status == "Pending")
    };
}
```

### Practical — set operations comparing two DAL result sets

```csharp
List<int> lastMonthProductIds = _dal.GetLastMonthCatalogIds();
List<int> thisMonthProductIds = _dal.GetThisMonthCatalogIds();

// Products that were discontinued (in last month's catalog, but NOT this month's)
var discontinuedIds = lastMonthProductIds.Except(thisMonthProductIds).ToList();

// Products that are genuinely NEW this month
var newProductIds = thisMonthProductIds.Except(lastMonthProductIds).ToList();

// Products that carried over both months
var carriedOverIds = lastMonthProductIds.Intersect(thisMonthProductIds).ToList();
```

### Custom equality for set operations on objects

```csharp
public class ProductIdComparer : IEqualityComparer<Product>
{
    public bool Equals(Product? x, Product? y) => x?.Id == y?.Id;
    public int GetHashCode(Product obj) => obj.Id.GetHashCode();
}

// Without this comparer, Intersect would use REFERENCE equality and likely find NO matches,
// even if two Product objects represent the "same" product with identical Ids
var commonProducts = listA.Intersect(listB, new ProductIdComparer());
```

## ⚡ Performance considerations

- `Count()`, `Sum()`, `Average()` etc. each fully enumerate the sequence — calling several of them back-to-back on the same source means **multiple full passes**. If you need several aggregates from the same data, consider computing them together in one pass (e.g., a single loop, or `Aggregate` with a custom accumulator) for very large collections.
- `Union`/`Intersect`/`Except` build an internal hash set for comparison — efficient (`O(n + m)`), but still an allocation and a full pass over both sequences; for very large sets, consider doing this comparison in SQL instead.
- For dashboard-style aggregates over large tables, it's usually far more efficient to compute `COUNT`/`SUM`/`AVG` directly in a **stored procedure** (`SELECT COUNT(*), SUM(Amount) FROM Orders WHERE ...`) than to pull all rows into memory and aggregate with LINQ.

## 🚨 Common mistakes

- ❌ Calling `.Average()`, `.Max()`, `.Min()`, `.First()`, or `.Single()` on a sequence that MIGHT be empty without checking `.Any()` first — a very common source of `InvalidOperationException` in production.
- ❌ Using `Intersect`/`Except`/`Union` on custom objects without overriding equality or supplying an `IEqualityComparer<T>` — silently gets reference equality, producing wrong (usually empty) results.
- ❌ Confusing `Union` (removes duplicates) with `Concat` (keeps duplicates) — using the wrong one when merging two lists.
- ❌ Computing multiple aggregates (`Count`, `Sum`, `Average`) via separate LINQ calls on a large in-memory collection when a single stored-procedure query could return all three in one pass.

## 💡 Best practices

- ✅ Always guard `Average`/`Max`/`Min`/`First`/`Single` with an `.Any()` check (or use `FirstOrDefault`/`SingleOrDefault`) when the sequence could be empty.
- ✅ Use `Count`/`Sum` freely — they're safe on empty sequences by design.
- ✅ Supply an explicit `IEqualityComparer<T>` (or override `Equals`/`GetHashCode`) whenever using set operations on custom objects.
- ✅ For large-scale dashboard/reporting aggregates, compute them in the stored procedure (`SUM`, `COUNT`, `AVG` in SQL) rather than pulling all rows into memory for LINQ to aggregate.

## 🎤 Interview Quick-Fire Q&A

| Question                                                            | Answer                                                                                                                                                                      |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Which LINQ aggregate operators throw on an empty sequence?          | `Average`, `Max`, `Min`, `First`, and `Single` — always guard with `.Any()` or use an `OrDefault` variant                                                    |
| Which aggregate operators are SAFE on an empty sequence?            | `Count()` (returns 0) and `Sum()` (returns 0)                                                                                                                           |
| What's the difference between`Union` and `Concat`?              | `Union` removes duplicates across both sequences; `Concat` keeps all elements, including duplicates                                                                     |
| What does`Intersect` require to work correctly on custom objects? | A proper equality definition — either overridden`Equals`/`GetHashCode` or an explicit `IEqualityComparer<T>` — otherwise it defaults to reference equality          |
| What does`Aggregate` do, and when should you use it?              | A general-purpose accumulator that folds a sequence into a single value using custom logic — use it only when built-ins like Sum/Count/Average can't express what you need |

## 📝 30-second Revision Cheat Sheet

- Aggregates reduce a sequence to ONE value: `Count`, `Sum` (safe on empty) vs `Average`, `Max`, `Min`, `First` (THROW on empty — always guard with `.Any()`).
- `Aggregate` = general-purpose custom accumulator — rarely needed, but good to understand conceptually.
- Set operations compare TWO sequences: `Union` (unique, combined), `Intersect` (common only), `Except` (A minus B), `Concat` (everything, duplicates kept).
- Set operations need proper equality (`Equals`/`GetHashCode` or an `IEqualityComparer<T>`) for custom objects — default is reference equality.
- For large datasets, compute aggregates in SQL/stored procedures rather than pulling everything into memory for LINQ.

---

✅ **F.LINQ chapter complete** (01–06). Next up: **G.Modern_CSharp**.
