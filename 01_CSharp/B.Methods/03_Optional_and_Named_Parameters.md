# 🏷️ Optional & Named Parameters

## 📌 What is it?

> **Optional parameters** — have a default value, so callers can omit them.
> **Named parameters** — specify arguments by parameter name instead of position, in any order.

---

## 💻 Code Examples

**Basic — Optional parameters:**

```csharp
void CreateUser(string name, bool isActive = true, string role = "User")
{
    Console.WriteLine($"{name}, Active={isActive}, Role={role}");
}

CreateUser("Sumit");                          // Active=true, Role=User (defaults used)
CreateUser("Sumit", false);                   // Active=false, Role=User
CreateUser("Sumit", false, "Admin");          // all provided
```

**Intermediate — Named parameters (skip/reorder args):**

```csharp
CreateUser(name: "Sumit", role: "Admin");     // skips isActive, uses default
CreateUser(role: "Admin", name: "Sumit");     // order doesn't matter with names
```

**Practical — Combining both (common in real APIs/utility methods):**

```csharp
void SendEmail(string to, string subject, string body, bool isHtml = false, string cc = null)
{
    // send logic
}

SendEmail(
    to: "user@example.com",
    subject: "Order Confirmation",
    body: "<p>Thank you!</p>",
    isHtml: true
);   // cc omitted, uses default null
```

---

## 🚨 Common Mistakes

- ❌ Optional parameters must come **after** required ones in the signature — `void Foo(int a = 1, int b)` is a compile error
- ❌ Changing a default value in a shared/public library — recompiling callers is needed, or they silently use the OLD default (compiled into caller's IL) — dangerous in versioned libraries
- ❌ Overusing optional parameters instead of method overloading when behavior significantly differs — can hurt readability

---

## 🎤 Interview Questions

| Question                                             | Key Point                                                                                                            |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Can optional parameters appear before required ones? | No — they must be the last parameters in the signature                                                              |
| What's the benefit of named parameters?              | Improves readability and allows skipping optional parameters out of order                                            |
| Risk of default parameter values in public APIs?     | Default is baked into the caller's compiled IL at compile-time — changing it later requires recompiling all callers |

---

## 📝 30-Second Revision Cheat Sheet

- Optional parameters = default value, must be last in signature
- Named parameters = pass by name, any order, improves readability
- Great for methods with many optional settings (avoids overload explosio

# Optional and Named Parameters

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
