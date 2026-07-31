# ⚡ Compile-Time Polymorphism (Method Overloading)

## 📌 What is it?

> **Compile-time polymorphism** (also called static polymorphism) — the specific method to call is determined by the **compiler**, at compile-time, based on the method signature. Achieved via **method overloading** and **operator overloading**.

> 📝 Note: Method overloading mechanics are covered in detail in `B.Methods/05_Method_Overloading.md`. This file focuses on framing it as a **polymorphism concept** for interview purposes.

---

## 🧠 Intuition

```
Compiler sees the call site and decides WHICH overload to use
based on argument types/count — before the program even runs.

Add(2, 3)        → compiler picks Add(int, int)
Add(2.5, 3.5)    → compiler picks Add(double, double)
```

This is "compile-time" because the decision is baked into the compiled code — no runtime lookup needed.

---

## 💻 Code Examples

**Method overloading as compile-time polymorphism:**

```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
    public double Add(double a, double b) => a + b;
    public int Add(int a, int b, int c) => a + b + c;
}

var calc = new Calculator();
calc.Add(2, 3);         // compiler resolves to Add(int, int) — decided NOW, at compile-time
calc.Add(2.5, 3.5);      // compiler resolves to Add(double, double)
```

**Operator overloading (also compile-time polymorphism):**

```csharp
public class Money
{
    public decimal Amount;

    public static Money operator +(Money a, Money b)
    {
        return new Money { Amount = a.Amount + b.Amount };
    }
}

var total = new Money { Amount = 100 } + new Money { Amount = 200 };
Console.WriteLine(total.Amount);  // 300 — '+' resolved at compile-time to the custom operator
```

---

## 📊 Compile-Time vs Runtime Polymorphism (Preview)

| Aspect        | Compile-Time (Overloading)                         | Runtime (Overriding)                        |
| ------------- | -------------------------------------------------- | ------------------------------------------- |
| Resolved when | Compile-time                                       | Runtime                                     |
| Achieved via  | Method/operator overloading                        | `virtual`/`override`                    |
| Flexibility   | Fixed — determined by argument types at call site | Dynamic — determined by actual object type |
| Also known as | Static polymorphism / early binding                | Dynamic polymorphism / late binding         |

> Full runtime polymorphism details in `02_Runtime_Method_Overriding.md`.

---

## 🚨 Common Mistakes

- ❌ Confusing "compile-time polymorphism" with just "overloading exists" — the interview-relevant point is WHY it's called polymorphism: same name, different behavior, resolved differently than overriding
- ❌ Thinking compile-time polymorphism is "less important" — it's foundational and used constantly (e.g., `Console.WriteLine` has dozens of overloads)

---

## 🎤 Interview Questions

| Question                                              | Key Point                                                                                                              |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| What is compile-time polymorphism?                    | Same method/operator name, different behavior, resolved by the compiler based on signature — achieved via overloading |
| Give two examples of compile-time polymorphism in C#. | Method overloading and operator overloading                                                                            |
| Why is it called "early binding"?                     | The specific method to call is bound/decided before the program runs (at compile-time)                                 |

---

## 📝 30-Second Revision Cheat Sheet

- Compile-time polymorphism = method/operator overloading, resolved by compiler based on signature
- Also called: static polymorphism / early binding
- Contrast with runtime polymorphism (overriding) — resolved at runtime based on actual object typ

# Compile Time Method Overloading

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
