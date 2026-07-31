# 🔁 Method Overloading

## 📌 What is it?

> Defining multiple methods with the **same name** but **different parameter lists** (number, type, or order of parameters) within the same class.

This is **compile-time polymorphism** — the compiler decides which version to call based on the arguments you pass.

---

## 💻 Code Examples

**Basic — overloading by parameter count/type:**

```csharp
int Add(int a, int b) => a + b;
double Add(double a, double b) => a + b;
int Add(int a, int b, int c) => a + b + c;

Add(2, 3);          // calls int version
Add(2.5, 3.5);       // calls double version
Add(1, 2, 3);        // calls 3-param version
```

**Intermediate — overloading vs optional parameters (design choice):**

```csharp
// Overloading — different behavior per signature
void Log(string message) => Console.WriteLine(message);
void Log(string message, Exception ex) => Console.WriteLine($"{message}: {ex.Message}");

// vs Optional parameter — same behavior, just fewer args needed
void Log(string message, Exception ex = null) =>
    Console.WriteLine(ex == null ? message : $"{message}: {ex.Message}");
```

> 💡 Prefer optional parameters when logic is the same; prefer overloading when behavior genuinely differs per signature.

**Practical — real-world repository pattern:**

```csharp
public class OrderRepository
{
    public Order GetById(int id) { /* fetch by id */ return null; }
    public Order GetById(string orderCode) { /* fetch by code */ return null; }
    public List<Order> GetById(List<int> ids) { /* bulk fetch */ return null; }
}
```

---

## 🚨 Common Mistakes

- ❌ Trying to overload only by **return type** — not allowed; return type alone doesn't distinguish overloads
- ❌ Creating ambiguous overloads that confuse the compiler (e.g., overloads differing only by `out`/`ref` — legal but risky)
- ❌ Overloading when optional parameters would be simpler and cleaner

---

## 🎤 Interview Questions

| Question                                       | Key Point                                                                                                                                         |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| Can you overload by return type alone?         | No — parameter list (type/count/order) must differ                                                                                               |
| Overloading vs Overriding?                     | Overloading = compile-time, same class, different signature; Overriding = runtime, inherited class, same signature (see`C.OOP/05_Polymorphism`) |
| How does the compiler pick the right overload? | Matches argument types/count at compile-time to the closest matching signature                                                                    |

---

## 📝 30-Second Revision Cheat Sheet

- Same method name, different parameter list (type/count/order) = overloading
- Compile-time polymorphism — resolved by the compiler, not at runtime
- Cannot overload by return type alone
- Choose overloading for differing behavior; optional params for same behavior with fewer ar

# Method Overloading

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
