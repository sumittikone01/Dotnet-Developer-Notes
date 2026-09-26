# 📦 Array Basics

## 📌 What is it?

An **Array** is a fixed-size, ordered collection of elements of the same type, stored in **contiguous memory**, accessed by a numeric index starting at `0`.

```csharp
int[] numbers = new int[3];   // fixed size: 3
numbers[0] = 10;
numbers[1] = 20;
numbers[2] = 30;

int[] direct = { 1, 2, 3 };   // shorthand initialization
```

## 🤔 Why do we need it?

Arrays are the **most basic, lowest-overhead** way to store a fixed sequence of items. They're the building block many other collections (`List<T>`, `Dictionary<K,V>` internals, etc.) are built on top of. When you know the exact size upfront and won't need to grow/shrink it, an array is the leanest option.

## 🌍 Real-world analogy

A **row of numbered lockers** 🔒 in a school hallway — a fixed number of lockers, each with a specific number (index), and you go directly to locker #5 without checking lockers #1-4 first. That direct jump is exactly how array indexing works in memory.

## ⚙️ Key Characteristics

| Property                   | Detail                                                                                            |
| -------------------------- | ------------------------------------------------------------------------------------------------- |
| **Size**             | Fixed at creation — cannot grow or shrink                                                        |
| **Memory layout**    | Contiguous block — enables very fast indexed access                                              |
| **Default indexing** | Zero-based (`array[0]` is the first element)                                                    |
| **Type**             | Reference type (the array object itself), but can hold value types or reference types as elements |
| **Default values**   | Numeric types →`0`; `bool` → `false`; reference types → `null`                         |

## 🖼 Memory Layout — Why Indexing is O(1)

```
int[] arr = {10, 20, 30, 40};

Memory:  [10][20][30][40]
Address: 1000 1004 1008 1012   (each int = 4 bytes)

arr[2] → address = base_address + (2 * element_size) = 1000 + 8 = 1008 → value 30
```

Because elements are stored **contiguously** and are all the same fixed size, the CPU can calculate any element's memory address directly with simple math — no searching needed. This is why array access is **O(1)**.

## 💻 Declaration & Initialization

```csharp
int[] a1 = new int[5];                  // 5 elements, all default to 0
int[] a2 = new int[] { 1, 2, 3 };       // explicit initializer
int[] a3 = { 1, 2, 3 };                 // shorthand — type inferred from context
string[] names = new string[3];          // all elements default to null
```

## 📊 Iterating an Array

```csharp
int[] numbers = { 10, 20, 30 };

for (int i = 0; i < numbers.Length; i++)
    Console.WriteLine(numbers[i]);

foreach (int n in numbers)
    Console.WriteLine(n);
```

`.Length` gives the array size — note: **not** `.Count` (that's for `List<T>` and other `ICollection<T>` types — see `02_List/01_List_T_Basics.md`).

## 📊 Array vs List — Quick Preview (full detail in `02_List/03_List_vs_Array.md`)

| Aspect      | Array                          | List\<T\>                          |
| ----------- | ------------------------------ | ---------------------------------- |
| Size        | Fixed                          | Dynamic (grows/shrinks)            |
| Performance | Slightly faster, less overhead | Small overhead from resizing logic |
| Flexibility | Low                            | High                               |

## 🚨 Common Mistakes

- ❌ **`IndexOutOfRangeException`** — accessing an index outside `0` to `Length - 1` (e.g. `arr[arr.Length]` is always out of bounds).
- ❌ Assuming arrays can be resized — they **can't**; `Array.Resize()` actually creates a **brand-new array** and copies elements over, it doesn't grow the original in place.
- ❌ Forgetting reference-type array elements default to `null`, not an empty instance — iterating without a null-check can throw `NullReferenceException`.
- ❌ Confusing `.Length` (arrays) with `.Count` (most other collections) — a very common beginner slip.

## 💡 Best Practices

- Use arrays when the **size is known and fixed** — e.g. days of the week, RGB values, a fixed-size buffer.
- Use `List<T>` instead when the collection needs to **grow or shrink** dynamically.
- Prefer `foreach` for simple iteration (cleaner); use indexed `for` loops when you need the index itself or need to modify elements in place.

## 🎤 Interview Questions

1. Why is array element access O(1)?
2. What happens internally when you call `Array.Resize()`?
3. What's the default value of elements in an array of reference types vs value types?
4. Why would you choose an array over a `List<T>`, and vice versa?

## 📝 30-second Revision Cheat Sheet

- Array = fixed-size, contiguous-memory, zero-indexed collection.
- Access is **O(1)** thanks to contiguous memory + fixed element size.
- Use `.Length` (not `.Count`) for size.
- Fixed size — can't grow/shrink; `Array.Resize()` creates a new array under the hood.
- Use when size is known upfront; use `List<T>` when it needs to change.
