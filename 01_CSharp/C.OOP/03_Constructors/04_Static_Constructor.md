# ⚡ Static Constructor

## 📌 What is it?

> A constructor that initializes **static members** of a class. It runs **automatically, exactly once**, before the class is used for the first time (before any instance is created or static member is accessed).

---

## 💻 Code Examples

**Basic:**

```csharp
public class AppConfig
{
    public static string ConnectionString;

    static AppConfig()   // STATIC CONSTRUCTOR — no access modifier, no parameters
    {
        ConnectionString = "Server=.;Database=MyApp;Trusted_Connection=True;";
        Console.WriteLine("AppConfig initialized once");
    }
}

// First access triggers the static constructor automatically:
Console.WriteLine(AppConfig.ConnectionString);
// "AppConfig initialized once" prints first, only ONE time, no matter how many times accessed after
```

**Intermediate — proving it runs only once:**

```csharp
public class Counter
{
    public static int InitCount;

    static Counter()
    {
        InitCount++;
    }
}

var c1 = new Counter();
var c2 = new Counter();
var c3 = new Counter();
Console.WriteLine(Counter.InitCount);  // 1 — static constructor ran only ONCE, despite 3 instances
```

**Practical — loading configuration once at app startup pattern:**

```csharp
public class CacheManager
{
    private static readonly Dictionary<string, object> _cache;

    static CacheManager()
    {
        _cache = new Dictionary<string, object>();
        // could load initial cache data from a config file here
    }

    public static void Set(string key, object value) => _cache[key] = value;
    public static object Get(string key) => _cache.TryGetValue(key, out var val) ? val : null;
}
```

---

## 📊 Static vs Instance Constructor

| Aspect          | Instance Constructor                   | Static Constructor                       |
| --------------- | -------------------------------------- | ---------------------------------------- |
| Runs when       | Every time`new` is called            | Once, automatically, before first use    |
| Access modifier | Can have any (`public`, `private`) | None allowed                             |
| Parameters      | Can accept parameters                  | Cannot accept any parameters             |
| Purpose         | Initialize instance-level fields       | Initialize static fields, one-time setup |

---

## 🚨 Common Mistakes

- ❌ Trying to add parameters to a static constructor — not allowed, since it's called automatically by the runtime, not explicitly by you
- ❌ Adding an access modifier (`public static AppConfig()`) — not allowed, static constructors have no access modifier
- ❌ Assuming it runs at app startup — it actually runs lazily, right before the class is first used (first instance created OR first static member accessed)

---

## 🎤 Interview Questions

| Question                                    | Key Point                                                                                              |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| When does a static constructor run?         | Automatically, once, right before the class is first used (instance created or static member accessed) |
| Can a static constructor take parameters?   | No                                                                                                     |
| Can you call a static constructor manually? | No — it's invoked automatically by the CLR                                                            |

---

## 📝 30-Second Revision Cheat Sheet

- Static constructor = runs once, automatically, before first use of the class
- No access modifier, no parameters
- Used for one-time static field initialization / setup logic
- Different from instance constructors, which run every time `new` is calle

# Static Constructor

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
