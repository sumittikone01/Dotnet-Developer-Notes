
# 01 — Model Responsibilities

---

## 🎯 One-Line Definition

> **The Model represents your data and its rules — it defines what an entity looks like, what values are valid, and nothing else. It has zero knowledge of HTTP, views, or databases.**

---

## 🔷 What the Model Is Responsible For

```
✅ MODEL OWNS:
  → Defining the shape of data (properties)
  → Validation rules ([Required], [Range], [EmailAddress])
  → Display names ([Display(Name = "...")])
  → Data type hints ([DataType(DataType.Date)])
  → Business constraints expressed as annotations

❌ MODEL NEVER DOES:
  → SQL queries or database calls
  → HTTP request/response handling
  → HTML rendering
  → ViewBag, TempData, ModelState
  → Calling services or other layers
```

---

## 🔷 The Three Types of Models in MVC

```
┌────────────────────────────────────────────────────────────────┐
│  TYPE 1 — Domain Model (Entity)                               │
│  Represents a real-world thing that maps to a DB table        │
│  e.g.: Employee, Department, Product                          │
│                                                               │
│  TYPE 2 — ViewModel                                           │
│  Designed for a specific View — combines multiple entities    │
│  or adds display-only properties                              │
│  e.g.: EmployeeFormViewModel (with DepartmentList for dropdown)│
│                                                               │
│  TYPE 3 — DTO (Data Transfer Object)                         │
│  Carries data between layers — stripped down, no annotations  │
│  e.g.: EmployeeDto (only Id, Name, Salary — no validation)   │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔷 Type 1 — Domain Model

The entity that represents a database table:

```csharp
// Models/Employee.cs
using System.ComponentModel.DataAnnotations;

public class Employee
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2,
        ErrorMessage = "Name must be between 2 and 100 characters")]
    [Display(Name = "Full Name")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Role is required")]
    [StringLength(50)]
    public string Role { get; set; }

    [EmailAddress(ErrorMessage = "Enter a valid email address")]
    [Display(Name = "Email Address")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Salary is required")]
    [Range(10000, 1000000,
        ErrorMessage = "Salary must be between ₹10,000 and ₹10,00,000")]
    [Display(Name = "Salary (₹)")]
    public decimal Salary { get; set; }

    [DataType(DataType.Date)]
    [Display(Name = "Hire Date")]
    public DateTime HireDate { get; set; }

    [Display(Name = "Active")]
    public bool IsActive { get; set; } = true;  // default value
}
```

---

## 🔷 Data Annotations — Full Reference

### Validation Annotations

| Annotation                               | What It Validates        | Example                                        |
| ---------------------------------------- | ------------------------ | ---------------------------------------------- |
| `[Required]`                           | Field must have a value  | `[Required(ErrorMessage = "Name required")]` |
| `[StringLength(max)]`                  | Max character length     | `[StringLength(100)]`                        |
| `[StringLength(max, MinimumLength=n)]` | Min and max length       | `[StringLength(100, MinimumLength=2)]`       |
| `[Range(min, max)]`                    | Numeric range            | `[Range(10000, 1000000)]`                    |
| `[EmailAddress]`                       | Valid email format       | `[EmailAddress]`                             |
| `[Phone]`                              | Valid phone format       | `[Phone]`                                    |
| `[Url]`                                | Valid URL format         | `[Url]`                                      |
| `[RegularExpression(pattern)]`         | Custom regex             | `[RegularExpression(@"^[A-Z]\d{5}$")]`       |
| `[Compare("OtherProp")]`               | Must match another field | `[Compare("Password")]`on ConfirmPassword    |
| `[MinLength(n)]`                       | Minimum length           | `[MinLength(8)]`                             |
| `[MaxLength(n)]`                       | Maximum length           | `[MaxLength(200)]`                           |

### Display Annotations

| Annotation                            | What It Does             | Example                                          |
| ------------------------------------- | ------------------------ | ------------------------------------------------ |
| `[Display(Name = "...")]`           | Label in form/table      | `[Display(Name = "Full Name")]`                |
| `[DataType(DataType.X)]`            | Hints the data type      | `[DataType(DataType.Date)]`                    |
| `[DisplayFormat(DataFormatString)]` | Format when displayed    | `[DisplayFormat(DataFormatString = "{0:C0}")]` |
| `[HiddenInput]`                     | Renders as hidden field  | `[HiddenInput]`                                |
| `[ScaffoldColumn(false)]`           | Exclude from scaffolding | `[ScaffoldColumn(false)]`                      |

---

## 🔷 Type 2 — ViewModel

When a View needs more than one entity — create a ViewModel:

```csharp
// Models/ViewModels/EmployeeFormViewModel.cs
// Used on the Create/Edit form — needs employee data AND dropdowns

public class EmployeeFormViewModel
{
    // The employee being edited
    public Employee Employee { get; set; }

    // Dropdown data — departments and roles
    public List<SelectListItem> Departments { get; set; }
    public List<SelectListItem> Roles       { get; set; }
}

// Controller builds and passes it:
public IActionResult Create()
{
    var vm = new EmployeeFormViewModel
    {
        Employee    = new Employee(),
        Departments = _bal.GetDepartments()
                          .Select(d => new SelectListItem(d.Name, d.Id.ToString()))
                          .ToList(),
        Roles       = new List<SelectListItem>
        {
            new("Developer",  "Developer"),
            new("QA Engineer","QA Engineer"),
            new("Manager",    "Manager")
        }
    };
    return View(vm);
}

// View uses:
@model EmployeeFormViewModel
<input asp-for="Employee.Name" class="form-control" />
<select asp-for="Employee.DeptId" asp-items="Model.Departments"></select>
```

---

## 🔷 Type 3 — DTO (Data Transfer Object)

Lightweight object to carry data between layers — no validation annotations:

```csharp
// Models/DTOs/EmployeeDto.cs
// Used when you only need a subset of Employee data
// e.g., for a dropdown list or a summary API response

public class EmployeeDto
{
    public int    Id     { get; set; }
    public string Name   { get; set; }
    public string Role   { get; set; }
    public decimal Salary { get; set; }
}

// DAL returns EmployeeDto for read-only lists
// No heavy annotations, faster to work with
```

---

## 🔷 Model + Controller + View — How They Connect

```
CONTROLLER creates/receives the Model:
  Employee emp = new Employee();          ← create for form
  List<Employee> list = _bal.GetAll();    ← read for display
  return View(emp);                       ← pass to View
  return View(list);

VIEW declares the Model type:
  @model Employee                         ← single object
  @model List<Employee>                   ← collection
  @model EmployeeFormViewModel            ← viewmodel

  Uses Model properties directly:
  @Model.Name
  @Model.Salary.ToString("C0")
  @foreach (var emp in Model) { ... }

  asp-for tag helper binds to Model:
  <input asp-for="Name" />               ← binds to Model.Name
  <span asp-validation-for="Name" />     ← shows Name's error
```

---

## 🔷 ModelState — Where Validation Results Go

```
When a form is submitted with [HttpPost]:
  1. ASP.NET reads the form fields
  2. Populates the Model with values (model binding)
  3. Checks every [Required], [Range], [EmailAddress] etc.
  4. Results go into ModelState

ModelState.IsValid = true   → all annotations passed
ModelState.IsValid = false  → at least one annotation failed

Controller checks it:
  if (!ModelState.IsValid)
      return View(emp);     // return same view with errors shown
```

---

## 🔷 Complete Domain Model — Employee with All Annotations

```csharp
public class Employee
{
    [HiddenInput]  // render as hidden field in forms
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2, ErrorMessage = "2–100 characters")]
    [Display(Name = "Full Name")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Role is required")]
    [StringLength(50)]
    public string Role { get; set; }

    [EmailAddress(ErrorMessage = "Invalid email format")]
    [Display(Name = "Email Address")]
    public string Email { get; set; }

    [Required(ErrorMessage = "Phone is required")]
    [Phone(ErrorMessage = "Invalid phone number")]
    [Display(Name = "Phone Number")]
    public string Phone { get; set; }

    [Required]
    [Range(10000, 1000000, ErrorMessage = "₹10,000 – ₹10,00,000")]
    [DisplayFormat(DataFormatString = "{0:C0}")]
    [Display(Name = "Salary (₹)")]
    public decimal Salary { get; set; }

    [DataType(DataType.Date)]
    [Display(Name = "Hire Date")]
    [DisplayFormat(DataFormatString = "{0:dd/MM/yyyy}", ApplyFormatInEditMode = true)]
    public DateTime HireDate { get; set; }

    [Display(Name = "Department")]
    public int DeptId { get; set; }

    [Display(Name = "Active")]
    public bool IsActive { get; set; } = true;
}
```

---

## ⚠️ Common Model Mistakes

| Mistake                                            | What Happens                                                           | Fix                            |
| -------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------ |
| Putting SQL in the Model                           | Model becomes a data access class — hard to test and reuse            | SQL only in DAL                |
| No `[Required]`on critical fields                | Invalid data reaches DAL → DB errors                                  | Annotate all required fields   |
| Using Model directly when View needs dropdown data | ViewBag needed for dropdown → messy                                   | Create a ViewModel             |
| No `[Display(Name)]`annotations                  | Form labels show raw property names ("DeptId" instead of "Department") | Always add `[Display(Name)]` |

---

## ⭐ Interview Quick-Fire

| Question                                    | Answer                                                                             |
| ------------------------------------------- | ---------------------------------------------------------------------------------- |
| What is a Model in MVC?                     | Represents data and defines validation rules — nothing else                       |
| Three types of Models?                      | Domain Model (entity), ViewModel (for a specific View), DTO (lightweight transfer) |
| What namespace are Data Annotations in?     | `System.ComponentModel.DataAnnotations`                                          |
| What does `[Required]`do?                 | Makes field mandatory — ModelState.IsValid becomes false if empty                 |
| What does `[Range(10000, 1000000)]`do?    | Validates that the value is between 10,000 and 1,000,000                           |
| What is ModelState?                         | Dictionary holding validation results for every field after model binding          |
| When to use a ViewModel instead of a Model? | When the View needs data from multiple models or extra display properties          |
| Can a Model have SQL?                       | ❌ Never — SQL only belongs in DAL                                                |
