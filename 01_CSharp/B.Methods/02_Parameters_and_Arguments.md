# 📥 Parameters & Arguments

## 📌 What is it?

> **Parameter** — the variable declared in a method's signature (placeholder).
> **Argument** — the actual value passed in when calling the method.

```csharp
void Greet(string name)   // 'name' is a PARAMETER
{
    Console.WriteLine($"Hello {name}");
}

Greet("Sumit");            // "Sumit" is the ARGUMENT
```

---

## 📊 Parameter Passing Mechanisms

| Mechanism                  | Keyword    | Behavior                                                                       |
| -------------------------- | ---------- | ------------------------------------------------------------------------------ |
| By value (default)         | *(none)* | Copy passed — changes inside method don't affect caller                       |
| By reference               | `ref`    | Caller's variable is directly modified                                         |
| Output only                | `out`    | Method must assign a value; used to return extra values                        |
| Input reference (readonly) | `in`     | Passed by reference but method can't modify it (performance for large structs) |
| Variable count             | `params` | Accepts a variable number of arguments as an array                             |

---

## 💻 Code Examples

**Basic — pass by value (default):**

```csharp
void Increment(int x)
{
    x++;
}

int num = 5;
Increment(num);
Console.WriteLine(num);  // 5 — unaffected, x was a copy
```

**Intermediate — params for flexible argument count:**

```csharp
int Sum(params int[] numbers)
{
    int total = 0;
    foreach (var n in numbers) total += n;
    return total;
}

Sum(1, 2, 3);        // 6
Sum(1, 2, 3, 4, 5);  // 15
Sum();               // 0
```

**Practical — real-world logging helper using params:**

```csharp
void LogError(string context, params string[] details)
{
    Console.WriteLine($"[ERROR] {context}: {string.Join(", ", details)}");
}

LogError("OrderService.GetOrderById", "OrderId=105", "User=Sumit");
```

---

## 🚨 Common Mistakes

- ❌ Confusing `ref` and `out` — `ref` requires the variable to be initialized before passing; `out` doesn't (but must be assigned inside the method)
- ❌ Overusing `params` for performance-critical code — it allocates an array each call
- ❌ Forgetting `params` must be the **last** parameter in the method signature

---

## 🎤 Interview Questions

| Question                                             | Key Point                                                                                                     |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Difference between parameter and argument?           | Parameter = declared placeholder; Argument = actual value passed                                              |
| What does`params` allow?                           | Variable number of arguments passed as an array                                                               |
| Is C# pass-by-value or pass-by-reference by default? | Pass-by-value (even reference types pass the reference*by value* — see `05_Value_vs_Reference_Types.md`) |

---

## 📝 30-Second Revision Cheat Sheet

- Parameter = placeholder in signature | Argument = actual value passed at call site
- Default = pass by value (copy) | `ref`/`out`/`in` = explicit reference-based passing
- `params` = variable-length argument list, must be last paramet

# Parameters and Arguments

---

## 📌 Overview

> Write your notes here.

---

## 🔑 Key Concepts

---

## 💻 Code Example

```csharp
```

---

## ❓ Interview Questions

---

## 🔗 Related Topics

---

*Last updated: 2026-03-15*
