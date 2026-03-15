
# 04 — Exception Handling in ADO.NET

---

## 🎯 One-Line Definition

> **`SqlException` is the specific exception ADO.NET throws for all database errors — its `Number` property identifies the exact SQL Server error code, letting you handle connection failures, constraint violations, and timeouts differently.**

---

## 🔷 Why ADO.NET Needs Its Own Exception Handling

```
General code exceptions:
  DivideByZeroException, NullReferenceException...
  → Caught with catch (Exception ex)

ADO.NET database exceptions are DIFFERENT:
  → Wrong connection string → SqlException
  → SQL syntax error       → SqlException
  → Table doesn't exist    → SqlException
  → Constraint violated    → SqlException
  → Query timeout          → SqlException (Number = -2)
  → Duplicate key          → SqlException (Number = 2627)
  → Deadlock               → SqlException (Number = 1205)

All the same type — but SqlException.Number tells you WHICH error.
```

---

## 🔷 The SqlException Class

```csharp
catch (SqlException ex)
{
    Console.WriteLine(ex.Number);       // SQL Server error number (most important)
    Console.WriteLine(ex.Message);      // Human-readable error message
    Console.WriteLine(ex.Severity);     // 0-25 — higher = more severe
    Console.WriteLine(ex.State);        // Additional state code
    Console.WriteLine(ex.Procedure);    // SP name if error from stored procedure
    Console.WriteLine(ex.LineNumber);   // Line number in the SQL where error occurred
    Console.WriteLine(ex.Server);       // SQL Server instance name
    Console.WriteLine(ex.Source);       // Provider source ("Core .Net SqlClient...")
}
```

---

## 🔷 Most Important SqlException Error Numbers

| Number    | Error                       | What Happened                             |
| --------- | --------------------------- | ----------------------------------------- |
| `-2`    | Timeout                     | Query took longer than `CommandTimeout` |
| `4060`  | Database not found          | Wrong database name in connection string  |
| `18456` | Login failed                | Wrong username or password                |
| `2`     | Server not found            | Wrong server name or server offline       |
| `2627`  | Primary key violation       | Inserting duplicate primary key value     |
| `2601`  | Unique constraint violation | Inserting duplicate unique column value   |
| `547`   | Foreign key violation       | Inserting/deleting breaks FK constraint   |
| `1205`  | Deadlock                    | Two transactions blocking each other      |
| `208`   | Table not found             | `SELECT * FROM NonExistent`             |
| `207`   | Invalid column name         | Column doesn't exist in the table         |

---

## 🔷 Basic Exception Handling Pattern

```csharp
public int Insert(Employee emp)
{
    string sql = "INSERT INTO Employee(Name, Role, Salary) VALUES(@N, @R, @S)";

    try
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@N", emp.Name);
                cmd.Parameters.AddWithValue("@R", emp.Role);
                cmd.Parameters.AddWithValue("@S", emp.Salary);

                return cmd.ExecuteNonQuery();
            }
        }
    }
    catch (SqlException ex)
    {
        // SQL Server-specific error — check the number
        Console.Error.WriteLine($"SQL Error {ex.Number}: {ex.Message}");
        throw;   // re-throw so the caller (BAL) knows it failed
    }
    catch (Exception ex)
    {
        // Any other unexpected error
        Console.Error.WriteLine($"Unexpected error: {ex.Message}");
        throw;
    }
}
```

---

## 🔷 Handling Specific Errors by Number

```csharp
public int Insert(Employee emp)
{
    try
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(
                "INSERT INTO Employee(Name, Email, Salary) VALUES(@N, @E, @S)", con))
            {
                cmd.Parameters.AddWithValue("@N", emp.Name);
                cmd.Parameters.AddWithValue("@E", emp.Email);
                cmd.Parameters.AddWithValue("@S", emp.Salary);
                return cmd.ExecuteNonQuery();
            }
        }
    }
    catch (SqlException ex) when (ex.Number == 2627 || ex.Number == 2601)
    {
        // Duplicate key or unique constraint — business-level message
        throw new InvalidOperationException("An employee with this email already exists.", ex);
    }
    catch (SqlException ex) when (ex.Number == -2)
    {
        // Timeout — query took too long
        throw new TimeoutException("The insert operation timed out. Please try again.", ex);
    }
    catch (SqlException ex) when (ex.Number == 547)
    {
        // Foreign key violation
        throw new InvalidOperationException("Invalid department or manager reference.", ex);
    }
    catch (SqlException ex) when (ex.Number == 18456 || ex.Number == 4060)
    {
        // Connection / auth issue — don't expose details to the user
        Console.Error.WriteLine($"DB Connection failed: {ex.Message}");
        throw new Exception("Database connection error. Contact administrator.", ex);
    }
    catch (SqlException ex)
    {
        // Any other SQL error — log and re-throw
        Console.Error.WriteLine($"Unexpected SQL Error {ex.Number}: {ex.Message}");
        throw;
    }
}
```

---

## 🔷 Exception Handling Across All Three Layers

```csharp
// ── DAL — handles database exceptions ────────────────────────────
public int Insert(Employee emp)
{
    try
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(
                "INSERT INTO Employee(Name,Email,Salary) VALUES(@N,@E,@S)", con))
            {
                cmd.Parameters.AddWithValue("@N", emp.Name);
                cmd.Parameters.AddWithValue("@E", emp.Email);
                cmd.Parameters.AddWithValue("@S", emp.Salary);
                return cmd.ExecuteNonQuery();
            }
        }
    }
    catch (SqlException ex) when (ex.Number == 2627 || ex.Number == 2601)
    {
        throw new InvalidOperationException("Duplicate email.", ex);
        // ↑ Converts DB exception into a business-level exception
        // BAL doesn't need to know it was a constraint violation
    }
    catch (SqlException ex)
    {
        Console.Error.WriteLine($"DB Error {ex.Number}: {ex.Message}");
        throw;
    }
}


// ── BAL — handles business exceptions from DAL ────────────────────
public string Insert(Employee emp)
{
    if (string.IsNullOrWhiteSpace(emp.Name)) return "Name is required.";
    if (emp.Salary < 10000) return "Salary too low.";

    try
    {
        int rows = _dal.Insert(emp);
        return rows > 0 ? "success" : "Insert failed.";
    }
    catch (InvalidOperationException ex)
    {
        // DAL translated SQL constraint → business exception
        return ex.Message;   // "Duplicate email."
    }
    catch (TimeoutException)
    {
        return "Operation timed out. Please try again.";
    }
    catch (Exception)
    {
        return "An unexpected error occurred. Please contact support.";
    }
}


// ── Controller — handles results from BAL ─────────────────────────
[HttpPost]
public IActionResult Create(Employee emp)
{
    if (!ModelState.IsValid) return View(emp);

    string result = _bal.Insert(emp);   // BAL already handles exceptions

    if (result == "success")
    {
        TempData["Success"] = "Employee added successfully!";
        return RedirectToAction(nameof(Index));
    }

    ModelState.AddModelError("", result);
    return View(emp);
}
```

---

## 🔷 Exception Handling in Transactions

```csharp
public bool TransferDepartment(int empId, int newDeptId)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        SqlTransaction tx = con.BeginTransaction();

        try
        {
            using (SqlCommand cmd1 = new SqlCommand(
                "UPDATE Employee SET DeptId = @DeptId WHERE Id = @Id", con, tx))
            {
                cmd1.Parameters.AddWithValue("@DeptId", newDeptId);
                cmd1.Parameters.AddWithValue("@Id",     empId);
                cmd1.ExecuteNonQuery();
            }

            using (SqlCommand cmd2 = new SqlCommand(
                "INSERT INTO DeptHistory(EmpId,DeptId,Date) VALUES(@E,@D,GETDATE())",
                con, tx))
            {
                cmd2.Parameters.AddWithValue("@E", empId);
                cmd2.Parameters.AddWithValue("@D", newDeptId);
                cmd2.ExecuteNonQuery();
            }

            tx.Commit();
            return true;
        }
        catch (SqlException ex)
        {
            // Rollback before logging or re-throwing
            try { tx.Rollback(); }
            catch (SqlException rbEx)
            {
                // Rollback itself failed (connection broken?)
                Console.Error.WriteLine($"Rollback failed: {rbEx.Message}");
            }

            Console.Error.WriteLine($"Transfer failed (SQL {ex.Number}): {ex.Message}");
            return false;
        }
        catch (Exception ex)
        {
            try { tx.Rollback(); }
            catch { /* log rollback failure */ }

            Console.Error.WriteLine($"Transfer failed: {ex.Message}");
            return false;
        }
    }
}
```

---

## 🔷 Exception Handling with SqlErrors Collection

`SqlException` can contain multiple errors (SQL Server sends multiple error messages sometimes):

```csharp
catch (SqlException ex)
{
    // ex.Errors = collection of SqlError objects
    foreach (SqlError error in ex.Errors)
    {
        Console.Error.WriteLine(
            $"  Error {error.Number} (Severity {error.Class}): " +
            $"{error.Message} " +
            $"[Procedure: {error.Procedure}, Line: {error.LineNumber}]"
        );
    }
}
```

---

## 🔷 What NOT to Do With Exceptions

```csharp
// ❌ Swallow exception silently — the worst practice
try { _dal.Insert(emp); }
catch { }   // Exception disappears — nobody knows something failed

// ❌ Show DB error details to the user
catch (SqlException ex)
{
    return View("Error", ex.Message);
    // "Violation of PRIMARY KEY constraint 'PK_Employee'..."
    // ← Exposes DB schema to users — security risk
}

// ❌ Catch Exception before SqlException
catch (Exception ex) { }      // ← catches everything first
catch (SqlException ex) { }   // ← NEVER REACHED — compiler warning

// ✅ Most specific first, least specific last:
catch (SqlException ex) { }   // ← specific DB exceptions first
catch (Exception ex)    { }   // ← general fallback last
```

---

## 🔷 Exception Handling Best Practices

```
✅ DO:
  Catch SqlException separately from Exception
  Check ex.Number for specific SQL errors
  Log the full exception (number, message, stack trace)
  Rollback transaction before re-throwing in catch
  Convert DB exceptions to user-friendly messages in BAL
  Never expose SQL error messages to end users

❌ DON'T:
  Swallow exceptions silently (empty catch block)
  Show SqlException.Message directly to users
  Catch Exception before SqlException
  Forget to rollback on exception in transactions
```

---

## ⭐ Interview Quick-Fire

| Question                                                       | Answer                                                                                                                                             |
| -------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| What exception does ADO.NET throw for DB errors?               | `SqlException`                                                                                                                                   |
| How to identify the specific SQL Server error?                 | `ex.Number`— each error has a unique integer code                                                                                               |
| What is `ex.Number = -2`?                                    | Query timeout — query exceeded `CommandTimeout`                                                                                                 |
| What is `ex.Number = 2627`?                                  | Primary key violation — duplicate key inserted                                                                                                    |
| What is `ex.Number = 547`?                                   | Foreign key violation — referential integrity broken                                                                                              |
| What is `ex.Number = 18456`?                                 | Login failed — wrong username or password                                                                                                         |
| Should you show `ex.Message`to users?                        | ❌ No — log it internally, show a friendly message to the user                                                                                    |
| Where should exceptions be caught in three-layer architecture? | DAL catches `SqlException`and converts to business exceptions. BAL catches those and returns friendly messages. Controller never sees DB errors. |
