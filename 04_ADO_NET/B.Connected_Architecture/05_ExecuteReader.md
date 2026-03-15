
# 05 — ExecuteReader

---

## 🎯 One-Line Definition

> **`ExecuteReader()` executes a SELECT query and returns a `SqlDataReader` — a fast, forward-only, read-only cursor that reads result rows one at a time from the database.**

---

## 🔷 What is ExecuteReader?

`ExecuteReader()` is the method you call when your SQL returns  **multiple rows** .
It does not load all rows into memory at once — it reads one row at a time as you call `reader.Read()`.

> 💡 Think of it like reading a book page by page — you don't photocopy the whole book first, you just turn pages one at a time.

---

## 🔷 Basic Syntax

```csharp
using (SqlCommand cmd = new SqlCommand("SELECT Id, Name, Salary FROM Employee", con))
{
    using (SqlDataReader reader = cmd.ExecuteReader())
    {
        while (reader.Read())   // ← Read() moves to next row, returns false at end
        {
            // access columns here
        }
    }
}
```

---

## 🔷 How ExecuteReader Works — Step by Step

```
Your Code                  ADO.NET                     SQL Server
─────────────────────────────────────────────────────────────────────
cmd.ExecuteReader()
                           Sends SQL →                 Executes SELECT
                                       ←─────────────  Returns result set
                                                        (streaming, not all at once)
reader.Read()              ← reads Row 1 from stream
Console.WriteLine(...)     ← you use the values
reader.Read()              ← reads Row 2
Console.WriteLine(...)
reader.Read()              ← reads Row 3
...
reader.Read()              ← returns false — no more rows
while loop exits
reader.Close()             ← closes the stream
```

---

## 🔷 `reader.Read()` — The Key Method

```csharp
while (reader.Read())
{
    // ...
}
```

| Call              | What Happens          | Returns                |
| ----------------- | --------------------- | ---------------------- |
| First `Read()`  | Moves cursor to Row 1 | `true`               |
| Second `Read()` | Moves cursor to Row 2 | `true`               |
| ...               | ...                   | `true`               |
| Last `Read()`   | No more rows          | `false`→ loop exits |

> ⚠️ **You must call `Read()` before accessing any column.** The reader starts BEFORE the first row, not ON the first row.

---

## 🔷 Accessing Column Values — Three Ways

```csharp
while (reader.Read())
{
    // ── Way 1: By column name (most readable) ─────────────────────
    string name   = reader["Name"].ToString();
    decimal salary = Convert.ToDecimal(reader["Salary"]);

    // ── Way 2: By column index (fastest) ──────────────────────────
    int    id   = (int)reader[0];
    string name2 = (string)reader[1];

    // ── Way 3: Typed Get methods (cleanest — no casting) ──────────
    int     id2     = reader.GetInt32(0);       // column 0 → int
    string  name3   = reader.GetString(1);      // column 1 → string
    decimal salary2 = reader.GetDecimal(2);     // column 2 → decimal
    DateTime date   = reader.GetDateTime(3);    // column 3 → DateTime
    bool    active  = reader.GetBoolean(4);     // column 4 → bool
}
```

### Typed Get Methods Reference

| Method                       | C# Type      | Use For                    |
| ---------------------------- | ------------ | -------------------------- |
| `reader.GetInt32(i)`       | `int`      | INT columns                |
| `reader.GetInt64(i)`       | `long`     | BIGINT columns             |
| `reader.GetString(i)`      | `string`   | VARCHAR, NVARCHAR          |
| `reader.GetDecimal(i)`     | `decimal`  | DECIMAL, MONEY             |
| `reader.GetDouble(i)`      | `double`   | FLOAT                      |
| `reader.GetBoolean(i)`     | `bool`     | BIT                        |
| `reader.GetDateTime(i)`    | `DateTime` | DATETIME, DATE             |
| `reader.GetGuid(i)`        | `Guid`     | UNIQUEIDENTIFIER           |
| `reader.GetOrdinal("col")` | `int`      | Get index from column name |

---

## 🔷 Handling NULL Values

```csharp
while (reader.Read())
{
    // ⚠️ Calling .ToString() on a DB NULL throws an exception

    // ── Check for NULL before reading ─────────────────────────────
    string email = reader["Email"] == DBNull.Value
                   ? null
                   : reader["Email"].ToString();

    // ── Using IsDBNull (index-based) ──────────────────────────────
    string phone = reader.IsDBNull(reader.GetOrdinal("Phone"))
                   ? "No phone"
                   : reader.GetString(reader.GetOrdinal("Phone"));

    // ── Null-coalescing shortcut ───────────────────────────────────
    string dept = reader["Department"] as string ?? "Unknown";
}
```

---

## 🔷 Complete Example — Read All Employees

```csharp
public List<Employee> GetAllEmployees()
{
    var employees = new List<Employee>();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();

        string sql = "SELECT Id, Name, Role, Salary, HireDate FROM Employee ORDER BY Name";

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    employees.Add(new Employee
                    {
                        Id       = reader.GetInt32(0),
                        Name     = reader.GetString(1),
                        Role     = reader.GetString(2),
                        Salary   = reader.GetDecimal(3),
                        HireDate = reader.GetDateTime(4)
                    });
                }
            }
        }
    }

    return employees;
}
```

---

## 🔷 Read a Single Row — Use `if` Not `while`

```csharp
public Employee GetEmployeeById(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();

        string sql = "SELECT Id, Name, Role, Salary FROM Employee WHERE Id = @Id";

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                if (reader.Read())   // ← if for single row, not while
                {
                    return new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Salary = reader.GetDecimal(3)
                    };
                }
                return null;   // not found
            }
        }
    }
}
```

---

## 🔷 SqlDataReader Key Properties

| Property                   | Type     | What It Returns                                          |
| -------------------------- | -------- | -------------------------------------------------------- |
| `reader.FieldCount`      | `int`  | Number of columns in the result                          |
| `reader.HasRows`         | `bool` | `true`if query returned at least one row               |
| `reader.IsClosed`        | `bool` | `true`if the reader has been closed                    |
| `reader.RecordsAffected` | `int`  | Rows affected (for non-queries — usually -1 for SELECT) |

```csharp
using (SqlDataReader reader = cmd.ExecuteReader())
{
    if (!reader.HasRows)
    {
        Console.WriteLine("No employees found.");
        return;
    }

    Console.WriteLine($"Columns: {reader.FieldCount}");

    while (reader.Read()) { ... }
}
```

---

## 🔷 ExecuteReader with CommandBehavior

```csharp
// Standard — connection stays open, you manage it
SqlDataReader reader = cmd.ExecuteReader();

// CloseConnection — auto-closes connection when reader closes
SqlDataReader reader2 = cmd.ExecuteReader(CommandBehavior.CloseConnection);
// Useful when returning the reader from a method

// SingleRow — hint to DB: only one row expected (small optimisation)
SqlDataReader reader3 = cmd.ExecuteReader(CommandBehavior.SingleRow);

// SchemaOnly — returns column info only, no rows (for metadata inspection)
SqlDataReader reader4 = cmd.ExecuteReader(CommandBehavior.SchemaOnly);
```

---

## 🔷 SqlDataReader Characteristics

| Characteristic       | Detail                                           |
| -------------------- | ------------------------------------------------ |
| Direction            | Forward-only — cannot go back to a previous row |
| Editing              | Read-only — cannot modify data                  |
| Memory               | Low — one row in memory at a time               |
| Speed                | ⚡ Fastest way to read data in ADO.NET           |
| Connection           | Requires open connection throughout              |
| Multiple result sets | ✅ Supported via `reader.NextResult()`         |

---

## ⚠️ Common Mistakes

| Mistake                              | What Happens                           | Fix                                                                |
| ------------------------------------ | -------------------------------------- | ------------------------------------------------------------------ |
| Not calling `reader.Read()`first   | No data — cursor starts before row 1  | Always call `Read()`before accessing columns                     |
| Accessing column after reader closed | `InvalidOperationException`          | Use `using`or access inside the `while`loop                    |
| Not handling `DBNull`              | `InvalidCastException`on null column | Check `reader.IsDBNull()`or use `as`operator                   |
| Wrong column name spelling           | `IndexOutOfRangeException`           | Match column name exactly — case-insensitive but spelling matters |
| `while`when expecting 1 row        | Reads all rows unnecessarily           | Use `if (reader.Read())`for single-row queries                   |

---

## ⭐ Interview Quick-Fire

| Question                              | Answer                                                             |
| ------------------------------------- | ------------------------------------------------------------------ |
| What does `ExecuteReader()`return?  | `SqlDataReader`                                                  |
| When do you use `ExecuteReader()`?  | SELECT queries returning multiple rows                             |
| What does `reader.Read()`return?    | `true`if row available,`false`at end                           |
| Is SqlDataReader forward-only?        | ✅ Yes — cannot go back                                           |
| Is SqlDataReader read-only?           | ✅ Yes — cannot modify data                                       |
| How to handle NULL columns?           | `reader.IsDBNull(i)`or `reader["col"] == DBNull.Value`         |
| What is `reader.HasRows`?           | `bool`— true if at least one row returned                       |
| Fastest way to read a column?         | Typed Get methods —`reader.GetInt32(0)`,`reader.GetString(1)` |
| Use `if`or `while`for single row? | `if (reader.Read())`— use `if`                                |
| Does reader need open connection?     | ✅ Yes — connection must stay open while reading                  |
