
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
