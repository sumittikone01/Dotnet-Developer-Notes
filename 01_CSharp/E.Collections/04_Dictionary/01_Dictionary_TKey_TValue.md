# 🔑 Dictionary<TKey, TValue>

## 📌 What is it?

`Dictionary<TKey, TValue>` stores data as **key-value pairs**, letting you look up a value **directly by its unique key** — instead of searching through a list item-by-item.

```csharp
Dictionary<string, int> ages = new Dictionary<string, int>();
ages["Rohit"] = 25;
ages["Priya"] = 28;

Console.WriteLine(ages["Rohit"]);   // 25 — direct lookup, no searching needed
```

## 🤔 Why do we need it?

Searching a `List<T>` for a match is **O(n)** — you may have to check every element. A `Dictionary` gives you **O(1) average-case lookup** by key, because it uses a **hash table** internally instead of scanning sequentially.

```csharp
// List — O(n) search
var employee = employeeList.FirstOrDefault(e => e.Id == 42);

// Dictionary — O(1) average lookup, if keyed by Id
var employee = employeeDict[42];
```

## 🌍 Real-world analogy

A **phone book** 📖 organized alphabetically by name — you don't read every entry from the start; you jump almost directly to "Rohit" because you know exactly where to look. A `Dictionary` gives you that same direct-jump lookup, but even faster (hash-based, not just alphabetical).

## ⚙️ Internal Working — Hash Table

```
Key "Rohit" → hashed → bucket #3
Key "Priya" → hashed → bucket #7

Buckets: [ ][ ][ ][Rohit:25][ ][ ][ ][Priya:28][ ]
              bucket 3                  bucket 7
```

1. The key is passed through a **hash function**, producing a numeric hash code.
2. That hash code determines which **bucket** the key-value pair is stored in.
3. Looking up a key re-hashes it and jumps **directly** to the right bucket — no scanning required.

### 🖼 Collision Handling

Two different keys can occasionally hash to the **same bucket** (a "collision"). `Dictionary<TKey,TValue>` handles this internally (via chaining) — you don't need to manage it yourself, but it's why worst-case lookup can degrade to O(n) if hashing is poor or the dictionary is misused.

## 💻 Basic Operations

```csharp
var dict = new Dictionary<string, int>();

dict.Add("Rohit", 25);          // add — throws if key already exists
dict["Priya"] = 28;              // add or update — safe either way

bool exists = dict.ContainsKey("Rohit");   // true
bool removed = dict.Remove("Priya");        // true

// SAFE lookup — avoids exception if key is missing
if (dict.TryGetValue("Rohit", out int age))
{
    Console.WriteLine(age);   // 25
}
```

## 🚨 `dict["missing_key"]` Throws!

```csharp
int age = dict["Someone Not There"];  
// 💥 KeyNotFoundException — direct indexer access assumes the key exists
```

Always use `TryGetValue()` or `ContainsKey()` first when you're not certain a key exists.

## 💻 Iterating a Dictionary

```csharp
foreach (KeyValuePair<string, int> kvp in dict)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}

// Or, using deconstruction (C# 7+)
foreach (var (name, age) in dict)
{
    Console.WriteLine($"{name}: {age}");
}

// Iterate keys or values only
foreach (string key in dict.Keys) { ... }
foreach (int value in dict.Values) { ... }
```

## 📊 Time Complexity

| Operation                                | Average Case | Worst Case                        |
| ---------------------------------------- | ------------ | --------------------------------- |
| Lookup (`dict[key]` / `TryGetValue`) | O(1)         | O(n) — with many hash collisions |
| Insert (`Add`)                         | O(1)         | O(n)                              |
| Remove                                   | O(1)         | O(n)                              |

⚠️ Worst-case O(n) only happens with a poorly-distributed hash function causing excessive collisions — in practice, with well-designed key types (like `string` or `int`, which have good built-in hash implementations), average-case O(1) is what you should expect.

## 🚨 Common Mistakes

- ❌ Using `dict[key]` without first checking existence — throws `KeyNotFoundException` if the key isn't there. Use `TryGetValue()` instead.
- ❌ Using a **mutable object** as a dictionary key, then changing it after insertion — its hash code changes, and the dictionary can no longer find it, effectively "losing" the entry.
- ❌ Assuming dictionary iteration order is guaranteed — it's **not** guaranteed by the API contract, even though it often appears insertion-ordered in practice (use `SortedDictionary` if you need guaranteed ordering — see `02_SortedDictionary.md`).
- ❌ Forgetting to override `GetHashCode()` and `Equals()` consistently on a **custom class** used as a dictionary key — mismatched implementations cause lookups to silently fail.

## 💡 Best Practices

- Use `TryGetValue()` for safe lookups instead of wrapping indexer access in a try/catch.
- Use **immutable types** (strings, value types, or records) as dictionary keys — avoid mutable reference types as keys.
- When using a custom class as a key, always override both `GetHashCode()` and `Equals()` together, consistently.
- Reach for `Dictionary<TKey,TValue>` whenever you find yourself repeatedly searching a `List<T>` by some identifying field — it's almost always the better structural fit.

## 🎤 Interview Questions

1. Why is dictionary lookup average-case O(1) while list search is O(n)?
2. What happens internally when two different keys produce the same hash (a collision)?
3. Why does using a mutable object as a dictionary key cause problems?
4. What's the difference between `dict[key]` and `TryGetValue()`, and when should you use each?

## 📝 30-second Revision Cheat Sheet

- `Dictionary<TKey, TValue>` = hash-table-backed key-value store — average O(1) lookup/insert/remove.
- Use `TryGetValue()` for safe access — `dict[key]` throws if missing.
- Iteration order is **not guaranteed**.
- Keys should be immutable; custom key classes need consistent `GetHashCode()` + `Equals()`.
- Use whenever you'd otherwise repeatedly search a list by a specific field.
