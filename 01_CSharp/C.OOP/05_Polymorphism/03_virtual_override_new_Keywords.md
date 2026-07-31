# 🔑 virtual, override, new Keywords

## 📌 What is it?

> Three keywords controlling how derived classes interact with base class methods:
>
> - `virtual` — marks a base method as **overridable**
> - `override` — provides a new implementation that **replaces** the base version (polymorphic)
> - `new` — provides a separate implementation that **hides** the base version (non-polymorphic)

---

## 📊 Side-by-Side Comparison

| Keyword             | Placed on                             | Effect                                                                 |
| ------------------- | ------------------------------------- | ---------------------------------------------------------------------- |
| `virtual`         | Base class method                     | Allows derived classes to override it                                  |
| `override`        | Derived class method                  | Replaces base implementation — polymorphic (runtime-resolved)         |
| `new`             | Derived class method                  | Hides base implementation — NOT polymorphic (reference-type resolved) |
| `abstract`        | Base class method (in abstract class) | MUST be overridden by derived classes — no default implementation     |
| `sealed override` | Derived class method                  | Overrides, but prevents further overriding in deeper subclasses        |

---

## 💻 Code Examples

**All three together:**

```csharp
public class Base
{
    public virtual void MethodA() => Console.WriteLine("Base.MethodA");
    public void MethodB() => Console.WriteLine("Base.MethodB");   // not virtual
}

public class Derived : Base
{
    public override void MethodA() => Console.WriteLine("Derived.MethodA");   // overrides — polymorphic
    public new void MethodB() => Console.WriteLine("Derived.MethodB");        // hides — non-polymorphic
}

Base b = new Derived();
b.MethodA();   // "Derived.MethodA" — override wins, polymorphism works
b.MethodB();   // "Base.MethodB"    — new hides, but reference type (Base) decides
```

**`sealed override` — preventing further overriding:**

```csharp
public class Animal
{
    public virtual void Speak() => Console.WriteLine("Animal sound");
}

public class Dog : Animal
{
    public sealed override void Speak() => Console.WriteLine("Dog barks");
    // sealed here means NO further class can override Speak() again
}

public class Puppy : Dog
{
    // public override void Speak() { }   ❌ Compile error — Speak() is sealed in Dog
}
```

---

## 🖼 Decision Flowchart

```
Do you want the derived class to be ABLE to change this method's behavior?
        │
        ├── No  → leave base method as-is (no virtual)
        │
        └── Yes → mark base method 'virtual'
                    │
                    Do you want polymorphism (correct version via base reference)?
                    │
                    ├── Yes → derived uses 'override'
                    │
                    └── No (rare, usually accidental) → derived uses 'new'
```

---

## 🚨 Common Mistakes

- ❌ Using `new` instead of `override` by mistake (missing the `override` keyword entirely defaults to hiding with a compiler warning, not overriding)
- ❌ Forgetting `override` requires the base to be `virtual`/`abstract`/`override` already — can't override a plain method
- ❌ Overusing `sealed override` without a clear reason — locks out legitimate future extension

---

## 🎤 Interview Questions

| Question                                                                                 | Key Point                                                                                                     |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| What happens if you don't mark a base method`virtual` but try `override` in derived? | Compile error —`override` requires the base method to be `virtual`, `abstract`, or itself `override` |
| What does`sealed override` do?                                                         | Overrides the method but prevents any further overriding in deeper subclasses                                 |
| What warning appears if you redefine a base method without`new` or `override`?       | Compiler warning: hides inherited member, suggests adding`new` to make it explicit                          |

---

## 📝 30-Second Revision Cheat Sheet

- `virtual` (base) → allows overriding
- `override` (derived) → replaces base, polymorphic, runtime-resolved
- `new` (derived) → hides base, NOT polymorphic, reference-type-resolved
- `sealed override` → overrides but blocks further overriding down the chai

# virtual override new Keywords

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
