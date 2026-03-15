
# 03 — CancellationToken in ADO.NET

---

## 🎯 One-Line Definition

> **`CancellationToken` lets you cancel a running async database operation — when the user navigates away, the request times out, or the server shuts down, you stop the query immediately instead of letting it run and waste resources.**

---

## 🔷 The Problem CancellationToken Solves

```
WITHOUT CancellationToken:
──────────────────────────────────────────────────────────────
User opens a report page that runs a heavy SQL query (5 seconds)
User immediately clicks Back / navigates away
HTTP request cancelled by browser

BUT:
  The database query is STILL RUNNING on the server
  Thread is STILL WAITING for the result
  CPU and DB resources are STILL BEING CONSUMED
  Memory will hold the results that nobody needs

Result: wasted resources on every cancelled request

WITH CancellationToken:
──────────────────────────────────────────────────────────────
User clicks Back → browser cancels HTTP request
ASP.NET Core detects cancellation → triggers CancellationToken
ADO.NET receives the token → cancels the query mid-execution
SQL Server stops processing → resources freed immediately
```

---

## 🔷 What is CancellationToken?

```
CancellationTokenSource  →  creates and controls the token
       │
       │  .Token property
       ▼
CancellationToken        →  passed to async methods
                            (read-only — can only listen, not cancel)
       │
       │  when .Cancel() called on Source
       ▼
OperationCanceledException → thrown from the awaited method
```

---

## 🔷 CancellationToken in All Async ADO.NET Methods

Every async ADO.NET method accepts an optional `CancellationToken`:

```csharp
await con.OpenAsync(cancellationToken);
await cmd.ExecuteReaderAsync(cancellationToken);
await cmd.ExecuteNonQueryAsync(cancellationToken);
await cmd.ExecuteScalarAsync(cancellationToken);
await reader.ReadAsync(cancellationToken);
await reader.NextResultAsync(cancellationToken);
```

---

## 🔷 Pattern 1 — Manual CancellationToken (Console / Tests)

```csharp
public async Task RunWithTimeoutAsync()
{
    // Create a token that auto-cancels after 5 seconds
    using CancellationTokenSource cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    CancellationToken token = cts.Token;

    try
    {
        await GetAllEmployeesAsync(token);
        Console.WriteLine("✅ Query completed.");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("⏱️ Query cancelled — took too long.");
    }
}

public async Task<List<Employee>> GetAllEmployeesAsync(CancellationToken token = default)
{
    var employees = new List<Employee>();
    string sql = "SELECT Id, Name, Role, Salary FROM Employee";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        await con.OpenAsync(token);   // ← pass token here

        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            using (SqlDataReader reader = await cmd.ExecuteReaderAsync(token))
            {
                while (await reader.ReadAsync(token))   // ← and here
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

## 🔷 Pattern 2 — ASP.NET Core `HttpContext.RequestAborted` (Real-World)

In ASP.NET Core, every request has a built-in `CancellationToken` — `HttpContext.RequestAborted`.
It fires automatically when the client disconnects or navigates away.

```csharp
// Controller — pass the request token to all async operations
public class EmployeeController : Controller
{
    private readonly EmployeeRepository _repo;

    public EmployeeController(EmployeeRepository repo) => _repo = repo;

    public async Task<IActionResult> Index()
    {
        // HttpContext.RequestAborted cancels if user navigates away
        var employees = await _repo.GetAllAsync(HttpContext.RequestAborted);
        return View(employees);
    }

    public async Task<IActionResult> Report()
    {
        var data = await _repo.GetReportDataAsync(HttpContext.RequestAborted);
        return View(data);
    }
}
```

```csharp
// Repository — accept the token, pass it to every async operation
public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    public async Task<List<Employee>> GetAllAsync(CancellationToken token = default)
    {
        var employees = new List<Employee>();

        using (SqlConnection con = new SqlConnection(_cs))
        {
            await con.OpenAsync(token);

            using (SqlCommand cmd = new SqlCommand(
                "SELECT Id, Name, Role, Salary FROM Employee", con))
            {
                using (SqlDataReader reader = await cmd.ExecuteReaderAsync(token))
                {
                    while (await reader.ReadAsync(token))
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

    public async Task<int> InsertAsync(Employee emp, CancellationToken token = default)
    {
        string sql = @"INSERT INTO Employee (Name, Role, Salary)
                       VALUES (@Name, @Role, @Salary)";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            await con.OpenAsync(token);

            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Name",   emp.Name);
                cmd.Parameters.AddWithValue("@Role",   emp.Role);
                cmd.Parameters.AddWithValue("@Salary", emp.Salary);

                return await cmd.ExecuteNonQueryAsync(token);
            }
        }
    }
}
```

---

## 🔷 Pattern 3 — Manual Cancel from Code

```csharp
public async Task ManualCancelDemoAsync()
{
    CancellationTokenSource cts = new CancellationTokenSource();

    // Cancel after 3 seconds
    _ = Task.Run(async () =>
    {
        await Task.Delay(3000);
        cts.Cancel();
        Console.WriteLine("⏱️ Cancellation triggered.");
    });

    try
    {
        await RunLongQueryAsync(cts.Token);
        Console.WriteLine("✅ Query completed.");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("❌ Query was cancelled.");
    }
    finally
    {
        cts.Dispose();
    }
}
```

---

## 🔷 Handling OperationCanceledException

```csharp
public async Task<List<Employee>> GetAllAsync(CancellationToken token = default)
{
    var employees = new List<Employee>();

    try
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            await con.OpenAsync(token);

            using (SqlCommand cmd = new SqlCommand("SELECT * FROM Employee", con))
            using (SqlDataReader reader = await cmd.ExecuteReaderAsync(token))
            {
                while (await reader.ReadAsync(token))
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
    catch (OperationCanceledException)
    {
        // Request was cancelled — normal in web apps when user navigates away
        // No need to log this as an error — it's expected behaviour
        Console.WriteLine("ℹ️ Request was cancelled by client.");
        // Return empty list — caller will discard the response anyway
    }
    catch (SqlException ex)
    {
        Console.Error.WriteLine($"Database error: {ex.Message}");
        throw;
    }

    return employees;
}
```

---

## 🔷 CancellationToken with Timeout — Three Ways

```csharp
// Way 1: Timeout on the token source (cancels after duration)
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
await GetAllAsync(cts.Token);

// Way 2: Timeout on the SqlCommand (SQL Server stops the query)
cmd.CommandTimeout = 10;   // seconds — set on the command object

// Way 3: Combine both (belt and suspenders)
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
cmd.CommandTimeout = 10;
await cmd.ExecuteReaderAsync(cts.Token);
```

### Difference: CommandTimeout vs CancellationToken

|             | `CommandTimeout`         | `CancellationToken`            |
| ----------- | -------------------------- | -------------------------------- |
| Set on      | `SqlCommand`object       | Passed to async method           |
| Who cancels | SQL Server stops the query | ADO.NET / .NET runtime           |
| Triggers    | `SqlException`           | `OperationCanceledException`   |
| Scope       | This one command           | Entire request chain             |
| Best for    | Slow query timeout         | User cancellation, request abort |

---

## 🔷 `token = default` — The Best Practice

Always use `CancellationToken token = default` as the parameter default:

```csharp
// ✅ Best practice — token is optional
public async Task<List<Employee>> GetAllAsync(CancellationToken token = default)
{
    // default(CancellationToken) = CancellationToken.None = never cancels
    // So calling without a token is safe:
    var list = await _repo.GetAllAsync();          // works — uses CancellationToken.None
    var list2 = await _repo.GetAllAsync(myToken);  // works — uses the token
}
```

---

## 🔷 Complete Repository with CancellationToken

```csharp
public class EmployeeRepository
{
    private readonly string _cs;

    public EmployeeRepository(IConfiguration config)
        => _cs = config.GetConnectionString("DefaultConnection");

    public async Task<List<Employee>> GetAllAsync(CancellationToken token = default)
    {
        var list = new List<Employee>();

        using var con = new SqlConnection(_cs);
        await con.OpenAsync(token);

        using var cmd = new SqlCommand("SELECT Id, Name, Role, Salary FROM Employee", con);
        using var reader = await cmd.ExecuteReaderAsync(token);

        while (await reader.ReadAsync(token))
        {
            list.Add(new Employee
            {
                Id     = reader.GetInt32(0),
                Name   = reader.GetString(1),
                Role   = reader.GetString(2),
                Salary = reader.GetDecimal(3)
            });
        }

        return list;
    }

    public async Task<Employee> GetByIdAsync(int id, CancellationToken token = default)
    {
        using var con = new SqlConnection(_cs);
        await con.OpenAsync(token);

        using var cmd = new SqlCommand(
            "SELECT Id, Name, Role, Salary FROM Employee WHERE Id = @Id", con);
        cmd.Parameters.AddWithValue("@Id", id);

        using var reader = await cmd.ExecuteReaderAsync(token);
        if (await reader.ReadAsync(token))
        {
            return new Employee
            {
                Id = reader.GetInt32(0), Name = reader.GetString(1),
                Role = reader.GetString(2), Salary = reader.GetDecimal(3)
            };
        }
        return null;
    }

    public async Task<int> InsertAsync(Employee emp, CancellationToken token = default)
    {
        using var con = new SqlConnection(_cs);
        await con.OpenAsync(token);

        using var cmd = new SqlCommand(
            "INSERT INTO Employee(Name,Role,Salary) VALUES(@Name,@Role,@Salary)", con);
        cmd.Parameters.AddWithValue("@Name",   emp.Name);
        cmd.Parameters.AddWithValue("@Role",   emp.Role);
        cmd.Parameters.AddWithValue("@Salary", emp.Salary);

        return await cmd.ExecuteNonQueryAsync(token);
    }

    public async Task<int> DeleteAsync(int id, CancellationToken token = default)
    {
        using var con = new SqlConnection(_cs);
        await con.OpenAsync(token);

        using var cmd = new SqlCommand("DELETE FROM Employee WHERE Id=@Id", con);
        cmd.Parameters.AddWithValue("@Id", id);

        return await cmd.ExecuteNonQueryAsync(token);
    }
}
```

---

## 🔷 Where CancellationToken Flows in ASP.NET Core

```
Browser sends request
    ↓
ASP.NET Core creates HttpContext
    ↓
HttpContext.RequestAborted = CancellationToken (starts not-cancelled)
    ↓
Controller receives request
    ↓
Passes HttpContext.RequestAborted to Repository
    ↓
Repository passes to all async ADO.NET calls
    ↓
                    ← User navigates away / browser cancels
    ↓
HttpContext.RequestAborted triggers → IsCancellationRequested = true
    ↓
Next awaited operation sees the token → throws OperationCanceledException
    ↓
ADO.NET cancels the query → SQL Server stops the query
    ↓
Resources freed immediately ✅
```

---

## ⚠️ Common Mistakes

| Mistake                                                      | What Happens                                           | Fix                                         |
| ------------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------- |
| Passing token to `OpenAsync`but not `ExecuteReaderAsync` | Connection open cancelled but query still runs         | Pass token to EVERY async call in the chain |
| Catching `OperationCanceledException`and logging as error  | Noise in logs — this is expected on client disconnect | Catch separately, log as Info not Error     |
| Using `cts.Cancel()`after `cts.Dispose()`                | `ObjectDisposedException`                            | Cancel before disposing                     |
| Not using `token = default`                                | Callers must always pass a token                       | `default`makes token optional             |

---

## ⭐ Interview Quick-Fire

| Question                                               | Answer                                                                                                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| What is CancellationToken?                             | A signal that tells async operations to stop — passed through the call chain                                                              |
| In ASP.NET Core, where does the token come from?       | `HttpContext.RequestAborted`— fires when client disconnects                                                                             |
| Which exception is thrown on cancellation?             | `OperationCanceledException`                                                                                                             |
| Difference:`CommandTimeout`vs `CancellationToken`? | `CommandTimeout`= SQL Server stops the query (SqlException). CancellationToken = .NET cancels the operation (OperationCanceledException) |
| What is `CancellationToken.None`?                    | A token that never cancels — safe default when no token is needed                                                                         |
| Why use `token = default`as parameter default?       | Makes the token optional — caller passes it when available, not required always                                                           |
| What does `CancellationTokenSource`do?               | Creates and controls the token — calling `.Cancel()`signals all listeners                                                               |
