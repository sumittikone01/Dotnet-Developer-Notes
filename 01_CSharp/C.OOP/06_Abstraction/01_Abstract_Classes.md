# 🎨 Abstract Classes

## 📌 What is it?

> An **abstract class** is a class that **cannot be instantiated directly** and may contain both fully-implemented methods AND `abstract` methods (no implementation) that derived classes MUST implement.

Used to achieve **Abstraction** — the 4th OOP pillar — by defining a common contract while allowing shared implementation.

---

## 🧠 Intuition

```
abstract class Shape
{
    public abstract double GetArea();      // NO implementation — derived MUST provide it
    public void PrintInfo()                 // full implementation — shared by all
    {
        Console.WriteLine($"Area: {GetArea()}");
    }
}
```

- `Shape shape = new Shape();` → ❌ Compile error — can't instantiate abstract class
- Every derived class MUST implement `GetArea()` or itself be abstract

---

## 💻 Code Examples

**Basic:**

```csharp
public abstract class Shape
{
    public abstract double GetArea();          // must be implemented by derived classes
    public abstract double GetPerimeter();

    public void Display()                       // shared, concrete implementation
    {
        Console.WriteLine($"Area: {GetArea()}, Perimeter: {GetPerimeter()}");
    }
}

public class Circle : Shape
{
    public double Radius;
    public override double GetArea() => Math.PI * Radius * Radius;
    public override double GetPerimeter() => 2 * Math.PI * Radius;
}

// var shape = new Shape();  ❌ Compile error
var circle = new Circle { Radius = 5 };
circle.Display();   // uses inherited Display(), calls overridden GetArea()/GetPerimeter()
```

**Practical — real-world use case (common architecture pattern):**

```csharp
public abstract class ReportGenerator
{
    public void GenerateReport()             // template method — shared workflow
    {
        FetchData();
        FormatData();
        Export();
    }

    protected abstract void FetchData();      // each report type implements differently
    protected abstract void FormatData();

    protected virtual void Export()           // has default, but CAN be overridden
    {
        Console.WriteLine("Exporting as PDF (default)");
    }
}

public class SalesReportGenerator : ReportGenerator
{
    protected override void FetchData() => Console.WriteLine("Fetching sales data");
    protected override void FormatData() => Console.WriteLine("Formatting sales report");
}
```

> 💡 This is the **Template Method design pattern** — abstract class defines the workflow skeleton, derived classes fill in the specifics. Very common in real-world reporting/processing pipelines.

---

## 📊 Abstract Class Rules

| Rule                                              | Detail                                              |
| ------------------------------------------------- | --------------------------------------------------- |
| Cannot be instantiated                            | `new AbstractClass()` is always a compile error   |
| Can have constructors                             | Yes — called via derived class's`base()`         |
| Can have fields/properties                        | Yes — fully supported, unlike interfaces (pre-C#8) |
| Can mix abstract + concrete methods               | Yes — this is its key advantage over interfaces    |
| Derived class must implement all abstract members | Unless the derived class is ALSO abstract           |

---

## 🚨 Common Mistakes

- ❌ Trying to instantiate an abstract class directly
- ❌ Forgetting derived classes must implement ALL abstract members, or be abstract themselves
- ❌ Using abstract classes when a simple interface would suffice (see `03_Abstract_Class_vs_Interface.md` for when to choose which)

---

## 🎤 Interview Questions

| Question                                                | Key Point                                                                   |
| ------------------------------------------------------- | --------------------------------------------------------------------------- |
| Can you instantiate an abstract class?                  | No, never — even if it has no abstract members                             |
| Can an abstract class have a constructor?               | Yes — used when a derived class calls`base()`                            |
| What's the benefit of abstract classes over interfaces? | Can provide shared implementation (concrete methods) alongside the contract |

---

## 📝 30-Second Revision Cheat Sheet

- Abstract class = cannot instantiate, mix of abstract (no body) + concrete (full body) methods
- Derived class MUST implement all abstract members (or be abstract itself)
- Great for shared logic + enforced contract (Template Method pattern)
- Different from interfaces — can hold fields, constructors, and concrete method bodie

# Abstract Classes

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
