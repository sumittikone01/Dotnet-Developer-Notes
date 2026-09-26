# 01_Null_Conditional_and_Coalescing

> **Null-conditional operator (`?.`)** = safely access a member ONLY if the object isn't null, short-circuiting to `null` otherwise. **Null-coalescing operator (`??`)** = provide a fallback value when something IS null. Together, the two most-used tools for writing concise, safe null-handling code in modern C#.

> New chapter: **G.Modern_CSharp**. Where `06_Aggregates_and_Set_Operations.md` closed out LINQ, this chapter starts a run of everyday C# syntax sugar that makes code shorter and safer — starting with null-safety, the single most common source of runtime crashes (`NullReferenceException`) in C#.

## 📌 What is it?

```csharp
// WITHOUT null-conditional — verbose, easy to forget a check
string city = null;
if (customer != null)
{
    if (customer.Address != null)
    {
        city = customer.Address.City;
    }
}

// WITH null-conditional — one line, same safety
string city = customer?.Address?.City;
```

```csharp
// WITHOUT null-coalescing
string displayName;
if (customer.Name != null)
    displayName = customer.Name;
else
    displayName = "Guest";

// WITH null-coalescing
string displayName = customer.Name ?? "Guest";
```

## 🤔 Why do we need them?

| Problem                                                                                         | How these operators help                                      |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `NullReferenceException` is the #1 most common C# runtime crash                               | `?.` short-circuits safely instead of throwing              |
| Deeply nested null-checks (`if (a != null) if (a.B != null) ...`) are verbose and error-prone | `?.` chains as many levels deep as needed, in one line      |
| Providing a default/fallback value requires an`if/else`                                       | `??` expresses "use this, or fall back to that" in one line |
| Updating a value only if it's currently null needs an extra`if`                               | `??=` (null-coalescing assignment) does it in one line      |

## 🌍 Real-world analogy

`?.` is like a mail carrier who **checks if a house exists before trying to deliver a package** — if there's no house at that address, they simply skip it and don't crash trying to walk through a door that isn't there. `??` is like saying "give me my usual coffee order, or if they're out, just give me whatever's the closest match" — always end up with SOMETHING usable.

## 📊 Operator Reference

| Operator | Name                               | Behavior                                                                            |
| -------- | ---------------------------------- | ----------------------------------------------------------------------------------- |
| `?.`   | Null-conditional (safe navigation) | Access a member ONLY if the object isn't null; otherwise, short-circuits to`null` |
| `?[]`  | Null-conditional indexer           | Same idea, for indexers:`list?[0]`                                                |
| `??`   | Null-coalescing                    | Returns the left operand if not null; otherwise, returns the right operand          |
| `??=`  | Null-coalescing assignment         | Assigns the right-hand value ONLY IF the variable is currently null                 |

## ⚙️ Internal working — short-circuiting through a chain

```csharp
string city = customer?.Address?.City;
```

```
Step 1: Is 'customer' null?
   YES → STOP immediately. Entire expression evaluates to null. 'Address' and 'City' are NEVER accessed.
   NO  → continue to Step 2

Step 2: Is 'customer.Address' null?
   YES → STOP immediately. Entire expression evaluates to null. 'City' is NEVER accessed.
   NO  → continue to Step 3

Step 3: Return 'customer.Address.City'
```

```
┌─────────────────────────────────────────────────────────────┐
│  customer?.Address?.City                                       │
│           │         │                                          │
│           ▼         ▼                                          │
│   "if customer   "if Address                                   │
│    is null,       is null,                                     │
│    STOP HERE"     STOP HERE"                                   │
│                                                                  │
│  Any ?. in the chain short-circuits the ENTIRE remaining chain  │
│  the moment it hits a null — nothing after it gets evaluated.   │
└─────────────────────────────────────────────────────────────┘
```

## ⚙️ `??` vs `??=` — the subtle but important difference

```csharp
string name = customer.Name ?? "Guest";
// Reads customer.Name — if null, EVALUATES TO "Guest" (customer.Name itself is NOT changed)

customer.Name ??= "Guest";
// If customer.Name IS null, ASSIGNS "Guest" TO customer.Name (mutates it!)
// If customer.Name is NOT null, does NOTHING — no assignment happens at all
```

```
??   → "use THIS value, or fall back to THAT value" (read-only fallback, doesn't mutate anything)
??=  → "if this is currently null, SET it to that value" (an actual assignment/mutation)
```

## 💻 Code examples

### Basic — safe navigation through a nested object graph

```csharp
public class Address { public string City { get; set; } }
public class Customer { public Address Address { get; set; } public string Name { get; set; } }

Customer customer = _dal.GetCustomerById(5); // might have a null Address if incomplete data

string city = customer?.Address?.City ?? "Unknown"; // combines BOTH operators in one line
// If customer is null → "Unknown"
// If customer.Address is null → "Unknown"
// Otherwise → the actual city
```

### Intermediate — calling a method safely, and null-conditional with events

```csharp
// Safe method call — only invoked if 'logger' isn't null
logger?.LogInformation("Product saved successfully");

// A very common, IMPORTANT pattern: safely invoking an event
public event EventHandler? ProductUpdated;

public void UpdateProduct(Product product)
{
    _dal.UpdateProduct(product);
    ProductUpdated?.Invoke(this, EventArgs.Empty); // avoids a NullReferenceException if NO ONE subscribed
}
```

### Practical — using `??=` to lazily initialize a field

```csharp
public class ProductCache
{
    private List<Product>? _cachedProducts;

    public List<Product> GetProducts()
    {
        _cachedProducts ??= _dal.GetAllProducts(); // fetch ONLY the first time; reuse afterward
        return _cachedProducts;
    }
}
```

### Practical — combining with LINQ (from earlier `F.LINQ` chapters)

```csharp
// Safely get a count even if the list itself might be null
int productCount = products?.Count() ?? 0;

// Safely access the first item's name, with a fallback
string firstProductName = products?.FirstOrDefault()?.Name ?? "No products available";
```

## ⚡ Performance considerations

- `?.` and `??` compile down to simple null-checks (a couple of IL instructions) — there's no meaningful performance cost versus writing the equivalent `if` statements by hand. Use them freely for readability.
- `??=` avoids an unnecessary assignment when the value is already non-null — technically slightly more efficient than an unconditional `if (x == null) x = ...` pattern written out longhand, though the difference is negligible in practice.

## 🚨 Common mistakes

- ❌ Confusing `??` (read-only fallback expression) with `??=` (an actual assignment/mutation) — using `??` when you actually meant to update the variable.
- ❌ Overusing `?.` to silently swallow null situations that actually indicate a real bug — sometimes a null SHOULD throw/be investigated, not be quietly bypassed everywhere.
- ❌ Chaining `?.` on a value type member access that then gets used without a final `??` fallback — e.g., `int? length = customer?.Name?.Length;` — forgetting that the RESULT is now nullable (`int?`, not `int`) because ANY link in the chain could have short-circuited.
- ❌ Using `event?.Invoke(...)` incorrectly by forgetting the `?.` entirely — calling `ProductUpdated.Invoke(...)` directly throws a `NullReferenceException` if nobody has subscribed to the event.

## 💡 Best practices

- ✅ Use `?.` for safe navigation through object graphs where a null at any level is a legitimate, expected possibility (e.g., optional related data).
- ✅ Always safely invoke events with `?.Invoke(...)` — this is closer to a mandatory idiom than a style choice in C#.
- ✅ Use `??` to provide sensible defaults; use `??=` specifically for lazy initialization or "set once if not already set" patterns.
- ✅ Don't overuse `?.` to hide null situations that represent genuine bugs — sometimes an explicit null-check with a thrown exception or logged warning is the more honest choice.

## 🎤 Interview Quick-Fire Q&A

| Question                                                          | Answer                                                                                                                                                      |
| ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| What does the null-conditional operator (`?.`) do?              | Accesses a member only if the object isn't null; short-circuits the entire remaining chain to`null` if it is                                              |
| What's the difference between`??` and `??=`?                  | `??` returns a fallback value as part of an expression (no mutation); `??=` assigns the fallback value TO the variable, but only if it's currently null |
| Why is`event?.Invoke(...)` an important pattern?                | Calling an event directly when no subscribers exist throws a`NullReferenceException`; the `?.` safely no-ops instead                                    |
| If`customer?.Name?.Length` is used, what's the result type?     | `int?` (nullable int) — not `int` — because any link in the chain could short-circuit to null                                                         |
| Is there a meaningful performance cost to using`?.` and `??`? | No — they compile to simple, efficient null-checks; use them freely for readability                                                                        |

## 📝 30-second Revision Cheat Sheet

- `?.` = safe navigation — access a member only if not null, short-circuits the WHOLE remaining chain to null otherwise.
- `??` = fallback value in an expression (no mutation); `??=` = assign the fallback ONLY if currently null (mutation).
- Always use `?.Invoke(...)` for events — calling directly risks a `NullReferenceException`.
- A `?.` chain's result type becomes nullable — remember to handle that (often with a trailing `??`).
- Both operators are effectively free performance-wise — use them for clarity and safety without hesitation.
