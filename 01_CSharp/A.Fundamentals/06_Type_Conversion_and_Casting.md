# 🔄 Type Conversion & Casting

## 📌 What is it?

> Converting a value from one data type to another — either automatically (implicit) or manually (explicit).

---

## 🧠 Intuition

```
Implicit Conversion  →  Safe, no data loss  →  Compiler does it automatically
   int → long → float → double

Explicit Conversion (Casting)  →  Possible data loss  →  YOU must ask for it
   double → int   (decimal part is chopped off)
```

---

## 📊 Conversion Methods — Comparison Table

| Method                             | When to use                                                     | Behavior on failure                                                                                      |
| ---------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Implicit conversion**      | Widening (small type → big type), no data loss                 | Never fails — compiler-safe                                                                             |
| **Explicit cast** `(int)x` | Narrowing (big type → small type)                              | Throws`OverflowException` (in `checked` context) or silently truncates (default `unchecked`)       |
| `Convert.ToXxx()`                | Converting between unrelated types (e.g.,`string` → `int`) | Throws`FormatException`/`InvalidCastException` on invalid input; handles `null` → returns default |
| `Parse()`                        | `string` → number/type                                       | Throws`FormatException` if invalid; `ArgumentNullException` if null                                  |
| `TryParse()`                     | Safe parsing without exceptions                                 | Returns`bool`; no exception — **preferred for user input**                                      |
| `as` operator                    | Reference type casting                                          | Returns`null` instead of throwing (only for reference types/nullable)                                  |

---

## 💻 Code Examples

**Basic — Implicit vs Explicit:**

```csharp
// Implicit — safe, automatic
int a = 100;
long b = a;        // int → long, no data loss, no cast needed
double c = b;       // long → double, no cast needed

// Explicit — must cast, possible data loss
double price = 99.99;
int roundedDown = (int)price;   // 99 — decimal part truncated (not rounded!)
```

**Intermediate — Convert vs Parse vs TryParse:**

```csharp
// Convert.ToInt32 — handles null gracefully (returns 0)
string input1 = null;
int x = Convert.ToInt32(input1);   // 0, no exception

// int.Parse — throws if invalid or null
string input2 = "123";
int y = int.Parse(input2);         // 123

// int.TryParse — SAFEST for user input (e.g., form fields, query params)
string input3 = "abc";
bool success = int.TryParse(input3, out int result);
if (!success)
    Console.WriteLine("Invalid number!");   // this runs — result = 0
```

**Practical — Reading from ADO.NET `SqlDataReader` (your daily use case):**

```csharp
// Common real-world pattern in DAL layer
while (reader.Read())
{
    int id = Convert.ToInt32(reader["Id"]);
    string name = reader["Name"].ToString();
    decimal price = Convert.ToDecimal(reader["Price"]);
    DateTime createdDate = Convert.ToDateTime(reader["CreatedDate"]);

    // Handling nullable DB columns safely
    int? quantity = reader["Quantity"] != DBNull.Value
        ? Convert.ToInt32(reader["Quantity"])
        : (int?)null;
}
```

> 💡 This exact pattern shows up constantly in your ADO.NET + stored procedure workflow — `Convert.ToXxx()` is preferred over casting when reading `object`-typed DB values, since it handles `DBNull`/type mismatches more gracefully than a direct cast.

---

## 🚨 Common Mistakes

- ❌ Using `(int)` cast on a `string` — casting only works between compatible types (numeric-to-numeric, or class hierarchies); `string` → `int` needs `Parse`/`Convert`, not a cast
- ❌ Using `int.Parse()` on unpredictable user input without try/catch → crashes app; use `TryParse()` instead
- ❌ Forgetting that casting `double` → `int` **truncates**, doesn't round (`(int)9.9` = `9`, not `10`) — use `Math.Round()` first if rounding is intended
- ❌ Using `Convert.ToInt32()` on a `DBNull.Value` — throws `InvalidCastException`; always null-check DB values first

---

## 🎤 Interview Questions

| Question                                                     | Key Point                                                                                    |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| Difference between implicit and explicit conversion?         | Implicit = safe/automatic (widening); Explicit = manual, possible data loss (narrowing)      |
| Difference between`Convert.ToInt32()` and `int.Parse()`? | `Convert` handles `null` (returns 0); `Parse` throws on `null`                       |
| Why prefer`TryParse` over `Parse`?                       | Avoids exceptions for invalid input — better performance & control flow for untrusted input |
| Does casting`double` to `int` round or truncate?         | Truncates (chops decimal part) — does NOT round                                             |

---

## 📝 30-Second Revision Cheat Sheet

- **Implicit** = automatic, safe, widening (`int → double`)
- **Explicit/cast** = manual, risky, narrowing (`double → int`, truncates!)
- `Convert.ToXxx()` → null-safe | `Parse()` → throws on null/invalid | `TryParse()` → safest, no exceptions
- ADO.NET tip: always null-check (`DBNull.Value`) before converting reader valu

# Type Conversion and Casting

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
