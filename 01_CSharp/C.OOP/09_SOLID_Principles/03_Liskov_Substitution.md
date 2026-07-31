# 3️⃣ Liskov Substitution Principle (LSP)

## 📌 What is it?

> **"Objects of a derived class must be substitutable for objects of the base class, without breaking the application."** If `B` inherits from `A`, you should be able to use `B` anywhere `A` is expected, and everything should still work correctly.

The **L** in **SOLID**.

---

## 🧠 Intuition — The Classic Violation: Square/Rectangle

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public int GetArea() => Width * Height;
}

public class Square : Rectangle   // "mathematically" a Square IS-A Rectangle...
{
    public override int Width
    {
        get => base.Width;
        set { base.Width = value; base.Height = value; }   // forces both to stay equal
    }
    public override int Height
    {
        get => base.Height;
        set { base.Width = value; base.Height = value; }
    }
}

void TestRectangle(Rectangle r)
{
    r.Width = 5;
    r.Height = 10;
    Console.WriteLine(r.GetArea());   // expected: 50
}

TestRectangle(new Rectangle());   // 50 ✅ correct
TestRectangle(new Square());      // 100 ❌ WRONG! Violates caller's expectation — LSP broken!
```

> The `Square` breaks the expected behavior of `Rectangle` when substituted in — even though it's technically a valid inheritance relationship on paper.

---

## 💻 Better Design — Avoiding the Violation

```csharp
public abstract class Shape
{
    public abstract int GetArea();
}

public class Rectangle : Shape
{
    public int Width, Height;
    public override int GetArea() => Width * Height;
}

public class Square : Shape   // no longer inherits Rectangle — sibling relationship instead
{
    public int Side;
    public override int GetArea() => Side * Side;
}
```

> By making both `Rectangle` and `Square` independent implementations of a common `Shape` abstraction (rather than Square inheriting Rectangle), the substitution problem disappears entirely.

---

## 📊 Signs of LSP Violation

| Sign                                                                    | Explanation                                          |
| ----------------------------------------------------------------------- | ---------------------------------------------------- |
| Derived class throws`NotImplementedException` for an inherited method | The derived class can't honor the full base contract |
| Derived class overrides a method to do LESS or behave unexpectedly      | Breaks caller assumptions built around the base type |
| Client code needs`if (obj is DerivedType)` checks to work correctly   | Signals the substitution isn't truly seamless        |

---

## 🚨 Common Mistakes

- ❌ Modeling "IS-A" purely based on real-world/mathematical logic (Square IS mathematically a Rectangle) without considering BEHAVIORAL compatibility
- ❌ Overriding a method to throw an exception or do nothing just to "fit" an inheritance hierarchy — a red flag that inheritance is the wrong tool here
- ❌ Not testing whether a derived class truly satisfies ALL the base class's behavioral contracts, not just its structural signature

---

## 🎤 Interview Questions

| Question                                                   | Key Point                                                                                                         |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| What is the Liskov Substitution Principle?                 | Derived classes must be usable in place of their base class without breaking expected behavior                    |
| What's the classic example used to explain LSP violations? | Square inheriting Rectangle — breaks behavioral expectations despite being logically valid                       |
| How do you detect an LSP violation in code?                | Look for overridden methods that throw exceptions, do less than expected, or require type-checking in client code |

---

## 📝 30-Second Revision Cheat Sheet

- LSP = derived class must be a true behavioral substitute for its base class
- Classic violation example: Square inheriting Rectangle
- Red flags: `NotImplementedException` in overrides, unexpected behavior changes, client-side type checks
- Fix: reconsider the hierarchy — sometimes sibling classes under a shared abstraction work better than forced inheritanc

# Liskov Substitution Principle

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
