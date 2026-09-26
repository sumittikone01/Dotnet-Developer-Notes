# 02_String_Interpolation

> **String Interpolation** (`$"..."`) = embedding expressions directly inside a string literal, using `{expression}` placeholders — the modern, readable replacement for `string.Format()` and manual `+` concatenation.

> Continues **G.Modern_CSharp**, right after `01_Null_Conditional_and_Coalescing.md`. Same theme: syntax that makes everyday C# shorter, clearer, and less error-prone.

## 📌 What is it?

```csharp
string name = "Sumit";
int orderCount = 5;

// String concatenation (old style)
string message1 = "Hello " + name + ", you have " + orderCount + " orders.";

// string.Format (older, still common)
string message2 = string.Format("Hello {0}, you have {1} orders.", name, orderCount);

// String interpolation (modern) — the expression lives RIGHT WHERE it's used
string message3 = $"Hello {name}, you have {orderCount} orders.";
```

## 🤔 Why do we need it?

| Problem with the old approaches                                                                  | How interpolation helps                                                   |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------- |
| `+` concatenation gets unreadable with many pieces                                             | Reads like a normal sentence with values inline                           |
| `string.Format("{0} {1}", a, b)` requires mentally matching indices to arguments               | The value is written exactly where it appears — no index-matching needed |
| Easy to mismatch`{0}`/`{1}` placeholders with the wrong argument as a string grows           | Impossible to mismatch — the expression IS the placeholder               |
| Concatenating many types (`int`, `decimal`, `DateTime`) needs manual `.ToString()` calls | Automatic`ToString()` call on every embedded expression                 |

## 🌍 Real-world analogy

Filling out a **mail-merge template** where you write "Dear {FirstName}, your order {OrderId} shipped on {ShipDate}" directly into the letter — instead of writing "Dear [blank 1], your order [blank 2] shipped on [blank 3]" and keeping a separate numbered list of what goes in each blank (`string.Format`'s `{0}`, `{1}`, `{2}` approach).

## ⚙️ Internal working — what `$"..."` compiles to

```csharp
string message = $"Hello {name}, you have {orderCount} orders.";

// Compiles to roughly the equivalent of:
string message = string.Format("Hello {0}, you have {1} orders.", name, orderCount);

// (In many cases, the compiler further optimizes simple interpolations into
//  direct string.Concat calls when no special formatting is involved.)
```

## 📊 Formatting Inside Interpolation — the full toolkit

| Syntax                   | Purpose                             | Example                                           |
| ------------------------ | ----------------------------------- | ------------------------------------------------- |
| `{value}`              | Basic embed — calls`.ToString()` | `$"{price}"`                                    |
| `{value:FormatString}` | Apply a specific format             | `$"{price:C}"` → `$49.99` (currency)         |
| `{value,width}`        | Pad/align to a minimum width        | `$"{name,10}"` → right-aligns within 10 chars  |
| `{value,-width}`       | Left-align to a minimum width       | `$"{name,-10}"`                                 |
| `{value:N2}`           | Numeric with 2 decimal places       | `$"{price:N2}"` → `1,234.50`                 |
| `{value:yyyy-MM-dd}`   | Custom date format                  | `$"{date:yyyy-MM-dd}"` → `2026-09-26`        |
| `{{` and `}}`        | Literal curly braces (escaping)     | `$"{{literal braces}}"` → `{literal braces}` |

```csharp
decimal price = 49.9m;
DateTime orderDate = new DateTime(2026, 9, 26);

Console.WriteLine($"Price: {price:C}");                 // Price: $49.90
Console.WriteLine($"Date: {orderDate:yyyy-MM-dd}");       // Date: 2026-09-26
Console.WriteLine($"{"Name",-10}|{"Price",10}");          // "Name      |     Price"  (aligned columns)
```

## 🖼 Verbatim + Interpolated strings combined (`$@"..."`)

```csharp
string path = "products";
string fullPath = $@"C:\Data\{path}\export.csv";
// Combines verbatim string (@"...", no need to escape backslashes)
// WITH interpolation ({path} still gets substituted)
// Result: C:\Data\products\export.csv
```

## 💻 Code examples

### Basic — building dynamic messages in a BAL/Controller

```csharp
public string BuildOrderConfirmationMessage(Order order, Customer customer)
{
    return $"Hi {customer.Name}, your order #{order.Id} for {order.Items.Count} item(s) " +
           $"totaling {order.TotalAmount:C} has been confirmed and will arrive by " +
           $"{order.EstimatedDelivery:MMMM dd, yyyy}.";
}
```

### Intermediate — interpolation with embedded expressions and method calls

```csharp
public string GetProductStatusLabel(Product product)
{
    // Any valid C# expression can go inside {} — including ternaries and method calls
    return $"{product.Name}: {(product.Stock > 0 ? "In Stock" : "Out of Stock")} " +
           $"({product.Reviews.Count()} reviews, avg {product.Reviews.Average(r => r.Rating):F1}★)";
}
```

### Practical — interpolated strings in SQL logging (NOT for actual query building!)

```csharp
// ✅ SAFE — interpolation used ONLY for a LOG MESSAGE, not the actual SQL command
_logger.LogInformation($"Fetching product with Id={productId} at {DateTime.UtcNow:HH:mm:ss}");

// ❌ DANGEROUS — NEVER interpolate user input directly into a SQL string — classic SQL injection risk
// var sql = $"SELECT * FROM Products WHERE Id = {productId}";  // DO NOT DO THIS

// ✅ CORRECT for actual SQL — always use parameterized queries (per ADO.NET/DAL practices)
using var cmd = new SqlCommand("SELECT * FROM Products WHERE Id = @Id", conn);
cmd.Parameters.AddWithValue("@Id", productId);
```

## 📊 Raw String Literals (C# 11+) — a related, newer feature worth knowing

```csharp
// Traditional escaping — awkward for JSON/paths with lots of quotes/backslashes
string json1 = "{\"name\": \"Widget\", \"price\": 49.99}";

// Raw string literal (triple-quote) — NO escaping needed at all
string json2 = """{"name": "Widget", "price": 49.99}""";

// Raw + interpolated (C# 11+) — use extra $ signs to control which braces are literal
decimal price = 49.99m;
string json3 = $$"""{"name": "Widget", "price": {{price}}}""";
```

## ⚡ Performance considerations

- Simple interpolated strings compile efficiently (often to `string.Concat` for simple cases with no format specifiers) — no meaningful overhead versus manual concatenation for typical usage.
- Building strings in a **tight loop** (thousands of iterations) via repeated interpolation/concatenation is still inefficient — prefer a `StringBuilder` there, since each `+`/interpolation creates a NEW string object (strings are immutable in C#).
- `string.Format`/interpolation with heavy custom format strings has slightly more overhead than raw concatenation — negligible for typical UI/logging messages, worth knowing for extremely hot paths.

## 🚨 Common mistakes

- ❌ **Interpolating raw user input directly into a SQL query string** — a critical SQL injection vulnerability; always use parameterized `SqlCommand` parameters instead (never build SQL text with `$"..."`).
- ❌ Using string interpolation/concatenation inside a large loop instead of `StringBuilder` — creates many discarded intermediate string objects.
- ❌ Forgetting to escape literal curly braces (`{{` / `}}`) when a string genuinely needs to display a literal `{` or `}`.
- ❌ Embedding complex, hard-to-read logic inside `{}` (e.g., deeply nested ternaries or long LINQ chains) — hurts readability; consider computing the value in a separate variable first.

## 💡 Best practices

- ✅ Default to string interpolation over `string.Format()` or `+` concatenation for readability in modern C# code.
- ✅ Use format specifiers (`:C`, `:N2`, `:yyyy-MM-dd`) directly inside the interpolation rather than manually formatting beforehand.
- ✅ NEVER use interpolation to build SQL/command text with untrusted input — always use parameterized queries.
- ✅ Switch to `StringBuilder` for string-building inside loops with many iterations.
- ✅ Consider raw string literals (`"""..."""`) for JSON, regex, or file-path-heavy strings that would otherwise need heavy escaping.

## 🎤 Interview Quick-Fire Q&A

| Question                                                                          | Answer                                                                                                                            |
| --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| What does string interpolation compile down to?                                   | Roughly equivalent to a`string.Format()` call (or `string.Concat` for simple cases with no format specifiers)                 |
| How do you apply a specific format (like currency) inside an interpolated string? | Using a format specifier after a colon inside the braces, e.g.,`{price:C}`                                                      |
| Why is it dangerous to interpolate user input directly into a SQL query string?   | It creates a SQL injection vulnerability — always use parameterized queries instead                                              |
| How do you include a literal curly brace in an interpolated string?               | Escape it by doubling it:`{{` for a literal `{`, `}}` for a literal `}`                                                   |
| Why should you avoid string interpolation inside a large loop?                    | Strings are immutable — each interpolation/concatenation creates a new string object; use`StringBuilder` for repeated building |

## 📝 30-second Revision Cheat Sheet

- `$"...{expression}..."` embeds expressions directly in a string — replaces `string.Format`/`+` concatenation.
- Format specifiers go after a colon: `{price:C}`, `{date:yyyy-MM-dd}`, `{value,width}` for alignment.
- Compiles roughly to `string.Format`/`string.Concat` — negligible overhead for typical use.
- NEVER interpolate untrusted input into SQL text — always use parameterized queries.
- Use `StringBuilder` instead of interpolation/concatenation inside large loops.
- Raw string literals (`"""..."""`) avoid heavy escaping for JSON/paths/regex.
