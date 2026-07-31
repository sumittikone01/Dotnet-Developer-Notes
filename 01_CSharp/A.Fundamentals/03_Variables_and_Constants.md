# 📦 Variables & Constants

## 📌 What is it?

> **Variable** — a named storage location whose value can change during program execution.
> **Constant** — a named storage location whose value is fixed at compile-time and can never change.

---

## 🧠 Intuition

```
Variable:  int score = 10;   →   score = 20;   ✅ allowed
Constant:  const int MAX = 100;  →   MAX = 200;   ❌ compile error
```

---

## ⚙️ Types of "Unchangeable" Values in C#

| Keyword      | When value is set              | Can change later?                      | Belongs to                      |
| ------------ | ------------------------------ | -------------------------------------- | ------------------------------- |
| `const`    | Compile-time                   | ❌ Never                               | Type itself (implicitly static) |
| `readonly` | Runtime (in constructor)       | ❌ After construction                  | Instance or static              |
| `var`      | Runtime (compiler infers type) | ✅ Yes — it's still a normal variable | N/A (just type inference)       |

> ⚠️ `var` is NOT a constant or dynamic type — it's a compile-time type inference shortcut. `var x = 5;` makes `x` a strongly-typed `int`, just written less verbosely.

---

## 💻 Code Examples

**Basic — Declaration:**

```csharp
int age = 25;              // variable
string name = "Sumit";     // variable
const double Pi = 3.14159; // constant — must be assigned at declaration
```

**Intermediate — `readonly` vs `const`:**

```csharp
class Config
{
    public const int MaxRetries = 3;         // fixed forever, known at compile-time
    public readonly DateTime CreatedAt;      // fixed after constructor, but can differ per object

    public Config()
    {
        CreatedAt = DateTime.Now;  // ✅ allowed inside constructor
    }
}
```

**Practical — `var` with type inference:**

```csharp
var count = 10;             // inferred as int
var name = "Sumit";         // inferred as string
var list = new List<int>(); // inferred as List<int>

// count = "text";  ❌ Compile error — still strongly typed!
```

---

## 📊 const vs readonly — Deep Comparison

| Aspect                        | `const`                                          | `readonly`                                      |
| ----------------------------- | -------------------------------------------------- | ------------------------------------------------- |
| Assigned at                   | Compile-time (must be literal/constant expression) | Runtime (constructor or field initializer)        |
| Can depend on runtime values? | ❌ No                                              | ✅ Yes (e.g.,`DateTime.Now`)                    |
| Static by default?            | ✅ Implicitly static                               | ❌ No — instance-level (unless marked`static`) |
| Can be per-object different?  | ❌ No — same for all                              | ✅ Yes — each object can have different value    |

---

## 🚨 Common Mistakes

- ❌ Using `const` for values that might change across versions/deployments (e.g., API URLs) — use config files instead
- ❌ Thinking `var` makes C# dynamically typed — it doesn't; type is fixed at compile-time
- ❌ Trying to assign a runtime value (like `DateTime.Now`) to `const` — only `readonly` supports this

---

## 🎤 Interview Questions

| Question                                                    | Key Point                                                                                       |
| ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Difference between`const` and `readonly`?               | `const` = compile-time, implicitly static; `readonly` = runtime, can vary per instance      |
| Is`var` the same as `dynamic`?                          | No —`var` is resolved to a concrete type at compile-time; `dynamic` is resolved at runtime |
| Can you change a`readonly` field outside the constructor? | No — only inside the constructor (or field initializer)                                        |

---

## 📝 30-Second Revision Cheat Sheet

- `const` → compile-time fixed, implicitly static, must be literal
- `readonly` → runtime fixed (in constructor), can differ per object
- `var` → NOT dynamic — just compiler-inferred static typing
- Rule of thumb: use `const` for truly universal constants (`Math.PI`-like), `readonly` for per-instance fixed valu

# Variables and Constants

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
