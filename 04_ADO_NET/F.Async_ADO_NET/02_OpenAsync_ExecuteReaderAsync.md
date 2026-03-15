
# 02 — OpenAsync & ExecuteReaderAsync

---

## 🎯 One-Line Definition

> **`OpenAsync()` opens a database connection without blocking the thread — `ExecuteReaderAsync()` fires the SELECT query without blocking — together they make the two most common async operations in ADO.NET.**

---

## 🔷 Why These Two Methods Matter Most

```
Every database read goes through these two steps:

  Step 1: Open connection   →  con.Open()          (blocks)
                            →  con.OpenAsync()      (non-blocking ✅)

  Step 2: Execute query     →  cmd.ExecuteReader()  (blocks)
                            →  cmd.ExecuteReaderAsync() (non-blocking ✅)

These are the two biggest blocking points in ADO.NET reads.
Making both async = maximum scalability.
```

---

## 🔷 OpenAsync() — Async Connection Open

### What it does

`OpenAsync()` initiates the connection to SQL Server and **immediately returns control** to the caller.
The thread is free to do other work while TCP handshake, authentication, and pool checkout happen.

### Syntax

```csharp
// Synchronous — thread blocked during open
con.Open();

// Asynchronous — thread free while connecting
await con.OpenAsync();
```

### Connection Pooling with OpenAsync

```
With Connection Pooling (default):
  OpenAsync() checks the pool first.
  If a connection is available → reuses it instantly (microseconds).
  If pool is empty → creates new TCP connection (milliseconds) — this is where async saves time.

Async benefit is MOST noticeable when pool is empty and a real network connection must be made.
For pooled connections, the benefit is smaller but still present.
```

### Full Open Example

```csharp
public async Task<bool> TestConnectionAsync()
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        try
        {
            await con.OpenAsync();

            Console.WriteLine($"Connected to: {con.Database}");
            Console.WriteLine($"Server:       {con.DataSource}");
            Console.WriteLine($"Version:      {con.ServerVersion}");
            Console.WriteLine($"State:        {con.State}");   // Open

            return true;
        }
        catch (SqlException ex)
        {
            Console.Error.WriteLine($"Connection failed: {ex.Message}");
            return false;
        }
    }
}
```

---

## 🔷 ExecuteReaderAsync() — Async Query Execution

### What it does

`ExecuteReaderAsync()` sends the SQL query to SQL Server and  **immediately returns control** .
The thread waits for the query result without blocking — it's freed to handle other requests.

### Syntax

```csharp
// Synchronous — thread blocked while query executes
SqlDataReader reader = cmd.ExecuteReader();

// Asynchronous — thread free while query executes
SqlDataReader reader = await cmd.ExecuteReaderAsync();
```

### With CommandBehavior

```csharp
// CloseConnection — auto-closes connection when reader closes
SqlDataReader reader = await cmd.ExecuteReaderAsync(CommandBehavior.CloseConnection);

// SingleRow — hint to DB: only one row expected
SqlDataReader reader = await cmd.ExecuteReaderAsync(CommandBehavior.SingleRow);

// SequentialAccess — read large binary/text columns in order
SqlDataReader reader = await cmd.ExecuteReaderAsync(CommandBehavior.SequentialAccess);
```

---

## 🔷 ReadAsync() — Async Row Reading

`reader.ReadAsync()` advances to the next row without blocking:

```csharp
// Sync — blocks thread on each row advance
while (reader.Read()) { ... }

// Async — thread free between row reads
while (await reader.ReadAsync()) { ... }
```

> For small result sets (< 100 rows), the difference is negligible.
> For large result sets streaming thousands of rows, `ReadAsync()` makes a real difference.

---

## 🔷 Complete Async Read Pattern

```csharp
public async Task<List<Employee>> GetAllAsync()
{
    var employees = new List<Employee>();
    string sql = "SELECT Id, Name, Role, Email, Salary, HireDate FROM Employee";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();   // ← async open

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            using (SqlDataReader reader = await cmd.ExecuteReaderAsync())   // ← async execute
            {
                while (await reader.ReadAsync())   // ← async row advance
                {
                    employees.Add(new Employee
                    {
                        Id       = reader.GetInt32(reader.GetOrdinal("Id")),
                        Name     = reader.GetString(reader.GetOrdinal("Name")),
                        Role     = reader.GetString(reader.GetOrdinal("Role")),
                        Email    = reader["Email"] as string,
                        Salary   = reader.GetDecimal(reader.GetOrdinal("Salary")),
                        HireDate = reader.GetDateTime(reader.GetOrdinal("HireDate"))
                    });
                }
            }
        }
    }

    return employees;
}
```

---

## 🔷 Async GetById — Single Row

```csharp
public async Task<Employee> GetByIdAsync(int id)
{
    string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee WHERE Id = @Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
            {
                if (await reader.ReadAsync())   // ← if for single row
                {
                    return new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Email  = reader["Email"] as string,
                        Salary = reader.GetDecimal(4)
                    };
                }
                return null;
            }
        }
    }
}
```

---

## 🔷 Async with Multiple Result Sets

```csharp
public async Task<(List<Employee> employees, List<Department> departments)>
    GetFormDataAsync()
{
    string sql = "SELECT Id, Name FROM Employee; SELECT Id, Name FROM Department;";

    var employees   = new List<Employee>();
    var departments = new List<Department>();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand(sql, con))
        using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
        {
            // First result set
            while (await reader.ReadAsync())
            {
                employees.Add(new Employee
                {
                    Id   = reader.GetInt32(0),
                    Name = reader.GetString(1)
                });
            }

            // Move to second result set
            await reader.NextResultAsync();   // ← async NextResult

            // Second result set
            while (await reader.ReadAsync())
            {
                departments.Add(new Department
                {
                    Id   = reader.GetInt32(0),
                    Name = reader.GetString(1)
                });
            }
        }
    }

    return (employees, departments);
}
```

---

## 🔷 Async with Stored Procedure

```csharp
public async Task<List<Employee>> GetByDeptAsync(string dept)
{
    var employees = new List<Employee>();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand("sp_GetEmployeesByDept", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Dept", dept);

            using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    employees.Add(new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Salary = reader.GetDecimal(2)
                    });
                }
            }
        }
    }

    return employees;
}
```

---

## 🔷 All Async Methods at a Glance

| Method                        | Returns           | Async Pair                     | Returns                 |
| ----------------------------- | ----------------- | ------------------------------ | ----------------------- |
| `con.Open()`                | `void`          | `con.OpenAsync()`            | `Task`                |
| `cmd.ExecuteReader()`       | `SqlDataReader` | `cmd.ExecuteReaderAsync()`   | `Task<SqlDataReader>` |
| `cmd.ExecuteNonQuery()`     | `int`           | `cmd.ExecuteNonQueryAsync()` | `Task<int>`           |
| `cmd.ExecuteScalar()`       | `object`        | `cmd.ExecuteScalarAsync()`   | `Task<object?>`       |
| `reader.Read()`             | `bool`          | `reader.ReadAsync()`         | `Task<bool>`          |
| `reader.NextResult()`       | `bool`          | `reader.NextResultAsync()`   | `Task<bool>`          |
| `con.Close()`/`Dispose()` | `void`          | `con.CloseAsync()`(.NET 5+)  | `ValueTask`           |

---

## 🔷 The Thread Timeline — Visualised

```
WITHOUT async:
Thread 1: [Handle request]─[OpenDB]─────[WAIT DB]─────[ReadRows]─[Respond]
                                          ↑ blocked here

WITH async:
Thread 1: [Handle request]─[OpenDB]─╴[FREE]╶─[ReadRows]─[Respond]
                                    ↑ frees during DB wait
                                    ↑ handles other requests here
Thread 1: [Handle request 2]─[OpenDB]─╴[FREE]╶─[ReadRows]─[Respond]
Thread 1: [Handle request 3]─[OpenDB]─╴[FREE]╶─[ReadRows]─[Respond]
```

---

## ⚠️ Common Mistakes

| Mistake                                               | What Happens                                | Fix                                                   |
| ----------------------------------------------------- | ------------------------------------------- | ----------------------------------------------------- |
| `con.OpenAsync()`but `ExecuteReader()`(not async) | Thread freed on open but blocked on execute | Make ALL steps async consistently                     |
| `await reader.ReadAsync()`inside sync method        | Compile error                               | Mark the method `async`                             |
| Disposing reader before reading all rows              | `ObjectDisposedException`                 | Use `using`— reader disposed only when block exits |
| Calling `.Result`on `OpenAsync()`task             | Deadlock in ASP.NET                         | Always use `await`                                  |

---

## ⭐ Interview Quick-Fire

| Question                                                        | Answer                                                                |
| --------------------------------------------------------------- | --------------------------------------------------------------------- |
| What does `OpenAsync()`do differently from `Open()`?        | Frees the thread while the connection is being established            |
| What does `ExecuteReaderAsync()`return?                       | `Task<SqlDataReader>`— use `await`to get the `SqlDataReader`   |
| Should you use `ReadAsync()`for small result sets?            | Yes — it's good practice even for small sets; overhead is negligible |
| What is `NextResultAsync()`?                                  | Async version of `NextResult()`— moves to next result set          |
| Which three methods need to be async for a full async read?     | `OpenAsync()`,`ExecuteReaderAsync()`,`ReadAsync()`              |
| Can you mix sync `Open()`with async `ExecuteReaderAsync()`? | ✅ Works but defeats the purpose — make the whole chain async        |
