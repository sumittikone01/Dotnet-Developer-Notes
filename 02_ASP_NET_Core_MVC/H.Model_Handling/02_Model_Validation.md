
# 02 — Model Validation

---

## 🎯 One-Line Definition

> **Model Validation is the process of checking that data bound from the request meets your rules before you process it — ASP.NET Core runs it automatically after Model Binding using Data Annotations, stores the results in `ModelState`, and with `[ApiController]` returns 400 automatically when rules fail.**

---

## 🔷 Where Validation Fits in the Request Flow

```
HTTP Request arrives
        │
        ▼
Model Binding runs
  → reads form/JSON/query/route into your parameter objects
        │
        ▼
Validation runs  ← YOU ARE HERE
  → checks every Data Annotation on every bound property
  → populates ModelState.Errors if any rule fails
        │
        ├─ [ApiController] present AND ModelState invalid
        │    → 400 Bad Request returned AUTOMATICALLY
        │    → your action method is NEVER called
        │
        └─ No [ApiController] (MVC controllers)
             → your action method runs
             → you check ModelState.IsValid manually
        │
        ▼
Your Action Method Runs
```

---

## 🔷 ModelState — What It Is

```csharp
// ModelState is a dictionary on every Controller that holds:
//   1. The bound values for each property
//   2. Any validation errors for each property
//   3. An IsValid flag — false if ANY property has ANY error

// Type: ModelStateDictionary
// Accessible as: ModelState  (property on Controller/ControllerBase)

// Key properties:
ModelState.IsValid          // bool — true if zero errors
ModelState.ErrorCount       // int  — total number of errors
ModelState["EmpName"]       // entry for a specific property
ModelState["EmpName"].Errors // errors for that property
```

---

## 🔷 Validation in MVC Controller — Manual Check

```csharp
public class EmployeeController : Controller
{
    [HttpPost]
    public IActionResult Create(Employee emp)
    // ↑ Model Binding populates emp from the form
    // ↑ Validation runs automatically after binding
    // ↑ ModelState.IsValid is now set
    {
        // ── Manual check required in MVC controllers ───────────────
        if (!ModelState.IsValid)
        {
            // Validation failed — stay on the form, show errors
            // Re-populate any ViewBag data the view needs:
            ViewBag.DeptList = new SelectList(_deptBal.GetAll(), "Id", "Name");
            return View(emp);   // Return the same model so form retains values
        }

        // ── ModelState.IsValid = true — safe to process ────────────
        _bal.Add(emp);
        TempData["Success"] = "Employee created successfully!";
        return RedirectToAction("Index");
    }

    [HttpPost]
    public IActionResult Edit(int id, Employee emp)
    {
        // You can also add custom errors programmatically:
        if (_bal.EmailExistsForOtherEmployee(emp.Email, id))
        {
            ModelState.AddModelError("Email", "This email is already used by another employee.");
            // ↑ "Email" = property name, second arg = the error message
        }

        if (!ModelState.IsValid)
        {
            ViewBag.DeptList = new SelectList(_deptBal.GetAll(), "Id", "Name");
            return View(emp);
        }

        _bal.Update(emp);
        return RedirectToAction("Index");
    }
}
```

---

## 🔷 Validation in API Controller — Automatic

```csharp
[ApiController]
[Route("api/employees")]
public class EmployeeApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create([FromBody] Employee emp)
    {
        // With [ApiController]:
        // If ModelState is invalid → 400 returned automatically
        // This line is NEVER reached with invalid data
        // You write ZERO ModelState.IsValid checks

        int newId = _bal.Add(emp);
        emp.EmpId = newId;
        return CreatedAtAction(nameof(GetById), new { id = newId }, emp);
    }
}

// What the automatic 400 response looks like (ProblemDetails format):
// {
//   "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
//   "title": "One or more validation errors occurred.",
//   "status": 400,
//   "errors": {
//     "EmpName": ["Employee name is required."],
//     "Salary":  ["Salary must be between 0 and 999999."]
//   }
// }

// Your AJAX error handler reads this:
// xhr.responseJSON.errors["EmpName"][0]  → "Employee name is required."
```

---

## 🔷 Data Annotations — The Validation Rules on Your Model

```csharp
using System.ComponentModel.DataAnnotations;

public class Employee
{
    public int EmpId { get; set; }

    // ── [Required] — must not be null or empty ────────────────────
    [Required(ErrorMessage = "Employee name is required.")]
    public string EmpName { get; set; }

    // ── [StringLength] — min and max character length ─────────────
    [Required]
    [StringLength(100, MinimumLength = 2,
        ErrorMessage = "Name must be between 2 and 100 characters.")]
    public string EmpName { get; set; }

    // ── [Range] — numeric value within a range ────────────────────
    [Required]
    [Range(0, 999999.99,
        ErrorMessage = "Salary must be between 0 and 999,999.99.")]
    public decimal Salary { get; set; }

    // ── [Range] for integers ───────────────────────────────────────
    [Range(1, int.MaxValue,
        ErrorMessage = "Department ID must be a positive number.")]
    public int DepartmentId { get; set; }

    // ── [EmailAddress] — valid email format ───────────────────────
    [Required]
    [EmailAddress(ErrorMessage = "Please enter a valid email address.")]
    public string Email { get; set; }

    // ── [Phone] — valid phone number ──────────────────────────────
    [Phone(ErrorMessage = "Please enter a valid phone number.")]
    public string Phone { get; set; }

    // ── [Url] — valid URL ─────────────────────────────────────────
    [Url(ErrorMessage = "Please enter a valid URL.")]
    public string LinkedInProfile { get; set; }

    // ── [MinLength] / [MaxLength] — for strings and arrays ─────────
    [MinLength(2,  ErrorMessage = "At least 2 characters required.")]
    [MaxLength(50, ErrorMessage = "Cannot exceed 50 characters.")]
    public string Department { get; set; }

    // ── [RegularExpression] — must match a pattern ─────────────────
    [RegularExpression(@"^EMP-\d{4}$",
        ErrorMessage = "Employee code must be in format: EMP-0001")]
    public string EmpCode { get; set; }

    // ── [Compare] — must equal another property's value ───────────
    // Used for confirm-password scenarios:
    [Compare("Password",
        ErrorMessage = "Passwords do not match.")]
    public string ConfirmPassword { get; set; }

    // ── [DataType] — hints for display and input type ─────────────
    [DataType(DataType.Password)]
    public string Password { get; set; }
    // → <input type="password" /> generated by asp-for

    [DataType(DataType.Date)]
    public DateTime JoiningDate { get; set; }
    // → <input type="date" />

    [DataType(DataType.Currency)]
    public decimal Salary { get; set; }
    // → formatted as currency in display

    // ── Stacking multiple rules on one property ────────────────────
    [Required(ErrorMessage = "Email is required.")]
    [EmailAddress(ErrorMessage = "Invalid email format.")]
    [MaxLength(100, ErrorMessage = "Email cannot exceed 100 characters.")]
    public string Email { get; set; }
}
```

---

## 🔷 Displaying Validation Errors in Views

```html
<!-- Views/Employee/Create.cshtml -->
@model Employee

<form asp-action="Create" method="post">

    <!-- ── Summary of ALL errors at top of form ──────────────── -->
    <div asp-validation-summary="ModelOnly" class="alert alert-danger"></div>
    <!--
        "None"       → don't show anything
        "ModelOnly"  → show only non-property errors (added via
                       ModelState.AddModelError("", "message"))
        "All"        → show every error (property + model level)
    -->

    <!-- ── Per-field validation message ──────────────────────── -->
    <div class="form-group">
        <label asp-for="EmpName"></label>
        <input asp-for="EmpName" class="form-control" />
        <span asp-validation-for="EmpName" class="text-danger"></span>
        <!-- ↑ Shows error for EmpName field only when invalid -->
    </div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Salary"></label>
        <input asp-for="Salary" class="form-control" />
        <span asp-validation-for="Salary" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Save</button>
</form>

<!-- ── Client-side validation scripts — REQUIRED ─────────────── -->
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
    <!--
        Includes:
          jquery.validate.js
          jquery.validate.unobtrusive.js

        These read the data-val-* attributes generated by asp-for
        and validate client-side before form is even submitted.
        Server validates again regardless — client = UX, server = security.
    -->
}
```

---

## 🔷 Client-Side vs Server-Side Validation

```
CLIENT-SIDE (browser, JavaScript):
──────────────────────────────────────────────────────────────
When:    Before form is submitted
How:     jquery.validate reads data-val-* HTML attributes
         generated by asp-for Tag Helper
Example: <input data-val="true"
                data-val-required="Name is required"
                data-val-length="2 to 100 chars"
                id="EmpName" name="EmpName" />

Purpose: ✅ Better UX — instant feedback without round trip
         ✅ Reduces server load (catches obvious errors early)

Weakness:❌ Can be bypassed — user disables JS, uses Postman,
            or tampers with the request. NEVER rely on it alone.

SERVER-SIDE (ASP.NET Core):
──────────────────────────────────────────────────────────────
When:    After form/AJAX reaches your server
How:     Data Annotations on model, checked after Model Binding
Purpose: ✅ Always runs — cannot be bypassed
         ✅ The authoritative source of truth
         ✅ Required for security

GOLDEN RULE:
  Always validate server-side.
  Client-side is only for user experience, never for security.
```

---

## 🔷 Custom Validation — ValidationAttribute

```csharp
// When built-in attributes don't cover your rule:

// ── Custom attribute: date must be in the future ───────────────
public class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(
        object value, ValidationContext context)
    {
        if (value == null)
            return ValidationResult.Success; // [Required] handles nulls

        if (value is DateTime date)
        {
            return date > DateTime.Today
                ? ValidationResult.Success
                : new ValidationResult(
                    ErrorMessage ?? "Date must be in the future.");
        }
        return new ValidationResult("Invalid date value.");
    }
}

// ── Custom attribute: end date must be after start date ────────
public class DateAfterAttribute : ValidationAttribute
{
    private readonly string _startDateProperty;

    public DateAfterAttribute(string startDatePropertyName)
        => _startDateProperty = startDatePropertyName;

    protected override ValidationResult IsValid(
        object value, ValidationContext context)
    {
        var startProp = context.ObjectType.GetProperty(_startDateProperty);
        var startDate = startProp?.GetValue(context.ObjectInstance) as DateTime?;
        var endDate   = value as DateTime?;

        if (!startDate.HasValue || !endDate.HasValue)
            return ValidationResult.Success;

        return endDate > startDate
            ? ValidationResult.Success
            : new ValidationResult(
                ErrorMessage ?? $"Must be after {_startDateProperty}.");
    }
}

// ── Usage ─────────────────────────────────────────────────────
public class LeaveRequest
{
    [Required]
    [FutureDate(ErrorMessage = "Start date must be a future date.")]
    public DateTime StartDate { get; set; }

    [Required]
    [DateAfter("StartDate", ErrorMessage = "End date must be after start date.")]
    public DateTime EndDate { get; set; }
}
```

---

## 🔷 Business Rule Validation Inside the Action

```csharp
// Data Annotations cover FORMAT rules (required, range, email).
// Business rules that need DB checks go INSIDE the action:

[HttpPost]
public IActionResult Create([FromBody] CreateEmployeeRequest req)
{
    // [ApiController] already handled annotation validation.
    // Now check BUSINESS RULES:

    // Rule 1: Email must be unique in DB
    if (_bal.EmailExists(req.Email))
    {
        ModelState.AddModelError("Email",
            "An employee with this email already exists.");
        return BadRequest(ModelState);
    }

    // Rule 2: Employee code must match department prefix
    var dept = _deptBal.GetById(req.DepartmentId);
    if (!req.EmpCode.StartsWith(dept.CodePrefix))
    {
        return UnprocessableEntity(new
        {
            message = $"Employee code must start with '{dept.CodePrefix}' for this department."
        });
        // 422 Unprocessable Entity = valid format, business rule failed
    }

    // Rule 3: Salary must not exceed department budget cap
    if (_deptBal.ExceedsBudgetCap(req.DepartmentId, req.Salary))
    {
        return BadRequest(new
        {
            message = "Salary exceeds the budget cap for this department."
        });
    }

    // All validation passed — safe to create
    int newId = _bal.Add(req);
    return CreatedAtAction(nameof(GetById), new { id = newId }, new { empId = newId });
}
```

---

## 🔷 Validation in AJAX — Reading Errors

```javascript
$.ajax({
    url:         '/api/employees',
    type:        'POST',
    contentType: 'application/json',
    data:        JSON.stringify(empData),
    success: function(created) {
        alert('Created! ID: ' + created.empId);
        refreshGrid();
    },
    error: function(xhr) {
        if (xhr.status === 400) {
            var response = xhr.responseJSON;

            // ── [ApiController] ProblemDetails format ─────────────
            if (response.errors) {
                var messages = [];
                $.each(response.errors, function(field, errors) {
                    $.each(errors, function(i, msg) {
                        messages.push(field + ': ' + msg);
                        // Show inline next to the field:
                        $('#err' + field).text(msg).show();
                    });
                });
                alert('Validation errors:\n' + messages.join('\n'));
            }
            // ── Custom message format ────────────────────────────
            else if (response.message) {
                alert(response.message);
            }
        } else if (xhr.status === 422) {
            alert('Business rule error: ' + xhr.responseJSON.message);
        } else {
            alert('Server error: ' + xhr.status);
        }
    }
});
```

---

## 🔷 Validation Flow — The Complete Picture

```
Client submits:  POST /api/employees
Body: { "empName": "", "salary": -500, "email": "notanemail" }
                         ↓
┌──────────────────────────────────────────────────────────────┐
│  MODEL BINDING                                               │
│  emp.EmpName = ""                                           │
│  emp.Salary  = -500                                         │
│  emp.Email   = "notanemail"                                 │
└──────────────────────────────┬───────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────┐
│  DATA ANNOTATION VALIDATION                                  │
│  EmpName: [Required] FAILS — empty string                   │
│  Salary:  [Range(0,999999)] FAILS — negative                │
│  Email:   [EmailAddress] FAILS — not valid email            │
│                                                              │
│  ModelState.IsValid = false                                  │
│  ModelState.Errors has 3 entries                            │
└──────────────────────────────┬───────────────────────────────┘
                               │
              [ApiController] present?
                               │
               ┌───────────────┴─────────────────┐
               │ YES                              │ NO
               ▼                                  ▼
  400 returned automatically         Your action runs
  Action NEVER called                You check ModelState.IsValid
                                     Return View(emp) if invalid
               │
               ▼
  {
    "errors": {
      "EmpName": ["Employee name is required."],
      "Salary":  ["Salary must be between 0 and 999,999.99."],
      "Email":   ["Please enter a valid email address."]
    }
  }
               │
               ▼
  AJAX error callback
  xhr.status = 400
  xhr.responseJSON.errors
```

---

## 🔷 ModelState Methods — Complete Reference

```csharp
// ── Reading state ──────────────────────────────────────────────
ModelState.IsValid                    // true if zero errors
ModelState.ErrorCount                 // total error count
ModelState.ContainsKey("EmpName")    // true if key exists

// Iterate all errors:
foreach (var key in ModelState.Keys)
{
    foreach (var error in ModelState[key].Errors)
    {
        Console.WriteLine($"{key}: {error.ErrorMessage}");
    }
}

// ── Adding errors manually ─────────────────────────────────────
// Property-level error (shows next to that field's asp-validation-for):
ModelState.AddModelError("Email", "Email already exists.");

// Model-level error (shows in asp-validation-summary="ModelOnly"):
ModelState.AddModelError("", "Unable to save. Please try again.");

// ── Removing ──────────────────────────────────────────────────
ModelState.Remove("EmpId");           // remove errors for one key
ModelState.Clear();                   // remove ALL errors (resets IsValid to true)

// ── Manual validation ─────────────────────────────────────────
// Force re-validation of a specific object:
TryValidateModel(emp);               // populates ModelState for emp
```

---

## ⭐ Interview Quick-Fire

| Question                                                               | Answer                                                                                                                         |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| What is Model Validation?                                              | The automatic check of Data Annotation rules on a bound model — results stored in `ModelState`                              |
| What is `ModelState.IsValid`?                                        | A bool —`true`if zero validation errors,`false`if any rule failed                                                         |
| Does `[ApiController]`affect validation?                             | ✅ Yes — automatically returns 400 when `ModelState.IsValid`is false, before your action runs                               |
| How do you check validation in an MVC controller?                      | `if (!ModelState.IsValid) return View(model);`— manual check required                                                       |
| What is the difference between client-side and server-side validation? | Client-side = UX, runs in browser JS, can be bypassed. Server-side = security, always runs, can never be bypassed              |
| How do you show validation errors in a Razor view?                     | `<span asp-validation-for="PropName">`per field and `<div asp-validation-summary="ModelOnly">`for summary                  |
| How do you add a custom validation error inside an action?             | `ModelState.AddModelError("FieldName", "message")`then return `BadRequest(ModelState)`                                     |
| What HTTP status code does failed validation return?                   | 400 Bad Request                                                                                                                |
| What is the ProblemDetails format?                                     | Standardized JSON error response with `errors`dictionary —`{ "errors": { "EmpName": ["required"] } }`                     |
| When should you validate inside the action vs in Data Annotations?     | Annotations for format rules (required, range, email). Action for business rules needing DB checks (unique email, budget caps) |
