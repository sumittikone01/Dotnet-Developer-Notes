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
