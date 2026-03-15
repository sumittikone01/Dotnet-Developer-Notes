
# 03 — Data Annotations

---

## 🎯 One-Line Definition

> **Data Annotations are attributes placed directly on model properties to declare validation rules, display names, and data types — ASP.NET Core reads them automatically for both client-side and server-side validation.**

---

## 🔷 What Data Annotations Do

```
You write on the Model:
  [Required]
  [StringLength(100)]
  [Range(10000, 1000000)]
  public string Name { get; set; }

ASP.NET Core automatically:
  ✅ Validates on POST — populates ModelState with errors
  ✅ Generates client-side validation attributes on <input>
  ✅ Reads [Display(Name="...")] for labels and error messages
  ✅ Generates HTML type hints ([DataType(DataType.Date)] → type="date")

One annotation → two benefits: server + client validation
```

---

## 🔷 Namespace

```csharp
using System.ComponentModel.DataAnnotations;
// All standard annotations live here
// No NuGet package needed — built into .NET
```

---

## 🔷 Category 1 — Validation Annotations

### `[Required]`

```csharp
[Required]
public string Name { get; set; }
// → field must have a value (not null, not empty string)

[Required(ErrorMessage = "Employee name is required")]
public string Name { get; set; }
// → custom error message shown to user

[Required(ErrorMessage = "Name is required",
           AllowEmptyStrings = false)]
public string Name { get; set; }
// → AllowEmptyStrings = false (default) — whitespace-only strings fail too
```

### `[StringLength]`

```csharp
[StringLength(100)]
public string Name { get; set; }
// → max 100 characters

[StringLength(100, MinimumLength = 2,
    ErrorMessage = "Name must be between 2 and 100 characters")]
public string Name { get; set; }
// → min 2, max 100

[StringLength(50, ErrorMessage = "Role cannot exceed 50 characters")]
public string Role { get; set; }
```

### `[MinLength]` and `[MaxLength]`

```csharp
[MinLength(8, ErrorMessage = "Password must be at least 8 characters")]
public string Password { get; set; }

[MaxLength(200, ErrorMessage = "Notes cannot exceed 200 characters")]
public string Notes { get; set; }
```

### `[Range]`

```csharp
[Range(10000, 1000000,
    ErrorMessage = "Salary must be between ₹10,000 and ₹10,00,000")]
public decimal Salary { get; set; }

[Range(1, int.MaxValue, ErrorMessage = "ID must be a positive number")]
public int DeptId { get; set; }

[Range(0.0, 100.0, ErrorMessage = "Percentage must be between 0 and 100")]
public double PercentageScore { get; set; }

// Date range (using type double for DateTime comparison)
[Range(typeof(DateTime), "2000-01-01", "2099-12-31",
    ErrorMessage = "Date must be between 2000 and 2099")]
public DateTime HireDate { get; set; }
```

### `[EmailAddress]`

```csharp
[EmailAddress(ErrorMessage = "Please enter a valid email address")]
public string Email { get; set; }
// → validates format: user@domain.com
// → does NOT verify email actually exists
```

### `[Phone]`

```csharp
[Phone(ErrorMessage = "Please enter a valid phone number")]
public string Phone { get; set; }
// → validates common phone formats
```

### `[Url]`

```csharp
[Url(ErrorMessage = "Please enter a valid URL")]
public string WebsiteUrl { get; set; }
// → must start with http:// or https://
```

### `[Compare]`

```csharp
public string Password { get; set; }

[Compare("Password", ErrorMessage = "Passwords do not match")]
public string ConfirmPassword { get; set; }
// → ConfirmPassword must equal Password
// → perfect for registration / change password forms
```

### `[RegularExpression]`

```csharp
[RegularExpression(@"^[A-Z]{2}[0-9]{4}$",
    ErrorMessage = "Code must be 2 uppercase letters + 4 digits (e.g. AB1234)")]
public string EmployeeCode { get; set; }

[RegularExpression(@"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$",
    ErrorMessage = "Password needs 8+ chars, uppercase, lowercase, and a digit")]
public string Password { get; set; }
```

---

## 🔷 Category 2 — Display Annotations

### `[Display]`

```csharp
[Display(Name = "Full Name")]
public string Name { get; set; }
// → <label> shows "Full Name" instead of "Name"
// → error messages say "Full Name is required" instead of "Name is required"

[Display(Name = "Department")]
public int DeptId { get; set; }

[Display(Name = "Date of Joining")]
public DateTime HireDate { get; set; }
```

### `[DataType]`

```csharp
[DataType(DataType.Date)]
public DateTime HireDate { get; set; }
// → renders <input type="date"> → browser shows date picker

[DataType(DataType.Password)]
public string Password { get; set; }
// → renders <input type="password"> → hides input

[DataType(DataType.EmailAddress)]
public string Email { get; set; }
// → renders <input type="email"> → mobile keyboard shows @ key

[DataType(DataType.PhoneNumber)]
public string Phone { get; set; }
// → renders <input type="tel">

[DataType(DataType.Currency)]
public decimal Salary { get; set; }
// → formats for display as currency

[DataType(DataType.MultilineText)]
public string Notes { get; set; }
// → renders <textarea> in scaffolded forms
```

### `[DisplayFormat]`

```csharp
// Format value when displaying
[DisplayFormat(DataFormatString = "{0:C0}")]
public decimal Salary { get; set; }
// → displays as "₹75,000"

[DisplayFormat(DataFormatString = "{0:dd/MM/yyyy}", ApplyFormatInEditMode = true)]
public DateTime HireDate { get; set; }
// → displays as "15/06/2021"
// → ApplyFormatInEditMode = true → also formats in edit inputs

[DisplayFormat(NullDisplayText = "Not assigned")]
public string Department { get; set; }
// → shows "Not assigned" when value is null

// Common format strings:
//   {0:C0}          → currency, no decimals: ₹75,000
//   {0:N2}          → number, 2 decimals: 75,000.00
//   {0:P0}          → percentage: 75%
//   {0:dd/MM/yyyy}  → date: 15/06/2021
//   {0:MMM yyyy}    → date: Jun 2021
```

---

## 🔷 Category 3 — Schema / Scaffolding Annotations

```csharp
// Hidden in forms but still submitted
[HiddenInput]
public int Id { get; set; }

// Exclude from scaffolded forms entirely
[ScaffoldColumn(false)]
public DateTime CreatedAt { get; set; }

// Mark as primary key (for scaffolding / EF)
[Key]
public int Id { get; set; }
```

---

## 🔷 Complete Employee Model with All Annotations

```csharp
using System.ComponentModel.DataAnnotations;

public class Employee
{
    [HiddenInput]
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2,
        ErrorMessage = "Name must be 2–100 characters")]
    [Display(Name = "Full Name")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Role is required")]
    [StringLength(50)]
    [Display(Name = "Job Role")]
    public string Role { get; set; }

    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Enter a valid email address")]
    [StringLength(150)]
    [Display(Name = "Email Address")]
    public string Email { get; set; }

    [Phone(ErrorMessage = "Enter a valid phone number")]
    [Display(Name = "Phone Number")]
    public string Phone { get; set; }

    [Required(ErrorMessage = "Salary is required")]
    [Range(10000, 1000000,
        ErrorMessage = "Salary must be between ₹10,000 and ₹10,00,000")]
    [DisplayFormat(DataFormatString = "{0:C0}")]
    [Display(Name = "Monthly Salary")]
    public decimal Salary { get; set; }

    [DataType(DataType.Date)]
    [Display(Name = "Date of Joining")]
    [DisplayFormat(DataFormatString = "{0:dd/MM/yyyy}", ApplyFormatInEditMode = true)]
    public DateTime HireDate { get; set; }

    [Display(Name = "Department")]
    [Range(1, int.MaxValue, ErrorMessage = "Please select a department")]
    public int DeptId { get; set; }

    [Display(Name = "Active Employee")]
    public bool IsActive { get; set; } = true;

    [MaxLength(500, ErrorMessage = "Notes cannot exceed 500 characters")]
    [DataType(DataType.MultilineText)]
    [Display(Name = "Additional Notes")]
    public string Notes { get; set; }
}
```

---

## 🔷 How Annotations Connect to the View

```cshtml
@model Employee

<div class="mb-3">
    @* [Display(Name="Full Name")] → label reads "Full Name" automatically *@
    <label asp-for="Name" class="form-label"></label>

    @* [Required] → generates: required="required"
       [StringLength(100)] → generates: maxlength="100"
       [StringLength(MinimumLength=2)] → generates: minlength="2"
       All client-side attributes generated from annotations *@
    <input asp-for="Name" class="form-control" />

    @* Shows the [Required] error message if Name is empty *@
    <span asp-validation-for="Name" class="text-danger small"></span>
</div>

<div class="mb-3">
    @* [Range(10000, 1000000)] → generates: min="10000" max="1000000" *@
    <label asp-for="Salary" class="form-label"></label>
    <input asp-for="Salary" type="number" class="form-control" />
    <span asp-validation-for="Salary" class="text-danger small"></span>
</div>

@* [DataType(DataType.Date)] → generates: type="date" → date picker *@
<div class="mb-3">
    <label asp-for="HireDate" class="form-label"></label>
    <input asp-for="HireDate" class="form-control" />
    <span asp-validation-for="HireDate" class="text-danger small"></span>
</div>
```

---

## 🔷 All Validation Annotations — Quick Reference

| Annotation                       | Validates                   | Example                                   |
| -------------------------------- | --------------------------- | ----------------------------------------- |
| `[Required]`                   | Field has a value           | `[Required(ErrorMessage = "Required")]` |
| `[StringLength(max)]`          | Max + optional min length   | `[StringLength(100, MinimumLength=2)]`  |
| `[MinLength(n)]`               | Minimum length              | `[MinLength(8)]`                        |
| `[MaxLength(n)]`               | Maximum length              | `[MaxLength(200)]`                      |
| `[Range(min, max)]`            | Numeric range               | `[Range(1, 100)]`                       |
| `[EmailAddress]`               | Valid email format          | `[EmailAddress]`                        |
| `[Phone]`                      | Valid phone format          | `[Phone]`                               |
| `[Url]`                        | Valid URL                   | `[Url]`                                 |
| `[Compare("Prop")]`            | Must equal another property | `[Compare("Password")]`                 |
| `[RegularExpression(pattern)]` | Matches regex               | `[RegularExpression(@"^\d{6}$")]`       |

---

## 🔷 All Display Annotations — Quick Reference

| Annotation                  | Purpose                   | Example                                        |
| --------------------------- | ------------------------- | ---------------------------------------------- |
| `[Display(Name="...")]`   | Label text + error prefix | `[Display(Name = "Full Name")]`              |
| `[DataType(DataType.X)]`  | Input type hint           | `[DataType(DataType.Date)]`                  |
| `[DisplayFormat(...)]`    | Format when displaying    | `[DisplayFormat(DataFormatString="{0:C0}")]` |
| `[HiddenInput]`           | Render as hidden field    | `[HiddenInput]`                              |
| `[ScaffoldColumn(false)]` | Exclude from scaffolding  | `[ScaffoldColumn(false)]`                    |

---

## ⚠️ Common Data Annotation Mistakes

| Mistake                                      | What Happens                                                                    | Fix                                                           |
| -------------------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| No `[Required]`on non-nullable value types | `int`,`decimal`,`DateTime`are required by default — no annotation needed | Only add `[Required]`to `string`and nullable types        |
| `[Required]`on `int Id`                  | Id = 0 fails Required — always required even when valid                        | Id is `int`— already non-nullable. No `[Required]`needed |
| Missing `ErrorMessage`                     | Default messages like "The Name field is required" — not user-friendly         | Always add `ErrorMessage`for user-facing forms              |
| `[StringLength]`without `MinimumLength`  | Only max enforced                                                               | Add `MinimumLength`to also enforce minimum                  |

---

## ⭐ Interview Quick-Fire

| Question                                              | Answer                                                                        |
| ----------------------------------------------------- | ----------------------------------------------------------------------------- |
| What namespace are Data Annotations in?               | `System.ComponentModel.DataAnnotations`                                     |
| What does `[Required]`do?                           | Makes the field mandatory — ModelState.IsValid = false if empty              |
| What does `[StringLength(100, MinimumLength=2)]`do? | Validates text is between 2 and 100 characters                                |
| What does `[Range(10000, 1000000)]`do?              | Validates the value is between 10,000 and 1,000,000                           |
| What does `[Compare("Password")]`do?                | Validates that this field's value equals the Password field's value           |
| What does `[DataType(DataType.Date)]`do?            | Hints the input type — renders `type="date"`for date picker                |
| What does `[Display(Name="Full Name")]`do?          | Sets the label text and error message prefix to "Full Name"                   |
| Do annotations provide client-side validation?        | ✅ Yes — ASP.NET Core generates HTML5 validation attributes from annotations |
