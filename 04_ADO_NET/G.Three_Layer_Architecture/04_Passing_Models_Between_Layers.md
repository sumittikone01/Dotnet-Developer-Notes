# Passing Models Between Layers

# 04 — Passing Models Between Layers

---

## 🎯 One-Line Definition

> **Model classes are the shared language between all three layers — the Controller creates a model from HTTP input, passes it to BAL for validation, BAL passes it to DAL for database work, and the result travels back up the same way.**

---

## 🔷 Why Models Are the Bridge

```
WITHOUT a shared model:
────────────────────────────────────────────────────────────────
Controller:  passes  string name, string role, decimal salary, string email
BAL:         receives string name, string role, decimal salary, string email
DAL:         receives string name, string role, decimal salary, string email

Problems:
  ❌ Add one new field → update every method signature in every layer
  ❌ 10+ parameters per method — hard to read, easy to get order wrong
  ❌ No single source of truth for what an Employee looks like

WITH a shared model:
────────────────────────────────────────────────────────────────
Controller:  passes  Employee emp
BAL:         receives Employee emp
DAL:         receives Employee emp

Benefits:
  ✅ Add new field → update Employee.cs only
  ✅ Clean, readable method signatures
  ✅ One definition of "Employee" used everywhere
```

---

## 🔷 The Model Class — Defined Once, Used Everywhere

```csharp
// Models/Employee.cs
// This class is shared across Controller, BAL, and DAL

public class Employee
{
    public int     Id       { get; set; }
    public string  Name     { get; set; }
    public string  Role     { get; set; }
    public string  Email    { get; set; }
    public decimal Salary   { get; set; }
    public string  Department { get; set; }
    public DateTime HireDate { get; set; }
    public bool    IsActive  { get; set; }
}
```

---

## 🔷 How Models Flow Through the Layers

```
CONTROLLER
  Employee emp = new Employee from form (model binding)
       │
       │  passes Employee emp DOWN
       ▼
BAL
  Validates emp.Salary, emp.Email etc.
  Calls _dal.Insert(emp)
       │
       │  passes Employee emp DOWN
       ▼
DAL
  Opens SqlConnection
  Reads emp.Name, emp.Salary etc.
  Executes INSERT

──────────── result flows UP ────────────

DAL
  Returns int (rows affected)
       │
       │  returns int UP
       ▼
BAL
  Checks if rows > 0
  Returns "success" or error message
       │
       │  returns string UP
       ▼
CONTROLLER
  Shows success view or error to user
```

---

## 🔷 Complete Example — Insert Flow All Three Layers

### Model

```csharp
// Models/Employee.cs
public class Employee
{
    public int     Id     { get; set; }
    public string  Name   { get; set; }
    public string  Role   { get; set; }
    public string  Email  { get; set; }
    public decimal Salary { get; set; }
}
```

### DAL — receives Employee, talks to DB

```csharp
// DAL/EmployeeDAL.cs
public class EmployeeDAL
{
    private readonly string _cs;

    public EmployeeDAL(IConfiguration config)
        => _cs = config.GetConnectionString("DefaultConnection");

    public int Insert(Employee emp)   // ← receives Employee model
    {
        string sql = @"INSERT INTO Employee (Name, Role, Email, Salary)
                       VALUES (@Name, @Role, @Email, @Salary)";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            {
                cmd.Parameters.Add("@Name",   SqlDbType.NVarChar, 100).Value = emp.Name;
                cmd.Parameters.Add("@Role",   SqlDbType.NVarChar, 50).Value  = emp.Role;
                cmd.Parameters.Add("@Email",  SqlDbType.NVarChar, 150).Value =
                                    (object)emp.Email ?? DBNull.Value;
                cmd.Parameters.Add("@Salary", SqlDbType.Decimal).Value       = emp.Salary;

                return cmd.ExecuteNonQuery();  // ← returns int UP to BAL
            }
        }
    }

    public List<Employee> GetAll()   // ← returns List<Employee> UP to BAL
    {
        var list = new List<Employee>();
        string sql = "SELECT Id, Name, Role, Email, Salary FROM Employee";

        using (SqlConnection con = new SqlConnection(_cs))
        {
            con.Open();
            using (SqlCommand cmd = new SqlCommand(sql, con))
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    list.Add(new Employee          // ← creates Employee objects
                    {
                        Id     = reader.GetInt32(0),
                        Name   = reader.GetString(1),
                        Role   = reader.GetString(2),
                        Email  = reader["Email"] as string,
                        Salary = reader.GetDecimal(4)
                    });
                }
            }
        }

        return list;
    }
}
```

### BAL — receives Employee from Controller, validates, passes to DAL

```csharp
// BAL/EmployeeBAL.cs
public class EmployeeBAL
{
    private readonly EmployeeDAL _dal;

    public EmployeeBAL(EmployeeDAL dal) => _dal = dal;

    public string Insert(Employee emp)   // ← receives same Employee model
    {
        // Validate
        if (string.IsNullOrWhiteSpace(emp.Name))   return "Name is required.";
        if (emp.Salary < 10000)                     return "Minimum salary is ₹10,000.";
        if (_dal.EmailExists(emp.Email))             return "Email already registered.";

        // Transform before saving
        emp.Name  = emp.Name.Trim();
        emp.Email = emp.Email?.Trim().ToLower();

        int rows = _dal.Insert(emp);               // ← passes Employee model DOWN to DAL
        return rows > 0 ? "success" : "Insert failed.";
    }

    public List<Employee> GetAll()   // ← returns same List<Employee> UP to Controller
    {
        return _dal.GetAll();    // pass-through (no business logic for read-all)
    }
}
```

### Controller — gets Employee from model binding, passes to BAL

```csharp
// Controllers/EmployeeController.cs
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;

    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    // GET: /Employee
    public IActionResult Index()
    {
        List<Employee> employees = _bal.GetAll();  // ← receives List<Employee> from BAL
        return View(employees);
    }

    // GET: /Employee/Create
    public IActionResult Create()
    {
        return View(new Employee());
    }

    // POST: /Employee/Create
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(Employee emp)      // ← ASP.NET binds form → Employee
    {
        if (!ModelState.IsValid) return View(emp);

        string result = _bal.Insert(emp);          // ← passes Employee DOWN to BAL

        if (result == "success")
        {
            TempData["Success"] = "Employee added successfully!";
            return RedirectToAction(nameof(Index));
        }

        ModelState.AddModelError("", result);
        return View(emp);
    }
}
```

---

## 🔷 Read Flow — Data Coming Back Up

```
DAL creates Employee objects from DataReader rows
       │
       │  returns List<Employee> UP
       ▼
BAL receives List<Employee>
  May filter or transform (e.g. show only active)
       │
       │  returns List<Employee> UP
       ▼
Controller receives List<Employee>
  Passes to View as Model
       │
       │  View(employees)
       ▼
Razor View renders each Employee as a table row
```

---

## 🔷 Complete Write Flow — Update

```csharp
// ── CONTROLLER: receives form data → Employee model ───────────────
[HttpPost]
public IActionResult Edit(Employee emp)
{
    if (!ModelState.IsValid) return View(emp);

    string result = _bal.Update(emp);   // ← Employee flows down

    if (result == "success") return RedirectToAction(nameof(Index));

    ModelState.AddModelError("", result);
    return View(emp);
}

// ── BAL: validates Employee, passes to DAL ────────────────────────
public string Update(Employee emp)
{
    if (emp.Id <= 0)                    return "Invalid ID.";
    if (string.IsNullOrWhiteSpace(emp.Name)) return "Name is required.";
    if (emp.Salary < 10000)             return "Salary too low.";
    if (_dal.EmailExists(emp.Email, emp.Id)) return "Email already in use.";

    emp.Name  = emp.Name.Trim();
    int rows  = _dal.Update(emp);       // ← Employee flows down to DAL

    return rows > 0 ? "success" : "Employee not found.";
}

// ── DAL: reads from Employee, executes SQL ────────────────────────
public int Update(Employee emp)         // ← receives Employee, returns int
{
    string sql = @"UPDATE Employee
                   SET Name=@Name, Role=@Role, Email=@Email, Salary=@Salary
                   WHERE Id=@Id";

    using (SqlConnection con = new SqlConnection(_cs))
    {
        con.Open();
        using (SqlCommand cmd = new SqlCommand(sql, con))
        {
            cmd.Parameters.AddWithValue("@Name",   emp.Name);
            cmd.Parameters.AddWithValue("@Role",   emp.Role);
            cmd.Parameters.AddWithValue("@Email",  (object)emp.Email ?? DBNull.Value);
            cmd.Parameters.AddWithValue("@Salary", emp.Salary);
            cmd.Parameters.AddWithValue("@Id",     emp.Id);

            return cmd.ExecuteNonQuery();   // ← int flows back UP to BAL
        }
    }
}
```

---

## 🔷 What Each Layer Passes and Returns

```
┌──────────────────────────────────────────────────────────────────┐
│                    PASSING DIRECTION                             │
│                                                                  │
│  DOWN (Controller → BAL → DAL):                                  │
│  ─────────────────────────────                                   │
│  Employee emp          ← for INSERT / UPDATE                     │
│  int id                ← for GET BY ID / DELETE                  │
│  string searchTerm     ← for filtered reads                      │
│                                                                  │
│  UP (DAL → BAL → Controller):                                     │
│  ────────────────────────────                                     │
│  List<Employee>        ← for read all / read filtered            │
│  Employee              ← for read by ID                         │
│  int                   ← rows affected (from DAL)               │
│  bool                  ← exists check (from DAL)                │
│  string                ← result message (from BAL)              │
│  DataTable             ← for Kendo Grid (from DAL)              │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Data Annotations on the Model

Model annotations serve TWO purposes: Controller validation AND DB column hints:

```csharp
using System.ComponentModel.DataAnnotations;

public class Employee
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, ErrorMessage = "Max 100 characters")]
    [Display(Name = "Full Name")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Role is required")]
    public string Role { get; set; }

    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; }

    [Required]
    [Range(10000, 1000000, ErrorMessage = "Salary must be between ₹10,000 and ₹10,00,000")]
    [Display(Name = "Salary (₹)")]
    public decimal Salary { get; set; }

    [DataType(DataType.Date)]
    [Display(Name = "Hire Date")]
    public DateTime HireDate { get; set; }
}
```

```
[Required]    → Controller checks via ModelState.IsValid
              → BAL also checks (defence in depth)

[Range]       → Controller validates range via ModelState
              → BAL enforces business range rules

These annotations mean the Controller catches obvious issues
before even calling the BAL — BAL is the second line of defence.
```

---

## 🔷 Using ViewModels for Different Scenarios

Sometimes the model passed between layers is NOT the same as the DB entity.
Use a ViewModel when the View needs different/extra data:

```csharp
// ViewModel for the employee list page — combines two models
public class EmployeeListViewModel
{
    public List<Employee>   Employees   { get; set; }
    public List<Department> Departments { get; set; }
    public string           SearchTerm  { get; set; }
    public int              TotalCount  { get; set; }
}

// BAL builds and returns the ViewModel
public EmployeeListViewModel GetListData(string searchTerm = "")
{
    return new EmployeeListViewModel
    {
        Employees   = _dal.GetAll(searchTerm),
        Departments = _dal.GetAllDepartments(),
        SearchTerm  = searchTerm,
        TotalCount  = _dal.GetCount()
    };
}

// Controller uses ViewModel
public IActionResult Index(string search = "")
{
    EmployeeListViewModel vm = _bal.GetListData(search);
    return View(vm);
}
```

---

## 🔷 Full Picture — All Three Layers Together

```csharp
// ── PROGRAM.CS ────────────────────────────────────────────────────
builder.Services.AddControllersWithViews();
builder.Services.AddScoped<EmployeeDAL>();
builder.Services.AddScoped<EmployeeBAL>();


// ── MODEL ─────────────────────────────────────────────────────────
public class Employee
{
    public int     Id     { get; set; }
    [Required] public string  Name   { get; set; }
    [Required] public string  Role   { get; set; }
    public string  Email  { get; set; }
    [Range(10000, 1000000)] public decimal Salary { get; set; }
}


// ── DAL ───────────────────────────────────────────────────────────
public class EmployeeDAL
{
    private readonly string _cs;
    public EmployeeDAL(IConfiguration cfg) => _cs = cfg.GetConnectionString("DefaultConnection");

    public List<Employee> GetAll()
    {
        var list = new List<Employee>();
        using var con = new SqlConnection(_cs);
        con.Open();
        using var cmd = new SqlCommand("SELECT Id, Name, Role, Email, Salary FROM Employee", con);
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
            list.Add(new Employee { Id = reader.GetInt32(0), Name = reader.GetString(1),
                                    Role = reader.GetString(2), Salary = reader.GetDecimal(4) });
        return list;
    }

    public int Insert(Employee emp)
    {
        using var con = new SqlConnection(_cs);
        con.Open();
        using var cmd = new SqlCommand(
            "INSERT INTO Employee(Name,Role,Email,Salary) VALUES(@N,@R,@E,@S)", con);
        cmd.Parameters.AddWithValue("@N", emp.Name);
        cmd.Parameters.AddWithValue("@R", emp.Role);
        cmd.Parameters.AddWithValue("@E", (object)emp.Email ?? DBNull.Value);
        cmd.Parameters.AddWithValue("@S", emp.Salary);
        return cmd.ExecuteNonQuery();
    }

    public bool EmailExists(string email, int excludeId = 0)
    {
        using var con = new SqlConnection(_cs);
        con.Open();
        using var cmd = new SqlCommand(
            "SELECT COUNT(*) FROM Employee WHERE Email=@E AND Id!=@Id", con);
        cmd.Parameters.AddWithValue("@E",  email);
        cmd.Parameters.AddWithValue("@Id", excludeId);
        return (int)cmd.ExecuteScalar() > 0;
    }
}


// ── BAL ───────────────────────────────────────────────────────────
public class EmployeeBAL
{
    private readonly EmployeeDAL _dal;
    public EmployeeBAL(EmployeeDAL dal) => _dal = dal;

    public List<Employee> GetAll() => _dal.GetAll();

    public string Insert(Employee emp)
    {
        if (string.IsNullOrWhiteSpace(emp.Name))   return "Name is required.";
        if (emp.Salary < 10000)                    return "Salary too low.";
        if (!string.IsNullOrEmpty(emp.Email) && _dal.EmailExists(emp.Email))
            return "Email already registered.";

        emp.Name  = emp.Name.Trim();
        emp.Email = emp.Email?.Trim().ToLower();

        return _dal.Insert(emp) > 0 ? "success" : "Insert failed.";
    }
}


// ── CONTROLLER ────────────────────────────────────────────────────
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    public EmployeeController(EmployeeBAL bal) => _bal = bal;

    public IActionResult Index()
        => View(_bal.GetAll());

    public IActionResult Create() => View(new Employee());

    [HttpPost, ValidateAntiForgeryToken]
    public IActionResult Create(Employee emp)
    {
        if (!ModelState.IsValid) return View(emp);

        string result = _bal.Insert(emp);
        if (result == "success") return RedirectToAction(nameof(Index));

        ModelState.AddModelError("", result);
        return View(emp);
    }
}
```

---

## 🔷 Model Passing Summary Table

| Operation | Controller passes down | BAL passes down  | DAL returns up     | BAL returns up     |
| --------- | ---------------------- | ---------------- | ------------------ | ------------------ |
| Get All   | —                     | —               | `List<Employee>` | `List<Employee>` |
| Get By Id | `int id`             | `int id`       | `Employee`       | `Employee`       |
| Insert    | `Employee emp`       | `Employee emp` | `int` rows       | `string` result  |
| Update    | `Employee emp`       | `Employee emp` | `int` rows       | `string` result  |
| Delete    | `int id`             | `int id`       | `int` rows       | `string` result  |

---

## ⚠️ Common Mistakes

| Mistake                                            | What Happens                                             | Fix                                                        |
| -------------------------------------------------- | -------------------------------------------------------- | ---------------------------------------------------------- |
| Passing 10 separate parameters instead of model    | Fragile — adding a field breaks every layer             | Always wrap in a model class                               |
| BAL returning `DataTable` to Controller          | Leaks DAL-specific type into upper layers                | BAL returns `List<Employee>` or model objects            |
| Controller accessing `_dal` directly             | Skips BAL — business rules bypassed                     | Controller only injects and calls BAL                      |
| No `using` directives for the model namespace    | Model not found at compile time                          | Model in `Models/` folder, `using` at top of each file |
| ViewModel not used when page needs multiple models | Controller forces View to use `ViewBag` for extra data | Create a ViewModel class combining the data                |

---

## ⭐ Interview Quick-Fire

| Question                                        | Answer                                                                                        |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------- |
| What travels down through layers?               | Model objects (Employee), primitive types (int id, string term)                               |
| What travels up through layers?                 | List`<Model>`, single Model, int (rows), string (result message)                            |
| Why use a model instead of separate parameters? | Cleaner signatures, one change point, type safety                                             |
| What is a ViewModel?                            | A model designed for a specific View — combines multiple entities or adds display properties |
| Where are model classes defined?                | In the `Models/` folder — shared across all three layers                                   |
| Should BAL return DataTable to Controller?      | ❌ No — BAL works with model objects. DataTable is a DAL concern                             |
| How does Controller get Employee from a form?   | ASP.NET Core model binding reads form fields and populates an Employee object automatically   |

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
