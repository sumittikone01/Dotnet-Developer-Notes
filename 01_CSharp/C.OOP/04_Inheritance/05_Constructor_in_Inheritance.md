# 🏗️ Constructor in Inheritance

## 📌 What is it?

> How constructors behave when inheritance is involved — specifically, the guaranteed **execution order**: base class constructor always runs before the derived class constructor's body.

---

## 🧠 Intuition

```
new Employee("Sumit", "IT")
        │
        ▼
Step 1: Base class (Person) constructor runs FIRST
        │
        ▼
Step 2: Derived class (Employee) constructor body runs
```

This happens **even if you don't explicitly write `base(...)`** — C# implicitly calls the base's parameterless constructor if you don't specify one.

---

## 💻 Code Examples

**Basic — implicit base constructor call:**

```csharp
public class Person
{
    public Person()
    {
        Console.WriteLine("Person constructor");
    }
}

public class Employee : Person   // no explicit base() call
{
    public Employee()
    {
        Console.WriteLine("Employee constructor");
    }
}

new Employee();
// Output:
// Person constructor      ← runs first, automatically
// Employee constructor
```

**Intermediate — explicit base constructor call with parameters:**

```csharp
public class Person
{
    public string Name;
    public Person(string name)
    {
        Name = name;
        Console.WriteLine($"Person constructor: {name}");
    }
}

public class Employee : Person
{
    public string Department;

    public Employee(string name, string department) : base(name)   // MUST explicitly call since Person has no parameterless constructor
    {
        Department = department;
        Console.WriteLine($"Employee constructor: {department}");
    }
}

new Employee("Sumit", "IT");
// Output:
// Person constructor: Sumit
// Employee constructor: IT
```

**Practical — compile error scenario (common beginner mistake):**

```csharp
public class Person
{
    public Person(string name) { }   // NO parameterless constructor exists
}

public class Employee : Person
{
    public Employee()   // ❌ Compile error!
    {
        // C# tries to implicitly call base() — but Person has no parameterless constructor
    }
}

// Fix: explicitly chain
public class Employee : Person
{
    public Employee() : base("Unknown") { }   // ✅ now compiles
}
```

---

## 🚨 Common Mistakes

- ❌ Assuming a derived class automatically gets a matching parameterless constructor — if the base class only has parameterized constructors, the derived class MUST explicitly chain via `base(...)`
- ❌ Thinking constructor logic order is derived-first — it's always base-first
- ❌ Forgetting fields initialized inline (`public bool IsActive = true;`) run as part of the constructor sequence too, in declaration order, before the constructor body executes

---

## 🎤 Interview Questions

| Question                                                         | Key Point                                                                                        |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| In inheritance, which constructor executes first?                | The base class constructor always executes before the derived class constructor's body           |
| What happens if the base class has no parameterless constructor? | The derived class MUST explicitly call`base(...)` with matching arguments, or it won't compile |
| Is the base constructor called even without writing`base()`?   | Yes — C# implicitly calls the base's parameterless constructor if none is explicitly specified  |

---

## 📝 30-Second Revision Cheat Sheet

- Execution order: Base constructor → Derived constructor body (always, no exceptions)
- If base has no parameterless constructor, derived MUST explicitly use `base(...)`
- Implicit `base()` call happens automatically if not specified (only works if base has a parameterless constructo

# Constructor in Inheritance

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
