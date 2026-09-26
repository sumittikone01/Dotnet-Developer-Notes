# ⚖️ List<T></t> vs Array

## 📌 What is it?

Both `List<T>` and arrays (`T[]`) store ordered, indexable collections of the same type — but they differ in **size flexibility, performance, and API richness**. Choosing between them is one of the most common everyday decisions in C#.

## 📊 Head-to-Head Comparison

| Aspect                             | `Array`                                            | `List<T>`                                                            |
| ---------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------- |
| **Size**                     | Fixed at creation                                    | Dynamic — grows/shrinks automatically                                 |
| **Size property**            | `.Length`                                          | `.Count`                                                             |
| **Memory overhead**          | Minimal — just the raw data                         | Slightly more — internal capacity management                          |
| **Performance (raw access)** | Marginally faster (no resizing logic to account for) | Very close, negligible difference in practice                          |
| **Adding/removing elements** | Not supported — must create a new array             | `.Add()`, `.Remove()`, `.Insert()` built in                      |
| **Built-in methods**         | Fewer (via`System.Array` static methods)           | Rich instance methods (`Sort`, `Find`, `AddRange`, etc.)         |
| **Multidimensional support** | Yes (`int[,]`, jagged `int[][]`)                 | No native multidimensional equivalent (use`List<List<T>>` if needed) |
| **Best for**                 | Fixed-size, performance-critical, low-level data     | General-purpose, everyday collections where size varies                |

## 🌍 Real-world analogy

An **array** is a rigid **ice cube tray** — a fixed number of slots, and you can't add "one more slot" without buying a whole new tray. A **`List<T>`** is a **stretchy elastic bag** — it expands as you add more items, adjusting its own capacity behind the scenes.

## 🧠 The Core Tradeoff

```
Array:  Fixed size          →  Slightly leaner, faster, but rigid
List<T>: Dynamic size        →  Slightly more overhead, but flexible and far more convenient
```

In modern C#, `List<T>` is used **far more often** than raw arrays in everyday application code — the performance difference is usually negligible for typical business logic, and the flexibility/convenience wins out. Arrays remain common in **performance-critical**, **fixed-size**, or **low-level/interop** scenarios.

## 💻 Side-by-Side Example

```csharp
// Array — must know the size upfront, can't grow
int[] arr = new int[3];
arr[0] = 1; arr[1] = 2; arr[2] = 3;
// arr[3] = 4; ❌ IndexOutOfRangeException — no room to grow

// List<T> — grows as needed
List<int> list = new List<int> { 1, 2, 3 };
list.Add(4);   // ✅ just works — {1, 2, 3, 4}
```

## ⚡ Performance Considerations

- **Iteration speed** (`foreach`) is essentially identical between `Array` and `List<T>` for practical purposes — both are backed by contiguous memory internally.
- **`List<T>`'s overhead** comes almost entirely from **resizing operations** (doubling capacity + copying elements) — which, as covered in `01_List_T_Basics.md`, is *amortized* and rarely a real bottleneck.
- If you're in a genuinely performance-critical hot path (e.g. a tight game loop, high-frequency trading code) and the size is truly fixed, an array may squeeze out marginally better performance — but this is a micro-optimization that rarely matters in typical business applications.

## 🔄 Converting Between Them

```csharp
int[] array = { 1, 2, 3 };
List<int> list = array.ToList();      // array → list

List<int> list2 = new List<int> { 4, 5, 6 };
int[] array2 = list2.ToArray();       // list → array
```

Common pattern: accept data as a flexible `List<T>` internally, but expose a finalized, immutable-feeling result as an array (`.ToArray()`) from a public API — signals "this is fixed/done" to callers.

## 🚨 Common Mistakes

- ❌ Defaulting to arrays out of habit when the collection's size genuinely isn't known upfront — leads to awkward manual resizing code that `List<T>` already solves.
- ❌ Defaulting to `List<T>` even when a small, truly fixed-size, performance-sensitive array would be simpler and marginally faster (e.g. RGB values, days of week).
- ❌ Assuming `List<T>` is dramatically slower than arrays in typical code — the real-world difference is usually negligible and rarely the actual bottleneck in an application.

## 💡 Best Practices

- Default to `List<T>` for general application code — it's the pragmatic, flexible choice for most everyday scenarios.
- Use arrays when: size is truly fixed and known, you're in a performance-critical path, or you're interfacing with APIs/interop that specifically expect arrays.
- Use `.ToArray()` / `.ToList()` to convert between the two when crossing between "building/mutating" code (List) and "finalized/exposed" code (Array).

## 🎤 Interview Questions

1. What is the fundamental structural difference between an array and a `List<T>`?
2. Why is `List<T>`'s `Add()` considered close to O(1) in practice despite occasional resizing?
3. In what scenario would you deliberately choose an array over a `List<T>`?
4. How would you convert a `List<T>` to an array and vice versa?

## 📝 30-second Revision Cheat Sheet

- Array = fixed size, `.Length`, minimal overhead.
- `List<T>` = dynamic size, `.Count`, richer built-in methods, small resizing overhead.
- Default to `List<T>` for general use; reach for arrays when size is fixed/known or in performance-critical paths.
- Convert with `.ToArray()` / `.ToList()` as needed.
