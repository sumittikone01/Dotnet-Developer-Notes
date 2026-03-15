# JSON in ASP.NET Core

---

## 📌 Overview

In ASP.NET Core, JSON is the default format for Web API communication. The framework handles most serialization automatically — but you need to understand **how and where** to configure it, what the defaults are, and what common issues arise.

```
┌──────────────────────────────────────────────────────────────┐
│                    JSON FLOW IN ASP.NET CORE                  │
│                                                               │
│  Browser                      Server (Controller)            │
│  ──────────                   ──────────────────             │
│                                                               │
│  JS Object  → stringify() → [JSON string] → deserialize →   │
│                                              C# Object       │
│                                              (Model)         │
│                                                               │
│  JS Object  ← parse()    ← [JSON string] ← serialize   ←    │
│                                              C# Object       │
└──────────────────────────────────────────────────────────────┘
```

ASP.NET Core uses **System.Text.Json** (built-in, .NET 5+) by default.
Optionally, you can use **Newtonsoft.Json** (more features, more control).

---

## 🔑 Default Serializer — System.Text.Json

### What it does automatically

* C# objects → JSON when you `return Ok(object)` or `return Json(object)`
* JSON request body → C# objects via `[FromBody]`
* Handles primitives, nested objects, lists, enums
* Built into .NET — no NuGet package needed

### Default Behavior (Important to Know)

```csharp
public class Employee
{
    public int    Id         { get; set; }   // → "id"       (camelCase by default in some configs)
    public string FirstName  { get; set; }   // → "firstName"
    public string Department { get; set; }   // → "department"
    public decimal Salary    { get; set; }   // → "salary"
}
```

**Default System.Text.Json produces PascalCase:**

```json
{ "Id": 1, "FirstName": "Alice", "Department": "IT", "Salary": 75000 }
```

**After configuring camelCase (recommended for JavaScript):**

```json
{ "id": 1, "firstName": "Alice", "department": "IT", "salary": 75000 }
```

---

## 🔑 Configuring JSON in Program.cs

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews()
    .AddJsonOptions(options =>
    {
        // ── Naming Policy ───────────────────────────────────
        // CamelCase: "FirstName" → "firstName"
        // Matches JavaScript conventions and Kendo UI expectations
        options.JsonSerializerOptions.PropertyNamingPolicy =
            JsonNamingPolicy.CamelCase;

        // ── Null Handling ────────────────────────────────────
        // Ignore null values — don't include them in JSON output
        // "salary": null won't appear if you set this
        options.JsonSerializerOptions.DefaultIgnoreCondition =
            System.Text.Json.Serialization.JsonIgnoreCondition.WhenWritingNull;

        // ── Case-insensitive Deserialization ─────────────────
        // Accepts "Name", "name", "NAME" all as the same property
        options.JsonSerializerOptions.PropertyNameCaseInsensitive = true;

        // ── Number Handling ───────────────────────────────────
        // Allow reading numbers written as strings: "salary": "75000"
        options.JsonSerializerOptions.NumberHandling =
            System.Text.Json.Serialization.JsonNumberHandling.AllowReadingFromString;

        // ── Enum Handling ─────────────────────────────────────
        // Serialize enums as their string names, not integers
        // Status.Active → "Active" instead of 0
        options.JsonSerializerOptions.Converters.Add(
            new System.Text.Json.Serialization.JsonStringEnumConverter());

        // ── Circular Reference Handling ───────────────────────
        // Prevents stack overflow when objects reference each other
        options.JsonSerializerOptions.ReferenceHandler =
            System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles;
    });
```

---

## 🔑 Returning JSON from Controllers

### Method 1 — return Ok() from ApiController

```csharp
[ApiController]
[Route("api/[controller]")]
public class EmployeeApiController : ControllerBase
{
    // Returns single object as JSON
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var emp = _db.Employees.Find(id);
        if (emp == null) return NotFound();
        return Ok(emp);
        // Automatically serialized to JSON:
        // { "id": 1, "name": "Alice", "salary": 75000 }
    }

    // Returns list as JSON array
    [HttpGet]
    public IActionResult GetAll()
    {
        var list = _db.Employees.ToList();
        return Ok(list);
        // [ {"id":1,...}, {"id":2,...} ]
    }

    // Returns anonymous object
    [HttpGet("summary")]
    public IActionResult GetSummary()
    {
        return Ok(new
        {
            total       = _db.Employees.Count(),
            avgSalary   = _db.Employees.Average(e => e.Salary),
            departments = _db.Employees.Select(e => e.Department).Distinct().ToList()
        });
    }
}
```

### Method 2 — return Json() from MVC Controller (for Kendo)

```csharp
public class EmployeeController : Controller
{
    // For Kendo Grid — returns DataSourceResult wrapped JSON
    [HttpPost]
    public JsonResult Read([DataSourceRequest] DataSourceRequest request)
    {
        var result = _db.Employees.ToDataSourceResult(request);
        return Json(result);
        // { "Data": [...], "Total": 150, "Errors": null }
    }

    // For dropdowns — returns simple array
    public JsonResult GetDepartments()
    {
        var depts = _db.Departments
            .Select(d => new { value = d.Id, text = d.Name })
            .ToList();
        return Json(depts);
        // [ {"value":1,"text":"IT"}, {"value":2,"text":"HR"} ]
    }
}
```

---

## 🔑 Receiving JSON in Controllers — [FromBody]

When an AJAX POST sends a JSON body, use `[FromBody]` to deserialize it into a C# model.

```csharp
// ── Simple model binding ─────────────────────────────────────
[HttpPost]
public IActionResult Create([FromBody] Employee employee)
{
    // ASP.NET Core automatically deserializes the JSON body
    // into the Employee object

    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Ok(employee);
}

// ── Anonymous / dynamic body (avoid in production) ───────────
[HttpPost]
public IActionResult UpdateSalary([FromBody] dynamic body)
{
    // Works but loses type safety — prefer a typed DTO
    int id       = (int)body.id;
    decimal sal  = (decimal)body.salary;
    // ...
}

// ── DTO (Data Transfer Object) approach — best practice ──────
public class UpdateSalaryRequest
{
    public int     EmployeeId { get; set; }
    public decimal NewSalary  { get; set; }
    public string  Reason     { get; set; }
}

[HttpPost("update-salary")]
public IActionResult UpdateSalary([FromBody] UpdateSalaryRequest request)
{
    // Clean, typed, validated
    var emp = _db.Employees.Find(request.EmployeeId);
    emp.Salary = request.NewSalary;
    _db.SaveChanges();
    return Ok(new { success = true, newSalary = emp.Salary });
}
```

---

## 🔑 Manual Serialization with System.Text.Json

For cases outside of controllers where you need to serialize/deserialize manually:

```csharp
using System.Text.Json;

// ── Serialize (C# → JSON string) ────────────────────────────
var employee = new Employee { Id = 1, Name = "Alice", Salary = 75000 };

string json = JsonSerializer.Serialize(employee);
// '{"Id":1,"Name":"Alice","Salary":75000}'

// With options (camelCase + pretty print)
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    WriteIndented = true
};
string prettyJson = JsonSerializer.Serialize(employee, options);
// {
//   "id": 1,
//   "name": "Alice",
//   "salary": 75000
// }

// ── Deserialize (JSON string → C#) ────────────────────────────
string jsonInput = '{"id":1,"name":"Alice","salary":75000}';

var emp = JsonSerializer.Deserialize<Employee>(jsonInput);
Console.WriteLine(emp.Name);    // "Alice"

// Deserialize a list
string listJson = '[{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]';
var list = JsonSerializer.Deserialize<List<Employee>>(listJson);
Console.WriteLine(list.Count);  // 2

// Deserialize to anonymous type (using a template)
var template = new { Id = 0, Name = "" };
var result = JsonSerializer.Deserialize(jsonInput, template.GetType());
```

---

## 🔑 Data Annotations That Affect JSON

```csharp
using System.Text.Json.Serialization;

public class Employee
{
    public int Id { get; set; }

    // Change the JSON key name
    [JsonPropertyName("full_name")]
    public string Name { get; set; }
    // → "full_name": "Alice"   (instead of "name")

    // Always include even if null
    [JsonIgnoreCondition(JsonIgnoreCondition.Never)]
    public string? Department { get; set; }

    // Exclude this property from JSON entirely
    [JsonIgnore]
    public string PasswordHash { get; set; }
    // ← never appears in JSON output

    // Include only when not null
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public string? Notes { get; set; }

    // Serialize enum as string
    [JsonConverter(typeof(JsonStringEnumConverter))]
    public EmployeeStatus Status { get; set; }
    // → "status": "Active"  (instead of "status": 0)
}

public enum EmployeeStatus { Active, Inactive, OnLeave }
```

---

## 🔑 Newtonsoft.Json (Alternative — More Features)

For complex scenarios, many teams use Newtonsoft.Json (Json.NET):

```csharp
// Install: dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson

// Program.cs
builder.Services.AddControllersWithViews()
    .AddNewtonsoftJson(options =>
    {
        // camelCase
        options.SerializerSettings.ContractResolver =
            new Newtonsoft.Json.Serialization.CamelCasePropertyNamesContractResolver();

        // Ignore nulls
        options.SerializerSettings.NullValueHandling =
            Newtonsoft.Json.NullValueHandling.Ignore;

        // Enum as string
        options.SerializerSettings.Converters.Add(
            new Newtonsoft.Json.Converters.StringEnumConverter());

        // Date format
        options.SerializerSettings.DateFormatString = "yyyy-MM-dd";
    });
```

```csharp
// Manual serialization with Newtonsoft
using Newtonsoft.Json;

string json  = JsonConvert.SerializeObject(employee);
var    emp   = JsonConvert.DeserializeObject<Employee>(json);
var    list  = JsonConvert.DeserializeObject<List<Employee>>(jsonArray);

// Newtonsoft also has [JsonProperty], [JsonIgnore] attributes
public class Employee
{
    [JsonProperty("full_name")]
    public string Name { get; set; }

    [JsonIgnore]
    public string Password { get; set; }
}
```

### System.Text.Json vs Newtonsoft.Json

| Feature                     | System.Text.Json | Newtonsoft.Json   |
| --------------------------- | ---------------- | ----------------- |
| Included in .NET            | ✅ Yes           | ❌ NuGet package  |
| Performance                 | Faster           | Slower            |
| Features                    | Fewer            | More              |
| Kendo .ToDataSourceResult() | Works            | Works             |
| Dynamic object support      | Limited          | Full              |
| Date handling               | Basic            | Advanced          |
| Use for                     | New projects     | Complex scenarios |

---

## ⚠️ Common JSON Issues in ASP.NET Core

### Issue 1 — Dates serialize with timezone offset

```json
// Problem:
"hireDate": "2021-06-15T00:00:00+05:30"

// Fix in Program.cs:
options.JsonSerializerOptions.Converters.Add(new DateOnlyJsonConverter());
// Or store dates as strings in "yyyy-MM-dd" format
```

### Issue 2 — Circular reference (Employee → Department → Employees...)

```csharp
// Error: A possible object cycle was detected

// Fix 1: Ignore cycles
options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;

// Fix 2: Use DTOs that don't have circular references
// Fix 3: Add [JsonIgnore] to navigation property
public class Department
{
    public int Id { get; set; }
    public string Name { get; set; }

    [JsonIgnore]  // ← breaks the circle
    public List<Employee> Employees { get; set; }
}
```

### Issue 3 — PascalCase vs camelCase mismatch with Kendo

```csharp
// Kendo expects camelCase. If you return PascalCase:
// { "Id": 1, "Name": "Alice" }
// Kendo can't find the data because it looks for "id", "name"

// Fix — add to Program.cs:
options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
```

### Issue 4 — [FromBody] returns null

```javascript
// Client must set Content-Type header
$.ajax({
    contentType: "application/json",   // ← required!
    data: JSON.stringify(myObject),    // ← must stringify!
});
```

```csharp
// [FromBody] will be null if Content-Type is missing
// or if the body is form-encoded instead of JSON
```

---

## ❓ Interview Questions

**Q: What is the default JSON serializer in ASP.NET Core?**

> System.Text.Json, introduced in .NET 5. Before that, Newtonsoft.Json was the default.

**Q: How do you configure camelCase JSON in ASP.NET Core?**

> In `Program.cs`, call `.AddJsonOptions(o => o.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase)`.

**Q: What does [JsonIgnore] do?**

> It excludes a property from JSON serialization entirely — the property won't appear in the JSON output and won't be read from JSON input.

**Q: What attribute changes the JSON key name for a property?**

> `[JsonPropertyName("new_name")]` for System.Text.Json or `[JsonProperty("new_name")]` for Newtonsoft.Json.

**Q: How do you prevent circular reference errors in JSON serialization?**

> Set `ReferenceHandler = ReferenceHandler.IgnoreCycles` in JsonSerializerOptions, or use DTOs that don't have circular navigation properties.

**Q: What is a DTO?**

> Data Transfer Object — a simple class designed specifically for transferring data (usually as JSON). It contains only the properties needed for that specific request/response, avoiding circular references and exposing only what is necessary.

---
