
# 🔢 Data Types

## 📌 What is it?

> A **data type** defines what kind of value a variable can hold, how much memory it uses, and what operations are valid on it.

C# is **statically & strongly typed** — every variable's type is known at compile-time and can't silently change.

---

## 🧠 Intuition

```
Data Types
│
├── Value Types (stored on Stack*)
│   ├── Numeric (int, double, decimal, float, long...)
│   ├── bool
│   ├── char
│   └── struct / enum
│
└── Reference Types (stored on Heap*)
    ├── string
    ├── class
    ├── array
    ├── interface
    └── delegate
```

> \* Simplified — actual storage location depends on context (local var vs field). Full details in `05_Value_vs_Reference_Types.md` and `J.Memory_and_Runtime/03_Stack_vs_Heap.md`.

---

## 📊 Built-in Numeric Types

| Type        | Size     | Range (approx)           | Use Case                                |
| ----------- | -------- | ------------------------ | --------------------------------------- |
| `byte`    | 1 byte   | 0 to 255                 | Small positive numbers, raw binary data |
| `short`   | 2 bytes  | -32K to 32K              | Rarely used directly                    |
| `int`     | 4 bytes  | -2.1B to 2.1B            | Default choice for whole numbers        |
| `long`    | 8 bytes  | Very large               | IDs, timestamps, large counters         |
| `float`   | 4 bytes  | ~7 digit precision       | Rarely used in business apps            |
| `double`  | 8 bytes  | ~15-16 digit precision   | Scientific/general decimal math         |
| `decimal` | 16 bytes | ~28-29 digit precision   | **Money/financial calculations**  |
| `bool`    | 1 byte   | `true` / `false`     | Conditions/flags                        |
| `char`    | 2 bytes  | Single Unicode character | Single character storage                |

> 💡 **Real-world rule (important for your stack):** Always use `decimal` for money-related fields (prices, salaries, quantities in billing) — never `float`/`double`, due to rounding errors.

---

## 💻 Code Examples

**Basic — Declaring different types:**

```csharp
int age = 25;
long population = 8_000_000_000;   // digit separator for readability
double pi = 3.14159;
decimal price = 499.99m;           // 'm' suffix required for decimal literals
bool isActive = true;
char grade = 'A';
string name = "Sumit";
```

**Intermediate — Why `decimal` matters:**

```csharp
double d = 0.1 + 0.2;
Console.WriteLine(d);      // 0.30000000000000004 ❌ floating-point error

decimal m = 0.1m + 0.2m;
Console.WriteLine(m);      // 0.3 ✅ exact
```

**Practical — Typical ADO.NET/SQL mapping:**

```csharp
// SQL: DECIMAL(18,2) → C#: decimal
// SQL: INT           → C#: int
// SQL: DATETIME      → C#: DateTime
// SQL: BIT           → C#: bool
// SQL: NVARCHAR      → C#: string

decimal totalAmount = Convert.ToDecimal(reader["TotalAmount"]);
```

---

## 🚨 Common Mistakes

- ❌ Using `float`/`double` for money → rounding errors compound over many transactions
- ❌ Forgetting the `m` suffix on decimal literals (`decimal x = 5.5;` → compiles as double, then implicit conversion — better to write `5.5m`)
- ❌ Assuming `int` can hold any large number — overflows silently unless `checked` context is used

---

## 🎤 Interview Questions

| Question                                               | Key Point                                                                         |
| ------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Why use`decimal` instead of `double` for money?    | `decimal` has higher precision and avoids binary floating-point rounding errors |
| Default type of a floating-point literal like`3.14`? | `double` — must add `f` (float) or `m` (decimal) suffix otherwise          |
| Difference between`int` and `long`?                | `long` is 8 bytes (larger range) vs `int`'s 4 bytes                           |

---

## 📝 30-Second Revision Cheat Sheet

- Types split into **Value Types** (stack-ish) vs **Reference Types** (heap-ish)
- Money → always `decimal` | General math → `double` | Whole numbers → `int`/`long`
- Literal suffixes: `m` = decimal, `f` = float, `d` = double (default)
- C# is statically & strongly typed — no silent type changes
