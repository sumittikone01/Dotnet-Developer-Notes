
# 01 — Async/Await with ADO.NET

---

## 🎯 One-Line Definition

> **Async/Await in ADO.NET lets your application send a database query and immediately free the thread to handle other requests — the thread resumes only when the database responds, making your app far more scalable.**

---

## 🔷 Why Async Matters for Database Code

```
SYNCHRONOUS (blocking):
──────────────────────────────────────────────────────────────
Request 1: Thread 1 → Open → Execute SQL → [WAITING 200ms] → Read → Respond
Request 2: Thread 2 → Open → Execute SQL → [WAITING 200ms] → Read → Respond
Request 3: Thread 3 → Open → Execute SQL → [WAITING 200ms] → Read → Respond

Each thread is BLOCKED while waiting for the database.
100 concurrent requests = 100 blocked threads = server under strain.

ASYNCHRONOUS (non-blocking):
──────────────────────────────────────────────────────────────
Request 1: Thread 1 → OpenAsync → [database working] → Thread 1 FREE to handle other requests
Request 2: Thread 1 → OpenAsync → [database working] → Thread 1 FREE again
Request 3: Thread 1 → OpenAsync → [database working] → Thread 1 FREE again
                         ↓ when DB responds ↓
             Thread resumes, reads data, responds

ONE thread handles many requests → same server handles far more load.
```

---

## 🔷 Sync vs Async — Every Method Has a Pair

| Synchronous               | Async Equivalent               | Returns                 |
| ------------------------- | ------------------------------ | ----------------------- |
| `con.Open()`            | `con.OpenAsync()`            | `Task`                |
| `cmd.ExecuteReader()`   | `cmd.ExecuteReaderAsync()`   | `Task<SqlDataReader>` |
| `cmd.ExecuteNonQuery()` | `cmd.ExecuteNonQueryAsync()` | `Task<int>`           |
| `cmd.ExecuteScalar()`   | `cmd.ExecuteScalarAsync()`   | `Task<object>`        |
| `reader.Read()`         | `reader.ReadAsync()`         | `Task<bool>`          |
| `reader.NextResult()`   | `reader.NextResultAsync()`   | `Task<bool>`          |

---

## 🔷 The async/await Pattern — Rules

```csharp
// Rule 1: Method must be marked async
public async Task<List<Employee>> GetAllAsync()

// Rule 2: Return type wraps in Task<T>
// void   → Task
// int    → Task<int>
// List<T>→ Task<List<T>>

// Rule 3: await before every async operation
await con.OpenAsync();
await cmd.ExecuteNonQueryAsync();

// Rule 4: await unwraps the Task — gives you the result directly
int rows   = await cmd.ExecuteNonQueryAsync();    // int, not Task<int>
var reader = await cmd.ExecuteReaderAsync();       // SqlDataReader, not Task<SqlDataReader>
bool next  = await reader.ReadAsync();             // bool, not Task<bool>
```

---

## 🔷 Basic Async Pattern — SELECT (Read)

```csharp
public async Task<List<Employee>> GetAllAsync()
{
    var employees = new List<Employee>();
    string sql = "SELECT Id, Name, Role, Salary FROM Employee ORDER BY Name";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();   // ← async open — thread not blocked

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())   // ← async read
                {
                    employees.Add(new Employee
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Salary = reader.GetDecimal(3)
                    });
                }
            }
        }
    }

    return employees;
}
```

---

## 🔷 Basic Async Pattern — INSERT

```csharp
public async Task<int> InsertAsync(Employee emp)
{
    string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                   VALUES (@Name, @Role, @Email, @Salary)";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.Add("@Name",   SqlDbType.NVarChar, 100).Value = emp.Name;
            cmd.Parameters.Add("@Role",   SqlDbType.NVarChar, 50).Value  = emp.Role;
            cmd.Parameters.Add("@Email",  SqlDbType.NVarChar, 150).Value =
                                (object)emp.Email ?? DBNull.Value;
            cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value       = emp.Salary;

            return await cmd.ExecuteNonQueryAsync();
        }
    }
}
```

---

## 🔷 Basic Async Pattern — UPDATE

```csharp
public async Task<int> UpdateAsync(Employee emp)
{
    string sql = @"UPDATE Employee
                   SET Name=@Name, Role=@Role, Salary=@Salary
                   WHERE Id=@Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Name",   emp.Name);
            cmd.Parameters.AddWithValue("@Role",   emp.Role);
            cmd.Parameters.AddWithValue("@Salary", emp.Salary);
            cmd.Parameters.AddWithValue("@Id",     emp.Id);

            return await cmd.ExecuteNonQueryAsync();
        }
    }
}
```

---

## 🔷 Basic Async Pattern — DELETE

```csharp
public async Task<int> DeleteAsync(int id)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand(
            "DELETE FROM Employee WHERE Id = @Id", con))
        {
            cmd.Parameters.AddWithValue("@Id", id);
            return await cmd.ExecuteNonQueryAsync();
        }
    }
}
```

---

## 🔷 Basic Async Pattern — COUNT (ExecuteScalar)

```csharp
public async Task<int> GetCountAsync()
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync();

        using (SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Employee", con))
        {
            object result = await cmd.ExecuteScalarAsync();
            return Convert.ToInt32(result);
        }
    }
}
```

---

## 🔷 Async in ASP.NET Core Controller

```csharp
public class EmployeeController : Controller
{
    private readonly EmployeeRepository _repo;

    public EmployeeController(EmployeeRepository repo) => _repo = repo;

    // ── GET: /Employee ────────────────────────────────────────────
    public async Task<IActionResult> Index()
    {
        var employees = await _repo.GetAllAsync();
        return View(employees);
    }

    // ── POST: /Employee/Create ────────────────────────────────────
    [HttpPost]
    public async Task<IActionResult> Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);

        await _repo.InsertAsync(emp);
        return RedirectToAction(nameof(Index));
    }

    // ── POST: /Employee/Delete/5 ──────────────────────────────────
    [HttpPost]
    public async Task<IActionResult> Delete(int id)
    {
        await _repo.DeleteAsync(id);
        return RedirectToAction(nameof(Index));
    }
}
```

---

## 🔷 Sync vs Async — What Changes

```csharp
// SYNC version                      // ASYNC version
public Employee GetById(int id)      public async Task<Employee> GetByIdAsync(int id)
{                                    {
    using (var con = new ...)           using (var con = new ...)
    {                                   {
        con.Open();              →          await con.OpenAsync();
        using (var cmd = ...)               using (var cmd = ...)
        {                                   {
            var reader =                        var reader =
              cmd.ExecuteReader(); →              await cmd.ExecuteReaderAsync();
            if (reader.Read())   →              if (await reader.ReadAsync())
            {                                   {
                return MapEmployee(reader);         return MapEmployee(reader);
            }                                   }
            return null;                        return null;
        }                                   }
    }                                   }
}                                    }

Changes: 3 lines changed → add async/await/Task
Everything else stays identical.
```

---

## 🔷 Exception Handling in Async

```csharp
public async Task<List<Employee>> GetAllAsync()
{
    var employees = new List<Employee>();

    try
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            await con.OpenAsync();

            using (SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con))
            using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
            {
                while (await reader.ReadAsync())
                {
                    employees.Add(new Employee
                    {
                        Id   = reader.GetInt32(0),
                        Name = reader.GetString(1)
                    });
                }
            }
        }
    }
    catch (SqlException ex)
    {
        Console.Error.WriteLine($"Database error: {ex.Message}");
        throw;   // re-throw so the caller knows something went wrong
    }

    return employees;
}
```

---

## 🔷 Sync vs Async — Key Differences

|                      | Synchronous                     | Asynchronous                      |
| -------------------- | ------------------------------- | --------------------------------- |
| Thread while waiting | ❌ Blocked — can't do anything | ✅ Free — handles other requests |
| Scalability          | Lower                           | Higher                            |
| Code complexity      | Simpler                         | Slightly more syntax              |
| Return type          | `T`                           | `Task<T>`                       |
| Method keyword       | (none)                          | `async`                         |
| Method call          | `method()`                    | `await method()`                |
| Best for             | Simple scripts                  | ASP.NET Core web apps             |

---

## ⚠️ Common Mistakes

| Mistake                                       | What Happens                                        | Fix                                                            |
| --------------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------- |
| Calling async method without `await`        | Returns a `Task`, not the result — silent bug    | Always `await`async calls                                    |
| `async void`on non-event methods            | Exceptions can't be caught by caller                | Use `async Task`— never `async void`except event handlers |
| Not marking method `async`                  | Compiler error —`await`used in non-async context | Add `async`to the method signature                           |
| Blocking async with `.Result`or `.Wait()` | Deadlock in ASP.NET — app hangs forever            | Always `await`, never `.Result`or `.Wait()`              |
| Ignoring the returned Task                    | Fire-and-forget — errors silently lost             | `await`the call or explicitly handle the Task                |

---

## ⭐ Interview Quick-Fire

| Question                                          | Answer                                                                        |
| ------------------------------------------------- | ----------------------------------------------------------------------------- |
| Why use async in ADO.NET?                         | Frees the thread while waiting for the DB — handles more concurrent requests |
| Async version of `con.Open()`?                  | `await con.OpenAsync()`                                                     |
| Async version of `cmd.ExecuteNonQuery()`?       | `await cmd.ExecuteNonQueryAsync()`— returns `int`                        |
| Async version of `reader.Read()`?               | `await reader.ReadAsync()`— returns `bool`                               |
| Return type of async method that returns `int`? | `Task<int>`                                                                 |
| Return type of async void method?                 | Use `Task`—`async void`only for event handlers                           |
| What does `await`do?                            | Suspends the method until the awaited Task completes, freeing the thread      |
| Deadlock risk?                                    | `.Result`/`.Wait()`on async in ASP.NET — always use `await`            |
