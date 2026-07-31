# ⚡ Static Classes & Members

## 📌 What is it?

> `static` means a member (or entire class) belongs to the **type itself**, not to any specific instance. There's only ONE copy, shared across the whole application.

---

## 🧠 Intuition

```
Instance member:  each object gets its OWN copy
Static member:    ONE shared copy for the entire class, accessed via ClassName.Member
```

---

## 💻 Code Examples

**Basic — static field vs instance field:**

```csharp
public class Counter
{
    public static int TotalCount;    // shared across ALL instances
    public int InstanceId;           // unique per instance

    public Counter()
    {
        TotalCount++;                 // increments the SHARED counter
        InstanceId = TotalCount;
    }
}

var c1 = new Counter();
var c2 = new Counter();
var c3 = new Counter();

Console.WriteLine(Counter.TotalCount);  // 3 — shared across all instances
Console.WriteLine(c1.InstanceId);       // 1
Console.WriteLine(c3.InstanceId);       // 3
```

**Intermediate — static class (utility/helper pattern):**

```csharp
public static class MathHelper   // entire class is static — cannot be instantiated
{
    public static double CalculateDiscount(decimal price, double percent)
    {
        return (double)price * (1 - percent / 100);
    }
}

// var m = new MathHelper();  ❌ Compile error — cannot instantiate a static class
double discounted = MathHelper.CalculateDiscount(1000, 10);   // called directly on the class
```

**Practical — real-world static utility (common in your projects):**

```csharp
public static class ValidationHelper
{
    public static bool IsValidEmail(string email) =>
        !string.IsNullOrEmpty(email) && email.Contains("@");

    public static bool IsValidPhoneNumber(string phone) =>
        !string.IsNullOrEmpty(phone) && phone.Length == 10;
}

// Used anywhere without creating an object:
if (ValidationHelper.IsValidEmail(userEmail))
{
    // proceed
}
```

---

## 📊 Static vs Instance — Comparison

| Aspect                       | Instance Member       | Static Member                                                  |
| ---------------------------- | --------------------- | -------------------------------------------------------------- |
| Belongs to                   | Each object           | The class/type itself                                          |
| Access via                   | `object.Member`     | `ClassName.Member`                                           |
| Memory                       | New copy per object   | Single shared copy                                             |
| Can access instance members? | Yes                   | ❌ No — static methods can't directly access instance members |
| Common use                   | Object-specific state | Utility methods, shared counters, constants                    |

---

## 🚨 Common Mistakes

- ❌ Trying to access an instance member from within a static method — not allowed, static methods have no `this` context
- ❌ Overusing static classes for things that should be instance-based (e.g., stateful services) — static state is shared globally and can cause threading/testability issues
- ❌ Using static fields for mutable shared state in a web app (ASP.NET Core) — dangerous, since multiple concurrent requests could interfere with each other (not thread-safe by default)

---

## 🎤 Interview Questions

| Question                                                | Key Point                                                                                        |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| What does`static` mean?                               | The member/class belongs to the type itself, not any instance — single shared copy              |
| Can a static class have instance methods?               | No — all members of a static class must also be static                                          |
| Why be cautious with static state in a web application? | Shared across all requests/users — can cause race conditions and unexpected shared state issues |

---

## 📝 30-Second Revision Cheat Sheet

- `static` = belongs to the type, ONE shared copy, accessed via `ClassName.Member`
- Static class → cannot be instantiated, all members must be static (great for utility/helper classes)
- Static methods CANNOT access instance members directly
- ⚠️ Avoid mutable static state in web apps — not safe across concurrent request

# Static Classes and Members

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
