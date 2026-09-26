# 📜 List Methods

## 📌 What is it?

Beyond basic `Add`/`Remove`, `List<T>` provides a rich set of built-in methods for searching, sorting, transforming, and querying its contents — covering nearly everything you'd otherwise hand-write a loop for.

## 📊 Method Categories At a Glance

| Category                  | Methods                                                                     |
| ------------------------- | --------------------------------------------------------------------------- |
| **Search**          | `Contains`, `IndexOf`, `Find`, `FindAll`, `FindIndex`, `Exists` |
| **Sort/Order**      | `Sort`, `Reverse`                                                       |
| **Bulk operations** | `AddRange`, `InsertRange`, `RemoveRange`, `RemoveAll`               |
| **Transform**       | `ForEach`, `ConvertAll`                                                 |
| **Conversion**      | `ToArray`, `ToList` (from LINQ)                                         |

## 💻 Searching

```csharp
var names = new List<string> { "Alice", "Bob", "Charlie", "Dave" };

bool has = names.Contains("Bob");           // true
int idx = names.IndexOf("Charlie");          // 2

// Find — first match based on a condition (predicate)
string found = names.Find(n => n.StartsWith("D"));   // "Dave"

// FindAll — ALL matches, returns a new List<T>
List<string> allCs = names.FindAll(n => n.Contains("a"));   // {Alice, Charlie, Dave}

// FindIndex — index of first match
int index = names.FindIndex(n => n.Length > 5);   // 2 (Charlie)

// Exists — just a true/false check, no need to retrieve the match itself
bool anyLong = names.Exists(n => n.Length > 6);   // false
```

## 💻 Sorting & Reversing

```csharp
var numbers = new List<int> { 5, 2, 8, 1 };

numbers.Sort();                          // {1, 2, 5, 8} — ascending, in place
numbers.Sort((a, b) => b.CompareTo(a));   // {8, 5, 2, 1} — custom comparer, descending
numbers.Reverse();                       // reverses current order, in place
```

## 💻 Bulk Operations

```csharp
var list = new List<int> { 1, 2, 3 };

list.AddRange(new[] { 4, 5, 6 });         // {1,2,3,4,5,6} — append multiple at once
list.InsertRange(0, new[] { -1, 0 });     // {-1,0,1,2,3,4,5,6} — insert multiple at index
list.RemoveRange(0, 2);                    // removes 2 elements starting at index 0

list.RemoveAll(x => x % 2 == 0);           // removes ALL matches (unlike Remove(), which removes only the first)
```

⚠️ Prefer `AddRange()` over calling `Add()` in a loop — it's more efficient because `List<T>` can calculate the required capacity once, instead of potentially triggering multiple resize operations.

## 💻 Transform Operations

```csharp
var names = new List<string> { "alice", "bob" };

// ForEach — runs an action on every element (note: NOT the same as foreach loop — this is a method call)
names.ForEach(n => Console.WriteLine(n.ToUpper()));

// ConvertAll — transforms every element into a NEW list of a possibly different type
List<int> lengths = names.ConvertAll(n => n.Length);   // {5, 3}
```

## 💻 Conversion

```csharp
List<int> list = new List<int> { 1, 2, 3 };

int[] arr = list.ToArray();        // convert to a fixed-size array
```

## 📊 `Find`/`FindAll` vs LINQ `Where`/`FirstOrDefault` — Which to Use?

| Task        | `List<T>` method     | LINQ equivalent               |
| ----------- | ---------------------- | ----------------------------- |
| First match | `Find(predicate)`    | `FirstOrDefault(predicate)` |
| All matches | `FindAll(predicate)` | `Where(predicate)`          |

Both work fine — `Find`/`FindAll` are slightly older, `List<T>`-specific methods; LINQ equivalents work across **any** `IEnumerable<T>`, not just lists (see `F.LINQ/01_LINQ_Overview.md`). Modern code often prefers LINQ for consistency across collection types.

## 🚨 Common Mistakes

- ❌ Using `Remove()` expecting it to delete **all** matches — it only removes the **first** occurrence; use `RemoveAll(predicate)` for removing every match.
- ❌ Calling `Add()` in a loop to append many items instead of using `AddRange()` — misses the single-capacity-calculation optimization.
- ❌ Confusing `.ForEach()` (a `List<T>` method taking an `Action<T>`) with a `foreach` loop (a language construct) — they look similar but are different mechanisms; `.ForEach()` cannot use `break`/`continue`.
- ❌ Modifying the list inside `.ForEach()` or while iterating — same "collection modified" exception risk as a regular `foreach`.

## 💡 Best Practices

- Use `RemoveAll(predicate)` instead of manually looping to remove multiple matching items.
- Use `AddRange()`/`InsertRange()` for bulk additions rather than repeated single `Add()` calls.
- For anything beyond simple List-specific operations, consider **LINQ** — it's more expressive and consistent across all collection types.
- Avoid `.ForEach()` for anything beyond a trivial, simple action per element — a regular `foreach` loop is usually clearer and supports `break`/`continue`.

## 🎤 Interview Questions

1. What's the difference between `Remove()` and `RemoveAll()`?
2. Why is `AddRange()` more efficient than calling `Add()` repeatedly in a loop?
3. What's the practical difference between `List<T>.Find()` and LINQ's `FirstOrDefault()`?
4. Why can't you use `break` inside a `.ForEach()` call the way you can in a `foreach` loop?

## 📝 30-second Revision Cheat Sheet

- Search: `Contains`, `Find` (first match), `FindAll` (all matches), `Exists` (bool check).
- Bulk ops: `AddRange`/`InsertRange`/`RemoveRange` — more efficient than one-by-one loops.
- `Remove()` = first match only; `RemoveAll(predicate)` = every match.
- `.ForEach()` is a method call, not the `foreach` language construct — no `break`/`continue` support.
- LINQ (`Where`, `FirstOrDefault`) offers equivalent, more universal alternatives.
