# 03_Method_Syntax

> **Method Syntax** = writing LINQ as a chain of extension method calls — `.Where(...).Select(...).OrderBy(...)` — using lambda expressions. The dominant, most idiomatic style in real-world modern C#.

> Direct counterpart to `02_Query_Syntax.md`. Both compile to identical results; this chapter covers Method Syntax's structure, its use of lambdas, and exactly why it's the default choice for most LINQ code you'll write.

## 📌 What is it?

Method Syntax chains LINQ's extension methods (from `01_LINQ_Overview.md`'s `IEnumerable<T>` foundation) directly, each one taking a **lambda expression** describing what to do with each element.

```csharp
var result = products
    .Where(p => p.Price > 100)       // lambda: keep if price > 100
    .OrderBy(p => p.Name)             // lambda: sort key = Name
    .Select(p => p.Name);             // lambda: project to just the name
```

## 🤔 Why do we need it?

- It's the **default style** most C# developers reach for — more common in real-world codebases, tutorials, and library documentation than Query Syntax.
- Every LINQ operator has a method form, but **not every operator has a Query Syntax keyword** (`Count`, `Any`, `First`, `Sum`, etc. are method-only) — so you need Method Syntax fluency regardless of which style you prefer for the basics.
- Chaining reads naturally as a **pipeline**: each method transforms the output of the one before it, left to right.

## 🌍 Real-world analogy

An **assembly line**. Each station (method call) takes what came off the previous station, does one specific job (filter out defects, paint, label), and passes it to the next station. `products.Where(...).OrderBy(...).Select(...)` is exactly that: raw materials in, transformed product out, one station at a time.

## ⚙️ Internal working — lambdas as the "instructions" for each method

```csharp
products.Where(p => p.Price > 100)
```

```
   .Where(                    predicate: Func<Product, bool>
          p                    =  the CURRENT element being examined
          =>                   =  "goes to" / "maps to"
          p.Price > 100        =  the CONDITION — return true to KEEP this element
   )
```

Every LINQ method has a specific **delegate signature** it expects:

| Method            | Lambda signature     | Meaning                                                                     |
| ----------------- | -------------------- | --------------------------------------------------------------------------- |
| `Where`         | `Func<T, bool>`    | Given an element, return true/false — keep it or not                       |
| `Select`        | `Func<T, TResult>` | Given an element, return a (possibly different) result                      |
| `OrderBy`       | `Func<T, TKey>`    | Given an element, return the value to sort BY                               |
| `Any` / `All` | `Func<T, bool>`    | Given an element, return true/false — used to check existence/universality |

## 📊 Method Chaining — the pipeline mental model

```
products                                  (IEnumerable<Product>)
   │
   ▼ .Where(p => p.Price > 100)           (IEnumerable<Product> — still Products, filtered)
   │
   ▼ .OrderBy(p => p.Name)                (IEnumerable<Product> — still Products, now sorted)
   │
   ▼ .Select(p => p.Name)                 (IEnumerable<string> — TRANSFORMED into strings!)
   │
   ▼ .ToList()                            (List<string> — IMMEDIATE execution, concrete result)
```

> Notice the TYPE can change mid-chain (`Product` → `string` after `.Select()`) — this is completely normal and one of LINQ's most powerful features: each method just needs to accept whatever `IEnumerable<T>` the previous one produced.

## 📊 Lambda Expression Syntax Quick Reference

| Form                | Example                                             | When to use                                                                       |
| ------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------- |
| Expression lambda   | `p => p.Price > 100`                              | Single expression, most common                                                    |
| Block lambda        | `p => { var x = p.Price * 0.9m; return x > 50; }` | Needs multiple statements/local variables                                         |
| No parameters       | `() => DateTime.Now`                              | Rare in LINQ (most operators need the element)                                    |
| Multiple parameters | `(x, y) => x + y`                                 | Used in operators like`Aggregate` (see `06_Aggregates_and_Set_Operations.md`) |

## 💻 Code examples

### Basic — a typical Method Syntax chain

```csharp
List<Product> products = _dal.GetAllProducts();

var expensiveProductNames = products
    .Where(p => p.Price > 100)
    .OrderByDescending(p => p.Price)
    .Select(p => p.Name)
    .ToList();
```

### Intermediate — chaining multiple filters and using method-only operators

```csharp
// Query Syntax has NO direct keyword for these — Method Syntax is required (or a trailing call)
bool hasExpensiveProduct = products.Any(p => p.Price > 1000);
int countInStock = products.Count(p => p.Stock > 0);
Product? cheapest = products.OrderBy(p => p.Price).FirstOrDefault();

// Multiple .Where() calls chain as AND conditions
var results = products
    .Where(p => p.IsActive)
    .Where(p => p.Category == "Electronics")
    .Where(p => p.Price < 500)
    .ToList();
```

### Practical — projecting into a DTO shaped for the Kendo Grid

```csharp
public List<ProductGridRowDto> GetGridData()
{
    List<Product> products = _dal.GetAllProducts();

    return products
        .Where(p => p.IsActive)
        .OrderBy(p => p.Name)
        .Select(p => new ProductGridRowDto   // Select's lambda body can construct a whole new object
        {
            Id = p.Id,
            Name = p.Name,
            FormattedPrice = p.Price.ToString("C"),
            StockStatus = p.Stock > 0 ? "In Stock" : "Out of Stock"
        })
        .ToList();
}
```

### Using method references instead of inline lambdas (a clean-up technique)

```csharp
// Inline lambda:
var active = products.Where(p => IsActiveProduct(p));

// Method group — cleaner when the logic already exists as a named method:
var active = products.Where(IsActiveProduct);

private bool IsActiveProduct(Product p) => p.IsActive && p.Stock > 0;
```

## ⚡ Performance considerations

- Chaining many `.Where()` calls is functionally identical to one `.Where()` with combined `&&` conditions — but multiple small `.Where()`s can sometimes read more clearly without any real performance cost, since deferred execution still processes the whole chain in a single pass per element under the hood.
- Every lambda captures its enclosing scope (a **closure**) if it references outside variables — be aware this can keep those variables alive longer than expected, and can allocate a closure object per call in performance-critical loops.
- As with Query Syntax, deferred execution rules apply identically — avoid re-enumerating a chain multiple times without `.ToList()`.

## 🚨 Common mistakes

- ❌ Writing `.Where(p => p.Price > 100 && p.IsActive && p.Category == "X" && ...)` as one giant unreadable condition — consider breaking into named boolean helper methods or multiple `.Where()` calls for readability.
- ❌ Forgetting that `.Select()` can change the element type entirely — trying to chain a `Product`-specific method after a `.Select(p => p.Name)` that already turned it into a `string`.
- ❌ Confusing `FirstOrDefault()` (returns `null`/`default` if not found) with `First()` (throws an exception if not found) — using the wrong one for the situation.
- ❌ Overusing block-lambdas with multiple statements when a simple expression lambda would be clearer — hurts LINQ's normally very readable, declarative style.

## 💡 Best practices

- ✅ Default to Method Syntax for typical filtering/projection/sorting — it's the more idiomatic, widely-used style in modern C#.
- ✅ Keep lambda bodies short and focused; extract complex conditions into a named private method and pass it as a method group if it improves readability.
- ✅ Know your operator's exact behavior on "not found" cases: `First()`/`Single()` throw, `FirstOrDefault()`/`SingleOrDefault()` return default — pick deliberately.
- ✅ Reach for Query Syntax (`02_Query_Syntax.md`) specifically when a query involves heavy joins/grouping — otherwise Method Syntax stays the default.

## 🎤 Interview Quick-Fire Q&A

| Question                                                               | Answer                                                                                                                                        |
| ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| What is Method Syntax?                                                 | Writing LINQ as a chain of extension method calls (Where, Select, OrderBy, etc.) using lambda expressions                                     |
| Why does Method Syntax fluency matter even if you prefer Query Syntax? | Some operators (Count, Any, First, Sum) have no Query Syntax keyword — they must be called as methods                                        |
| What's the difference between`First()` and `FirstOrDefault()`?     | `First()` throws an exception if no match is found; `FirstOrDefault()` returns the type's default value (e.g., null) instead              |
| Can the element type change partway through a LINQ chain?              | Yes — e.g.,`.Select(p => p.Name)` turns an `IEnumerable<Product>` into an `IEnumerable<string>`; later methods operate on the NEW type |
| What's a "closure" in the context of a LINQ lambda?                    | When a lambda references a variable from its enclosing scope, that variable is "captured" and kept alive as part of the lambda's context      |

## 📝 30-second Revision Cheat Sheet

- Method Syntax = chained extension methods + lambdas (`.Where().Select().OrderBy()`) — the default, most common LINQ style.
- Every method expects a specific lambda signature (`Func<T,bool>` for Where/Any, `Func<T,TResult>` for Select, etc.).
- The element type can change mid-chain (e.g., after `.Select()`).
- Some operators (Count, Any, First, Sum) are method-only — no Query Syntax equivalent.
- Know `First()` vs `FirstOrDefault()` (throws vs returns default) — a very common real bug source.
