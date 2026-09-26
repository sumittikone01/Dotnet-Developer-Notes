# 📋 List<T></t> Basics

## 📌 What is it?

`List<T>` is a **dynamically resizable** collection — the generic, general-purpose "go-to" collection in C#, similar to an array but able to **grow and shrink automatically** as items are added or removed.

```csharp
List<int> numbers = new List<int>();
numbers.Add(10);
numbers.Add(20);
numbers.Add(30);

Console.WriteLine(numbers.Count);   // 3 — note: .Count, NOT .Length (that's arrays)
```

## 🤔 Why do we need it?

Arrays (see `01_Arrays/01_Array_Basics.md`) have a **fixed size** decided at creation — adding a new element beyond that size requires manually creating a bigger array and copying everything over. `List<T>` handles all of that resizing logic **automatically** behind the scenes, making it the practical default choice whenever the collection's size isn't fixed/known upfront.

## 🌍 Real-world analogy

An **expandable file folder** 📁 vs a **rigid box**. A rigid box (array) holds a fixed number of items — once full, you need a whole new box. An expandable folder (`List<T>`) can grow to fit more documents as needed, without you manually managing that growth.

## ⚙️ Internal Working — How Growth Actually Happens

Internally, `List<T>` is backed by a regular **array**. When you add an item and the backing array is full, `List<T>` doesn't grow it by 1 — it:

1. Allocates a **new, larger array** (typically **double** the current capacity).
2. Copies all existing elements into the new array.
3. Discards the old array.

```
Capacity: 4, Count: 4 → [1][2][3][4]
Add(5) → triggers growth:
New array, Capacity: 8 → [1][2][3][4][5][ ][ ][ ]
```

This doubling strategy means resizing happens **less and less often** as the list grows, keeping the *average* cost of `Add()` very low (amortized O(1)) even though any single resizing operation is O(n).

## 📊 `Count` vs `Capacity`

| Property     | Meaning                                                                      |
| ------------ | ---------------------------------------------------------------------------- |
| `Count`    | Number of elements**actually stored**                                  |
| `Capacity` | Size of the**underlying backing array** (may be larger than `Count`) |

```csharp
var list = new List<int>();
list.Add(1);
Console.WriteLine(list.Count);      // 1
Console.WriteLine(list.Capacity);   // often 4 (implementation detail — starts with headroom)
```

## 💻 Basic Operations

```csharp
List<string> names = new List<string> { "Alice", "Bob" };

names.Add("Charlie");            // append: {Alice, Bob, Charlie}
names.Insert(1, "David");        // insert at index 1: {Alice, David, Bob, Charlie}
names.Remove("Bob");             // removes first match: {Alice, David, Charlie}
names.RemoveAt(0);                // removes by index: {David, Charlie}

string first = names[0];          // indexed access, like an array
bool has = names.Contains("David");  // true
```

## 📊 List<T></t> vs Array — Quick Preview (full comparison in `03_List_vs_Array.md`)

| Aspect        | `List<T>`                   | `Array`                          |
| ------------- | ----------------------------- | ---------------------------------- |
| Size          | Dynamic                       | Fixed                              |
| Size property | `.Count`                    | `.Length`                        |
| Performance   | Slight overhead from resizing | Marginally faster, no resize logic |

## ⚡ Performance Considerations

- If you know the **approximate final size** upfront, pass it to the constructor: `new List<int>(1000)` — this pre-allocates capacity and **avoids repeated resize-and-copy operations**, which is a meaningful performance win for large lists built incrementally.
- `Add()` is **amortized O(1)** — occasionally O(n) when a resize is triggered, but averaged over many calls it behaves like constant time.
- `Insert()`/`RemoveAt()` in the **middle** of a list are O(n) — every subsequent element must shift to fill/make the gap.

## 🚨 Common Mistakes

- ❌ Using `.Length` instead of `.Count` — a very common slip when switching between arrays and lists.
- ❌ Not pre-sizing a `List<T>` with a known/expected capacity when adding a large, known number of items in a loop — causes unnecessary repeated resizing.
- ❌ Modifying a list (`Add`/`Remove`) **while** iterating over it with `foreach` — throws `InvalidOperationException` ("Collection was modified"); use a regular `for` loop backward, or build a new list, instead.
- ❌ Assuming `Remove()` removes **all** matching elements — it only removes the **first** occurrence; use `RemoveAll()` for removing every match.

## 💡 Best Practices

- Use `List<T>` as your **default** collection choice whenever size isn't fixed/known — it's the most commonly used collection type in everyday C# code.
- Pre-size with an initial capacity (`new List<T>(capacity)`) when you know roughly how many items you'll add — avoids wasted resize operations.
- Use `RemoveAll(predicate)` instead of manually looping and removing to avoid the "modifying while iterating" pitfall.

## 🎤 Interview Questions

1. How does `List<T>` grow internally when it runs out of capacity?
2. What's the difference between `Count` and `Capacity`?
3. Why is `Add()` considered "amortized O(1)" rather than strictly O(1)?
4. What happens if you try to modify a `List<T>` while iterating over it with `foreach`?

## 📝 30-second Revision Cheat Sheet

- `List<T>` = dynamically resizable array — the everyday default collection.
- Backed internally by an array that **doubles in size** when it needs to grow.
- Use `.Count` (not `.Length`!) for size.
- `Add()` = amortized O(1); `Insert()`/`RemoveAt()` in the middle = O(n).
- Pre-size with `new List<T>(capacity)` when the approximate size is known upfront.
