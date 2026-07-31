# ➕ Operators

## 📌 What is it?

> Symbols that perform operations on operands (variables/values) — arithmetic, comparison, logic, assignment, and more.

---

## 📊 Operator Categories

| Category                      | Operators                | Example                           |
| ----------------------------- | ------------------------ | --------------------------------- |
| **Arithmetic**          | `+ - * / %`            | `10 % 3` → `1`               |
| **Assignment**          | `= += -= *= /= %= ??=` | `x += 5;` → `x = x + 5`      |
| **Comparison**          | `== != > < >= <=`      | `5 == 5` → `true`            |
| **Logical**             | `&& \|\| !`              | `true && false` → `false`    |
| **Bitwise**             | `& \| ^ ~ << >>`        | `5 & 3` → `1`                |
| **Null-related**        | `?? ??= ?.`            | `x ?? 0`                        |
| **Ternary**             | `condition ? a : b`    | `age >= 18 ? "Adult" : "Minor"` |
| **Increment/Decrement** | `++ --`                | `x++`                           |

---

## 🧠 Intuition — Short-Circuit Evaluation

```csharp
bool result = IsValid() && IsAuthorized();
```

- If `IsValid()` returns `false`, **`IsAuthorized()` is never called** — `&&` short-circuits.
- Same for `||`: if the first operand is `true`, the second is skipped.

> 💡 This matters in real code: put cheaper/more-likely-to-fail checks first for performance and to avoid null-reference errors:

```csharp
if (user != null && user.IsActive)   // safe — short-circuits before accessing .IsActive if user is null
```

---

## 💻 Code Examples

**Basic — Arithmetic & Comparison:**

```csharp
int a = 10, b = 3;
Console.WriteLine(a / b);   // 3  (integer division — truncates!)
Console.WriteLine(a % b);   // 1  (remainder)
Console.WriteLine(a / (double)b);  // 3.333... (cast one operand to get decimal result)
```

**Intermediate — Null-coalescing operators:**

```csharp
string name = null;
string displayName = name ?? "Guest";        // "Guest"

name ??= "Default";     // assigns only if name is null
Console.WriteLine(name);   // "Default"

// Null-conditional — avoids NullReferenceException
int? length = name?.Length;   // safely returns null if name is null, instead of throwing
```

**Practical — Ternary in real-world MVC/business logic:**

```csharp
string status = order.IsDelivered ? "Delivered" : "Pending";

decimal discount = customer.IsPremium ? 0.20m : 0.05m;

// Common in Kendo Grid data shaping
string badgeClass = item.Quantity > 0 ? "badge-success" : "badge-danger";
```

---

## ⚡ Performance Considerations

- Short-circuit (`&&`, `||`) evaluation avoids unnecessary/expensive function calls — order conditions cheapest-first
- Integer division truncates — always cast to `double`/`decimal` if a fractional result is expected

---

## 🚨 Common Mistakes

- ❌ `a / b` with two ints expecting a decimal result (`10 / 3` = `3`, not `3.33`) — must cast
- ❌ Using `=` instead of `==` in a condition (`if (x = 5)` is a compile error in C#, unlike some languages — but still a common typo to watch for)
- ❌ Forgetting `??=` only assigns if the variable is currently `null` — not for other falsy values like `0` or `""`

---

## 🎤 Interview Questions

| Question                             | Key Point                                                                                                            |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| What is short-circuit evaluation?    | `&&`/`\|\|` skip evaluating the second operand if the result is already determined by the first                    |
| Difference between`??` and `?.`? | `??` provides a fallback value if left side is null; `?.` safely accesses a member only if the object isn't null |
| Why does`10 / 3` give `3` in C#? | Integer division truncates the decimal part; cast to`double`/`decimal` for precise division                      |

---

## 📝 30-Second Revision Cheat Sheet

- `&&` / `\|\|` → short-circuit (skip 2nd operand when result is already known)
- `??` → fallback value | `??=` → assign only if null | `?.` → safe navigation (no null exception)
- Integer division truncates — cast for decimal precision
- Ternary `? :` → concise if-else for simple value assignme

# Operators

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
