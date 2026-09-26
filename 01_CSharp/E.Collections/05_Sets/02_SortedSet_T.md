# 🌳 SortedSet<T></t>

## 📌 What is it?

`SortedSet<T>` combines the two ideas from the previous two notes: it stores **unique elements** (like `HashSet<T>`) while **always keeping them sorted** (like `SortedDictionary<TKey,TValue>`) — implemented internally as a self-balancing binary search tree (Red-Black Tree).

```csharp
var sorted = new SortedSet<int> { 5, 1, 4, 2, 3 };

foreach (var num in sorted)
    Console.Write(num + " ");
// Output: 1 2 3 4 5  ← always sorted, regardless of insertion order
```

## 🤔 Why do we need it?

`HashSet<T>` gives you uniqueness and fast lookups, but **no ordering guarantee at all**. `SortedSet<T>` is for when you need **both**: no duplicates, AND a collection that's always traversable in sorted order — e.g. maintaining a running leaderboard, a sorted range of unique timestamps, or a priority-ordered unique task list.

## 🌍 Real-world analogy

`HashSet<T>` is like a **jar of unique marbles thrown in randomly** — no duplicates, but no particular order when you pour them out. `SortedSet<T>` is like **marbles arranged on a sorted display shelf** — still no duplicates, but always retrievable from smallest to largest.

## 📊 Time Complexity Comparison

| Operation         | `HashSet<T>` (hash table) | `SortedSet<T>` (tree)                                        |
| ----------------- | --------------------------- | -------------------------------------------------------------- |
| Add               | O(1) average                | **O(log n)**                                             |
| Remove            | O(1) average                | **O(log n)**                                             |
| Contains          | O(1) average                | **O(log n)**                                             |
| Iteration order   | Not guaranteed              | **Always sorted**                                        |
| Min/Max retrieval | O(n) — must scan all       | **O(log n)** (or O(1) with `.Min`/`.Max` properties) |

Same fundamental tradeoff as `Dictionary` vs `SortedDictionary`: give up O(1) average speed for guaranteed ordering.

## 💻 Basic Usage

```csharp
var scores = new SortedSet<int> { 85, 42, 99, 67 };

scores.Add(75);       // inserted in correct sorted position automatically
scores.Remove(42);

Console.WriteLine(scores.Min);   // 67 — fast, O(log n) (or better)
Console.WriteLine(scores.Max);   // 99 — fast, O(log n) (or better)
```

## 💻 Range Queries — A Standout Feature

`SortedSet<T>` supports efficiently retrieving a **subset within a range** — something `HashSet<T>` cannot do at all without scanning everything.

```csharp
var numbers = new SortedSet<int> { 1, 5, 10, 15, 20, 25, 30 };

// GetViewBetween — returns a "view" of elements within [lower, upper], inclusive
var range = numbers.GetViewBetween(10, 25);

foreach (var n in range)
    Console.Write(n + " ");
// Output: 10 15 20 25
```

This makes `SortedSet<T>` genuinely useful for range-based queries (e.g. "give me all scheduled event times between 9 AM and 5 PM") — something you'd otherwise need to manually filter+sort a `HashSet`/`List` for, at higher cost.

## 📊 Custom Sort Order

```csharp
var descending = new SortedSet<int>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
descending.Add(3);
descending.Add(1);
descending.Add(2);

foreach (var n in descending)
    Console.Write(n + " ");
// Output: 3 2 1
```

## 📊 `SortedSet<T>` vs `HashSet<T>` vs `List<T>` (sorted manually) — When to Use Which

| Need                                                          | Best Choice                                      |
| ------------------------------------------------------------- | ------------------------------------------------ |
| Fast uniqueness checks, order doesn't matter                  | `HashSet<T>`                                   |
| Uniqueness +**always sorted**, frequent inserts/removes | `SortedSet<T>`                                 |
| Need to sort**once** and don't expect frequent changes  | `List<T>` + `.Sort()` or LINQ `.OrderBy()` |
| Need efficient**range queries** on unique sorted data   | `SortedSet<T>` (`GetViewBetween`) ⭐         |

## 🚨 Common Mistakes

- ❌ Using `SortedSet<T>` by default without actually needing sorted iteration — pays unnecessary O(log n) cost vs `HashSet<T>`'s O(1) for no real benefit.
- ❌ Manually re-sorting a `List<T>` after every insertion to keep it ordered — `SortedSet<T>` (or `SortedList<T>`) already maintains order incrementally and far more efficiently.
- ❌ Forgetting custom element types need `IComparable<T>` implemented (or a custom `IComparer<T>` supplied) — otherwise `SortedSet<T>` doesn't know how to order them.
- ❌ Not knowing about `GetViewBetween()` and manually filtering + sorting instead — misses out on `SortedSet<T>`'s most distinctive, efficient feature.

## 💡 Best Practices

- Use `SortedSet<T>` when you need **both** uniqueness and continuous sorted order, especially with frequent inserts/removes.
- Use `.Min` / `.Max` properties instead of manually finding extremes — they're efficient on the tree structure.
- Use `GetViewBetween()` for range queries instead of manual filtering — it's the standout capability this collection offers over `HashSet<T>`.
- If you only need a one-time sort with no further mutation, a `List<T>` + LINQ `.OrderBy()` is usually simpler and sufficient.

## 🎤 Interview Questions

1. What's the core structural and performance difference between `HashSet<T>` and `SortedSet<T>`?
2. How does `GetViewBetween()` work, and why is it more efficient than manually filtering a sorted list?
3. Why would `.Min`/`.Max` be more efficient on a `SortedSet<T>` than on a `HashSet<T>`?
4. When would you choose `SortedSet<T>` over simply calling `.OrderBy()` on a `HashSet<T>` each time you need sorted output?

## 📝 30-second Revision Cheat Sheet

- `SortedSet<T>` = unique elements, **always sorted**, backed by a Red-Black Tree.
- O(log n) for add/remove/contains — trades `HashSet`'s O(1) speed for guaranteed order.
- `.Min` / `.Max` are efficient (not a full scan).
- `GetViewBetween(low, high)` — efficient range queries, a standout feature.
- Use only when you need both uniqueness AND continuous sorted order; otherwise `HashSet<T>` is faster.
