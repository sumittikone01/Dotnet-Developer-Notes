# 🗃️ SortedDictionary<TKey, TValue>

## 📌 What is it?

`SortedDictionary<TKey, TValue>` is a key-value collection that **automatically keeps its entries sorted by key** at all times — unlike the regular `Dictionary<TKey,TValue>`, which makes no ordering guarantee at all.

```csharp
var sorted = new SortedDictionary<string, int>();
sorted["Charlie"] = 3;
sorted["Alice"] = 1;
sorted["Bob"] = 2;

foreach (var kvp in sorted)
    Console.WriteLine(kvp.Key);
// Output: Alice, Bob, Charlie  ← always alphabetically sorted, regardless of insertion order!
```

## 🤔 Why do we need it?

Sometimes you need key-value lookup **and** a guaranteed, always-sorted iteration order — e.g. displaying a leaderboard by name, or processing entries in a deterministic sequence. Regular `Dictionary<TKey,TValue>` explicitly does **not** guarantee any particular order.

## 🌍 Real-world analogy

A regular `Dictionary` is like a **pile of index cards thrown in a drawer** — fast to find one specific card, but no particular order when you pull them all out. `SortedDictionary` is like a **card catalog at a library** — always alphabetically filed, so pulling cards out in order gives you sorted results automatically, at the cost of extra effort every time a new card is filed.

## ⚙️ Internal Working — Binary Search Tree (Red-Black Tree)

Unlike `Dictionary<TKey,TValue>` (hash table), `SortedDictionary` is internally implemented as a **self-balancing binary search tree** (specifically a Red-Black Tree). This structure inherently keeps elements ordered as they're inserted.

```
          Bob
         /    \
     Alice    Charlie

In-order traversal → Alice, Bob, Charlie (always sorted)
```

## 📊 Time Complexity — The Key Tradeoff

| Operation       | `Dictionary<K,V>` (hash table) | `SortedDictionary<K,V>` (tree) |
| --------------- | -------------------------------- | -------------------------------- |
| Lookup          | O(1) average                     | **O(log n)**               |
| Insert          | O(1) average                     | **O(log n)**               |
| Remove          | O(1) average                     | **O(log n)**               |
| Iteration order | Not guaranteed                   | **Always sorted by key**   |

**The core tradeoff**: you give up `Dictionary`'s O(1) average speed for `SortedDictionary`'s guaranteed ordering, at O(log n) — still very fast, just structurally different.

## 💻 Basic Usage

```csharp
var sorted = new SortedDictionary<int, string>();
sorted.Add(3, "Three");
sorted.Add(1, "One");
sorted.Add(2, "Two");

foreach (var kvp in sorted)
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
// Output (always in key order):
// 1: One
// 2: Two
// 3: Three
```

## 📊 Custom Sort Order — Using an `IComparer<T>`

```csharp
var descending = new SortedDictionary<int, string>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
descending.Add(1, "One");
descending.Add(3, "Three");
descending.Add(2, "Two");

foreach (var kvp in descending)
    Console.WriteLine(kvp.Key);
// Output: 3, 2, 1 — descending order via custom comparer
```

## 📊 `SortedDictionary` vs `SortedList` (a close relative, not covered here in depth)

| Aspect             | `SortedDictionary<K,V>`                         | `SortedList<K,V>`                       |
| ------------------ | ------------------------------------------------- | ----------------------------------------- |
| Internal structure | Red-Black Tree                                    | Sorted array pair                         |
| Insert/Remove      | O(log n)                                          | O(n) — shifting required                 |
| Memory usage       | Higher (tree node overhead)                       | Lower (array-backed)                      |
| Best for           | Frequent inserts/removes, still need sorted order | Mostly static/rarely-changing sorted data |

## 🚨 Common Mistakes

- ❌ Defaulting to `SortedDictionary` when you don't actually need sorted iteration — pays an unnecessary O(log n) cost instead of `Dictionary`'s O(1) for no real benefit.
- ❌ Assuming `SortedDictionary` sorts by **value** — it always sorts by **key**; if you need value-based ordering, sort a projection separately (e.g. via LINQ `OrderBy`).
- ❌ Forgetting that a custom key type needs to implement `IComparable<T>` (or you must supply an `IComparer<T>`) — without one, the tree doesn't know how to order your keys.

## 💡 Best Practices

- Use `SortedDictionary` only when you **genuinely need** the collection to stay sorted by key at all times.
- If you don't need frequent inserts/removes and just need a one-time sorted view, it's often simpler/cheaper to use a regular `Dictionary` and call `.OrderBy(kvp => kvp.Key)` (LINQ) when you need sorted output — avoiding the ongoing O(log n) overhead entirely.
- Supply a custom `IComparer<TKey>` when you need non-default sort order (e.g. descending, or a custom priority scheme).

## 🎤 Interview Questions

1. What's the core structural difference between `Dictionary<K,V>` and `SortedDictionary<K,V>`?
2. Why is `SortedDictionary` lookup O(log n) instead of O(1)?
3. When would you choose `SortedDictionary` over simply sorting a `Dictionary`'s entries with LINQ when needed?
4. What's required of a key type to be usable in a `SortedDictionary`?

## 📝 30-second Revision Cheat Sheet

- `SortedDictionary<K,V>` = key-value store, **always sorted by key**, backed by a Red-Black Tree.
- O(log n) for lookup/insert/remove — trades speed for guaranteed ordering.
- Regular `Dictionary` gives no ordering guarantee at all, but is O(1) average.
- Use only when you truly need continuous sorted-by-key iteration; otherwise sort on-demand with LINQ instead.
