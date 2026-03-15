
# 04 — FluentValidation

---

## 🎯 One-Line Definition

> **FluentValidation is a NuGet library that moves all validation rules OUT of the model into a dedicated Validator class — rules are written as readable C# method chains, keeping models clean and enabling complex validation logic that Data Annotations can't express.**

---

## 🔷 Data Annotations vs FluentValidation — The Key Problem

```
DATA ANNOTATIONS (on the model):
──────────────────────────────────────────────────────────
public class Employee
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    [RegularExpression(@"complicated pattern")]
    public string Name { get; set; }

    [Required]
    [Range(10000, 1000000)]
    public decimal Salary { get; set; }

    // Model is now cluttered with validation noise
    // Can't express: "If Role is Manager, Salary must be > 50000"
    // Can't DB-check: "Email must not already exist"
    // Can't reuse rules across different models
}

FLUENTVALIDATION (separate class):
──────────────────────────────────────────────────────────
public class Employee
{
    public string Name   { get; set; }   // clean — no annotations
    public decimal Salary { get; set; }  // clean
    public string Role   { get; set; }   // clean
}

public class EmployeeValidator : AbstractValidator<Employee>
{
    public EmployeeValidator()
    {
        RuleFor(e => e.Name).NotEmpty().Length(2, 100);
        RuleFor(e => e.Salary).InclusiveBetween(10000, 1000000);
        // Conditional:
        When(e => e.Role == "Manager", () => {
            RuleFor(e => e.Salary).GreaterThan(50000);
        });
    }
}
```

---

## 🔷 Why Use FluentValidation?

```
✅ Rules in a dedicated class — model stays clean
✅ Complex conditions — When(), Unless(), depends on other fields
✅ Async rules — check DB for unique email asynchronously
✅ Reusable validators — one validator used in multiple places
✅ Strongly typed — full IntelliSense on property names
✅ Better error messages — full sentence construction
✅ Easier unit testing — test the validator independently
✅ No limit — Data Annotations can't express "Salary > 50k if Manager"
```

---

## 🔷 Step 1 — Install FluentValidation

```bash
# NuGet Package Manager Console:
Install-Package FluentValidation.AspNetCore

# .NET CLI:
dotnet add package FluentValidation.AspNetCore
```

---

## 🔷 Step 2 — Register in Program.cs

```csharp
// Program.cs
using FluentValidation;
using FluentValidation.AspNetCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();

// Register FluentValidation — scans assembly for all validators
builder.Services
    .AddFluentValidationAutoValidation()     // auto-validate on model binding
    .AddFluentValidationClientsideAdapters() // client-side validation support
    .AddValidatorsFromAssemblyContaining<EmployeeValidator>();
    // ↑ finds all classes that extend AbstractValidator<T> in the assembly

var app = builder.Build();
// ... rest of Program.cs
```

---

## 🔷 Step 3 — Create a Validator Class

```csharp
using FluentValidation;

public class EmployeeValidator : AbstractValidator<Employee>
{
    private readonly EmployeeDAL _dal;

    // Inject DAL for async DB checks (optional)
    public EmployeeValidator(EmployeeDAL dal)
    {
        _dal = dal;
        BuildRules();
    }

    private void BuildRules()
    {
        // ── Name ─────────────────────────────────────────────────
        RuleFor(e => e.Name)
            .NotEmpty().WithMessage("Name is required")
            .Length(2, 100).WithMessage("Name must be 2–100 characters")
            .Matches(@"^[a-zA-Z\s]+$").WithMessage("Name can only contain letters");

        // ── Role ──────────────────────────────────────────────────
        RuleFor(e => e.Role)
            .NotEmpty().WithMessage("Role is required")
            .MaximumLength(50).WithMessage("Role cannot exceed 50 characters");

        // ── Email ─────────────────────────────────────────────────
        RuleFor(e => e.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Enter a valid email address");

        // ── Salary ────────────────────────────────────────────────
        RuleFor(e => e.Salary)
            .NotEmpty().WithMessage("Salary is required")
            .InclusiveBetween(10000, 1000000)
            .WithMessage("Salary must be between ₹10,000 and ₹10,00,000");

        // ── HireDate ──────────────────────────────────────────────
        RuleFor(e => e.HireDate)
            .NotEmpty().WithMessage("Hire date is required")
            .LessThanOrEqualTo(DateTime.Today)
            .WithMessage("Hire date cannot be in the future");
    }
}
```

---

## 🔷 Core Rule Methods — All You Need

### Nulls and Emptiness

```csharp
RuleFor(e => e.Name).NotNull();           // not null
RuleFor(e => e.Name).NotEmpty();          // not null AND not ""
RuleFor(e => e.Name).Empty();             // must be empty (rare)
```

### String Rules

```csharp
RuleFor(e => e.Name)
    .MinimumLength(2)                     // min 2 chars
    .MaximumLength(100)                   // max 100 chars
    .Length(2, 100)                       // between 2 and 100
    .Matches(@"^[A-Z]")                   // starts with uppercase
    .EmailAddress()                       // email format
    .Url()                                // URL format
    .Must(n => !n.Contains("admin"))      // custom check — no "admin" in name
    .WithMessage("Name cannot contain 'admin'");
```

### Numeric Rules

```csharp
RuleFor(e => e.Salary)
    .GreaterThan(0)                       // > 0
    .GreaterThanOrEqualTo(10000)          // >= 10000
    .LessThan(1000001)                    // < 1000001
    .LessThanOrEqualTo(1000000)           // <= 1000000
    .InclusiveBetween(10000, 1000000)     // 10000 ≤ x ≤ 1000000
    .ExclusiveBetween(9999, 1000001);     // 9999 < x < 1000001
```

### Date Rules

```csharp
RuleFor(e => e.HireDate)
    .NotEmpty().WithMessage("Hire date is required")
    .GreaterThan(new DateTime(2000, 1, 1))
    .WithMessage("Hire date must be after year 2000")
    .LessThanOrEqualTo(DateTime.Today)
    .WithMessage("Hire date cannot be in the future");
```

### Equal / NotEqual

```csharp
RuleFor(e => e.ConfirmPassword)
    .Equal(e => e.Password)
    .WithMessage("Passwords must match");

RuleFor(e => e.NewRole)
    .NotEqual(e => e.OldRole)
    .WithMessage("New role must be different from current role");
```

---

## 🔷 `Must()` — Custom Inline Rule

```csharp
// Must() takes a function that returns bool
// true = valid, false = invalid

RuleFor(e => e.Email)
    .NotEmpty()
    .EmailAddress()
    .Must(email => !email.EndsWith("@blocked.com"))
    .WithMessage("Emails from blocked.com are not allowed");

RuleFor(e => e.Salary)
    .Must((employee, salary) => {
        // Has access to the whole employee object
        if (employee.Role == "Intern") return salary <= 30000;
        return true;
    })
    .WithMessage("Intern salary cannot exceed ₹30,000");
```

---

## 🔷 `When()` / `Unless()` — Conditional Rules

```csharp
// Rule only applies WHEN condition is true
When(e => e.Role == "Manager", () =>
{
    RuleFor(e => e.Salary)
        .GreaterThan(50000)
        .WithMessage("Manager salary must be above ₹50,000");

    RuleFor(e => e.Phone)
        .NotEmpty()
        .WithMessage("Phone is required for Managers");
});

// Rule only applies UNLESS condition is true
Unless(e => e.IsContractor, () =>
{
    RuleFor(e => e.DeptId)
        .GreaterThan(0)
        .WithMessage("Full-time employees must be assigned a department");
});

// On a specific rule:
RuleFor(e => e.ManagerId)
    .NotNull()
    .When(e => e.Role != "CEO")
    .WithMessage("All employees except CEO must have a manager");
```

---

## 🔷 `MustAsync()` — Async Rules (DB Checks)

```csharp
public class EmployeeValidator : AbstractValidator<Employee>
{
    private readonly EmployeeDAL _dal;

    public EmployeeValidator(EmployeeDAL dal)
    {
        _dal = dal;

        // Async DB check — email must be unique
        RuleFor(e => e.Email)
            .NotEmpty()
            .EmailAddress()
            .MustAsync(async (employee, email, cancellationToken) =>
            {
                // Return true = valid (email does NOT exist for another employee)
                bool exists = await _dal.EmailExistsAsync(email, employee.Id);
                return !exists;
            })
            .WithMessage("This email is already registered");
    }
}
```

---

## 🔷 Using FluentValidation in the Controller

Once registered in `Program.cs` with `AddFluentValidationAutoValidation()`, it works exactly like Data Annotations — no controller changes needed:

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Employee emp)
{
    // ModelState.IsValid is automatically populated by FluentValidation
    if (!ModelState.IsValid) return View(emp);

    string result = _bal.Insert(emp);

    if (result == "success")
    {
        TempData["Success"] = "Employee added!";
        return RedirectToAction(nameof(Index));
    }

    ModelState.AddModelError("", result);
    return View(emp);
}
```

---

## 🔷 Using FluentValidation Manually (Without Auto-Registration)

```csharp
// Inject and call manually when you need full control
public class EmployeeController : Controller
{
    private readonly EmployeeBAL _bal;
    private readonly IValidator<Employee> _validator;

    public EmployeeController(EmployeeBAL bal, IValidator<Employee> validator)
    {
        _bal       = bal;
        _validator = validator;
    }

    [HttpPost]
    public async Task<IActionResult> Create(Employee emp)
    {
        // Manually validate
        ValidationResult result = await _validator.ValidateAsync(emp);

        if (!result.IsValid)
        {
            // Add errors to ModelState manually
            result.AddToModelState(ModelState, null);
            return View(emp);
        }

        _bal.Insert(emp);
        TempData["Success"] = "Employee added!";
        return RedirectToAction(nameof(Index));
    }
}
```

---

## 🔷 Unit Testing a Validator

```csharp
// One of the main advantages of FluentValidation:
// test your validation rules without HTTP or a database

[Test]
public void Salary_Below_Minimum_Should_Fail()
{
    var validator = new EmployeeValidator(/* mock dal */);
    var emp       = new Employee { Name = "Alice", Role = "Dev", Salary = 5000 };

    ValidationResult result = validator.Validate(emp);

    Assert.That(result.IsValid, Is.False);
    Assert.That(result.Errors.Any(e => e.PropertyName == "Salary"), Is.True);
}

[Test]
public void Valid_Employee_Should_Pass()
{
    var validator = new EmployeeValidator(/* mock dal */);
    var emp       = new Employee
    {
        Name     = "Alice",
        Role     = "Developer",
        Email    = "alice@company.com",
        Salary   = 75000,
        HireDate = DateTime.Today.AddMonths(-6)
    };

    ValidationResult result = validator.Validate(emp);

    Assert.That(result.IsValid, Is.True);
}
```

---

## 🔷 Data Annotations vs FluentValidation — When to Use Which

|                    | Data Annotations                  | FluentValidation                            |
| ------------------ | --------------------------------- | ------------------------------------------- |
| Location           | On the model                      | Separate Validator class                    |
| Complex conditions | ❌ Not possible                   | ✅`When()`,`Unless()`                   |
| Async/DB checks    | ❌ Not supported                  | ✅`MustAsync()`                           |
| Cross-field rules  | ❌ Only `[Compare]`             | ✅ Full access to whole model               |
| Reusability        | ❌ Tied to the model              | ✅ Reuse across controllers                 |
| Unit testing       | Hard — needs HTTP stack          | ✅ Easy — instantiate and validate         |
| Setup required     | ❌ None — built-in               | ✅ NuGet + Program.cs                       |
| Best for           | Simple models with standard rules | Complex forms, DB checks, conditional rules |

---

## 🔷 Common Rule Chaining Patterns

```csharp
// Full employee validator with all common patterns:
public class EmployeeValidator : AbstractValidator<Employee>
{
    public EmployeeValidator(EmployeeDAL dal)
    {
        RuleFor(e => e.Name)
            .NotEmpty().WithMessage("Name is required")
            .Length(2, 100).WithMessage("2–100 characters")
            .Matches(@"^[a-zA-Z\s\-']+$").WithMessage("Letters only");

        RuleFor(e => e.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MustAsync(async (emp, email, ct) =>
                !(await dal.EmailExistsAsync(email, emp.Id)))
            .WithMessage("Email already registered");

        RuleFor(e => e.Salary)
            .InclusiveBetween(10000, 1000000)
            .WithMessage("₹10,000 – ₹10,00,000");

        RuleFor(e => e.HireDate)
            .LessThanOrEqualTo(DateTime.Today)
            .WithMessage("Cannot be in the future");

        When(e => e.Role == "Manager", () =>
        {
            RuleFor(e => e.Salary)
                .GreaterThan(50000)
                .WithMessage("Manager salary must exceed ₹50,000");
        });
    }
}
```

---

## ⚠️ Common FluentValidation Mistakes

| Mistake                                                        | What Happens                                    | Fix                                                                     |
| -------------------------------------------------------------- | ----------------------------------------------- | ----------------------------------------------------------------------- |
| Not calling `AddFluentValidationAutoValidation()`            | Validators registered but never run             | Call both auto-validation and cli-side adapters in `Program.cs`       |
| Using both Data Annotations AND FluentValidation on same model | Duplicate validation — confusing double errors | Pick one approach per model — FluentValidation can replace annotations |
| Forgetting `WithMessage()`                                   | Default message — property name only: "Salary" | Always add `WithMessage`for user-friendly text                        |
| Async validator without `await`                              | Deadlock or sync-over-async issues              | Use `async/await`properly in `MustAsync()`                          |

---

## ⭐ Interview Quick-Fire

| Question                                                 | Answer                                                                                                 |
| -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| What is FluentValidation?                                | NuGet library that moves validation into a dedicated Validator class with method-chain rules           |
| What class do you extend to create a validator?          | `AbstractValidator<T>`where T is the model                                                           |
| How do you apply rules?                                  | `RuleFor(e => e.Name).NotEmpty().Length(2, 100).WithMessage("...")`                                  |
| What does `When()`do?                                  | Applies rules conditionally — only when the condition is true                                         |
| What does `MustAsync()`do?                             | Runs an async function for complex checks like DB uniqueness                                           |
| How does FluentValidation integrate with `ModelState`? | With `AddFluentValidationAutoValidation()`— validation errors automatically populate `ModelState` |
| Main advantage over Data Annotations?                    | Supports complex conditions, async DB checks, cross-field rules, is easily unit-testable               |
| NuGet package name?                                      | `FluentValidation.AspNetCore`                                                                        |
