# 🔗 Association

## 📌 What is it?

> **Association** is the most general relationship between two classes — they interact/use each other, but neither owns the other. Objects can exist completely independently.

---

## 🌍 Real-world analogy

A **Teacher** and a **Student** are associated — a teacher teaches students, students learn from teachers — but neither "owns" the other. If the teacher leaves, students still exist. If a student leaves, the teacher still exists.

---

## 💻 Code Example

```csharp
public class Teacher
{
    public string Name;
}

public class Student
{
    public string Name;
    public Teacher AssignedTeacher;   // association — Student references Teacher, but doesn't own it
}

var teacher = new Teacher { Name = "Mr. Sharma" };
var student = new Student { Name = "Sumit", AssignedTeacher = teacher };

// Teacher exists independently of Student
// Deleting the student doesn't delete the teacher
```

---

## 📊 Association Types

| Type                   | Description                             | Example                                                     |
| ---------------------- | --------------------------------------- | ----------------------------------------------------------- |
| **One-to-One**   | One object relates to exactly one other | `Person` ↔ `Passport`                                  |
| **One-to-Many**  | One object relates to many others       | `Teacher` → many `Students`                            |
| **Many-to-Many** | Many objects relate to many others      | `Student` ↔ `Course` (many students take many courses) |

---

## 🚨 Common Mistakes

- ❌ Confusing association with aggregation/composition — association has NO ownership implication at all; it's the loosest relationship
- ❌ Over-thinking simple "uses-a" relationships as something more complex than they are

---

## 🎤 Interview Questions

| Question                                     | Key Point                                                                     |
| -------------------------------------------- | ----------------------------------------------------------------------------- |
| What is association in OOP?                  | A general "uses-a" relationship between two independent classes, no ownership |
| Does association imply lifecycle dependency? | No — both objects can exist and be destroyed independently                   |
| Give a real-world example of association.    | Doctor and Patient — related, but independent                                |

---

## 📝 30-Second Revision Cheat Sheet

- Association = general "uses-a" relationship, no ownership
- Objects exist independently of each other
- Can be one-to-one, one-to-many, or many-to-many
- Loosest of the three object relationships (Association → Aggregation → Composition, in order of increasing coupling

# Association

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
