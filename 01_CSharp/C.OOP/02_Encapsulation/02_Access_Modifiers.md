# 🚪 Access Modifiers

## 📌 What is it?

> Keywords that control the **visibility/accessibility** of classes, methods, fields, and properties — from within the same class, same assembly, derived classes, or anywhere.

---

## 📊 Access Modifier Comparison

| Modifier               | Same Class | Derived Class (same assembly) | Same Assembly (non-derived) | Derived Class (different assembly) | Anywhere |
| ---------------------- | ---------- | ----------------------------- | --------------------------- | ---------------------------------- | -------- |
| `private`            | ✅         | ❌                            | ❌                          | ❌                                 | ❌       |
| `protected`          | ✅         | ✅                            | ❌                          | ✅                                 | ❌       |
| `internal`           | ✅         | ✅                            | ✅                          | ❌                                 | ❌       |
| `protected internal` | ✅         | ✅                            | ✅                          | ✅                                 | ❌       |
| `private protected`  | ✅         | ✅ (same assembly only)       | ❌                          | ❌                                 | ❌       |
| `public`             | ✅         | ✅                            | ✅                          | ✅                                 | ✅       |

---

## 🖼 Visual — Access Scope Hierarchy

```
public              ← accessible everywhere
  │
protected internal   ← same assembly OR derived class (any assembly)
  │
internal             ← same assembly only
  │
protected            ← same class + derived classes only
  │
private protected    ← same class + derived classes, SAME assembly only
  │
private              ← same class only (most restrictive)
```

---

## 💻 Code Examples

**Basic:**

```csharp
public class Employee
{
    private decimal salary;         // only accessible inside Employee class
    protected string department;    // accessible in Employee + derived classes
    internal string employeeCode;   // accessible anywhere in same project/assembly
    public string Name;             // accessible from anywhere
}
```

**Practical — typical BAL/DAL layering:**

```csharp
namespace MyApp.DAL
{
    internal class DatabaseHelper   // only accessible within the DAL assembly — hides implementation
    {
        internal SqlConnection GetConnection() { /* ... */ return null; }
    }

    public class OrderRepository    // public — the DAL's exposed API
    {
        public Order GetById(int id) { /* uses DatabaseHelper internally */ return null; }
    }
}
```

> 💡 Real-world pattern: mark internal implementation details (`DatabaseHelper`) as `internal`, and only expose the repository's public methods (`OrderRepository`) — this is encapsulation applied at the assembly/project level.

---

## 🚨 Common Mistakes

- ❌ Defaulting everything to `public` — increases coupling, makes refactoring risky
- ❌ Confusing `protected` (inheritance-based access) with `internal` (assembly-based access) — they solve different problems
- ❌ Forgetting default access levels: class members default to `private`; top-level classes default to `internal`

---

## 🎤 Interview Questions

| Question                                              | Key Point                                                                                                                                   |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Difference between`protected` and `internal`?     | `protected` = accessible in derived classes (any assembly); `internal` = accessible anywhere in same assembly (no inheritance required) |
| What is`protected internal`?                        | Union — accessible if EITHER same assembly OR derived class condition is true                                                              |
| What is`private protected`? (C# 7.2+)               | Intersection — accessible only if BOTH same assembly AND derived class                                                                     |
| What's the default access modifier for class members? | `private`                                                                                                                                 |

---

## 📝 30-Second Revision Cheat Sheet

- `private` → same class only | `public` → everywhere
- `protected` → same class + derived classes | `internal` → same assembly
- `protected internal` → OR of both | `private protected` → AND of both
- Default for class members = `private` | Default for top-level class = `interna`

# Access Modifiers

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
