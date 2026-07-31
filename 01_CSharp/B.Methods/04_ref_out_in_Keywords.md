# 🔗 ref, out, in Keywords

## 📌 What is it?

> Keywords that change how a parameter is passed to a method — allowing the method to read and/or modify the **caller's actual variable** instead of a copy.

---

## 📊 Comparison Table

| Keyword | Must be initialized before call? | Method must assign it? | Can method modify it? | Typical use case                                                   |
| ------- | -------------------------------- | ---------------------- | --------------------- | ------------------------------------------------------------------ |
| `ref` | ✅ Yes                           | ❌ No (optional)       | ✅ Yes                | Modify an existing value in place                                  |
| `out` | ❌ No                            | ✅ Yes (mandatory)     | ✅ Yes                | Return multiple values from a method                               |
| `in`  | ✅ Yes                           | ❌ No — cannot modify | ❌ No (read-only)     | Pass large structs by reference without copy, but prevent mutation |

---

## 💻 Code Examples

**`ref` — modify caller's variable:**

```csharp
void DoubleValue(ref int x)
{
    x *= 2;
}

int num = 5;
DoubleValue(ref num);
Console.WriteLine(num);  // 10 — modified directly
```

**`out` — return multiple values (very common with TryParse pattern):**

```csharp
bool TryDivide(int a, int b, out int result)
{
    if (b == 0)
    {
        result = 0;
        return false;
    }
    result = a / b;
    return true;
}

if (TryDivide(10, 2, out int answer))
    Console.WriteLine(answer);  // 5
```

**`in` — pass large struct efficiently, prevent mutation:**

```csharp
struct BigStruct { public int A, B, C, D, E; }

void Process(in BigStruct data)
{
    // data.A = 100;  ❌ Compile error — 'in' parameters are read-only
    Console.WriteLine(data.A);
}
```

**Practical — real ADO.NET pattern using `out` (TryParse family):**

```csharp
string input = "42";
if (int.TryParse(input, out int quantity))
{
    Console.WriteLine($"Valid quantity: {quantity}");
}
else
{
    Console.WriteLine("Invalid input");
}
```

---

## 🚨 Common Mistakes

- ❌ Using `ref` when `out` was intended (or vice versa) — `ref` requires pre-initialization, `out` doesn't
- ❌ Forgetting to assign the `out` parameter inside the method — compile error, "must be assigned before control leaves method"
- ❌ Using `in` and then trying to mutate the parameter — compile error, defeats the purpose of `in`

---

## 🎤 Interview Questions

| Question                               | Key Point                                                                                               |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Difference between`ref` and `out`? | `ref` requires the variable to already be initialized; `out` doesn't, but the method MUST assign it |
| Why use`out`?                        | To return multiple values from a single method call (e.g.,`TryParse` pattern)                         |
| What does`in` optimize for?          | Avoids copying large structs while still preventing the method from mutating them                       |

---

## 📝 30-Second Revision Cheat Sheet

- `ref` → must be initialized before call, method CAN modify
- `out` → doesn't need initialization, method MUST assign before returning
- `in` → read-only reference, avoids struct copy overhead
- `TryParse`-style methods are the classic real-world `out` use ca

# ref out in Keywords

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
