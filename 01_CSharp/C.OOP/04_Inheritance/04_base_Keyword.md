# 🔙 base Keyword

## 📌 What is it?

> `base` refers to the **immediate parent class**, used to access base class members (methods, constructors, properties) from within a derived class.

---

## 💻 Code Examples

**Basic — calling base method inside overridden method:**

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal makes a sound");
}

public class Dog : Animal
{
    public override void Speak()
    {
        base.Speak();                     // calls Animal's version first
        Console.WriteLine("Dog barks");   // then adds Dog-specific behavior
    }
}

new Dog().Speak();
// Output:
// Animal makes a sound
// Dog barks
```

**Intermediate — calling base constructor:**

```csharp
public class Person
{
    public string Name;
    public Person(string name) { Name = name; }
}

public class Employee : Person
{
    public string Department;

    public Employee(string name, string department) : base(name)   // calls Person's constructor
    {
        Department = department;
    }
}
```

**Practical — extending base validation logic (common real-world pattern):**

```csharp
public class BaseValidator
{
    public virtual bool Validate(Order order)
    {
        return order != null && order.Quantity > 0;
    }
}

public class PremiumOrderValidator : BaseValidator
{
    public override bool Validate(Order order)
    {
        // reuse base validation, then add extra rule
        return base.Validate(order) && order.CustomerTier == "Premium";
    }
}
```

---

## 📊 `base` vs `this` — Quick Comparison

| Aspect            | `this`                                                         | `base`                                                   |
| ----------------- | ---------------------------------------------------------------- | ---------------------------------------------------------- |
| Refers to         | Current object instance                                          | Immediate parent class                                     |
| Common use        | Resolve naming conflicts, constructor chaining within same class | Access parent's overridden method, call parent constructor |
| Constructor usage | `: this(...)`                                                  | `: base(...)`                                            |

---

## 🚨 Common Mistakes

- ❌ Forgetting to call `base.Speak()` when you actually want to EXTEND the base behavior, not fully replace it — leads to unintentionally dropping base logic
- ❌ Trying to use `base` inside a static method — not allowed, since `base` relates to instance context
- ❌ Confusing `base` with accessing a grandparent class — `base` only accesses the IMMEDIATE parent, not further up the chain

---

## 🎤 Interview Questions

| Question                                 | Key Point                                                             |
| ---------------------------------------- | --------------------------------------------------------------------- |
| What does`base` refer to?              | The immediate parent class of the current class                       |
| When would you use`base.MethodName()`? | To call/reuse the parent's implementation inside an overridden method |
| Can`base` be used in a static context? | No — it relates to instance-level inheritance                        |

---

## 📝 30-Second Revision Cheat Sheet

- `base` = reference to the immediate parent class
- `base.Method()` → call parent's version (often combined with `override` to extend behavior)
- `base(...)` in constructor → chain to parent's constructor
- Only accesses the IMMEDIATE parent, not further up multi-level hierarchie

# base Keyword

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
