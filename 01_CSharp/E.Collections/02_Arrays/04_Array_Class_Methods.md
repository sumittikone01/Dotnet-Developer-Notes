# 🛠️ Array Class Methods

## 📌 What is it?

The **`System.Array`** class provides a set of built-in **static and instance methods** for common array operations — sorting, searching, resizing, copying, reversing — so you don't have to hand-write loops for these everyday tasks.

## 🤔 Why do we need it?

Arrays are a fixed-size, low-level structure, but you still frequently need to sort them, search them, or resize them. The `Array` class centralizes these common operations as **optimized, well-tested static methods** rather than everyone reinventing sort/search algorithms by hand.

## 📊 Most-Used Methods At a Glance

| Method                                 | Purpose                                       | Example                            |
| -------------------------------------- | --------------------------------------------- | ---------------------------------- |
| `Array.Sort()`                       | Sorts elements in place                       | `Array.Sort(arr);`               |
| `Array.Reverse()`                    | Reverses element order in place               | `Array.Reverse(arr);`            |
| `Array.IndexOf()`                    | Finds the index of a value                    | `Array.IndexOf(arr, 5);`         |
| `Array.Find()` / `Array.FindAll()` | Finds element(s) matching a condition         | `Array.Find(arr, x => x > 10);`  |
| `Array.Copy()`                       | Copies elements from one array to another     | `Array.Copy(src, dest, length);` |
| `Array.Clear()`                      | Resets a range of elements to default values  | `Array.Clear(arr, 0, 3);`        |
| `Array.Resize()`                     | Creates a new array with a different size     | `Array.Resize(ref arr, 10);`     |
| `Array.Exists()`                     | Checks if any element matches a condition     | `Array.Exists(arr, x => x < 0);` |
| `Array.BinarySearch()`               | Fast search —**requires sorted array** | `Array.BinarySearch(arr, 7);`    |

## 💻 Sorting

```csharp
int[] numbers = { 5, 2, 8, 1 };
Array.Sort(numbers);
// numbers is now: { 1, 2, 5, 8 }

Array.Sort(numbers, (a, b) => b.CompareTo(a));   // custom comparer — descending order
// numbers is now: { 8, 5, 2, 1 }
```

## 💻 Searching

```csharp
int[] numbers = { 10, 20, 30, 40 };

int index = Array.IndexOf(numbers, 30);        // 2
int found = Array.Find(numbers, x => x > 25);  // 30 (first match)

// BinarySearch — MUCH faster than IndexOf for large sorted arrays: O(log n) vs O(n)
Array.Sort(numbers);                            // must be sorted first!
int pos = Array.BinarySearch(numbers, 30);      // 2
```

## 💻 Copying

```csharp
int[] source = { 1, 2, 3, 4, 5 };
int[] destination = new int[5];

Array.Copy(source, destination, 3);   // copies first 3 elements: {1, 2, 3, 0, 0}

int[] clone = (int[])source.Clone();  // full shallow copy of the whole array
```

## 💻 Resizing — "Resize" Actually Creates a NEW Array

```csharp
int[] arr = { 1, 2, 3 };
Array.Resize(ref arr, 5);
// arr is now: { 1, 2, 3, 0, 0 } — a BRAND NEW array under the hood, old one is discarded
```

⚠️ Important nuance: since arrays are fixed-size, `Array.Resize()` doesn't actually grow the original array in memory — it allocates a **new array** of the requested size, copies the old elements over, and reassigns your variable to point to it. This is why it requires `ref` — your variable itself needs to be updated to point somewhere new.

## 🖼 Array.Sort() — Under the Hood

```
Array.Sort() uses an introspective sort (a hybrid of QuickSort, HeapSort, and 
Insertion Sort depending on partition size) — average time complexity O(n log n).
```

## 🚨 Common Mistakes

- ❌ Using `Array.BinarySearch()` on an **unsorted** array — gives unreliable/incorrect results, since binary search assumes sorted input.
- ❌ Forgetting `Array.Resize()` requires the `ref` keyword and creates a whole new array — assuming it mutates the original in place is a common misunderstanding.
- ❌ Using `.Clone()` expecting a **deep copy** — it's a **shallow copy**; for arrays of reference types, the cloned array holds references to the *same* underlying objects, not copies of them.
- ❌ Reaching for manual loops to sort/search/reverse when a well-tested, optimized `Array` method already exists for exactly that purpose.

## 💡 Best Practices

- Prefer built-in `Array` methods (`Sort`, `Find`, `BinarySearch`) over hand-rolled loops — they're optimized and less error-prone.
- Use `Array.BinarySearch()` only after confirming the array is sorted — otherwise use `Array.IndexOf()`/`Array.Find()` (linear search, works on unsorted data).
- Remember `.Clone()` is shallow — deep-clone manually if the array holds mutable reference types and you need true independence.

## 🎤 Interview Questions

1. Why does `Array.BinarySearch()` require the array to be sorted first, and what's the performance benefit over a linear search?
2. What actually happens internally when you call `Array.Resize()`?
3. What's the difference between a shallow copy (`.Clone()`) and a deep copy, in the context of an array of reference types?
4. What sorting algorithm does `Array.Sort()` use internally, and what's its typical time complexity?

## 📝 30-second Revision Cheat Sheet

- `Array.Sort()`, `.Reverse()`, `.IndexOf()`, `.Find()`, `.Copy()`, `.Resize()`, `.BinarySearch()` — built-in, optimized operations.
- `BinarySearch` = O(log n) but **requires sorted input**; `IndexOf`/`Find` = O(n), works unsorted.
- `Array.Resize()` creates a **new array** internally (needs `ref`) — doesn't truly resize in place.
- `.Clone()` = shallow copy only.
