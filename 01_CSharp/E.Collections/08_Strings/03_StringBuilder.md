# 🧱 StringBuilder

## 📌 What is it?

`StringBuilder` is a **mutable** string-building class — unlike `string` (immutable, see `01_String_Basics.md`), a `StringBuilder` can be modified **in place** without creating a new object every time, making it far more efficient for building strings incrementally.

```csharp
StringBuilder sb = new StringBuilder();
sb.Append("Hello");
sb.Append(" ");
sb.Append("World");

string result = sb.ToString();   // "Hello World"
```

## 🤔 Why do we need it?

As covered in `01_String_Basics.md`, repeatedly concatenating strings with `+`/`+=` in a loop creates a **brand new string object every single time** — copying all previously accumulated characters again and again. For `n` concatenations, this becomes **O(n²)** overall.

```csharp
// ❌ Inefficient — string version, O(n²) overall
string result = "";
for (int i = 0; i < 10000; i++)
    result += i;   // new string object created EVERY iteration

// ✅ Efficient — StringBuilder version, O(n) overall
var sb = new StringBuilder();
for (int i = 0; i < 10000; i++)
    sb.Append(i);   // modifies the SAME internal buffer, no new object each time
string result = sb.ToString();
```

## 🌍 Real-world analogy

`string` is like **writing on individual printed pages** — every edit means printing a whole new page. `StringBuilder` is like **writing in pencil on a whiteboard** — you can erase, add, and extend directly on the same surface without starting over each time.

## ⚙️ Internal Working — A Resizable Character Buffer

`StringBuilder` maintains an internal, resizable **character buffer/array** (conceptually similar to how `List<T>` grows — see `E.Collections/02_List/01_List_T_Basics.md`). As you append content:

1. If there's enough remaining capacity, characters are added directly into the existing buffer — no new allocation.
2. If the buffer is full, a larger buffer is allocated (typically doubled), and existing content is copied over — but this happens far less often than with repeated string concatenation.

```
Capacity: 16
Append("Hello") → buffer: "Hello           " (5 used, 11 free) — no new allocation needed
Append(" World") → buffer: "Hello World     " (11 used, 5 free) — still fits!
```

## 💻 Core Methods

```csharp
var sb = new StringBuilder("Hello");

sb.Append(" World");              // "Hello World"
sb.AppendLine(" - done");         // adds text + newline
sb.Insert(5, ",");                 // "Hello, World - done" — insert at a specific index
sb.Replace("World", "C#");         // "Hello, C# - done"
sb.Remove(0, 6);                    // removes 6 characters starting at index 0

string result = sb.ToString();      // converts back to an immutable string when done
Console.WriteLine(sb.Length);        // current character count
```

## 📊 `StringBuilder` vs `string` Concatenation — When to Use Which

| Scenario                                                      | Best Choice                                                                            |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Building a string in a**loop** (many appends)           | `StringBuilder` ⭐                                                                   |
| One-off, simple concatenation of a few known values           | `string` interpolation (`$"..."`) — simpler, no meaningful performance difference |
| Building large text (e.g. generating a report, log file, CSV) | `StringBuilder` ⭐                                                                   |
| Final result needs to be passed around as`string`           | Call`.ToString()` once at the end                                                    |

## ⚡ Performance Considerations

- Pre-size the `StringBuilder` if you have a rough estimate of the final length: `new StringBuilder(capacity: 1000)` — avoids repeated internal buffer resizing.
- Don't overuse `StringBuilder` for **small, one-time** concatenations — plain string interpolation is simpler and the performance difference is negligible at that scale; `StringBuilder`'s benefit only really shows up with **many** sequential modifications (typically loops).
- Call `.ToString()` only **once**, at the very end — calling it repeatedly mid-process to "check progress" defeats some of the efficiency benefit if done excessively in a hot loop.

## 🚨 Common Mistakes

- ❌ Using `StringBuilder` for a single, simple concatenation of 2-3 known strings — unnecessary overhead; plain interpolation is simpler and just as fast for that case.
- ❌ Forgetting to call `.ToString()` at the end — `StringBuilder` itself is not a `string`; you must explicitly convert it when you need the final text.
- ❌ Not pre-sizing capacity when the approximate final length is known upfront in performance-critical code — misses an easy optimization.
- ❌ Passing a `StringBuilder` around across multiple methods/threads expecting `string`-like immutability guarantees — it's mutable, so shared references can be modified unexpectedly by other code, unlike a `string`.

## 💡 Best Practices

- Use `StringBuilder` specifically for **loops** or **many sequential modifications** — that's where its efficiency advantage actually matters.
- Pre-size the constructor capacity when you have a good estimate of the final string length.
- Use plain string interpolation for simple, one-off concatenation — don't reach for `StringBuilder` reflexively for every string operation.
- Always finish with `.ToString()` to get back an immutable `string` for use elsewhere.

## 🎤 Interview Questions

1. Why is `StringBuilder` more efficient than repeated string concatenation in a loop?
2. How does `StringBuilder`'s internal buffer grow, and how is that similar to `List<T>`'s resizing strategy?
3. When would using `StringBuilder` actually NOT be worth it compared to plain string concatenation?
4. Is `StringBuilder` thread-safe? Why does that matter if you share one across multiple threads?

## 📝 30-second Revision Cheat Sheet

- `StringBuilder` = mutable string buffer — modifies in place instead of creating new objects on every change.
- Use for **loops**/many sequential appends — turns O(n²) string concatenation into O(n).
- Not worth it for a single, simple concatenation — plain interpolation is simpler there.
- Pre-size capacity if the final length is roughly known.
- Call `.ToString()` once at the end to get the final immutable `string`.
