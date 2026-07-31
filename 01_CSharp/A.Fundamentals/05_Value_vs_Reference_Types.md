# 🎯 Value Types vs Reference Types

## 📌 What is it?

> **Value type** — variable directly holds the data.
> **Reference type** — variable holds a **pointer/reference** to the data, which lives elsewhere (heap).

This is one of the most important — and most interview-tested — concepts in C#.

---

## 🌍 Real-world analogy

- **Value type** = giving someone a **photocopy** of a document. They can scribble on it — your original is untouched.
- **Reference type** = giving someone the **address of your house**. If they change something inside the house, you see the change too (same house, same object).

---

## 🖼 Visual — Assignment Behavior

```
VALUE TYPE (int, struct, bool, etc.)
─────────────────────────────────────
int a = 10;
int b = a;      // b gets a COPY of value 10
b = 20;

a = 10   b = 20     ← independent copies


REFERENCE TYPE (class, array, string*)
─────────────────────────────────────
Person p1 = new Person("Sumit");
Person p2 = p1;      // p2 points to SAME object as p1
p2.Name = "Rahul";

p1 ──┐
     ├──► [ Person { Name: "Rahul" } ]   ← both point to same object
p2 ──┘
```

> \* `string` is technically a reference type but **behaves immutably** — see note below.

---

## 📊 Comparison Table

| Aspect              | Value Type                                          | Reference Type                                                |
| ------------------- | --------------------------------------------------- | ------------------------------------------------------------- |
| Stores              | Actual data                                         | Reference (address) to data                                   |
| Memory (typical)    | Stack                                               | Heap                                                          |
| Assignment behavior | Copies the value                                    | Copies the reference (both point to same object)              |
| Examples            | `int`, `double`, `bool`, `struct`, `enum` | `class`, `array`, `string`, `interface`, `delegate` |
| Default value       | `0`, `false`, etc.                              | `null`                                                      |
| Passed to methods   | By value (copy), unless`ref`/`out` used         | Reference is passed by value (see note below)                 |

---

## 💻 Code Examples

**Basic — Value type independence:**

```csharp
int a = 10;
int b = a;
b = 99;

Console.WriteLine(a);  // 10 (unchanged)
Console.WriteLine(b);  // 99
```

**Intermediate — Reference type sharing:**

```csharp
class Person
{
    public string Name;
}

Person p1 = new Person { Name = "Sumit" };
Person p2 = p1;         // p2 references the SAME object
p2.Name = "Rahul";

Console.WriteLine(p1.Name);  // "Rahul" — p1 changed too!
```

**Practical — The `string` special case:**

```csharp
string s1 = "Hello";
string s2 = s1;
s2 = "World";           // this creates a NEW string, doesn't mutate the old one

Console.WriteLine(s1);  // "Hello" — unaffected!
Console.WriteLine(s2);  // "World"
```

> 💡 `string` is a reference type, but it's **immutable** — every "change" actually creates a new string object. This is why it *behaves* like a value type in practice. Deep dive in `E.Collections/07_Strings/04_String_Interning_and_Immutability.md`.

---

## ⚙️ Important Nuance: Passing to Methods

```csharp
void Modify(int x) { x = 100; }              // copy — caller's value unaffected
void Modify(Person p) { p.Name = "Changed"; } // reference copy — but points to SAME object, so mutation IS visible
void Replace(Person p) { p = new Person(); }  // reassigning the LOCAL reference — caller's variable unaffected
```

| Scenario                                      | Does caller see the change?          |
| --------------------------------------------- | ------------------------------------ |
| Value type passed normally                    | ❌ No                                |
| Reference type — mutating a property         | ✅ Yes (same object)                 |
| Reference type — reassigning to a new object | ❌ No (only local reference changes) |

---

## 🚨 Common Mistakes

- ❌ Assuming reassigning a reference type parameter inside a method affects the caller's variable — it doesn't (only mutation does)
- ❌ Forgetting `string` is immutable — repeated concatenation in a loop creates many objects (use `StringBuilder` — see `07_Strings/03_StringBuilder.md`)
- ❌ Copying a reference type variable and expecting independent objects (`p2 = p1` does NOT clone)

---

## 🎤 Interview Questions

| Question                                                                         | Key Point                                                                                                                  |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Where are value types vs reference types stored?                                 | Value types typically on stack (or inline in heap if part of an object); reference types' data on heap, reference on stack |
| Is`string` a value type or reference type?                                     | Reference type, but immutable — behaves like a value type due to no in-place mutation                                     |
| If you pass an object to a method and mutate a property, does the caller see it? | Yes — because both point to the same heap object                                                                          |
| If you reassign the object reference inside the method, does caller see it?      | No — only the local copy of the reference is reassigned                                                                   |

---

## 📝 30-Second Revision Cheat Sheet

- Value type = copy of data | Reference type = copy of the address (points to same object)
- `int`, `struct`, `bool`, `enum` → value types
- `class`, `array`, `string` (immutable exception) → reference types
- Mutating a reference type's property → visible to caller
- Reassigning a reference type variable inside a method → NOT visible to call

# Value vs Reference Types

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
