# 📮 Delegates

## 📌 What is it?

> A **delegate** is a type-safe function pointer — a variable that can hold a reference to a method, and be called/invoked like a method itself.

---

## 🌍 Real-world analogy

Think of a delegate like a **mail forwarding address**. You give someone an address, and they don't need to know WHO lives there or HOW mail gets delivered — they just send it, and whatever method is "registered" at that address handles it.

---

## 🧠 Intuition

```csharp
delegate int MathOperation(int a, int b);   // defines a "shape" of method it can point to

int Add(int a, int b) => a + b;
int Multiply(int a, int b) => a * b;

MathOperation op = Add;         // delegate now points to Add
Console.WriteLine(op(3, 4));    // 7 — invokes Add via the delegate

op = Multiply;                  // now points to a DIFFERENT method
Console.WriteLine(op(3, 4));    // 12 — invokes Multiply
```

---

## 💻 Code Examples

**Basic — custom delegate:**

```csharp
public delegate void Notify(string message);   // delegate declaration

public class OrderProcessor
{
    public void Process(Notify callback)
    {
        Console.WriteLine("Processing order...");
        callback("Order processed successfully");   // invoke the delegate
    }
}

void LogToConsole(string msg) => Console.WriteLine($"LOG: {msg}");

var processor = new OrderProcessor();
processor.Process(LogToConsole);   // passes method as a delegate argument
```

**Intermediate — built-in generic delegates (`Func`, `Action`, `Predicate`):**

```csharp
Func<int, int, int> add = (a, b) => a + b;         // returns a value
Action<string> log = msg => Console.WriteLine(msg); // returns void
Predicate<int> isEven = n => n % 2 == 0;             // returns bool

Console.WriteLine(add(3, 4));   // 7
log("Hello");                   // "Hello"
Console.WriteLine(isEven(4));   // true
```

**Practical — multicast delegates (calling multiple methods):**

```csharp
Action<string> logger = null;
logger += msg => Console.WriteLine($"Console: {msg}");
logger += msg => File.AppendAllText("log.txt", msg);   // '+=' adds another method to the chain

logger("Order placed");   // calls BOTH methods
```

---

## 📊 Built-in Delegate Types

| Delegate             | Signature                        | Use Case                                  |
| -------------------- | -------------------------------- | ----------------------------------------- |
| `Action`           | No params, returns`void`       | Simple callback with no return value      |
| `Action<T>`        | Takes params, returns`void`    | Callback needing input, no output         |
| `Func<T, TResult>` | Takes params, returns a value    | Callback that computes/returns something  |
| `Predicate<T>`     | Takes one param, returns`bool` | Condition-checking (used heavily in LINQ) |

---

## 🚨 Common Mistakes

- ❌ Defining a custom delegate type when `Func`/`Action`/`Predicate` already covers the need — prefer built-in generic delegates unless you need a specific, named delegate type for clarity
- ❌ Forgetting multicast delegates with a return value only return the LAST method's result (earlier results are discarded)
- ❌ Not checking for `null` before invoking a delegate that might have no subscribers — use `?.Invoke()` to be safe

---

## 🎤 Interview Questions

| Question                                   | Key Point                                                                                |
| ------------------------------------------ | ---------------------------------------------------------------------------------------- |
| What is a delegate?                        | A type-safe reference to a method, allowing methods to be passed as parameters/variables |
| Difference between`Func` and `Action`? | `Func` returns a value; `Action` returns void                                        |
| What is a multicast delegate?              | A delegate that references multiple methods, all invoked in sequence when called         |

---

## 📝 30-Second Revision Cheat Sheet

- Delegate = type-safe method reference/pointer
- `Func<T, TResult>` → returns value | `Action<T>` → void | `Predicate<T>` → returns bool
- Multicast delegates (`+=`) call multiple methods; only the LAST return value is kept
- Foundation for Events (next topic) and LINQ (predicates/selectors

# Delegates

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
