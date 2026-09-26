# 🧵 String Methods

## 📌 What is it?

The `string` class provides a rich set of built-in methods for searching, transforming, splitting, and formatting text — covering nearly everything you'd need for everyday string manipulation without writing custom character-by-character logic.

## 📊 Method Categories At a Glance

| Category             | Methods                                                                  |
| -------------------- | ------------------------------------------------------------------------ |
| **Search**     | `Contains`, `StartsWith`, `EndsWith`, `IndexOf`, `LastIndexOf` |
| **Transform**  | `ToUpper`, `ToLower`, `Trim`, `Replace`, `Substring`           |
| **Split/Join** | `Split`, `string.Join`                                               |
| **Check**      | `string.IsNullOrEmpty`, `string.IsNullOrWhiteSpace`                  |
| **Format**     | `string.Format`, interpolation (`$"..."`)                            |

## 💻 Searching

```csharp
string text = "Hello, World!";

bool contains = text.Contains("World");        // true
bool starts = text.StartsWith("Hello");         // true
bool ends = text.EndsWith("!");                 // true

int index = text.IndexOf("World");              // 7 — position of first match
int lastIndex = text.LastIndexOf("o");          // 8 — position of last match, searching from the end
int notFound = text.IndexOf("xyz");              // -1 — not found
```

## 💻 Transforming

```csharp
string text = "  Hello World  ";

string upper = text.ToUpper();          // "  HELLO WORLD  "
string lower = text.ToLower();          // "  hello world  "
string trimmed = text.Trim();            // "Hello World" — removes leading/trailing whitespace
string trimStart = text.TrimStart();     // "Hello World  "
string trimEnd = text.TrimEnd();          // "  Hello World"

string replaced = text.Replace("World", "C#");   // "  Hello C#  "

string sub = "Hello World".Substring(6);         // "World" — from index 6 to end
string sub2 = "Hello World".Substring(0, 5);     // "Hello" — from index 0, length 5
```

## 💻 Splitting & Joining

```csharp
string csv = "apple,banana,cherry";
string[] fruits = csv.Split(',');
// fruits = { "apple", "banana", "cherry" }

string rejoined = string.Join(" | ", fruits);
// "apple | banana | cherry"

// Split with options — removing empty entries (common with messy input data)
string messy = "a,,b,,c";
string[] clean = messy.Split(',', StringSplitOptions.RemoveEmptyEntries);
// clean = { "a", "b", "c" }
```

## 💻 Null/Empty Checks — Important Distinction

```csharp
string a = null;
string b = "";
string c = "   ";
string d = "Hello";

Console.WriteLine(string.IsNullOrEmpty(a));        // true  — null counts
Console.WriteLine(string.IsNullOrEmpty(b));        // true  — empty string counts
Console.WriteLine(string.IsNullOrEmpty(c));        // false — whitespace is NOT "empty"

Console.WriteLine(string.IsNullOrWhiteSpace(c));   // true  — catches whitespace-only strings too!
```

⚠️ **`IsNullOrWhiteSpace` is almost always the safer choice** for validating user input — `IsNullOrEmpty` alone will happily accept a string of just spaces as "valid."

## 💻 Formatting

```csharp
string name = "Rohit";
int age = 25;

// String interpolation — modern, most readable ⭐
string msg1 = $"{name} is {age} years old";

// string.Format — older style, still seen in legacy code
string msg2 = string.Format("{0} is {1} years old", name, age);

// Format specifiers within interpolation
double price = 1234.5;
string formatted = $"{price:C}";     // "$1,234.50" (currency, culture-dependent)
string formatted2 = $"{price:F2}";   // "1234.50" (2 decimal places)
```

## 📊 String Comparison Methods

```csharp
string a = "Hello";
string b = "hello";

bool exact = a == b;                                          // false — case-sensitive
bool ignoreCase = a.Equals(b, StringComparison.OrdinalIgnoreCase);  // true

int cmp = string.Compare(a, b, StringComparison.OrdinalIgnoreCase); // 0 (equal, ignoring case)
```

⚠️ For culture-sensitive user-facing text, consider `StringComparison.CurrentCultureIgnoreCase`; for internal/technical comparisons (IDs, codes, keys), prefer `StringComparison.Ordinal` / `OrdinalIgnoreCase` — it's faster and avoids unexpected culture-specific sorting quirks (e.g. Turkish "I" issues).

## 🚨 Common Mistakes

- ❌ Using `string.IsNullOrEmpty()` when you actually need to catch whitespace-only input — use `IsNullOrWhiteSpace()` instead.
- ❌ Forgetting that `Substring()`'s second parameter is a **length**, not an end index — `"Hello".Substring(1, 3)` gives `"ell"` (3 characters starting at index 1), not up to index 3.
- ❌ Using culture-sensitive comparison (`a == b` or default `.Equals()`) for things like internal codes/IDs, where `Ordinal` comparison would be faster and more predictable.
- ❌ Calling `.Split()` without `StringSplitOptions.RemoveEmptyEntries` on messy real-world input, ending up with unwanted empty string entries in the result.

## 💡 Best Practices

- Use `string.IsNullOrWhiteSpace()` for validating user input, not just `IsNullOrEmpty()`.
- Use string interpolation (`$"..."`) over `string.Format()` for new code — more readable.
- Use `StringComparison.Ordinal`/`OrdinalIgnoreCase` for internal, technical, non-user-facing comparisons.
- Remember `Substring(start, length)` — length, not end-index — to avoid off-by-N errors.

## 🎤 Interview Questions

1. What's the difference between `string.IsNullOrEmpty()` and `string.IsNullOrWhiteSpace()`?
2. What does the second parameter of `Substring()` represent?
3. When would you use `StringComparison.Ordinal` instead of the default culture-sensitive comparison?
4. What's the difference between `string.Format()` and string interpolation, functionally?

## 📝 30-second Revision Cheat Sheet

- Search: `Contains`, `StartsWith`, `EndsWith`, `IndexOf`.
- Transform: `ToUpper`, `ToLower`, `Trim`, `Replace`, `Substring(start, length)`.
- Split/Join: `.Split()` (use `RemoveEmptyEntries` for messy data), `string.Join()`.
- Validate input with `IsNullOrWhiteSpace()`, not just `IsNullOrEmpty()`.
- Use `StringComparison.Ordinal` for internal/technical string comparisons; prefer interpolation (`$"..."`) for formatting.
