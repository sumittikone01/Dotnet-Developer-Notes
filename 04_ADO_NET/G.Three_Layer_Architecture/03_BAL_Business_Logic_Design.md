# BAL Business Logic Design

# 03 — BAL Business Logic Design

---

## 🎯 One-Line Definition

> **The BAL (Business Access Layer) sits between the Controller and DAL — it validates input, enforces business rules, orchestrates multiple DAL calls, and returns meaningful results so the Controller stays clean and the DAL stays focused on data only.**

---

## 🔷 What the BAL Owns

```
BAL owns:
  ✅ Input validation (salary range, required fields)
  ✅ Business rules (no duplicate email, max 5 managers per dept)
  ✅ Orchestration (call DAL.Insert + DAL.UpdateDeptCount)
  ✅ Calculations (net salary after tax, bonus calculation)
  ✅ Transformations (format data before save or after load)
  ✅ Conditional logic (if department changed → update headcount)

BAL does NOT own:
  ❌ SqlConnection, SqlCommand — that's DAL
  ❌ HTTP request/response — that's Controller
  ❌ ViewBag, TempData, ModelState — that's Controller
  ❌ IActionResult — that's Controller
  ❌ Any direct SQL
```

---

## 🔷 Where BAL Sits

```
Controller                BAL                    DAL
──────────────            ──────────────         ──────────────────
Receives HTTP request  →  Validates input     →  Opens SqlConnection
Calls bal.Insert(emp)     Checks business        Executes INSERT
Returns View/JSON      ←  rules               ←  Returns rows affected
                          Calls dal.Insert()
                          Returns result to
                          Controller
```

---

## 🔷 The BAL Constructor — Dependency Injection

```csharp
using Microsoft.Extensions.Configuration;

public class EmployeeBAL
{
    private readonly EmployeeDAL _dal;

    // ASP.NET Core injects EmployeeDAL automatically
    public EmployeeBAL(EmployeeDAL dal)
    {
        _dal = dal;
    }
}
```

---

## 🔷 Complete BAL Class — All Operations

```csharp
public class EmployeeBAL
{
    private readonly EmployeeDAL _dal;

    public EmployeeBAL(EmployeeDAL dal)
    {
        _dal = dal;
    }

    // ── GET ALL ───────────────────────────────────────────────────
    public List<Employee> GetAll()
    {
        // No business rules for read-all — just pass through
        return _dal.GetAll();
    }

    // ── GET BY ID ─────────────────────────────────────────────────
    public Employee GetById(int id)
    {
        if (id <= 0)
            return null;   // invalid id — don't even call DAL

        return _dal.GetById(id);
    }

    // ── INSERT — with full validation and rules ────────────────────
    public string Insert(Employee emp)
    {
        // ── Validation ────────────────────────────────────────────
        if (string.IsNullOrWhiteSpace(emp.Name))
            return "Name is required.";

        if (string.IsNullOrWhiteSpace(emp.Role))
            return "Role is required.";

        if (emp.Salary < 10000)
            return "Salary must be at least ₹10,000.";

        if (emp.Salary > 1000000)
            return "Salary cannot exceed ₹10,00,000.";

        if (!string.IsNullOrEmpty(emp.Email) && !IsValidEmail(emp.Email))
            return "Invalid email format.";

        // ── Business Rule: Email must be unique ───────────────────
        if (!string.IsNullOrEmpty(emp.Email) && _dal.EmailExists(emp.Email))
            return "This email is already registered.";

        // ── Transformation: trim whitespace before saving ─────────
        emp.Name  = emp.Name.Trim();
        emp.Role  = emp.Role.Trim();
        emp.Email = emp.Email?.Trim().ToLower();

        // ── Call DAL ──────────────────────────────────────────────
        int rows = _dal.Insert(emp);
        return rows > 0 ? "success" : "Insert failed. Please try again.";
    }

    // ── UPDATE — validation + uniqueness check excluding self ──────
    public string Update(Employee emp)
    {
        if (emp.Id <= 0)
            return "Invalid employee ID.";

        if (string.IsNullOrWhiteSpace(emp.Name))
            return "Name is required.";

        if (emp.Salary < 10000)
            return "Salary must be at least ₹10,000.";

        if (!string.IsNullOrEmpty(emp.Email) && !IsValidEmail(emp.Email))
            return "Invalid email format.";

        // ── Business Rule: Email unique — exclude current employee ─
        if (!string.IsNullOrEmpty(emp.Email) && _dal.EmailExists(emp.Email, emp.Id))
            return "This email is already used by another employee.";

        emp.Name  = emp.Name.Trim();
        emp.Email = emp.Email?.Trim().ToLower();

        int rows = _dal.Update(emp);

        if (rows == 0)
            return "Employee not found.";

        return "success";
    }

    // ── DELETE — with guard ───────────────────────────────────────
    public string Delete(int id)
    {
        if (id <= 0)
            return "Invalid employee ID.";

        // ── Business Rule: Cannot delete managers ─────────────────
        Employee emp = _dal.GetById(id);
        if (emp == null)
            return "Employee not found.";

        if (emp.Role.ToLower() == "manager")
            return "Cannot delete an employee with Manager role.";

        int rows = _dal.Delete(id);
        return rows > 0 ? "success" : "Delete failed.";
    }

    // ── PRIVATE: Validate email format ────────────────────────────
    private bool IsValidEmail(string email)
    {
        try
        {
            var addr = new System.Net.Mail.MailAddress(email);
            return addr.Address == email;
        }
        catch
        {
            return false;
        }
    }
}
```

---

## 🔷 BAL with Multiple DAL Calls — Orchestration

A key BAL responsibility: coordinating multiple DAL operations together:

```csharp
// When an employee changes department:
//   1. Update employee's DeptId
//   2. Decrease old department's headcount
//   3. Increase new department's headcount
// All three must succeed together — BAL orchestrates this

public string TransferDepartment(int empId, int newDeptId)
{
    // Validate
    Employee emp = _dal.GetById(empId);
    if (emp == null)          return "Employee not found.";
    if (emp.DeptId == newDeptId) return "Employee is already in this department.";

    Department newDept = _dal.GetDepartmentById(newDeptId);
    if (newDept == null)      return "Department not found.";

    // Business Rule: max 50 employees per department
    if (newDept.HeadCount >= 50)
        return $"{newDept.Name} is at full capacity.";

    // Orchestrate multiple DAL calls (wrapped in DAL transaction)
    bool success = _dal.TransferDepartment(empId, emp.DeptId, newDeptId);
    return success ? "success" : "Transfer failed. Please try again.";
}
```

---

## 🔷 BAL Return Types — Common Patterns

### Pattern 1 — Return string (simple, readable)

```csharp
public string Insert(Employee emp)
{
    // ...
    return "success";        // ← Controller checks this
    return "Email exists.";  // ← Controller shows as error
}

// Controller:
string result = _bal.Insert(emp);
if (result == "success")
    return RedirectToAction("Index");

ModelState.AddModelError("", result);
return View(emp);
```

### Pattern 2 — Return bool (simple pass/fail)

```csharp
public bool Delete(int id)
{
    if (id <= 0) return false;
    return _dal.Delete(id) > 0;
}

// Controller:
if (_bal.Delete(id))
    TempData["Success"] = "Employee deleted.";
else
    TempData["Error"] = "Delete failed.";
```

### Pattern 3 — Return Result object (cleanest for APIs)

```csharp
public class OperationResult
{
    public bool    IsSuccess { get; set; }
    public string  Message   { get; set; }
    public object  Data      { get; set; }

    public static OperationResult Success(string msg = "Success", object data = null)
        => new OperationResult { IsSuccess = true, Message = msg, Data = data };

    public static OperationResult Fail(string msg)
        => new OperationResult { IsSuccess = false, Message = msg };
}

// BAL method:
public OperationResult Insert(Employee emp)
{
    if (string.IsNullOrWhiteSpace(emp.Name))
        return OperationResult.Fail("Name is required.");

    if (_dal.EmailExists(emp.Email))
        return OperationResult.Fail("Email already registered.");

    int newId = _dal.InsertGetId(emp);
    return newId > 0
        ? OperationResult.Success("Employee added.", newId)
        : OperationResult.Fail("Insert failed.");
}

// Controller:
var result = _bal.Insert(emp);
if (!result.IsSuccess)
{
    ModelState.AddModelError("", result.Message);
    return View(emp);
}
TempData["Success"] = result.Message;
return RedirectToAction("Index");
```

---

## 🔷 Registering BAL in Program.cs

```csharp
// Program.cs
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();

// Order matters: register DAL first so BAL can inject it
```

---

## 🔷 BAL Design Rules

```
IN the BAL:                           NOT in the BAL:
────────────────────────────          ─────────────────────────────────
Null/empty checks                     SqlConnection, SqlCommand
Range validation (salary 10k–1M)      con.Open(), ExecuteNonQuery()
Format validation (email regex)       Any SQL string
Business rules (unique email)         HttpContext, Request
Calculations (net salary)             ViewBag, TempData
Orchestration (multiple DAL calls)    ModelState
Transformations (trim, lowercase)     IActionResult, View(), Json()
```

---

## 🔷 BAL Responsibility Matrix

| Task                                   | Controller      | BAL                     | DAL |
| -------------------------------------- | --------------- | ----------------------- | --- |
| "Name is required" validation          | ✅ (ModelState) | ✅ (extra check)        | ❌  |
| "Salary must be > 10,000"              | ❌              | ✅                      | ❌  |
| "Email must be unique in DB"           | ❌              | ✅ (calls DAL to check) | ❌  |
| "Cannot delete a Manager"              | ❌              | ✅                      | ❌  |
| "Trim and lowercase email before save" | ❌              | ✅                      | ❌  |
| "Execute UPDATE SQL"                   | ❌              | ❌                      | ✅  |

---

## ⚠️ Common BAL Mistakes

| Mistake                              | What Happens                                 | Fix                                      |
| ------------------------------------ | -------------------------------------------- | ---------------------------------------- |
| Putting SQL in BAL                   | Layer boundary broken — hard to maintain    | All SQL only in DAL                      |
| No validation in BAL                 | Invalid data reaches DAL, causes DB errors   | Always validate before calling DAL       |
| Returning raw `DataTable` from BAL | Presentation leaked into business layer      | BAL should work with model objects       |
| BAL calling Controller methods       | Wrong direction — layers only call downward | Controller → BAL → DAL (never reverse) |

---

## ⭐ Interview Quick-Fire

# 02 — DAL Class Design

---

## 🎯 One-Line Definition

> **The DAL class is the only place in your app that talks to SQL Server — it opens connections, executes queries, and returns data as models or DataTables — nothing else belongs here.**

---

## 🔷 What the DAL Owns

```
DAL owns:
  ✅ Connection string (read from IConfiguration)
  ✅ SqlConnection — open and close
  ✅ SqlCommand — SELECT / INSERT / UPDATE / DELETE
  ✅ SqlDataReader — read rows
  ✅ SqlDataAdapter — fill DataTable
  ✅ SqlTransaction — wrap multiple operations
  ✅ Stored procedure calls
  ✅ Mapping DataReader rows → Employee objects

DAL does NOT own:
  ❌ Business rules (no "if salary > 100000 reject")
  ❌ HTTP request/response
  ❌ ModelState validation
  ❌ ViewBag / TempData
  ❌ Any UI concerns
```

---

## 🔷 DAL Constructor — Reading Connection String

```csharp
using System.Data;
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Configuration;

public class EmployeeDAL
{
    private readonly string _cs;

    // ASP.NET Core injects IConfiguration — reads appsettings.json
    public EmployeeDAL(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }
}
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.\\SQLEXPRESS;Database=EmployeeDB;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

---

## 🔷 Complete DAL Class — All CRUD Methods

```csharp
using System.Data;
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Configuration;

public class EmployeeDAL
{
    private readonly string _cs;

    public EmployeeDAL(IConfiguration config)
    {
        _cs = config.GetConnectionString("DefaultConnection");
    }

    // ── GET ALL ───────────────────────────────────────────────────
    public List<Employee> GetAll()
    {
        var list = new List<Employee>();
        string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee ORDER BY Name";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    list.Add(MapReader(reader));
                }
            }
        }

        return list;
    }

    // ── GET BY ID ─────────────────────────────────────────────────
    public Employee GetById(int id)
    {
        string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee WHERE Id = @Id";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Id", id);

                using (SqlDataReader reader = cmd.ExecuteReader())
                {
                    if (reader.Read())
                        return MapReader(reader);
                }
            }
        }

        return null;
    }

    // ── INSERT ────────────────────────────────────────────────────
    public int Insert(Employee emp)
    {
        string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                       VALUES (@Name, @Role, @Email, @Salary)";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                AddEmployeeParams(cmd, emp);
                return cmd.ExecuteNonQuery();
            }
        }
    }

    // ── INSERT AND GET NEW ID ─────────────────────────────────────
    public int InsertGetId(Employee emp)
    {
        string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                       VALUES (@Name, @Role, @Email, @Salary);
                       SELECT SCOPE_IDENTITY();";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                AddEmployeeParams(cmd, emp);
                return Convert.ToInt32(cmd.ExecuteScalar());
            }
        }
    }

    // ── UPDATE ────────────────────────────────────────────────────
    public int Update(Employee emp)
    {
        string sql = @"UPDATE Employee
                       SET Name  = @Name,
                           Role  = @Role,
                           Email = @Email,
                           Salary = @Salary
                       WHERE Id = @Id";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                AddEmployeeParams(cmd, emp);
                cmd.Parameters.AddWithValue("@Id", emp.Id);
                return cmd.ExecuteNonQuery();
            }
        }
    }

    // ── DELETE ────────────────────────────────────────────────────
    public int Delete(int id)
    {
        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(
                "DELETE FROM Employee WHERE Id = @Id", con))
            {
                cmd.Parameters.AddWithValue("@Id", id);
                return cmd.ExecuteNonQuery();
            }
        }
    }

    // ── CHECK EMAIL EXISTS ─────────────────────────────────────────
    // Called by BAL for uniqueness check
    public bool EmailExists(string email, int excludeId = 0)
    {
        string sql = "SELECT COUNT(*) FROM Employee WHERE Email = @Email AND Id != @ExcludeId";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.AddWithValue("@Email",     email);
                cmd.Parameters.AddWithValue("@ExcludeId", excludeId);
                return (int)cmd.ExecuteScalar() > 0;
            }
        }
    }

    // ── GET AS DATATABLE (for Kendo Grid) ────────────────────────
    public DataTable GetAllAsTable()
    {
        DataTable dt = new DataTable();
        string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee";

        using (SqlDataAdapter da = new SqlDataAdapter(sql, _cs))
        {
            da.Fill(dt);
        }

        return dt;
    }

    // ── PRIVATE: Map DataReader row → Employee object ─────────────
    private Employee MapReader(SqlDataReader reader)
    {
        return new Employee
        {
            Id     = reader.GetInt32(reader.GetOrdinal("Id")),
            Name   = reader.GetString(reader.GetOrdinal("Name")),
            Role   = reader.GetString(reader.GetOrdinal("Role")),
            Email  = reader["Email"] as string,          // nullable
            Salary = reader.GetDecimal(reader.GetOrdinal("Salary"))
        };
    }

    // ── PRIVATE: Add common employee parameters to a command ──────
    private void AddEmployeeParams(SqlCommand cmd, Employee emp)
    {
        cmd.Parameters.Add("@Name",   SqlDbType.NVarChar, 100).Value = emp.Name;
        cmd.Parameters.Add("@Role",   SqlDbType.NVarChar, 50).Value  = emp.Role;
        cmd.Parameters.Add("@Email",  SqlDbType.NVarChar, 150).Value =
                            (object)emp.Email ?? DBNull.Value;
        cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value       = emp.Salary;
    }
}
```

---

## 🔷 DAL with Stored Procedures

```csharp
// When your project uses SPs instead of inline SQL, the DAL looks like this:

public List<Employee> GetAll()
{
    var list = new List<Employee>();

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_GetAllEmployees", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                    list.Add(MapReader(reader));
            }
        }
    }

    return list;
}

public int Insert(Employee emp)
{
    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand("sp_InsertEmployee", con))
        {
            cmd.CommandType = CommandType.StoredProcedure;
            AddEmployeeParams(cmd, emp);
            return cmd.ExecuteNonQuery();
        }
    }
}
```

---

## 🔷 DAL Design Rules — What Goes In, What Stays Out

```
IN the DAL:                           NOT in the DAL:
─────────────────────────────────     ────────────────────────────────────
SqlConnection                         if (salary > 100000) return error
SqlCommand                            ViewBag, TempData
SqlDataReader                         ModelState
SqlDataAdapter                        HttpContext
ExecuteReader/NonQuery/Scalar         IActionResult
Stored procedure calls                Business calculations
MapReader private method              Email/SMS notification
AddParams private helper              IConfiguration for non-DB things
Connection string from config         Any layer-specific dependencies
```

---

## 🔷 DAL with Transaction

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
                "INSERT INTO DeptHistory (EmpId, DeptId, TransferDate) VALUES (@E, @D, GETDATE())",
                con, tx))
            {
                cmd2.Parameters.AddWithValue("@E", empId);
                cmd2.Parameters.AddWithValue("@D", newDeptId);
                cmd2.ExecuteNonQuery();
            }

            tx.Commit();
            return true;
        }
        catch
        {
            tx.Rollback();
            return false;
        }
    }
}
```

---

## 🔷 Registering DAL in Program.cs

```csharp
// Program.cs
builder.Services.AddScoped<EmployeeDAL>();
// AddScoped = one instance per HTTP request (correct for DB access)
// AddTransient = new instance every injection (avoid for connections)
// AddSingleton = one instance for app lifetime (wrong for DB classes)
```

---

## 🔷 DAL Naming Conventions

```
Common naming patterns teams use:

  EmployeeDAL.cs          ← "DAL" suffix (your current pattern)
  EmployeeRepository.cs   ← "Repository" suffix (popular in .NET)
  EmployeeData.cs         ← "Data" suffix
  EmployeeService.cs      ← "Service" suffix (sometimes used for combined BAL+DAL)

All correct — pick one and stay consistent across the project.
```

---

## ⚠️ Common DAL Mistakes

| Mistake                                                  | What Happens                                            | Fix                                                |
| -------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| Putting business rules in DAL                            | DAL becomes bloated, hard to reuse                      | Move rules to BAL                                  |
| Not using private helpers (`MapReader`, `AddParams`) | Duplicate code in every method                          | Extract reusable private methods                   |
| Opening connection in constructor                        | Stays open — connection leak                           | Open connection inside each method using `using` |
| Returning DataReader from DAL                            | Reader requires open connection — unusable outside DAL | Map to `List<Employee>` inside DAL, return list  |

---

## ⭐ Interview Quick-Fire

| Question                                       | Answer                                                                       |
| ---------------------------------------------- | ---------------------------------------------------------------------------- |
| What is DAL responsible for?                   | All database access — SqlConnection, SQL queries, mapping results           |
| Can DAL have business logic?                   | ❌ Never — business rules belong in BAL                                     |
| How does DAL get the connection string?        | `IConfiguration` injected in constructor, reads `appsettings.json`       |
| Why use private `MapReader` method?          | Avoids duplicating mapping code in every read method                         |
| Should DAL return `SqlDataReader` to caller? | ❌ No — map to List/object inside DAL, connection must stay open for reader |
| What lifetime to register DAL with?            | `AddScoped` — one per HTTP request                                        |

---

## 📌 Overview

> Write your notes here.

---

## 🔑 Key Concepts

---

## 💻 Code Example

```csharp

```

---

## ❓ Interview Questions

---

## 🔗 Related Topics

---

*Last updated: 2026-03-15*
