
# 📥 Parameters & Arguments in C#

## 📌 What is it?

> **Parameter** — the variable declared in a method's signature (a placeholder waiting for a value).
> **Argument** — the actual value supplied when the method is called.

```csharp
void Greet(string name)   // 'name' is a PARAMETER
{
    Console.WriteLine($"Hello {name}");
}

Greet("Sumit");            // "Sumit" is the ARGUMENT
```

## 🤔 Why do we need different passing mechanisms?

By default, C# copies data into a method (pass-by-value) — safe and predictable, but sometimes you genuinely need the method to:

- Modify the caller's original variable (`ref`).
- Return more than one value without wrapping them in a class/tuple (`out`).
- Avoid copying a large struct for performance, while guaranteeing it won't be changed (`in`).
- Accept a flexible, unknown number of arguments (`params`).

Each mechanism exists to solve a specific, real problem — not just "extra syntax."

## 🧠 Intuition

Think of a parameter as a **labeled box** on the method's desk. By default, the caller hands over a **photocopy** of their data to put in that box (`by value`) — the method can scribble on the photocopy all it wants, the original is untouched. `ref`/`out`/`in` change this: instead of a photocopy, the method gets a **direct line back to the original document** itself.

## 🌍 Real-world analogy

- **By value** = handing someone a photocopy of your ID — they can write on it, fold it, whatever; your real ID is unaffected.
- **`ref`** = handing someone your actual ID card with instructions "you may update this" — any change they make is permanent, on the real thing.
- **`out`** = handing someone a blank form and saying "fill this in before you give it back" — they *must* write something before returning it to you.
- **`in`** = showing someone your ID through a locked glass case — they can read every detail but cannot physically alter it, and you didn't have to make them a copy.
- **`params`** = a drop box that accepts any number of letters at once, from zero to however many you want to post.

---

## 📊 Parameter Passing Mechanisms — Overview

| Mechanism                  | Keyword    | Behavior                                                                                          |
| -------------------------- | ---------- | ------------------------------------------------------------------------------------------------- |
| By value (default)         | *(none)* | Copy passed — changes inside method don't affect caller                                          |
| By reference               | `ref`    | Caller's variable is directly modified; must be initialized before the call                       |
| Output only                | `out`    | Method must assign a value; used to return extra values; caller doesn't need to initialize first  |
| Input reference (readonly) | `in`     | Passed by reference but method can't modify it — avoids copying for performance on large structs |
| Variable count             | `params` | Accepts a variable number of arguments as an array                                                |

---

## 💻 Code Examples — Each Mechanism in Depth

### 1️⃣ By Value (default) — the baseline

```csharp
void Increment(int x)
{
    x++; // modifies the LOCAL COPY only
}

int num = 5;
Increment(num);
Console.WriteLine(num);  // 5 — unaffected, x was a copy
```

**What's happening internally:** when `Increment(num)` is called, the value `5` is copied into the method's local parameter `x`. `x` and `num` are now two completely separate storage locations. Changing `x` has zero effect on `num`.

> ⚠️ Gotcha with reference types: passing a `class` instance "by value" still copies the *reference* (pointer), not the object. So mutating the object's fields IS visible to the caller — but reassigning the parameter to a brand-new object is NOT. See `05_Value_vs_Reference_Types.md` for the full breakdown.

```csharp
void Rename(Person p)
{
    p.Name = "Changed";      // ✅ visible to caller — same object on the heap
    p = new Person();         // ❌ NOT visible to caller — only the local copy of the reference changed
}
```

---

### 2️⃣ `ref` — pass by reference, caller must initialize first

```csharp
void Increment(ref int x)
{
    x++; // modifies the ORIGINAL variable directly
}

int num = 5;
Increment(ref num);   // 'ref' required at BOTH declaration and call site
Console.WriteLine(num);  // 6 — the original was changed!
```

**What's happening internally:** instead of copying the value, `ref` passes the **memory address** of `num` itself. Inside the method, `x` and `num` refer to the exact same storage location — there is no copy at all.

**Rule:** the variable passed with `ref` **must already be assigned a value** before the call — the compiler won't let you pass an uninitialized variable.

```csharp
int a;          // uninitialized
Increment(ref a); // ❌ Compile error: Use of unassigned local variable 'a'
```

**Practical use case — swapping two values:**

```csharp
void Swap(ref int a, ref int b)
{
    int temp = a;
    a = b;
    b = temp;
}

int x = 1, y = 2;
Swap(ref x, ref y);
Console.WriteLine($"{x}, {y}"); // 2, 1
```

---

### 3️⃣ `out` — output parameter, method MUST assign it

```csharp
bool TryDivide(int numerator, int denominator, out int result)
{
    if (denominator == 0)
    {
        result = 0;       // must assign SOMETHING on every code path, even the failure case
        return false;
    }
    result = numerator / denominator;
    return true;
}

if (TryDivide(10, 2, out int quotient))
{
    Console.WriteLine($"Result: {quotient}"); // Result: 5
}
```

**What's happening internally:** like `ref`, `out` passes a memory address — but the compiler enforces a different contract: the caller does **not** need to initialize the variable beforehand, but the **method must assign it on every possible exit path** before returning. This is exactly why the entire `TryParse`/`TryGetValue` family of .NET APIs uses `out` — it lets a method signal "success/failure" via its return `bool`, while still handing back the actual computed value through the `out` parameter.

**Practical — the classic `TryParse` pattern you already use constantly:**

```csharp
if (int.TryParse(userInput, out int parsedValue))
{
    Console.WriteLine($"Valid number: {parsedValue}");
}
else
{
    Console.WriteLine("Invalid input.");
}
```

**Multiple `out` parameters — returning several values without a wrapper class:**

```csharp
void GetMinMax(int[] numbers, out int min, out int max)
{
    min = numbers.Min();
    max = numbers.Max();
}

GetMinMax(new[] { 3, 7, 1, 9, 4 }, out int smallest, out int largest);
Console.WriteLine($"Min: {smallest}, Max: {largest}"); // Min: 1, Max: 9
```

**C# 7+ inline out-variable declaration (shown above) vs. older style:**

```csharp
// Old style (pre C# 7) — had to declare the variable separately first
int result;
bool success = TryDivide(10, 2, out result);

// Modern style — declare inline at the call site
bool success2 = TryDivide(10, 2, out int result2);
```

---

### 4️⃣ `in` — readonly reference, avoids copying large data

```csharp
struct LargeStruct // imagine this has many fields, making it expensive to copy
{
    public double X, Y, Z, W;
}

void PrintStruct(in LargeStruct s)
{
    Console.WriteLine($"{s.X}, {s.Y}, {s.Z}, {s.W}");
    // s.X = 10; // ❌ Compile error — cannot assign to a member of an 'in' parameter
}

var large = new LargeStruct { X = 1, Y = 2, Z = 3, W = 4 };
PrintStruct(in large); // 'in' at the call site is optional (compiler infers it), but improves clarity
```

**What's happening internally:** `in` passes by reference (no copy of the struct's data is made — important for large structs), but the compiler **enforces read-only access** inside the method, guaranteeing the caller's data can't be modified. It's essentially `ref`'s performance benefit with `const`-like safety.

**When it actually matters:** for small structs (like a single `int`), `in` offers no real benefit and can even be slightly slower due to indirection — reserve `in` for **large, read-heavy structs** in performance-critical code (e.g., math/physics/game-loop code using `Vector3`, `Matrix4x4`, or similar heavy value types).

---

### 5️⃣ `params` — variable number of arguments

```csharp
int Sum(params int[] numbers)
{
    int total = 0;
    foreach (var n in numbers) total += n;
    return total;
}

Sum(1, 2, 3);        // 6 — compiler wraps these into an array automatically
Sum(1, 2, 3, 4, 5);  // 15
Sum();               // 0 — empty array is valid, no error
Sum(new[] { 1, 2, 3 }); // also valid — pass an actual array directly
```

**What's happening internally:** the compiler automatically wraps however many arguments you pass into a single array before the method ever runs — `Sum(1, 2, 3)` is literally rewritten to `Sum(new int[] { 1, 2, 3 })` behind the scenes. This means **every call allocates a new array** — worth knowing in hot-path/performance-sensitive code (see Performance section below).

**Rule:** `params` must be the **last** parameter in the method signature, and there can only be one per method.

```csharp
// ❌ Invalid — params must be last
void Invalid(params int[] numbers, string label) { }

// ✅ Valid — other fixed parameters can come BEFORE params
void LogError(string context, params string[] details)
{
    Console.WriteLine($"[ERROR] {context}: {string.Join(", ", details)}");
}

LogError("OrderService.GetOrderById", "OrderId=105", "User=Sumit");
// Output: [ERROR] OrderService.GetOrderById: OrderId=105, User=Sumit
```

**Practical — real-world logging/formatting helper:**

```csharp
void LogInfo(string messageTemplate, params object[] args)
{
    Console.WriteLine(string.Format(messageTemplate, args));
}

LogInfo("User {0} placed order #{1} at {2}", "Sumit", 1042, DateTime.Now);
```

---

## 📊 Side-by-Side Comparison Table

| Feature                         | `ref`                 | `out`                         | `in`                             | `params`                   |
| ------------------------------- | ----------------------- | ------------------------------- | ---------------------------------- | ---------------------------- |
| Must initialize before call?    | ✅ Yes                  | ❌ No                           | ✅ Yes                             | N/A (not reference-based)    |
| Must be assigned inside method? | ❌ No (optional)        | ✅ Yes (on every path)          | ❌ Can't assign at all             | N/A                          |
| Can modify caller's value?      | ✅ Yes                  | ✅ Yes (it's the whole point)   | ❌ No (read-only)                  | N/A — passes a new array    |
| Typical use case                | Swap, in-place mutation | `TryParse`-style multi-return | Large struct, read-only perf       | Flexible argument count      |
| Keyword required at call site?  | ✅ Yes                  | ✅ Yes                          | Optional (recommended for clarity) | ❌ No — just list arguments |

---

## ⚡ Performance considerations

- `ref`/`in` avoid copying data — valuable for large `struct`s; meaningless for small ones (primitives, small structs) where copying is already cheap/free.
- `out` has the same reference-passing mechanics as `ref` — no extra cost beyond that.
- `params` allocates a **new array on every call** — avoid in extremely hot loops; prefer an explicit overload taking a fixed number of parameters, or pass a pre-built array/`Span<T>` directly when performance matters.
- Passing reference types (`class`) "by value" is already cheap — you're only copying a small pointer-sized reference, not the whole object — so `ref`/`in` rarely matter for classes, mainly for structs.

## 🚨 Common mistakes

- ❌ Confusing `ref` and `out` — `ref` requires the variable to be initialized **before** passing; `out` doesn't, but the method **must** assign it before returning.
- ❌ Forgetting the `ref`/`out`/`in` keyword at the call site — C# requires it explicitly (unlike some languages) to make the behavior visible at a glance.
- ❌ Overusing `params` in performance-critical code — it allocates a new array every single call.
- ❌ Forgetting `params` must be the **last** parameter in the method signature.
- ❌ Trying to assign to an `in` parameter's members — compile error; `in` is strictly read-only inside the method.
- ❌ Assuming `ref` on a reference type (`class`) lets you mutate its fields differently than normal — you can already mutate a class's fields without `ref`; `ref` on a class parameter is only needed if you want to **reassign** the parameter itself to a different object and have that visible to the caller.

## 💡 Best practices

- Default to pass-by-value; only reach for `ref`/`out`/`in` when you have a concrete reason.
- Use `out` for the well-established "try-pattern" (`TryParse`, `TryGetValue`-style methods) — it's idiomatic and expected by other C# developers.
- Use `ref` sparingly — mutating caller state through parameters can make code harder to follow; a return value is usually clearer when only one value needs to come back.
- Reserve `in` for large, performance-sensitive structs in hot paths — don't apply it reflexively to every parameter.
- Prefer `params` for genuinely variable-length, order-independent data (log details, string formatting args); avoid it for performance-critical, high-frequency calls.

## 🎤 Interview Questions

| Question                                                                     | Key Point                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Difference between parameter and argument?                                   | Parameter = declared placeholder in the method signature; Argument = actual value passed at the call site                                                                                                                    |
| Difference between`ref` and `out`?                                       | `ref` requires the variable to be initialized before the call and modification is optional inside the method; `out` doesn't require prior initialization but the method MUST assign it before returning                  |
| What does`in` guarantee?                                                   | The parameter is passed by reference (no copy) but cannot be modified inside the method — read-only access for performance                                                                                                  |
| What does`params` allow?                                                   | A variable number of arguments, automatically collected into an array; must be the last parameter                                                                                                                            |
| Is C# pass-by-value or pass-by-reference by default?                         | Pass-by-value — even reference types pass the*reference itself* by value (the object isn't copied, but the pointer variable is)                                                                                           |
| Why does`TryParse` use `out` instead of just returning a nullable value? | Historical/idiomatic design predating nullable value types' common use for this purpose — lets the method return a`bool` for success/failure while still delivering the parsed value, avoiding boxing/null-check ceremony |
| Can you overload a method differing only by`ref`/`out`?                  | You can't have two overloads differing only by`ref` vs `out` on the same parameter position (they're treated as a conflict at the call site), but `ref`/`out` vs. plain value IS a valid overload distinction        |

---

## 📝 30-Second Revision Cheat Sheet

- **Parameter** = placeholder in signature | **Argument** = actual value passed at call site
- **Default** = pass by value (copy) | `ref`/`out`/`in` = explicit reference-based passing
- `ref` → must be initialized before call; method MAY modify it
- `out` → no init needed before call; method MUST assign it before returning
- `in` → passed by reference, but READ-ONLY inside the method (perf for large structs)
- `params` → variable-length argument list, auto-wrapped into an array, MUST be last parameter
- Classes passed "by value" still share the same object — only *reassignment* of the parameter itself is invisible to the caller without `ref`
