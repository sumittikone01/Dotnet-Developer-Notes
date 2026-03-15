
# 08 — SqlDataReader

---

## 🎯 One-Line Definition

> **`SqlDataReader` is a fast, forward-only, read-only cursor that streams rows from SQL Server one at a time — it's the most efficient way to read large result sets in ADO.NET.**

---

## 🔷 What is SqlDataReader?

`SqlDataReader` is the object that comes back from `ExecuteReader()`.
It sits on top of the database connection and reads rows  **one at a time, in one direction** .

> 💡 Think of it like a **conveyor belt** — items come one at a time, you take each one, you can't go back to get a previous item, and the belt stops when there are no more items.

---

## 🔷 Key Characteristics

| Characteristic       | Detail                                                     |
| -------------------- | ---------------------------------------------------------- |
| Direction            | **Forward-only**— can only move forward, never back |
| Editing              | **Read-only**— cannot modify data                   |
| Memory               | **Low**— one row in memory at a time                |
| Speed                | ⚡**Fastest**read method in ADO.NET                  |
| Connection           | Requires open connection**throughout**reading        |
| Navigation           | `Read()`— moves cursor to next row                      |
| Multiple result sets | ✅ Supported via `NextResult()`                          |

---

## 🔷 Creating a SqlDataReader

`SqlDataReader` is **never created with `new`** — it is returned by `cmd.ExecuteReader()`.

```csharp
// ❌ Cannot do this:
SqlDataReader reader = new SqlDataReader();   // error — no public constructor

// ✅ Always created this way:
SqlDataReader reader = cmd.ExecuteReader();
```

---

## 🔷 The Read Loop — How It Works

```
reader.Read()
    │
    ├── Returns TRUE  → cursor moved to next row → read column values
    │
    └── Returns FALSE → no more rows → loop exits

Cursor position before any Read():
  [Before Row 1] [Row 1] [Row 2] [Row 3] [End]
        ↑
     starts here — must call Read() to move to Row 1
```

```csharp
using (SqlDataReader reader = cmd.ExecuteReader())
{
    while (reader.Read())        // moves to Row 1, Row 2, Row 3...
    {                            // returns false when no more rows
        // access columns here
    }
}
```

---

## 🔷 Reading Column Values — All Methods

### Method 1 — By Column Name (most readable)

```csharp
while (reader.Read())
{
    int     id     = (int)reader["Id"];
    string  name   = reader["Name"].ToString();
    decimal salary = (decimal)reader["Salary"];
    bool    active = (bool)reader["IsActive"];
}
```

### Method 2 — By Column Index (fastest)

```csharp
while (reader.Read())
{
    int     id     = (int)reader[0];      // column 0
    string  name   = (string)reader[1];   // column 1
    decimal salary = (decimal)reader[2];  // column 2
}
```

### Method 3 — Typed Get Methods (cleanest — no casting needed)

```csharp
while (reader.Read())
{
    int      id       = reader.GetInt32(0);
    string   name     = reader.GetString(1);
    decimal  salary   = reader.GetDecimal(2);
    DateTime hireDate = reader.GetDateTime(3);
    bool     isActive = reader.GetBoolean(4);
    double   rating   = reader.GetDouble(5);
    long     bigId    = reader.GetInt64(6);
    Guid     guid     = reader.GetGuid(7);
}
```

### All Typed Get Methods

| Method             | C# Type      | SQL Type                  |
| ------------------ | ------------ | ------------------------- |
| `GetInt32(i)`    | `int`      | INT                       |
| `GetInt64(i)`    | `long`     | BIGINT                    |
| `GetInt16(i)`    | `short`    | SMALLINT                  |
| `GetString(i)`   | `string`   | VARCHAR, NVARCHAR, CHAR   |
| `GetDecimal(i)`  | `decimal`  | DECIMAL, NUMERIC, MONEY   |
| `GetDouble(i)`   | `double`   | FLOAT                     |
| `GetFloat(i)`    | `float`    | REAL                      |
| `GetBoolean(i)`  | `bool`     | BIT                       |
| `GetDateTime(i)` | `DateTime` | DATETIME, DATE, DATETIME2 |
| `GetGuid(i)`     | `Guid`     | UNIQUEIDENTIFIER          |
| `GetByte(i)`     | `byte`     | TINYINT                   |
| `GetChar(i)`     | `char`     | CHAR(1)                   |
| `GetValue(i)`    | `object`   | Any — generic fallback   |

---

## 🔷 Handling NULL Values

```csharp
while (reader.Read())
{
    // ── Method 1: Check DBNull.Value ──────────────────────────────
    string email = reader["Email"] == DBNull.Value
                   ? null
                   : reader["Email"].ToString();

    // ── Method 2: IsDBNull with column index ──────────────────────
    int phoneIdx = reader.GetOrdinal("Phone");  // get index from name
    string phone = reader.IsDBNull(phoneIdx)
                   ? "No phone"
                   : reader.GetString(phoneIdx);

    // ── Method 3: `as` operator — safest for strings ─────────────
    string dept = reader["Department"] as string;   // null if DBNull

    // ── Method 4: Convert — safe for numerics ─────────────────────
    decimal salary = Convert.ToDecimal(reader["Salary"]);
    // Convert handles DBNull by returning 0 for numeric types
}
```

---

## 🔷 SqlDataReader Properties

| Property            | Type     | What It Returns                            |
| ------------------- | -------- | ------------------------------------------ |
| `HasRows`         | `bool` | `true`if query returned at least one row |
| `FieldCount`      | `int`  | Number of columns in the result set        |
| `IsClosed`        | `bool` | `true`if reader has been closed          |
| `RecordsAffected` | `int`  | Rows affected (always -1 for SELECT)       |
| `Depth`           | `int`  | Nesting depth (0 for flat result sets)     |

```csharp
using (SqlDataReader reader = cmd.ExecuteReader())
{
    Console.WriteLine($"Has rows:  {reader.HasRows}");     // true/false
    Console.WriteLine($"Columns:   {reader.FieldCount}");  // 5

    if (!reader.HasRows)
    {
        Console.WriteLine("No results found.");
        return;
    }

    while (reader.Read()) { ... }
}
```

---

## 🔷 Column Metadata — GetName and GetOrdinal

```csharp
// GetOrdinal — get index from column name
int nameIdx   = reader.GetOrdinal("Name");    // e.g. returns 1
int salaryIdx = reader.GetOrdinal("Salary");  // e.g. returns 3

// GetName — get column name from index
string col0 = reader.GetName(0);   // "Id"
string col1 = reader.GetName(1);   // "Name"

// GetDataTypeName — get SQL type name
string type0 = reader.GetDataTypeName(0);   // "int"
string type1 = reader.GetDataTypeName(1);   // "nvarchar"

// GetFieldType — get C# Type object
Type t0 = reader.GetFieldType(0);   // typeof(int)
Type t1 = reader.GetFieldType(1);   // typeof(string)
```

---

## 🔷 Complete Example — Map to List of Objects

```csharp
public List<Employee> GetAllEmployees()
{
    var employees = new List<Employee>();

    string sql = "SELECT Id, Name, Role, Email, Salary, HireDate, IsActive FROM Employee";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    var emp = new Employee
                    {
                        Id       = reader.GetInt32(reader.GetOrdinal("Id")),
                        Name     = reader.GetString(reader.GetOrdinal("Name")),
                        Role     = reader.GetString(reader.GetOrdinal("Role")),
                        Email    = reader["Email"] as string,   // nullable
                        Salary   = reader.GetDecimal(reader.GetOrdinal("Salary")),
                        HireDate = reader.GetDateTime(reader.GetOrdinal("HireDate")),
                        IsActive = reader.GetBoolean(reader.GetOrdinal("IsActive"))
                    };
                    employees.Add(emp);
                }
            }
        }
    }

    return employees;
}
```

---

## 🔷 Multiple Result Sets — NextResult()

A single `SqlCommand` can return multiple SELECT result sets.
`reader.NextResult()` moves to the next one.

```csharp
string sql = @"SELECT Id, Name FROM Employee;
               SELECT Id, Name FROM Department;";

using (SqlCommand cmd = new SqlCommand(sql, con))
{
    using (SqlDataReader reader = cmd.ExecuteReader())
    {
        // ── First result set: Employees ───────────────────────────
        Console.WriteLine("── Employees ──");
        while (reader.Read())
        {
            Console.WriteLine($"{reader["Id"]}  {reader["Name"]}");
        }

        // ── Move to second result set ─────────────────────────────
        reader.NextResult();

        // ── Second result set: Departments ───────────────────────
        Console.WriteLine("── Departments ──");
        while (reader.Read())
        {
            Console.WriteLine($"{reader["Id"]}  {reader["Name"]}");
        }
    }
}
```

---

## 🔷 CommandBehavior — Fine-Tuning ExecuteReader

```csharp
// Default — standard reader, connection managed manually
cmd.ExecuteReader()

// CloseConnection — auto-closes connection when reader is closed
cmd.ExecuteReader(CommandBehavior.CloseConnection)
// Useful when returning the reader outside the method

// SingleRow — hint: only one row expected
cmd.ExecuteReader(CommandBehavior.SingleRow)

// SchemaOnly — returns column structure only, no rows
cmd.ExecuteReader(CommandBehavior.SchemaOnly)

// SequentialAccess — read large binary/text columns in chunks
cmd.ExecuteReader(CommandBehavior.SequentialAccess)
```

---

## 🔷 SqlDataReader vs DataTable — When to Use Which

|                   | SqlDataReader                    | DataTable                      |
| ----------------- | -------------------------------- | ------------------------------ |
| Architecture      | Connected                        | Disconnected                   |
| Connection needed | ✅ Open while reading            | ❌ Closed after fill           |
| Direction         | Forward-only                     | Random access (any row/column) |
| Editing data      | ❌ Read-only                     | ✅ Full edit                   |
| Memory            | Low — one row                   | Higher — all rows             |
| Speed             | ⚡ Faster                        | Slightly slower                |
| Best for          | Streaming large data, live reads | Kendo Grids, reports, editing  |

---

## ⚠️ Common Mistakes

| Mistake                                 | What Happens                           | Fix                                            |
| --------------------------------------- | -------------------------------------- | ---------------------------------------------- |
| Not calling `Read()`first             | No data — cursor starts before row 1  | Always call `Read()`before accessing columns |
| Accessing column after reader closes    | `InvalidOperationException`          | Read values inside the `while`loop           |
| Wrong column name                       | `IndexOutOfRangeException`           | Match name exactly — check with SQL query     |
| Not checking `IsDBNull`               | `InvalidCastException`on null column | Check `reader.IsDBNull(i)`before typed Get   |
| Closing reader while still using values | Object reference lost                  | Map to a class/list inside the loop            |
| Using reader after connection closes    | Reader becomes unusable                | Keep connection open for reader lifetime       |

---

## ⭐ Interview Quick-Fire

| Question                          | Answer                                                                |
| --------------------------------- | --------------------------------------------------------------------- |
| What is SqlDataReader?            | Fast, forward-only, read-only cursor for reading rows from SQL Server |
| How is SqlDataReader created?     | Returned by `cmd.ExecuteReader()`— never with `new`              |
| Is it forward-only?               | ✅ Yes — cannot go back to previous rows                             |
| Is it read-only?                  | ✅ Yes — cannot modify data                                          |
| Method to advance to next row?    | `reader.Read()`— returns true/false                                |
| Where does cursor start?          | Before the first row — must call `Read()`to move to Row 1          |
| How to check for NULL column?     | `reader.IsDBNull(i)`or `reader["col"] == DBNull.Value`            |
| What is `reader.HasRows`?       | `bool`— true if at least one row returned                          |
| What is `reader.FieldCount`?    | `int`— number of columns in the result                             |
| How to read multiple result sets? | `reader.NextResult()`to advance to the next SELECT                  |
| Does it need open connection?     | ✅ Yes — connection must stay open throughout reading                |
