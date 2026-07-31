# ❓ Nullable Types

## 📌 What is it?

> By default, **value types** (`int`, `bool`, `DateTime`, etc.) cannot be `null` — they always have a value. A **nullable type** (`int?`) allows a value type to also represent "no value."

---

## 🤔 Why do we need it?

Real-world data is often **optional**:

- A database column that allows `NULL` (e.g., `MiddleName`, `DeliveryDate`)
- A form field the user didn't fill in
- A value that "doesn't exist yet" vs "exists and is zero"

Without nullable types, you'd have to use magic values (`-1`, `DateTime.MinValue`) to represent "missing" — nullable types make this explicit and type-safe.

---

## 🧠 Intuition

```
int x = null;      ❌ Compile error — int is a value type, can't be null
int? x = null;      ✅ Works — Nullable<int>

Nullable<int> is a struct wrapper:
┌─────────────────────────┐
│ Nullable<T>              │
│  ├── HasValue : bool     │
│  └── Value    : T        │
└─────────────────────────┘
```

`int?` is just shorthand for `Nullable<int>`.

---

## 💻 Code Examples

**Basic — Declaring & checking:**

```csharp
int? age = null;

if (age.HasValue)
    Console.WriteLine(age.Value);
else
    Console.WriteLine("Age not provided");
```

**Intermediate — Null-coalescing shortcuts:**

```csharp
int? quantity = null;

int finalQty = quantity ?? 0;              // if null, default to 0
int finalQty2 = quantity.GetValueOrDefault(); // same effect, alternate syntax

Console.WriteLine(finalQty);  // 0
```

**Practical — Mapping nullable DB columns (your ADO.NET workflow):**

```csharp
// SQL column DeliveryDate allows NULL
DateTime? deliveryDate = reader["DeliveryDate"] != DBNull.Value
    ? Convert.ToDateTime(reader["DeliveryDate"])
    : (DateTime?)null;

if (deliveryDate.HasValue)
    Console.WriteLine($"Delivers on: {deliveryDate.Value:d}");
else
    Console.WriteLine("Not yet scheduled");
```

---

## 📊 Nullable Value Types vs Nullable Reference Types

| Aspect               | `int?` (Nullable Value Type)                            | `string?` (Nullable Reference Type — C# 8+)           |
| -------------------- | --------------------------------------------------------- | -------------------------------------------------------- |
| Applies to           | Value types (`struct`, `int`, `bool`, `DateTime`) | Reference types (`class`, `string`)                  |
| Underlying mechanism | `Nullable<T>` struct wrapper                            | Compiler flow-analysis + annotation (no runtime wrapper) |
| Default without`?` | Cannot be null                                            | Can already be null (reference types default to`null`) |
| Purpose              | Explicitly allow "no value" for value types               | Warn you at compile-time about possible null reference   |

> 📝 Note: Nullable Reference Types (`string?`) is a distinct, newer feature — covered separately in `D.Language_Features/09_Nullable_Reference_Types.md`. This file focuses only on nullable *value* types.

---

## 🚨 Common Mistakes

- ❌ Accessing `.Value` without checking `.HasValue` first → throws `InvalidOperationException` if null
- ❌ Using `??` on non-nullable types unnecessarily — only needed when the left side can actually be null
- ❌ Forgetting nullable value types need unwrapping before passing to methods expecting non-nullable types

---

## 🎤 Interview Questions

| Question                                    | Key Point                                                                                                                                  |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| What is`int?`?                            | Shorthand for`Nullable<int>` — allows value types to represent "no value"                                                               |
| How do you safely read a nullable value?    | Check`.HasValue` first, or use `??` / `GetValueOrDefault()`                                                                          |
| Difference between`int?` and `string?`? | `int?` wraps a value type in `Nullable<T>`; `string?` is compiler-level null-safety annotation on an already-nullable reference type |

---

## 📝 30-Second Revision Cheat Sheet

- `int?` = `Nullable<int>` → lets value types hold `null`
- Check with `.HasValue`, read with `.Value`, default with `??` or `.GetValueOrDefault()`
- Essential for mapping nullable DB columns in ADO.NET
- Different from Nullable **Reference** Types (`string?`) — that's a separate C# 8+ featu

# Nullable Types

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
