# 📝 Query Syntax

## 📌 What is it?

**Query Syntax** is a SQL-like way to write LINQ queries, using keywords like `from`, `where`, `select`, `orderby`, and `group by` — as an alternative to method (fluent chain) syntax.

```csharp
List<int> numbers = new List<int> { 5, 2, 8, 1, 9 };

var query = from n in numbers
            where n > 3
            orderby n
            select n;

// query: { 5, 8, 9 }
```

## 🤔 Why do we need it?

Some people find query syntax more **readable** for certain kinds of queries — especially ones involving **joins** and **grouping**, where the SQL-like structure closely mirrors how you'd think about the problem in database terms. It's also a gentler on-ramp for developers coming from a SQL background.

> Important: query syntax and method syntax (`03_Method_Syntax.md`) are **not two different features** — query syntax is literally **compiled into method syntax** by the compiler. Anything you write in query syntax could also be written in method syntax (though the reverse isn't always true — some methods have no query-syntax equivalent).

## 🌍 Real-world analogy

Query syntax is like **speaking a sentence in a very structured, formal grammar** ("Select all customers, where age is over 18, ordered by name") — closely mirroring how a SQL statement reads, which is comforting if SQL is already familiar territory.

## ⚙️ Core Keywords

| Keyword      | Purpose                                              | SQL Equivalent             |
| ------------ | ---------------------------------------------------- | -------------------------- |
| `from`     | Specifies the data source and a range variable       | `FROM`                   |
| `where`    | Filters elements                                     | `WHERE`                  |
| `select`   | Projects/transforms each element                     | `SELECT`                 |
| `orderby`  | Sorts results (`ascending`/`descending`)         | `ORDER BY`               |
| `group by` | Groups elements by a key                             | `GROUP BY`               |
| `join`     | Combines two sequences based on a matching key       | `JOIN`                   |
| `let`      | Introduces an intermediate variable within the query | (no direct SQL equivalent) |

## 💻 Basic Filtering & Projection

```csharp
var employees = GetEmployees();

var seniorNames = from e in employees
                   where e.YearsOfExperience > 5
                   select e.Name;
```

## 💻 Ordering — Ascending / Descending

```csharp
var sorted = from e in employees
             orderby e.Salary descending, e.Name ascending
             select e;
```

`orderby` supports **multiple sort keys**, comma-separated — the second key breaks ties from the first, exactly like SQL's `ORDER BY col1 DESC, col2 ASC`.

## 💻 `let` — Introducing an Intermediate Variable

```csharp
var result = from e in employees
             let bonus = e.Salary * 0.1m
             where bonus > 5000
             select new { e.Name, bonus };
```

`let` lets you compute a value once and reuse it (both in `where` and `select`) without repeating the calculation — a genuinely useful query-syntax-only convenience.

## 💻 Grouping

```csharp
var byDepartment = from e in employees
                    group e by e.Department into deptGroup
                    select new { Department = deptGroup.Key, Count = deptGroup.Count() };
```

`group ... by ... into` is the query-syntax way to bucket elements — full detail on grouping in `05_Grouping_and_Joining.md`.

## 💻 Joining Two Sequences

```csharp
var query = from order in orders
            join customer in customers on order.CustomerId equals customer.Id
            select new { order.OrderId, customer.Name };
```

This SQL-like `join ... on ... equals ...` structure is one of the biggest reasons people reach for query syntax — it's arguably more readable here than the equivalent method-syntax `.Join()` call.

## 📊 Query Syntax vs Method Syntax — Same Query, Both Ways

```csharp
// Query syntax
var query1 = from e in employees
             where e.YearsOfExperience > 5
             orderby e.Name
             select e.Name;

// Method syntax — functionally IDENTICAL, compiles to the same thing
var query2 = employees
    .Where(e => e.YearsOfExperience > 5)
    .OrderBy(e => e.Name)
    .Select(e => e.Name);
```

## 🚨 Not Every LINQ Method Has a Query Syntax Equivalent!

Many common operations — `.Count()`, `.Sum()`, `.First()`, `.Any()`, `.ToList()` — have **no** query-syntax keyword. You must call these as a method, often by wrapping a query-syntax expression in parentheses:

```csharp
int count = (from e in employees where e.YearsOfExperience > 5 select e).Count();
```

This is a major reason query syntax is used **less often** in modern C# code — most real-world code ends up mixing both, or just uses method syntax throughout for consistency.

## 🚨 Common Mistakes

- ❌ Assuming query syntax and method syntax are two entirely separate features — they're not; query syntax is purely **syntactic sugar** that compiles down to the same method calls.
- ❌ Trying to find a query-syntax keyword for methods like `.Sum()`, `.Count()`, `.Any()` — these don't exist in query syntax; you must call them as methods.
- ❌ Overusing query syntax for simple one-step filters where method syntax would be shorter and just as clear (`numbers.Where(n => n > 5)` vs the more verbose `from n in numbers where n > 5 select n`).

## 💡 Best Practices

- Reach for query syntax specifically when a query involves **joins** or complex **grouping** — it tends to read more naturally there.
- Use method syntax for everything else, especially simple filters/projections — it's more concise and is what most modern C# codebases use predominantly.
- Don't feel obligated to pick one style exclusively — mixing (writing a query-syntax expression, then calling `.Count()` on the result) is completely normal and common.

## 🎤 Interview Questions

1. Is query syntax a separate feature from method syntax, or does it compile down to the same thing?
2. Why do query syntax and joins/grouping tend to pair particularly well together?
3. Name a common LINQ operation that has NO query-syntax keyword equivalent.
4. What does the `let` keyword do in query syntax, and why is it useful?

## 📝 30-second Revision Cheat Sheet

- Query syntax = SQL-like LINQ syntax: `from`, `where`, `select`, `orderby`, `group by`, `join`, `let`.
- Compiles down to the **exact same method calls** as method syntax — purely syntactic sugar.
- Especially readable for **joins** and **grouping**.
- Methods like `.Count()`, `.Sum()`, `.Any()` have **no query-syntax equivalent** — must call as methods.
- Method syntax is more common overall in modern C# code.
