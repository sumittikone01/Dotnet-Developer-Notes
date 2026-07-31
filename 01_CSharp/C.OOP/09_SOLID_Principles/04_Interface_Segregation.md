# 4️⃣ Interface Segregation Principle (ISP)

## 📌 What is it?

> **"Clients should not be forced to depend on interfaces they don't use."** Prefer several small, focused interfaces over one large, general-purpose interface.

The **I** in **SOLID**.

---

## 🧠 Intuition

```
❌ VIOLATES ISP — fat interface forces unnecessary implementation:
interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

class RobotWorker : IWorker
{
    public void Work() { /* works fine */ }
    public void Eat() => throw new NotImplementedException();   // robots don't eat!
    public void Sleep() => throw new NotImplementedException(); // robots don't sleep!
}

✅ FOLLOWS ISP — segregated, focused interfaces:
interface IWorkable { void Work(); }
interface IFeedable { void Eat(); }
interface ISleepable { void Sleep(); }

class HumanWorker : IWorkable, IFeedable, ISleepable { /* implements all naturally */ }
class RobotWorker : IWorkable { /* only implements what actually applies */ }
```

---

## 💻 Code Example — Real Project Scenario

```csharp
// ❌ Fat interface
public interface IReportGenerator
{
    void GeneratePdf();
    void GenerateExcel();
    void GenerateCsv();
    void EmailReport();
    void PrintReport();
}
// Every report type must implement ALL of these, even if it only supports PDF!

// ✅ Segregated interfaces
public interface IPdfExportable { void GeneratePdf(); }
public interface IExcelExportable { void GenerateExcel(); }
public interface IEmailable { void EmailReport(); }

public class SalesReport : IPdfExportable, IEmailable   // only implements what it actually supports
{
    public void GeneratePdf() { /* ... */ }
    public void EmailReport() { /* ... */ }
}

public class InventoryReport : IPdfExportable, IExcelExportable   // different combination, no forced Email
{
    public void GeneratePdf() { /* ... */ }
    public void GenerateExcel() { /* ... */ }
}
```

---

## 📊 ISP vs SRP — Common Confusion

| Aspect     | SRP                                      | ISP                                                                   |
| ---------- | ---------------------------------------- | --------------------------------------------------------------------- |
| Applies to | Classes                                  | Interfaces                                                            |
| Concern    | A class should have one reason to change | An interface shouldn't force implementers to depend on unused members |
| Fix        | Split a class by responsibility          | Split an interface by client need                                     |

---

## 🚨 Common Mistakes

- ❌ Creating "fat" interfaces that try to cover every possible use case — forces implementers to write `NotImplementedException` stubs
- ❌ Over-segregating into too many single-method interfaces when cohesive grouping would be clearer — balance is key
- ❌ Confusing ISP with SRP — ISP is specifically about interface design, not class responsibility

---

## 🎤 Interview Questions

| Question                                     | Key Point                                                                                                  |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| What is the Interface Segregation Principle? | Interfaces should be small and focused — clients shouldn't be forced to implement methods they don't need |
| What's a sign that ISP is being violated?    | Classes implementing an interface method with`throw new NotImplementedException()`                       |
| How does ISP differ from SRP?                | ISP is about interface design (client-focused contracts); SRP is about class responsibility                |

---

## 📝 30-Second Revision Cheat Sheet

- ISP = prefer small, focused interfaces over large, general-purpose ones
- Red flag: `NotImplementedException` in interface implementations
- A class can implement MULTIPLE small interfaces to compose exactly the capabilities it need

# Interface Segregation Principle

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
