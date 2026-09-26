# 04_Filtering_Projection_Ordering

> The three most-used LINQ operation families: **Filtering** (keep only matching elements), **Projection** (transform each element into something else), and **Ordering** (sort elements) — the bread-and-butter operators behind nearly every LINQ query you'll write.

> Builds directly on `03_Method_Syntax.md`'s pipeline model — this chapter is the detailed reference for exactly which operator to reach for within each of these three families.

## 📌 What is it?

These three families cover the vast majority of everyday LINQ usage — before you ever need grouping/joining (`05_Grouping_and_Joining.md`) or aggregates (`06_Aggregates_and_Set_Operations.md`).

```
FILTERING:   products.Where(p => p.Price > 100)         → fewer elements, SAME type
PROJECTION:  products.Select(p => p.Name)                → same count, DIFFERENT type/shape
ORDERING:    products.OrderBy(p => p.Name)               → same elements, DIFFERENT sequence
```

## 🤔 Why do we need to know each family in depth?

Each family has more than one operator, and picking the *wrong* one is a common source of subtle bugs or unnecessarily verbose code — e.g., using `.Select().Where()` when `.Where().Select()` is both more efficient and clearer.

## 📊 Filtering Operators

| Operator                 | Behavior                                                               |
| ------------------------ | ---------------------------------------------------------------------- |
| `Where(predicate)`     | Keeps elements where the predicate returns`true`                     |
| `OfType<T>()`          | Filters a non-generic/mixed sequence to only elements of type`T`     |
| `Distinct()`           | Removes duplicate elements (uses default equality)                     |
| `Skip(n)`              | Skips the first`n` elements                                          |
| `Take(n)`              | Takes only the first`n` elements                                     |
| `SkipWhile(predicate)` | Skips elements while the predicate is true, then takes the rest        |
| `TakeWhile(predicate)` | Takes elements while the predicate is true, stops at the first failure |

```csharp
var pageTwo = products.Skip(20).Take(20);   // classic pagination pattern (ties into
                                              // 12_System_Design/I.API_Design/04_Pagination_and_Filtering.md)
```

```
SkipWhile vs Where — the key difference:

numbers = [1, 2, 3, 10, 1, 2]

.Where(n => n < 5)      → [1, 2, 3, 1, 2]     (checks EVERY element independently)
.TakeWhile(n => n < 5)  → [1, 2, 3]            (STOPS at the first element that fails — "10" — never looks further)
```

## 📊 Projection Operators

| Operator                 | Behavior                                                       |
| ------------------------ | -------------------------------------------------------------- |
| `Select(selector)`     | Transforms each element 1-to-1 into a new shape                |
| `SelectMany(selector)` | Flattens a sequence-of-sequences into one single flat sequence |

```csharp
// Select — ONE result per input element (1-to-1)
var names = products.Select(p => p.Name);  // IEnumerable<string>, same count as products

// SelectMany — flattens nested collections (1-to-MANY, then FLATTENED)
List<Order> orders = ...; // each Order has a List<OrderItem> Items
var allItems = orders.SelectMany(o => o.Items);  // ONE flat IEnumerable<OrderItem> across ALL orders
```

```
Select (1-to-1):                         SelectMany (1-to-many, FLATTENED):

Order A → [Item1, Item2]                 Order A → [Item1, Item2]  ┐
Order B → [Item3]                        Order B → [Item3]         ├─► [Item1, Item2, Item3, Item4]
Order C → [Item4]                        Order C → [Item4]         ┘   (ONE flat sequence)

Select(o => o.Items) gives:              SelectMany(o => o.Items) gives:
IEnumerable<List<OrderItem>>             IEnumerable<OrderItem>
  [ [Item1,Item2], [Item3], [Item4] ]      [ Item1, Item2, Item3, Item4 ]
  ← a sequence OF LISTS (nested!)          ← ONE flat sequence (flattened!)
```

> This nested-vs-flat distinction is the single most common `Select` vs `SelectMany` confusion — reach for `SelectMany` any time each element maps to a **collection** that you want merged into one flat result.

## 📊 Ordering Operators

| Operator                           | Behavior                                                          |
| ---------------------------------- | ----------------------------------------------------------------- |
| `OrderBy(keySelector)`           | Sorts ascending by the given key                                  |
| `OrderByDescending(keySelector)` | Sorts descending by the given key                                 |
| `ThenBy(keySelector)`            | Secondary ascending sort (after`OrderBy`/`OrderByDescending`) |
| `ThenByDescending(keySelector)`  | Secondary descending sort                                         |
| `Reverse()`                      | Reverses the current order of elements                            |

```csharp
// Multi-level sort — sort by Category first, then by Price WITHIN each category
var sorted = products
    .OrderBy(p => p.Category)
    .ThenByDescending(p => p.Price);
```

```
🚨 COMMON MISTAKE: chaining multiple .OrderBy() calls instead of using .ThenBy()

products.OrderBy(p => p.Category).OrderBy(p => p.Price)
   → The SECOND .OrderBy() OVERRIDES the first sort entirely!
   → Result is sorted ONLY by Price — the Category sort is LOST.

products.OrderBy(p => p.Category).ThenBy(p => p.Price)
   → CORRECT: sorted by Category, and WITHIN each category, by Price.
```

## 💻 Code examples

### Basic — filter then project (correct order for efficiency)

```csharp
List<Product> products = _dal.GetAllProducts();

// GOOD: filter FIRST (fewer elements), then project
var names = products
    .Where(p => p.IsActive)
    .Select(p => p.Name)
    .ToList();

// LESS EFFICIENT (though same final result): project first, filter after
// var names = products.Select(p => p.Name).Where(n => ...); // can't even filter by IsActive anymore — lost that data!
```

### Intermediate — SelectMany flattening nested DAL results

```csharp
List<Order> orders = _orderDal.GetOrdersWithItems(); // each Order has List<OrderItem> Items

// Get a single flat list of ALL order items across ALL orders
var allItems = orders.SelectMany(o => o.Items).ToList();

// SelectMany with an index-mapping overload — keep a reference back to the parent order
var itemsWithOrderId = orders.SelectMany(
    o => o.Items,
    (order, item) => new { order.OrderId, item.ProductName, item.Quantity });
```

### Practical — multi-level sorting for a Kendo Grid

```csharp
public List<ProductGridRowDto> GetSortedGridData()
{
    List<Product> products = _dal.GetAllProducts();

    return products
        .Where(p => p.IsActive)
        .OrderBy(p => p.Category)           // primary sort
        .ThenByDescending(p => p.Price)      // secondary sort WITHIN each category
        .Select(p => new ProductGridRowDto { Id = p.Id, Name = p.Name, Category = p.Category, Price = p.Price })
        .ToList();
}
```

### Practical — pagination using Skip/Take (recap from Filtering)

```csharp
public List<Product> GetPage(int pageNumber, int pageSize)
{
    return products
        .OrderBy(p => p.Id)               // ALWAYS order before Skip/Take — otherwise page order isn't guaranteed
        .Skip((pageNumber - 1) * pageSize)
        .Take(pageSize)
        .ToList();
}
```

## ⚡ Performance considerations

- **Filter before projecting** whenever possible (`Where` then `Select`) — reduces the number of elements the projection has to process, and often preserves fields you might still need for the filter condition.
- `Skip`/`Take` over LINQ-to-Objects still has to enumerate (and discard) the skipped elements — for large in-memory collections this isn't free; for large DATABASE tables, push pagination into the SQL query (stored procedure's `OFFSET`/`FETCH`, per `12_System_Design/E.Database_Scaling`) rather than pulling everything into memory first.
- Always call `.OrderBy()` (or an equivalent stable ordering) before `Skip`/`Take` — without an explicit order, the sequence order is not guaranteed to be stable across calls, and pagination can behave unpredictably.

## 🚨 Common mistakes

- ❌ Chaining `.OrderBy().OrderBy()` instead of `.OrderBy().ThenBy()` — the second `OrderBy` silently discards the first sort.
- ❌ Using `.Select()` when the mapping actually produces a collection per element (should be `.SelectMany()`) — ending up with an awkward nested `IEnumerable<IEnumerable<T>>` instead of a flat sequence.
- ❌ Calling `.Skip()/.Take()` without an `.OrderBy()` first — pagination results can be inconsistent between calls since the underlying enumeration order isn't guaranteed.
- ❌ Projecting (`Select`) before filtering, accidentally discarding fields the filter condition still needed.

## 💡 Best practices

- ✅ Filter first, then project — keeps the pipeline efficient and preserves fields needed for filtering.
- ✅ Use `SelectMany` whenever each element maps to a collection you want flattened into one sequence.
- ✅ Always chain `ThenBy`/`ThenByDescending` for secondary sort keys — never stack multiple `OrderBy` calls.
- ✅ Always sort before paginating (`Skip`/`Take`) to guarantee consistent, predictable page contents.

## 🎤 Interview Quick-Fire Q&A

| Question                                                         | Answer                                                                                                                                                                                    |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What's the difference between`Select` and `SelectMany`?      | `Select` maps each element 1-to-1 (possibly to a nested collection); `SelectMany` flattens a sequence-of-collections into one single flat sequence                                    |
| What happens if you chain two`OrderBy()` calls?                | The second`OrderBy()` overrides the first entirely — use `ThenBy()` for a secondary sort key instead                                                                                 |
| Why should you always`OrderBy()` before `Skip()`/`Take()`? | Without an explicit order, enumeration order isn't guaranteed to be stable, so pagination results could be inconsistent                                                                   |
| What's the difference between`Where` and `TakeWhile`?        | `Where` checks every element independently; `TakeWhile` stops entirely at the first element that fails the condition, even if later elements would pass                               |
| When would`Distinct()` not behave as expected?                 | When comparing custom objects without overriding`Equals`/`GetHashCode` — reference types default to reference equality, so two "equal-looking" objects are still treated as distinct |

## 📝 30-second Revision Cheat Sheet

- Filtering: `Where`, `Distinct`, `Skip`/`Take`, `TakeWhile`/`SkipWhile` — fewer elements, same type.
- Projection: `Select` (1-to-1) vs `SelectMany` (1-to-many, FLATTENED) — same count vs flattened count.
- Ordering: `OrderBy`/`OrderByDescending` for primary sort, `ThenBy`/`ThenByDescending` for secondary — NEVER stack two `OrderBy`s.
- Always sort BEFORE `Skip`/`Take` for reliable pagination.
- Filter before you project — keeps the pipeline efficient and preserves needed fields.
