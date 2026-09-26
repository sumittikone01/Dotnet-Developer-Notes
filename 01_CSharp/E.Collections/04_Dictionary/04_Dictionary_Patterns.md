# 🧩 Dictionary Patterns

## 📌 What is it?

This note covers **common, recurring patterns** for using dictionaries effectively in real-world code — practical idioms that come up constantly once you move past basic `Add`/lookup usage.

## 🧠 Pattern 1: Counting / Frequency Map

**Use case**: Count occurrences of items (word frequency, vote tally, character counts).

```csharp
string text = "the quick brown fox the lazy dog the";
var wordCount = new Dictionary<string, int>();

foreach (var word in text.Split(' '))
{
    if (wordCount.ContainsKey(word))
        wordCount[word]++;
    else
        wordCount[word] = 1;
}
// Result: {"the": 3, "quick": 1, "brown": 1, ...}
```

**Cleaner version using `CollectionsMarshal` / `GetValueOrDefault`:**

```csharp
foreach (var word in text.Split(' '))
{
    wordCount[word] = wordCount.GetValueOrDefault(word, 0) + 1;
}
```

## 🧠 Pattern 2: Grouping Items by a Key

**Use case**: Organize a flat list into groups (e.g. employees by department).

```csharp
var employees = GetEmployees();  // List<Employee>

var byDepartment = new Dictionary<string, List<Employee>>();

foreach (var emp in employees)
{
    if (!byDepartment.ContainsKey(emp.Department))
        byDepartment[emp.Department] = new List<Employee>();
  
    byDepartment[emp.Department].Add(emp);
}
```

💡 In practice, LINQ's `.GroupBy()` handles this far more concisely (see `F.LINQ/05_Grouping_and_Joining.md`) — this manual pattern is worth understanding, but LINQ is usually preferred in real code.

## 🧠 Pattern 3: Caching / Memoization

**Use case**: Avoid recomputing expensive results for the same input.

```csharp
var cache = new Dictionary<int, long>();

long Fibonacci(int n)
{
    if (n <= 1) return n;
  
    if (cache.TryGetValue(n, out long cached))
        return cached;
  
    long result = Fibonacci(n - 1) + Fibonacci(n - 2);
    cache[n] = result;
    return result;
}
```

This turns an exponential-time naive recursive Fibonacci into a **linear-time** one — a classic, very common real-world/interview use of dictionaries.

## 🧠 Pattern 4: "Default Value" Lookup (Avoiding Exceptions)

```csharp
var settings = new Dictionary<string, string> { { "theme", "dark" } };

// Instead of risky direct indexing:
string theme = settings.TryGetValue("theme", out var value) ? value : "light";

// Or, more concisely with GetValueOrDefault:
string fontSize = settings.GetValueOrDefault("fontSize", "12px");
```

## 🧠 Pattern 5: Using a Dictionary as a Fast "Set Membership" Check

**Use case**: When you need O(1) "does this exist?" checks, and don't actually need the value itself, consider `HashSet<T>` instead (see `04_Sets/01_HashSet_T.md`) — but a `Dictionary` is sometimes used this way too when you need both the check **and** an associated value.

```csharp
var validCodes = new Dictionary<string, bool> { { "A1", true }, { "B2", true } };

if (validCodes.ContainsKey(userInput))  
{
    // valid
}
```

⚠️ If you don't actually need an associated value, `HashSet<T>` is the more appropriate and semantically clearer choice for pure membership checks.

## 🧠 Pattern 6: Two-Way Lookup (Bidirectional Mapping)

**Use case**: Need to look up by either key or value (e.g. mapping error codes ↔ error messages).

```csharp
var codeToMessage = new Dictionary<int, string> { { 404, "Not Found" }, { 500, "Server Error" } };
var messageToCode = codeToMessage.ToDictionary(kvp => kvp.Value, kvp => kvp.Key);

Console.WriteLine(codeToMessage[404]);          // "Not Found"
Console.WriteLine(messageToCode["Not Found"]);  // 404
```

There's no single built-in "bidirectional dictionary" in .NET — maintaining **two separate dictionaries in sync** (as above) is the common, pragmatic approach.

## 🧠 Pattern 7: Dictionary of Delegates (Command/Strategy Pattern)

**Use case**: Replace a long `if/else` or `switch` chain with a lookup table of behaviors.

```csharp
var operations = new Dictionary<string, Func<int, int, int>>
{
    { "add", (a, b) => a + b },
    { "subtract", (a, b) => a - b },
    { "multiply", (a, b) => a * b }
};

int result = operations["add"](5, 3);   // 8
```

This is a clean, extensible way to map string commands (or enum values) directly to behavior — commonly seen in calculators, command processors, and plugin-style architectures.

## 🚨 Common Mistakes

- ❌ Manually writing "check-then-increment" counting logic instead of using `GetValueOrDefault()` — more verbose and error-prone than necessary.
- ❌ Reimplementing grouping logic by hand when LINQ's `.GroupBy()` already does it more concisely and robustly.
- ❌ Using a `Dictionary<TKey, bool>` purely for membership checks when a `HashSet<T>` is the more appropriate, self-documenting tool.
- ❌ Building a bidirectional mapping but forgetting to keep **both** dictionaries in sync on every update — leads to subtle desync bugs.

## 💡 Best Practices

- Use `GetValueOrDefault()` to simplify "default if missing" logic instead of verbose `ContainsKey` + indexer patterns.
- Reach for LINQ's `.GroupBy()` over manually building grouping dictionaries when working with a `List<T>` of objects.
- Use `HashSet<T>` instead of a dictionary when you only need membership checks, not associated values.
- Dictionary-of-delegates is a clean pattern for replacing large `switch`/`if-else` chains — genuinely useful in production code, not just an academic exercise.

## 🎤 Interview Questions

1. How would you implement a word-frequency counter using a dictionary?
2. How does memoization with a dictionary improve the performance of a naive recursive algorithm like Fibonacci?
3. When would you choose a `Dictionary<TKey,bool>` over a `HashSet<T>` for membership checks, if ever?
4. How would you implement a simple command dispatcher using a dictionary of delegates?

## 📝 30-second Revision Cheat Sheet

- **Frequency count**: `dict[key] = dict.GetValueOrDefault(key, 0) + 1`.
- **Grouping**: manual dictionary-of-lists, or better — LINQ `.GroupBy()`.
- **Memoization**: cache expensive results keyed by input — turns exponential recursion into linear time.
- **Command dispatch**: `Dictionary<string, Func<...>>` replaces long `switch`/`if-else` chains.
- Use `HashSet<T>` instead of a dictionary for pure membership checks.
