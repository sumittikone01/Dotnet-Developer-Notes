# Boxing and Unboxing in C#

## 📌 What is it?

**Boxing** is converting a value type (e.g., `int`, `struct`) into a reference type (`object`) — the value gets wrapped in a heap-allocated box. **Unboxing** is the reverse — extracting the value type back out of that boxed `object`.

```csharp
int i = 42;
object boxed = i;        // boxing — heap allocation happens here
int unboxed = (int)boxed; // unboxing — extracts value back
```

## 🤔 Why do we need it?

- C#'s type system unifies value types and reference types under a common root (`object`), so you *can* treat an `int` as an `object` when needed (e.g., old-style non-generic collections, reflection, `ToString()` calls).
- But this unification has a real runtime cost — boxing allocates on the heap and copies the value, which matters a lot in performance-sensitive code.

## 🧠 Intuition

Think of a value type as a **loose coin** — cheap, lightweight, lives directly wherever it's used (stack, or inline in an object). Boxing is like **sealing that coin inside a labeled box** so it can be stored in a system designed only for boxes (`object` references) — like old non-generic collections (`ArrayList`) that only know how to hold references, not raw values.

## 🌍 Real-world analogy

Mailing a coin through a postal system that only accepts **packages** (references), not loose items. Boxing = wrapping the coin in a box so the postal system (a reference-based API) can handle it. Unboxing = opening the box back up to get the coin out. Every wrap/unwrap costs time and packaging material (heap allocation) — annoying if you're mailing thousands of coins one at a time instead of just sending them in bulk (generics avoid this entirely).

## ⚙️ Internal working

1. **Boxing**: CLR allocates a new object on the heap, copies the value type's data into it, and returns a reference to that box. This box contains type information (needed for the eventual unboxing type check) plus the value.
2. **Unboxing**: CLR checks that the boxed object's runtime type **exactly matches** the target value type, then copies the value back out. A mismatched cast throws `InvalidCastException`.
3. Each boxing operation is a **heap allocation** — subject to GC, adding pressure especially in loops.
4. Generics (`List<int>` vs `ArrayList`) were introduced specifically to **avoid** this — a `List<int>` stores actual `int` values directly, no boxing needed, unlike `ArrayList` which stores `object` references.

## 🖼 Diagram

```
STACK                          HEAP
┌───────────┐                 ┌──────────────────┐
│ int i = 42│                 │                    │
└───────────┘                 │                    │
      │ box                    │                    │
      ▼                        │                    │
┌───────────┐   reference      │  ┌──────────────┐  │
│object boxed│─────────────────┼─▶│ Type: Int32  │  │
└───────────┘                 │  │ Value: 42    │  │
                               │  └──────────────┘  │
                               └──────────────────┘
      │ unbox (with type check)
      ▼
┌───────────┐
│int unboxed│  ← value copied back out
└───────────┘
```

## 💻 Code Examples

### Basic — implicit boxing, explicit unboxing

```csharp
int number = 10;
object boxed = number;       // implicit boxing — heap allocation

int unboxed = (int)boxed;    // explicit unboxing — must cast to the EXACT original type
```

### ❌ Common mistake — mismatched unbox type throws

```csharp
object boxed = 42;          // boxed as int
long value = (long)boxed;   // ❌ throws InvalidCastException — must unbox to the EXACT original type (int)

long correct = (long)(int)boxed; // ✅ unbox to int first, THEN convert to long
```

### Boxing in non-generic collections (legacy pattern — avoid)

```csharp
ArrayList list = new ArrayList();
list.Add(1);      // boxing — int → object
list.Add(2);      // boxing again
int first = (int)list[0]; // unboxing

// ✅ Modern equivalent — no boxing at all
List<int> genericList = new List<int> { 1, 2 };
int firstGeneric = genericList[0]; // direct value access, no boxing
```

### Hidden boxing — a subtle gotcha

```csharp
int x = 5;
Console.WriteLine("Value: " + x); // x is boxed internally to build the string via object.ToString()

// Avoid via interpolation/formatting that doesn't box, or explicit ToString()
Console.WriteLine($"Value: {x}"); // string interpolation is generally efficient, minimal boxing
```

### Boxing cost demonstration

```csharp
// Heavy boxing — allocates on every iteration
object sum = 0;
for (int i = 0; i < 1_000_000; i++)
{
    sum = (int)sum + i; // box, unbox, box again — repeated per iteration!
}

// ✅ No boxing — pure value type arithmetic
int sum2 = 0;
for (int i = 0; i < 1_000_000; i++)
{
    sum2 += i;
}
```

## 📊 Comparison Table — Boxed vs Unboxed

| Aspect                | Value type directly                       | Boxed value type (`object`)                                                        |
| --------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ |
| Storage               | Stack (if local) or inline in heap object | Always a separate heap allocation                                                    |
| Allocation cost       | Cheap/free                                | Real heap allocation + GC pressure                                                   |
| Type safety on access | Compile-time checked                      | Runtime cast check (can throw`InvalidCastException`)                               |
| Used by               | Modern generic collections (`List<T>`)  | Legacy non-generic collections (`ArrayList`, `Hashtable`), `object`-typed APIs |

## ⚡ Performance considerations

- Every boxing operation is a heap allocation — in tight loops or high-frequency code, this adds significant, avoidable GC pressure.
- Generics (`List<T>`, `Dictionary<TKey,TValue>`) eliminate boxing for value types entirely compared to their non-generic predecessors (`ArrayList`, `Hashtable`) — always prefer generic collections.
- String interpolation/formatting of value types can trigger boxing internally in older patterns — modern .NET has optimized many common cases, but it's worth being aware of in hot paths.
- Boxing is a common hidden cost in code using `object`-typed parameters, non-generic interfaces, or reflection-heavy code.

## 🚨 Common mistakes

- ❌ Using legacy non-generic collections (`ArrayList`, `Hashtable`) with value types — causes boxing on every insert/read.
- ❌ Unboxing to the wrong type (e.g., boxed as `int`, unboxing as `long`) — throws `InvalidCastException` instead of silently converting.
- ❌ Not realizing boxing happens implicitly and repeatedly in loops doing `object`-based accumulation.
- ❌ Passing value types to methods with `object` parameters unnecessarily (e.g., some old logging/formatting APIs) without considering the boxing cost.
- ❌ Assuming boxing/unboxing is "free" like a simple cast — it's a real heap allocation plus a runtime type check.

## 💡 Best practices

- Always prefer generic collections (`List<T>`, `Dictionary<TKey,TValue>`) over their non-generic legacy counterparts to avoid boxing entirely.
- Be mindful of APIs that accept `object` parameters for value types — check if a generic overload exists.
- When unboxing, always cast to the **exact original type** the value was boxed as (a common two-step: unbox to original type, then convert).
- In performance-critical, high-frequency code paths, actively look for and eliminate unnecessary boxing (profilers can flag this).
- Use `Span<T>`/generic APIs and structs judiciously to keep value-type data boxing-free in hot paths.

## 🎤 Interview Questions

1. **What is boxing, and why does it have a performance cost?**
   → Boxing wraps a value type in a heap-allocated object so it can be treated as a reference type — the cost comes from the heap allocation and the eventual GC pressure, unlike a plain value type which can live cheaply on the stack.
2. **What's the exact rule for unboxing — can you unbox to any compatible numeric type?**
   → No — you must unbox to the **exact same type** it was boxed as; unboxing to a different type (even a wider numeric type like `long` from a boxed `int`) throws `InvalidCastException`.
3. **Why do generic collections avoid boxing while non-generic ones don't?**
   → Generic collections (`List<int>`) are compiled to store the actual value type directly; non-generic collections (`ArrayList`) only know how to store `object` references, forcing every value type insertion to be boxed.
4. **Give an example of hidden/implicit boxing in everyday code.**
   → Passing an `int` to a method expecting `object` (e.g., some legacy logging APIs, `string.Format` in older patterns), or storing value types in an `ArrayList`.
5. **Does boxing happen for reference types?**
   → No — boxing only applies to value types (`struct`, primitives, enums); reference types are already reference types, so assigning them to `object` doesn't box anything.

## 📝 30-second Revision Cheat Sheet

| Concept    | Key Point                                                                |
| ---------- | ------------------------------------------------------------------------ |
| Boxing     | Value type → heap-allocated`object` wrapper                           |
| Unboxing   | Extract value back — must match EXACT original type                     |
| Cost       | Real heap allocation + GC pressure per operation                         |
| Avoid via  | Generic collections (`List<T>`) instead of `ArrayList`/`Hashtable` |
| Gotcha     | Unboxing to wrong type throws`InvalidCastException`                    |
| Applies to | Value types only — reference types never get boxed                      |
