
# 06 — ExecuteScalar

---

## 🎯 One-Line Definition

> **`ExecuteScalar()` executes a SQL query and returns the value of the first column of the first row as a single `object` — perfect for COUNT, MAX, MIN, SUM, or any query that returns exactly one value.**

---

## 🔷 What is ExecuteScalar?

`ExecuteScalar()` is the method you call when your SQL returns **one single value** — not rows, not tables, just one number or string.

> 💡 If `ExecuteReader()` is reading a full report, `ExecuteScalar()` is asking **"just give me the total."**

---

## 🔷 Basic Syntax

```csharp
using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
{
    object result = cmd.ExecuteScalar();
    int count = (int)result;
    Console.WriteLine($"Total employees: {count}");
}
```

---

## 🔷 How ExecuteScalar Works

```
Your SQL returns:
┌─────────┐
│  Count  │  ← only this cell matters
├─────────┤
│   47    │  ← ExecuteScalar returns this value
│   ...   │  ← all other rows/columns IGNORED
│   ...   │
└─────────┘

ExecuteScalar() → reads Row 1, Column 1 → returns as object → you cast it
```

> If the query returns multiple rows and columns — only the **first row, first column** is returned. Everything else is ignored.

---

## 🔷 Return Type — Always `object`

`ExecuteScalar()` always returns `object`. You must **cast** it to the correct type.

```csharp
// ── COUNT → int ────────────────────────────────────────────────
int count = (int)cmd.ExecuteScalar();

// ── MAX Salary → decimal ───────────────────────────────────────
decimal maxSalary = (decimal)cmd.ExecuteScalar();

// ── MIN Date → DateTime ────────────────────────────────────────
DateTime earliest = (DateTime)cmd.ExecuteScalar();

// ── SUM → decimal ──────────────────────────────────────────────
decimal totalPayroll = (decimal)cmd.ExecuteScalar();

// ── Single string (e.g. name) → string ─────────────────────────
string name = cmd.ExecuteScalar().ToString();

// ── Get the newly inserted ID (SCOPE_IDENTITY) ─────────────────
int newId = Convert.ToInt32(cmd.ExecuteScalar());
```

---

## 🔷 Handling NULL — Always Check First

If the query returns **no rows** (empty table or no match), `ExecuteScalar()` returns **`null`** — not `DBNull.Value`, but actual C# `null`.

```csharp
// ❌ DANGEROUS — crashes if result is null
int count = (int)cmd.ExecuteScalar();

// ✅ SAFE — handle null before casting
object result = cmd.ExecuteScalar();

if (result != null && result != DBNull.Value)
{
    int count = (int)result;
}
else
{
    Console.WriteLine("No data returned.");
}

// ✅ Shortcut with Convert — handles null safely
int count2 = Convert.ToInt32(cmd.ExecuteScalar());
// Convert.ToInt32(null) = 0  — no exception
```

---

## 🔷 Common Use Cases

### COUNT — Total rows

```csharp
public int GetTotalEmployees()
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
        {
            return (int)cmd.ExecuteScalar();
        }
    }
}
```

### COUNT with filter

```csharp
public int GetCountByDepartment(string dept)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        string sql = "SELECT COUNT(*) FROM Employee WHERE Department = @Dept";

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Dept", dept);
            return (int)cmd.ExecuteScalar();
        }
    }
}
```

### MAX — Highest salary

```csharp
public decimal GetMaxSalary()
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("SELECT MAX(Salary) FROM Employee", con))
        {
            object result = cmd.ExecuteScalar();
            return result != null ? (decimal)result : 0;
        }
    }
}
```

### SUM — Total payroll

```csharp
public decimal GetTotalPayroll()
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("SELECT SUM(Salary) FROM Employee", con))
        {
            return Convert.ToDecimal(cmd.ExecuteScalar());
        }
    }
}
```

### Get newly inserted ID — SCOPE_IDENTITY()

```csharp
public int InsertEmployeeAndGetId(string name, decimal salary)
{
    string sql = @"INSERT INTO Employee(Name, Salary) VALUES(@Name, @Salary);
                   SELECT SCOPE_IDENTITY();";
    //                     ↑ SCOPE_IDENTITY() returns the newly generated ID

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Name",   name);
            cmd.Parameters.AddWithValue("@Salary", salary);

            int newId = Convert.ToInt32(cmd.ExecuteScalar());
            Console.WriteLine($"New employee ID: {newId}");
            return newId;
        }
    }
}
```

### Check if record exists

```csharp
public bool EmailExists(string email)
{
    string sql = "SELECT COUNT(*) FROM Employee WHERE Email = @Email";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Email", email);

            int count = (int)cmd.ExecuteScalar();
            return count > 0;   // true if at least one record found
        }
    }
}
```

### Get a single field by ID

```csharp
public string GetEmployeeName(int id)
{
    string sql = "SELECT Name FROM Employee WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Id", id);

            object result = cmd.ExecuteScalar();
            return result?.ToString() ?? "Not found";
        }
    }
}
```

---

## 🔷 ExecuteScalar vs ExecuteReader for Single Value

```csharp
// ❌ WRONG — using ExecuteReader for a single value (wasteful)
using (SqlDataReader reader = cmd.ExecuteReader())
{
    if (reader.Read())
    {
        int count = (int)reader[0];
    }
}

// ✅ CORRECT — use ExecuteScalar for single values
int count = (int)cmd.ExecuteScalar();
```

> Use `ExecuteScalar()` whenever your query returns ONE value — it's simpler and slightly more efficient.

---

## 🔷 All Three Execute Methods — When to Use Which

```
What does your SQL return?
│
├── Multiple rows                 →  ExecuteReader()    → SqlDataReader
│   SELECT * FROM Employee
│
├── Single value (aggregate)      →  ExecuteScalar()    → object (cast it)
│   SELECT COUNT(*) FROM Employee
│   SELECT MAX(Salary) FROM Employee
│   SELECT SCOPE_IDENTITY()
│
└── No rows (write operation)     →  ExecuteNonQuery()  → int (rows affected)
    INSERT / UPDATE / DELETE
```

---

## 🔷 ExecuteScalar Summary Table

| Scenario          | SQL                                       | Cast To                          |
| ----------------- | ----------------------------------------- | -------------------------------- |
| Count all rows    | `SELECT COUNT(*) FROM Table`            | `int`                          |
| Count with filter | `SELECT COUNT(*) WHERE col = @val`      | `int`                          |
| Check existence   | `SELECT COUNT(*) WHERE id = @id`        | `int`→`> 0`                 |
| Highest value     | `SELECT MAX(Salary) FROM Table`         | `decimal`                      |
| Lowest value      | `SELECT MIN(Salary) FROM Table`         | `decimal`                      |
| Total / Sum       | `SELECT SUM(Salary) FROM Table`         | `decimal`                      |
| Average           | `SELECT AVG(Salary) FROM Table`         | `decimal`                      |
| Get single string | `SELECT Name FROM Table WHERE Id = @Id` | `string`                       |
| New row identity  | `SELECT SCOPE_IDENTITY()`               | `int`via `Convert.ToInt32()` |

---

## ⭐ Interview Quick-Fire

| Question                                  | Answer                                                      |
| ----------------------------------------- | ----------------------------------------------------------- |
| What does `ExecuteScalar()`return?      | `object`— first column of first row                      |
| When to use `ExecuteScalar()`?          | Single value: COUNT, MAX, MIN, SUM, SCOPE_IDENTITY          |
| What if the query returns no rows?        | Returns `null`— always check before casting              |
| Safe way to cast to int?                  | `Convert.ToInt32(cmd.ExecuteScalar())`                    |
| What is `SCOPE_IDENTITY()`?             | Returns the last auto-generated ID in the current scope     |
| Why not use `ExecuteReader()`for COUNT? | Overkill —`ExecuteScalar()`is simpler and more efficient |
| What happens to other rows/columns?       | Ignored — only first row, first column is returned         |
| How to check if a record exists?          | `SELECT COUNT(*) WHERE...`→`ExecuteScalar()`→`> 0`  |
