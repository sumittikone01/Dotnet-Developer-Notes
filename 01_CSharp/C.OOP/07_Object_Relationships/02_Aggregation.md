# 🧩 Aggregation

## 📌 What is it?

> **Aggregation** is a "HAS-A" relationship with **weak ownership** — one object contains/uses another, but the contained object can exist **independently** and be shared across multiple owners.

---

## 🌍 Real-world analogy

A **Department** HAS **Employees**. But if the Department is dissolved, the Employees still exist — they can move to another department. The Employee's existence isn't tied to that specific Department.

---

## 💻 Code Example

```csharp
public class Employee
{
    public string Name;
}

public class Department
{
    public string Name;
    public List<Employee> Employees = new();   // aggregation — Department HAS Employees
}

var emp1 = new Employee { Name = "Sumit" };
var emp2 = new Employee { Name = "Rahul" };

var dept = new Department { Name = "Engineering" };
dept.Employees.Add(emp1);
dept.Employees.Add(emp2);

// Employees are created INDEPENDENTLY and passed in — Department doesn't create them
// If 'dept' is deleted, emp1 and emp2 still exist and could be assigned elsewhere
```

---

## 📊 Aggregation vs Association vs Composition (Preview)

| Aspect                                    | Association     | Aggregation          | Composition                         |
| ----------------------------------------- | --------------- | -------------------- | ----------------------------------- |
| Ownership                                 | None            | Weak ("has-a")       | Strong ("owns-a")                   |
| Lifecycle dependency                      | Independent     | Independent          | Dependent — child dies with parent |
| Can the "part" exist without the "whole"? | N/A             | ✅ Yes               | ❌ No                               |
| Example                                   | Doctor–Patient | Department–Employee | Car–Engine                         |

---

## 🚨 Common Mistakes

- ❌ Confusing aggregation with composition — the key test is: "Can the child object be reassigned to a different parent or exist without the parent?" If yes → aggregation. If no → composition.
- ❌ Assuming a `List<T>` field automatically means composition — it depends on whether the objects were created BY the container or passed in from outside (aggregation)

---

## 🎤 Interview Questions

| Question                                       | Key Point                                                                                                                             |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| What is aggregation?                           | A HAS-A relationship with weak ownership — the contained object can exist independently                                              |
| How is aggregation different from composition? | In aggregation, the "part" can exist without the "whole" and can be shared; in composition, the part's lifecycle is tied to the whole |
| Give a real-world aggregation example.         | A Department has Employees, but Employees exist independently of any specific Department                                              |

---

## 📝 30-Second Revision Cheat Sheet

- Aggregation = HAS-A, weak ownership, "part" can exist independently
- Objects are typically created OUTSIDE the container and passed in
- Contrast: Composition = strong ownership, "part" dies with the "whole

# Aggregation

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
