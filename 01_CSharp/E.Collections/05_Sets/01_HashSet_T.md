# 🎯 HashSet<T></t>

## 📌 What is it?

`HashSet<T>` is a collection that stores **only unique elements** — duplicates are automatically rejected — and gives **O(1) average-case** membership checks, just like the keys of a `Dictionary<TKey,TValue>` (which is exactly how it's implemented internally).

```csharp
var set = new HashSet<int> { 1, 2, 3 };
bool added = set.Add(2);   // false — 2 already exists, not added again
bool added2 = set.Add(4);  // true — new element, added

Console.WriteLine(set.Contains(3));   // true — O(1) average lookup
```

## 🤔 Why do we need it?

Two extremely common needs:

1. **Deduplication** — ensure a collection never contains repeated values.
2. **Fast membership testing** — "does this collection contain X?" in O(1) average time, instead of O(n) with a `List<T>`.

```csharp
// List<T> — O(n) membership check
bool exists = myList.Contains(value);

// HashSet<T> — O(1) average membership check
bool exists = mySet.Contains(value);
```

## 🌍 Real-world analogy

A **guest list at an exclusive event** — each name can appear **only once**, and the bouncer can instantly tell you "yes, they're on the list" or "no, they're not" without reading through the entire list from the top. That instant yes/no check, with guaranteed no duplicates, is exactly what `HashSet<T>` gives you.

## ⚙️ Internal Working

Just like `Dictionary<TKey,TValue>`, `HashSet<T>` is backed by a **hash table** — each element is hashed to determine its bucket, giving average O(1) add/remove/contains operations. Conceptually, you can think of `HashSet<T>` as a `Dictionary<T, (nothing)>` — it only stores the "keys," with no associated value.

## 💻 Basic Operations

```csharp
var fruits = new HashSet<string> { "apple", "banana" };

fruits.Add("cherry");         // true — added
fruits.Add("apple");          // false — already exists, ignored
fruits.Remove("banana");       // true — removed
bool has = fruits.Contains("apple");   // true
```

## 📊 Set Operations — The Real Power of `HashSet<T>`

`HashSet<T>` implements classic **mathematical set operations** directly as built-in methods:

| Method                    | Operation                                                    | Example Result                             |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| `UnionWith()`           | Combine all unique elements from both sets                   | `{1,2,3} ∪ {3,4,5}` = `{1,2,3,4,5}`   |
| `IntersectWith()`       | Keep only elements present in**both** sets             | `{1,2,3} ∩ {2,3,4}` = `{2,3}`         |
| `ExceptWith()`          | Remove elements found in the other set                       | `{1,2,3} − {2,3}` = `{1}`             |
| `SymmetricExceptWith()` | Keep elements in**either** set, but **not both** | `{1,2,3} ⊕ {2,3,4}` = `{1,4}`         |
| `IsSubsetOf()`          | Check if all elements exist in another set                   | `{1,2}.IsSubsetOf({1,2,3})` = `true`   |
| `IsSupersetOf()`        | Check if this set contains all of another's elements         | `{1,2,3}.IsSupersetOf({1,2})` = `true` |

```csharp
var setA = new HashSet<int> { 1, 2, 3 };
var setB = new HashSet<int> { 2, 3, 4 };

var union = new HashSet<int>(setA);
union.UnionWith(setB);          // {1, 2, 3, 4}

var intersection = new HashSet<int>(setA);
intersection.IntersectWith(setB); // {2, 3}

var difference = new HashSet<int>(setA);
difference.ExceptWith(setB);     // {1}
```

## 🖼 Visualizing Set Operations (Venn Diagram Style)

```
Set A: {1, 2, 3}       Set B: {2, 3, 4}

Union (A ∪ B):        [1] [2  3] [4]     → {1,2,3,4}
Intersection (A ∩ B):     [2  3]         → {2,3}
Except (A − B):        [1]               → {1}
SymmetricExcept:        [1]         [4]  → {1,4}
```

## 📊 `HashSet<T>` vs `List<T>` — When to Use Which

| Need                                             | Use                                         |
| ------------------------------------------------ | ------------------------------------------- |
| Preserve insertion order, allow duplicates       | `List<T>`                                 |
| Fast duplicate-free membership checks            | `HashSet<T>`                              |
| Frequent "does this exist?" checks on large data | `HashSet<T>` (O(1) vs `List<T>`'s O(n)) |
| Need indexed access (`list[i]`)                | `List<T>` (HashSet has no indexer)        |

## 🚨 Common Mistakes

- ❌ Using a `List<T>` + manual duplicate-checking (`if (!list.Contains(x)) list.Add(x)`) instead of just using a `HashSet<T>` — reinvents what the collection already does natively, and far less efficiently (O(n) check vs O(1)).
- ❌ Assuming `HashSet<T>` preserves insertion order — it does **not** guarantee any particular iteration order (in practice it often appears roughly insertion-ordered for small sets, but this is not a documented guarantee).
- ❌ Trying to access elements by index (`set[0]`) — `HashSet<T>` has **no indexer**; it's not meant for positional access.
- ❌ Using a mutable object as an element and modifying it after adding — like dictionary keys, this can break the set's ability to find/remove it later (hash code changes).

## 💡 Best Practices

- Reach for `HashSet<T>` whenever you need **fast duplicate elimination** or **frequent membership checks** — it's purpose-built for exactly this.
- Use the built-in set operations (`UnionWith`, `IntersectWith`, `ExceptWith`) instead of manually writing loops to compute these — cleaner and well-tested.
- Don't use `HashSet<T>` when you need ordering or indexed access — use `List<T>` or `SortedSet<T>` (see `02_SortedSet_T.md`) instead.

## 🎤 Interview Questions

1. How does `HashSet<T>` achieve O(1) average-case `Contains()`, compared to `List<T>`'s O(n)?
2. What's the difference between `UnionWith()` and `IntersectWith()`?
3. Why doesn't `HashSet<T>` support indexed access like `list[i]`?
4. When would you choose `HashSet<T>` over `List<T>` for storing a collection of items?

## 📝 30-second Revision Cheat Sheet

- `HashSet<T>` = unique elements, O(1) average add/remove/contains, hash-table backed.
- Built-in set operations: `UnionWith`, `IntersectWith`, `ExceptWith`, `SymmetricExceptWith`.
- No indexer — can't do `set[0]`.
- No guaranteed iteration order.
- Use whenever you need deduplication or fast membership checks.
