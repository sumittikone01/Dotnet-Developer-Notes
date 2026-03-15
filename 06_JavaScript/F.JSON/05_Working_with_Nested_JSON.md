
# Working with Nested JSON

---

## 📌 Overview

Nested JSON means objects inside objects, arrays inside objects, objects inside arrays — any combination of depth. Real APIs almost always return nested JSON. Knowing how to read, write, access, and map it in both JavaScript and C# is essential daily work.

---

## 🔑 Types of Nesting

```json
// Level 1 — Object with simple values (not nested)
{ "id": 1, "name": "Alice" }

// Level 2 — Object with nested object
{ "id": 1, "name": "Alice", "address": { "city": "NY" } }

// Level 2 — Object with array
{ "id": 1, "name": "Alice", "skills": ["C#", "SQL"] }

// Level 3 — Object → Array → Objects
{
  "id": 1,
  "name": "Alice",
  "projects": [
    { "projectId": 10, "title": "HR System" },
    { "projectId": 11, "title": "Payroll App" }
  ]
}

// Level 3 — Object → Object → Array
{
  "employee": {
    "id": 1,
    "contact": {
      "phones": ["555-1111", "555-2222"]
    }
  }
}
```

---

## 🔑 Accessing Nested JSON in JavaScript

### The Dot Chain Rule

To access nested values, chain dots (or brackets) as deep as needed:

```javascript
var data = {
  id:   1,
  name: "Alice",
  address: {
    street: "123 Main St",
    city:   "New York",
    state:  "NY",
    geo: {
      lat: 40.7128,
      lng: -74.0060
    }
  },
  skills: ["C#", "SQL", "JavaScript"],
  projects: [
    { id: 10, title: "HR System",   active: true  },
    { id: 11, title: "Payroll App", active: false }
  ]
};

// ── Accessing nested objects ───────────────────────────────
console.log(data.address.city);              // "New York"
console.log(data.address.geo.lat);           // 40.7128
console.log(data["address"]["city"]);        // "New York" (bracket)

// ── Accessing arrays inside objects ───────────────────────
console.log(data.skills[0]);                 // "C#"
console.log(data.skills[2]);                 // "JavaScript"
console.log(data.skills.length);             // 3

// ── Accessing objects inside arrays ───────────────────────
console.log(data.projects[0].title);         // "HR System"
console.log(data.projects[1].active);        // false

// ── Iterating nested array ─────────────────────────────────
data.projects.forEach(function(project) {
  console.log(project.id + ": " + project.title);
});

// ── Using map on nested array ──────────────────────────────
var titles = data.projects.map(p => p.title);
// ["HR System", "Payroll App"]

// ── Filtering nested array ─────────────────────────────────
var activeProjects = data.projects.filter(p => p.active === true);
// [{ id: 10, title: "HR System", active: true }]
```

---

## 🔑 Safe Navigation — Avoiding "Cannot read property of undefined"

This is the most common runtime error when working with nested JSON.

```javascript
var data = {
  employee: {
    name: "Alice"
    // address is missing — undefined
  }
};

// ❌ Unsafe — throws TypeError if address is undefined
console.log(data.employee.address.city);
// TypeError: Cannot read properties of undefined (reading 'city')

// ✅ Option 1 — Check each level manually (old way)
if (data.employee && data.employee.address && data.employee.address.city) {
  console.log(data.employee.address.city);
}

// ✅ Option 2 — Optional chaining ?. (modern, ES2020)
console.log(data.employee?.address?.city);
// undefined  (no error if any level is missing)

// ✅ Option 3 — Optional chaining with default value (?? operator)
var city = data.employee?.address?.city ?? "City not provided";
console.log(city);  // "City not provided"

// ✅ Option 4 — for nested arrays
var firstSkill = data.employee?.skills?.[0] ?? "No skills listed";
```

---

## 🔑 Building Nested JSON to Send to Server

```javascript
// Build a complex nested object before sending via AJAX
var employeePayload = {
    name:       $("#Name").val(),
    department: $("#Department").val(),
    salary:     parseFloat($("#Salary").val()),

    // Nested object
    address: {
        street: $("#Street").val(),
        city:   $("#City").val(),
        state:  $("#State").val(),
        zip:    $("#Zip").val()
    },

    // Array of strings
    skills: $("#Skills").val().split(",").map(s => s.trim()),

    // Array of objects
    emergencyContacts: [
        {
            name:         $("#Contact1Name").val(),
            relationship: $("#Contact1Rel").val(),
            phone:        $("#Contact1Phone").val()
        }
    ]
};

// Send to server
$.ajax({
    url:         "/Employee/Create",
    type:        "POST",
    contentType: "application/json",
    data:        JSON.stringify(employeePayload),
    success: function(response) {
        console.log("Employee created with ID:", response.id);
    }
});
```

---

## 🔑 Mapping Nested JSON to C# Classes

Each nested object in JSON needs a corresponding C# class.

### JSON → C# Class Mapping

```json
{
  "id":   1,
  "name": "Alice Johnson",
  "department": "IT",
  "address": {
    "street": "123 Main St",
    "city":   "New York",
    "state":  "NY",
    "zip":    "10001"
  },
  "skills": ["C#", "SQL", "JavaScript"],
  "projects": [
    { "projectId": 10, "title": "HR System",   "budget": 50000 },
    { "projectId": 11, "title": "Payroll App", "budget": 30000 }
  ]
}
```

```csharp
// One class per nested object level

public class Employee
{
    public int           Id         { get; set; }
    public string        Name       { get; set; }
    public string        Department { get; set; }

    // Nested object → separate class
    public Address       Address    { get; set; }

    // Array of strings → List<string>
    public List<string>  Skills     { get; set; }

    // Array of objects → List<ClassName>
    public List<Project> Projects   { get; set; }
}

public class Address
{
    public string Street { get; set; }
    public string City   { get; set; }
    public string State  { get; set; }
    public string Zip    { get; set; }
}

public class Project
{
    public int     ProjectId { get; set; }
    public string  Title     { get; set; }
    public decimal Budget    { get; set; }
}
```

```csharp
// Controller receiving nested JSON
[HttpPost]
public IActionResult Create([FromBody] Employee employee)
{
    // employee.Address.City is fully populated
    // employee.Skills is a List<string>
    // employee.Projects is a List<Project>

    Console.WriteLine(employee.Name);               // "Alice Johnson"
    Console.WriteLine(employee.Address.City);       // "New York"
    Console.WriteLine(employee.Skills[0]);          // "C#"
    Console.WriteLine(employee.Projects[0].Title);  // "HR System"

    _db.Employees.Add(employee);
    _db.SaveChanges();
    return Ok(employee);
}
```

---

## 🔑 Real API Response Patterns

### Paginated / Wrapped response

```json
{
  "success":    true,
  "page":       1,
  "pageSize":   10,
  "totalCount": 150,
  "data": [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
  ]
}
```

```csharp
// C# model for this shape
public class PagedResult<T>
{
    public bool       Success    { get; set; }
    public int        Page       { get; set; }
    public int        PageSize   { get; set; }
    public int        TotalCount { get; set; }
    public List<T>    Data       { get; set; }
}

// Deserialize:
var result = JsonSerializer.Deserialize<PagedResult<Employee>>(json);
Console.WriteLine(result.TotalCount);       // 150
Console.WriteLine(result.Data[0].Name);     // "Alice"
```

```javascript
// In JavaScript
$.get("/api/employees?page=1", function(response) {
    console.log(response.totalCount);        // 150
    console.log(response.data.length);       // 10

    response.data.forEach(function(emp) {
        console.log(emp.name);
    });
});
```

### Kendo DataSource result structure

```json
{
  "Data": [
    { "id": 1, "name": "Alice", "department": "IT", "salary": 75000 },
    { "id": 2, "name": "Bob",   "department": "HR", "salary": 55000 }
  ],
  "Total": 150,
  "AggregateResults": null,
  "Errors": null
}
```

```javascript
// Kendo DataSource schema maps the wrapper
schema: {
    data:   "Data",    // the array is in response.Data
    total:  "Total",   // count is in response.Total
    errors: "Errors"
}
```

---

## 🔑 Extracting Data from Deeply Nested JSON

A real-world response might look like this:

```json
{
  "company": {
    "name": "TechCorp",
    "departments": [
      {
        "name": "IT",
        "headCount": 15,
        "employees": [
          {
            "id": 1,
            "name": "Alice",
            "skills": [
              { "name": "C#",  "level": "Expert"  },
              { "name": "SQL", "level": "Advanced" }
            ]
          },
          {
            "id": 2,
            "name": "Bob",
            "skills": [
              { "name": "Python", "level": "Intermediate" }
            ]
          }
        ]
      },
      {
        "name": "HR",
        "headCount": 8,
        "employees": [ ... ]
      }
    ]
  }
}
```

```javascript
var response = { ...above... };

// Get company name
console.log(response.company.name);
// "TechCorp"

// Get first department name
console.log(response.company.departments[0].name);
// "IT"

// Get first employee in IT
console.log(response.company.departments[0].employees[0].name);
// "Alice"

// Get Alice's first skill
console.log(response.company.departments[0].employees[0].skills[0].name);
// "C#"

// ── Extract all employee names across all departments ──────
var allNames = [];
response.company.departments.forEach(function(dept) {
    dept.employees.forEach(function(emp) {
        allNames.push(emp.name);
    });
});
console.log(allNames);  // ["Alice", "Bob", ...]

// ── Using flatMap (modern way) ─────────────────────────────
var allEmployees = response.company.departments
    .flatMap(dept => dept.employees);

var allSkills = allEmployees
    .flatMap(emp => emp.skills)
    .map(skill => skill.name);

// ── Find IT department ─────────────────────────────────────
var itDept = response.company.departments.find(d => d.name === "IT");
var itEmps = itDept?.employees ?? [];
```

---

## 🔑 Modifying Nested JSON

```javascript
var employee = {
  id: 1,
  name: "Alice",
  address: { city: "New York", zip: "10001" },
  skills: ["C#", "SQL"]
};

// ── Update nested property ─────────────────────────────────
employee.address.city = "Chicago";

// ── Add new property to nested object ─────────────────────
employee.address.country = "USA";

// ── Add item to nested array ────────────────────────────────
employee.skills.push("JavaScript");

// ── Remove item from nested array ──────────────────────────
employee.skills = employee.skills.filter(s => s !== "SQL");

// ── Replace entire nested object ────────────────────────────
employee.address = { city: "Boston", zip: "02101", country: "USA" };

// ── Spread operator to update without mutating original ─────
var updated = {
    ...employee,
    address: { ...employee.address, city: "Boston" }
};
```

---

## 🔑 Transforming Nested JSON (reshaping for UI)

```javascript
// Server returns this shape:
var serverData = [
  { "emp_id": 1, "emp_nm": "Alice", "dept_cd": "IT",  "sal_amt": 75000 },
  { "emp_id": 2, "emp_nm": "Bob",   "dept_cd": "HR",  "sal_amt": 55000 }
];

// Transform to the shape your UI needs:
var uiData = serverData.map(function(item) {
    return {
        id:          item.emp_id,
        name:        item.emp_nm,
        department:  item.dept_cd,
        salary:      item.sal_amt,
        displayText: item.emp_nm + " (" + item.dept_cd + ")"
    };
});
// [{ id:1, name:"Alice", department:"IT", salary:75000, displayText:"Alice (IT)" }]
```

---

## ❓ Interview Questions

**Q: How do you access a property three levels deep in JSON?**

> Chain dots: `obj.level1.level2.level3` or use optional chaining: `obj?.level1?.level2?.level3` to avoid errors when any level might be undefined.

**Q: What is optional chaining and when should you use it?**

> `?.` — it returns `undefined` instead of throwing a TypeError if any part of the chain is null or undefined. Use it whenever accessing deeply nested properties from external/API data that might be incomplete.

**Q: How do you map a nested JSON array to C# classes?**

> Each nested object needs its own C# class. Arrays map to `List<T>`. Arrays of strings map to `List<string>`. ASP.NET Core deserializes these automatically.

**Q: What is `flatMap` in JavaScript?**

> It maps each element and then flattens the result by one level. Useful for extracting nested arrays: `departments.flatMap(d => d.employees)` gives you a single flat list of all employees from all departments.

**Q: How do you update a nested object without mutating the original?**

> Use spread operator: `{ ...original, nested: { ...original.nested, field: newValue } }`.

---
